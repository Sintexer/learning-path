
##  Migration Tools — Flyway vs. Liquibase

Both tools solve the fundamental problem of **Database Change Management (Schema as Code)**: ensuring that every environment (local, staging, production) has an identical, reproducible database schema state.

| Feature | **Flyway** (Industry Standard for Simplicity) | **Liquibase** (Enterprise / Multi-DB Engine) |
| :--- | :--- | :--- |
| **Format** | Pure SQL scripts (e.g., `.sql` files) | Formats: YAML, XML, JSON, or SQL |
| **Philosophy** | "SQL-first" — You write native, optimized dialect SQL. | "Abstraction-first" — Defines changes via XML/YAML tags; translates to underlying DB dialect. |
| **Tracking Table** | `flyway_schema_history` | `DATABASECHANGELOG` & `DATABASECHANGELOGLOCK` |
| **Rollback support**| Automated rollbacks require commercial edition (Free version relies on manual rollback scripts). | Built-in `<rollback>` tags in XML/YAML. |

*Recommendation for interviews:* Mention **Flyway** as your primary tool, as most high-performance engineering teams prefer writing raw SQL migrations to maintain absolute control over locking behavior and dialect-specific features.

## How Migration Tools Work Under the Hood

When an application starts up with Flyway enabled:
1. It looks for a metadata table called **`flyway_schema_history`**. (If missing, it creates it).
2. It acquires a database-level lock or advisory lock to prevent multiple app instances from migrating concurrently.
3. It scans the classpath (e.g., `src/main/resources/db/migration`) for migration files.
4. It reads the history table to see which migrations have already been applied and verifies their **checksums** (SHA-256).
   * *Trap:* If someone modifies an already-executed migration file, the checksum fails, and Flyway **refuses to boot** the application to prevent state corruption.
5. It applies new migration files in strict numerical order inside individual transactions.

## Versioned vs. Repeatable Migrations

This is a classic interview question.

### A. Versioned Migrations (`V...__description.sql`)
* **Naming convention:** `V1__init.sql`, `V2__add_users_table.sql`, `V2.1__add_index.sql`.
* **Execution:** Executed **exactly once** in their lifetime.
* **Use Cases:** **DDL changes** that mutate state irreversibly:
  * Creating tables (`CREATE TABLE`)
  * Adding/modifying columns (`ALTER TABLE`)
  * Creating indexes (`CREATE INDEX`)
  * Data migrations (backfilling rows)

### B. Repeatable Migrations (`R...__description.sql`)
* **Naming convention:** `R__create_user_views.sql`, `R__update_discount_function.sql` (No version number!).
* **Execution:** Executed **every time their file checksum changes**.
* **Order:** Always executed *after* all versioned migrations have completed.
* **Use Cases:** **Idempotent, state-less objects** that can be dropped and recreated safely:
  * SQL Views (`CREATE OR REPLACE VIEW ...`)
  * Stored procedures / Functions (`CREATE OR REPLACE FUNCTION ...`)
  * Triggers and permissions

## Rollback Strategy

> **Interview Question:** *"How do you handle rollbacks in production migrations?"*

Many junior developers answer: *"We write down-scripts (`U1__rollback.sql` or Liquibase `<rollback>`) to undo the changes."*

**The Senior/Staff Answer: Always Roll Forward.**
In modern high-scale continuous deployment (CI/CD), **automated down-migrations are an anti-pattern**.

### Why rollbacks fail in production:
1. **Data Loss:** If migration $V2$ drops a column or splits a table, running a down-migration cannot magically recreate the data written by live users in the intervening 10 minutes.
2. **Untested Path:** Rollback scripts are rarely tested in staging under real load. Running an untested destructive script during an active production outage is dangerous.
3. **App-DB Decoupling:** If the new version of your application code is already running, rolling back the database schema instantly crashes the application.

**The Production Strategy:** 
* Design migrations to be **backward-compatible** (non-breaking).
* If a release fails, roll back the *application code* first (which works because the DB is backward-compatible).
* If the schema must be changed, apply a **new, forward migration** (`V3__fix_previous_issue.sql`).

## How to Design Backward-Compatible Migrations

The fundamental principle of zero-downtime deployments is the **N−1/N Compatibility Rule**:

> At any given point in time during a deployment, **the database schema must work seamlessly with BOTH the old application version (N−1) and the new application version (N) simultaneously.**

Because modern deployments are Rolling Updates (Kubernetes, AWS ECS) or Canary Deployments, old pods and new pods run at the same time for several minutes.

#### The 3 Iron Rules of Backward Compatibility:

1. **Rule 1: Additive-Only Changes First**
    - You can add new nullable columns, add new tables, or add new indexes.
    - The old application simply ignores the new columns and keeps running.
2. **Rule 2: Never Add a `NOT NULL` Column in One Shot**
    - If you run `ALTER TABLE users ADD COLUMN phone VARCHAR NOT NULL;`, the old application version (which knows nothing about `phone`) will attempt to insert new users without a `phone` field.
    - Result: The database rejects all incoming inserts from the old app with a `null value in column "phone" violates not-null constraint` error!
3. **Rule 3: Decouple Code Deployments from Destructive DB Changes**
    - **Phase 1:** Apply schema change (Expand: purely additive).
    - **Phase 2:** Deploy application code (Starts reading/writing new schema, stops using old schema).
    - **Phase 3:** Apply schema cleanup (Contract: safely remove old columns/tables).
    - _Never combine Phase 1 and Phase 3 in the same deployment._

## Zero-Downtime Schema Changes

Why do database migrations take down production systems?  **Locks.**

### The Postgres Lock Queue

Every time you run an `ALTER TABLE` statement, [[PostgreSQL]] must acquire an **`ACCESS EXCLUSIVE` lock** on that table.
* While held, **no other transaction can read (`SELECT`) or write (`INSERT`, `UPDATE`, `DELETE`)** to that table.

1. A long-running `SELECT` is executing.
2. Your migration asks for `ACCESS EXCLUSIVE`. It must wait for the `SELECT` to finish.
3. **PostgreSQL prioritizes the lock queue:** Every normal application query arriving *after* the migration request will now **queue up behind the migration**.
4. Within seconds, your application connection pool is completely exhausted, and your entire site returns `504 Gateway Timeout`.

#### The Mitigation: Strict Lock Timeouts
Never run a production migration without setting a short lock timeout!
```sql
-- Tell Postgres: If you can't get the lock in 2 seconds, FAIL IMMEDIATELY.
-- Do not sit there blocking all other web traffic!
SET lock_timeout = '2s';

ALTER TABLE users ADD COLUMN age INT;
```

## Real-World Zero-Downtime Recipes

### Recipe 1: Adding a Column Safely
*Scenario: Adding a column with a default value to a 100-million row table.*

**The Anti-Pattern (Prior to Postgres 11 / older MySQL):**
```sql
ALTER TABLE users ADD COLUMN is_verified BOOLEAN DEFAULT false NOT NULL;
```
  
  *Why this broke production:* In older engines, this physically rewrote every single 8KB disk page on the storage engine to append the byte `0` to every row. On a 100M row table, this locked the table exclusively for 20 minutes!
  
**The Modern Way (Postgres 11+):**
  Postgres 11+ optimizes constant defaults. It records the default value in the system catalog (`pg_attribute`) as **metadata only**.  The table is updated in **0.001 milliseconds** without rewriting disk pages.

 **If adding a volatile default (e.g., `DEFAULT clock_timestamp()` or UUID):**
  Postgres *must* still rewrite the table! You must do it in steps:
  1. Add the column without default (`ALTER TABLE users ADD COLUMN token UUID;`).
  2. Set the default for *future* rows (`ALTER TABLE users ALTER COLUMN token SET DEFAULT gen_random_uuid();`).
  3. Backfill old rows in batches using an application script or a background worker loop (e.g., 5,000 rows per transaction) to prevent long locks.
  4. Finally, add the `NOT NULL` constraint safely:
  
 ```sql
 -- Step A: Add as NOT VALID (instant, doesn't lock for scan)
 ALTER TABLE users ADD CONSTRAINT chk_token_not_null CHECK (token IS NOT NULL) NOT VALID;
 -- Step B: Validate concurrently (scans heap without blocking writes)
 ALTER TABLE users VALIDATE CONSTRAINT chk_token_not_null;
 ```

### Recipe 2: Adding an Index Safely

*Scenario: Adding an index to a live table with millions of rows.*

**The Anti-Pattern:**
```sql
CREATE INDEX idx_users_email ON users (email);
```

Acquires a `SHARE` lock. It allows reads (`SELECT`), but **blocks all writes (`INSERT`, `UPDATE`, `DELETE`)** until the entire index is built. If it takes 15 minutes to scan the table, your application writes are dead for 15 minutes.

**The Production Recipe:**
```sql
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
```

  *How it works internally:*
  1. It performs **two sequential passes** over the table heap.
  2. It does **not** take an exclusive lock; reads and writes continue unimpeded.
  3. *Catch:* It cannot run inside a multi-statement transaction block (`BEGIN ... COMMIT`). Tools like Flyway must be configured with `non-transactional` execution for concurrent index files.

### Recipe 3: Renaming or Changing a Column (The Expand/Contract Pattern)

*Scenario: You have a column `phone_number VARCHAR(20)`. You want to rename it to `mobile_number`, or change it to an encrypted format.*

**Never run `ALTER TABLE users RENAME COLUMN phone_number TO mobile_number;` in production.** 

If you do, the moment the migration runs, any running application instance running the old code will throw exceptions (`Column "phone_number" does not exist`).

You must use the **Expand/Contract (Parallel Run) Pattern** across multiple deployments:

```
[Phase 1: Expand]  ----->  [Phase 2: Transition]  ----->  [Phase 3: Contract]
Add new column              App reads/writes both           Drop old column
```

#### The 4-Step Deployment Lifecycle:
1. **Migration 1 (Expand):**
   * Add the new column `mobile_number` (nullable).
   * Put a database trigger on the table so any `INSERT`/`UPDATE` to `phone_number` automatically mirrors into `mobile_number`.
2. **Data Backfill:**
   * Run a background batch worker to copy old `phone_number` values over to `mobile_number` for historical rows.
3. **Application Deploy (Transition):**
   * Deploy the new application version. It now writes directly to `mobile_number` (and falls back to reading `mobile_number`).
4. **Migration 2 (Contract):**
   * Drop the synchronization trigger.
   * Drop the old column: `ALTER TABLE users DROP COLUMN phone_number;` (or mark it as hidden/unused first).

### The Physical Truth About `DROP COLUMN`

In PostgreSQL, executing:

```sql
ALTER TABLE users DROP COLUMN middle_name;
```

**Does NOT rewrite the heap. It does NOT touch the table rows on disk.**

#### What Postgres actually does:
1. It acquires an `ACCESS EXCLUSIVE` lock on the table.
2. It goes to the metadata catalog (`pg_attribute`) and flips an internal boolean flag:  
    `attisdropped = true`
3. It removes the column from any indexes it was part of.
4. It releases the lock.

**Total time taken:** **~1 to 2 milliseconds.**  
The physical bytes of `middle_name` **remain inside the 8KB heap pages** on disk. When future `UPDATE`s happen or when `VACUUM` runs later, those dead bytes are gradually ignored or reclaimed.

#### So why is dropping a column dangerous?

 If the operation only takes 2 milliseconds, why does it crash production systems? Two reasons:

#### 1. The Application Crash (ORM / `SELECT *`)

If the currently running application has an entity mapping `middle_name` (e.g., Hibernate, JPA, or an explicit `SELECT id, name, middle_name FROM users`), the moment the column is dropped, every single read and write in your running app crashes with:  
`PSQLException: ERROR: column "middle_name" does not exist`

Even if your code doesn't use the column, if a developer wrote `SELECT * FROM users`, many frameworks will fail to deserialize the row because the column list changed under their feet.

#### 2. The Lock Queue Cascade

Even though the metadata update takes 2 milliseconds, **acquiring the `ACCESS EXCLUSIVE` lock** can get stuck behind an active, slow `SELECT` query. While the migration waits in line, **all other web traffic queues up behind it**, exhausting the connection pool.

### The Complete Zero-Downtime Recipe to Drop a Column

#### Deployment 1: Application-Level Deprecation

- **Objective:** Ensure no running application instance asks for the column.
- **Action:**
    - Remove `middle_name` from application queries.
    - In Hibernate/JPA, mark the field as ignored, or remove it from the `@Entity`:
```java
// Instead of mapping it:
// private String middleName; 

// In Hibernate 5/6, tell Hibernate to completely ignore its existence:
@Transient
private String middleName;
```
- **Deploy this code to production.**
- **Verify:** Verify all old pods/instances are terminated. Now, 100% of live traffic is completely unaware of the `middle_name` column.

#### Deployment 2: Safe Database Dropping

Now that no application code is querying the column, run the migration with a **safe lock timeout**:

```sql
-- Step 1: Tell Postgres not to wait longer than 2 seconds for a lock
SET lock_timeout = '2s';

-- Step 2: Drop the column (instant metadata change)
ALTER TABLE users DROP COLUMN middle_name;
```

_What if it fails due to lock timeout?_  The migration fails safely without blocking user traffic. The CI/CD pipeline retries when traffic is lower or background jobs are finished.