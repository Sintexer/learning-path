Object-level storage of any data type. Doesn't store files in a file system.

Object stored in S3 along with some metadata:

- **System-Defined** - used to maintain things such as created date, size and last modified.
- **User-Defined** - allows user to assign key-value pairs to the data.

	Amazon S3 provides SLA 99.999999999% (11 9's). This is possible due to high rate of data redundancy across at least 3 AZs in an S3 Region.

**Bucket** is the fundamental container for objects.

> **Buckets must have globally unique name.**
> **Buckets are defined at the region level.**

Buckets serve several purposes:

1. They organize the S3 namespace at the highest level.
2. They identify the account responsible for storage and data transfer charges.
3. They play role in access control.
4. They serve as the unit of aggregation for usage reporting.

**Object** is uniquely identified data stored inside the bucket. Has a key and a version ID.

Object consists of `key`, `version ID`, `value`, `metadata`, `subresources` and `access control information`.

**Buckets tied to the regions**. You might choose bucket region to optimize latency, minimize costs or address regulatory requirements.

#### S3 features

**S3 Storage classes:**

1. **S3 Intelligent-tiering** - automatic cost savings by auto-tiering data with any access pattern.
2. **S3 Standard** - General purpose storage for active, frequent data access.
3. **S3 Standard-Infrequent Access (S3 Standard-IA) - Low cost storage for data accessed monthly, and requires miliseconds retrieval.
4. **S3 Glacier Instant Retrieval** - Low cost storage for long-lived data, with retrieval in miliseconds.
5. **S3 Glacier Flexible Retrieval** - Long-term, low-cost storage for backups and archives with retrieval options from minutes to hours.
6. **S3 Glacier Deep Archive** - Lowest cost cloud storage for long-term, rarely accessed archive data, with retrieval in hours.
7. **S3 One Zone-Infrequent Access (S3 One Zone-IA) - Infrequently accessed data in a single AZ for cost savings.
8. **S3 on Outpost** - Delivers object storage to on-premises AWS Outposts environments to meet lcal data processing and data residency needs.

#### S3 encryption

1. SSE-S3 - encrypts S3 objects using keys handled and managed by AWS.
2. SSE-KMS - uses AWS KMS to manage encryption keys.
3. SSE-C - user manages keys by its own.
4. Client side encryption.

SSE stands for Server Side Encryption.

**1. SSE-S3.** Uses keys handled and managed by AWS. Uses **AES256** encryption type. Must set header `"x-amz-server-side-encryption":"AES256"`.

**2. SSE-KMS.** Uses KMS. Gives user control over who has an access. Plus audit trial. Must set header `"x-amz-server-side-encryption":"aws:kms"`.

**3. SSE-C.** Amazon doesn't store client key, client must handle that. Client must use HTTPS. Key must be provided in HTTP headers for every HTTP request.

**4. Client side encryption.** Client must encrypt data before sendind it to AWS and decrypt after retrieving it.

