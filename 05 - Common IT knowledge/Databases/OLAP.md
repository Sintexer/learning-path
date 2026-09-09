> Online Analytics Processing

Usually an analytic query needs to scan over a huge number of records, only reading a few columns per record, and calculates aggregate statistics (such as count, sum, or average) rather than returning the raw data to the user. For example, if your data is a table of sales [[Transaction|transactions]], then analytic queries

might be:
- What was the total revenue of each of our stores in January? 
- How many more bananas than usual did we sell during our latest promotion? 
- Which brand of baby food is most often purchased together with brand X diapers?

These queries are often written by business analysts, and feed into reports that help the management of a company make better decisions (business intelligence). In order to differentiate this pattern of using databases from transaction processing, it has been called **online analytic processing** (OLAP).

Usually OLAP systems are used for [[Data Warehouse]].

## Warehouse Tables schema

Usually the question is [[Star Schema vs Snowflake Schema]].

**Fact table:**

- Contains **metrics/measurements** (numbers you sum, average, count)
- One row per **event/transaction**
- Foreign keys pointing to dimensions
- Additive: you can sum amounts across rows

**Dimension table:**

- Contains **descriptive attributes** (who, what, when, where, why)
- One row per **entity** (one product, one customer, one store)
- Primary key only (no foreign keys to other dimensions)
- Non-additive: summing a name makes no sense

## Practical test: Can you sum it?

**Fact test:** Does it make sense to SUM this column?

```sql
-- These are facts (summing makes sense)
SUM(amount)          -- ✓ total sales
SUM(quantity)        -- ✓ total items sold
SUM(cost)            -- ✓ total cost
COUNT(*)             -- ✓ number of transactions
AVG(rating)          -- ✓ average customer rating

-- These are NOT facts (summing is nonsensical)
SUM(product_name)    -- ✗ what does "sum of names" mean?
SUM(customer_email)  -- ✗ nonsense
SUM(order_date)      -- ✗ summing dates is weird
```

If summing makes sense → **fact**  
If summing is nonsense → **dimension**

## The tricky cases: When it's not obvious

### Case 1: Time

**Is date a dimension or a fact?**

**Answer: Always dimension.**

Why? Because:

- You can't sum dates
- You filter/group by dates constantly
- Dates describe _when_ an event happened

```sql
-- WRONG: treating date as a fact
CREATE TABLE sales_fact (
  sale_id,
  amount,
  sale_date  -- This is a dimension attribute!
);

-- RIGHT: date is a dimension
CREATE TABLE sales_fact (
  sale_id,
  amount,
  sale_date_id  -- FK to date_dim
);

CREATE TABLE date_dim (
  date_id,
  date,
  day_of_week,
  month,
  quarter,
  year
);
```

### Case 2: Flags and classifications

**Is a "status" field a fact or dimension?**

**Answer: Usually dimension, sometimes fact.**

```sql
-- Example: Order status

-- DIMENSION: status describes the order
CREATE TABLE order_dim (
  order_id,
  status,  -- 'pending', 'shipped', 'delivered'
  customer_id,
  ...
);

-- FACT: if you count statuses as a metric
CREATE TABLE order_fact (
  order_id,
  amount,
  shipped_count,     -- 1 if shipped, 0 otherwise (SUMABLE)
  delivered_count,   -- 1 if delivered, 0 otherwise (SUMABLE)
  ...
);
```

**Rule:** If you COUNT or SUM it (like "count of shipped orders"), it belongs in the fact table as a flag. If you just filter by it, it's a dimension.

### Case 3: Slowly changing dimensions

**Is price a fact or dimension?**

**Tricky answer: Both, depending on context.**

```sql
-- Option 1: Price is a FACT (changes per transaction)
CREATE TABLE sales_fact (
  sale_id,
  product_id,
  price,             -- What was paid in THIS sale
  quantity,
  amount
);

-- Option 2: Price is a DIMENSION (historical prices)
CREATE TABLE product_dim (
  product_id,
  name,
  price,             -- Current price
  effective_from,
  effective_to
);
```

**Which to choose?**

- **Price as fact:** You care about "what price was paid in each sale" (most common)
- **Price as dimension:** You care about "how did product pricing evolve over time"

Usually: **Put price in the fact table** (what was actually charged), and keep product_dim clean (current attributes only). If you need history, add a separate `product_price_history` dimension.

### Case 4: Counts and aggregations

**Is "number of orders by customer" a fact or dimension?**

**Answer: Fact.**

Why? Because:

- It's derived from events
- It's summed/averaged in reports
- It changes as new orders arrive

```sql
CREATE TABLE customer_summary_fact (
  customer_id,
  year_month,
  order_count,       -- SUMABLE: total orders
  total_spent,       -- SUMABLE: total revenue
  avg_order_value    -- Can average these
);
```

This is a **grain** (aggregation level) of the fact table, not a dimension.