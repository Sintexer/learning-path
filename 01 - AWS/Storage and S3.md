
Data stored in AWS is safe. AWS ensures redundancy of your data to insure natural disaster or any other catastrophic event won't make you lose your data. Your data might be replicated across [[Region]]s, [[Availability Zone]]s, or [[Data Center]]s, or not even replicated at all. This is based on the service you are using and service settings. Some services require you to configure data redundancy, some handle this automatically. If a service is **Region-scoped**, it handles redundancy across AZs automatically.

## Block stores
### EBS - Elastic block store

Could be **SSD** or **HDD** backed. 
- SSD: *gp3, gp2, io1, io2.* 
- HDD: *st1, sc1.*

**EBS snapshots best practices:**

1. **Incremental** (only changed should be snapped).
2. Could be done during instance running, but **better not to do it during high load**.
3. Max **100\_000** snapshots.
4. **Can be duplicated** across AZ.
5. Should be warmed up after unsealing.
6. Could be automated using AWS Lifecycle management.
7. You can make AMI from snapshot.
8. Snapshots are stored in S3.

**EBS Encryption:**

1. Everything is encrypted (volume, data transferred between volume and instance, snapshots, volumes from snapshots).
2. Encryption is transparent, without your efforts.
3. Low impact on latency.
4. KMS keys could be used.
5. Snapshots of encrypted volumes are encrypted.

**EBS Migration:**

1. EBS volumes are locked to specific AZ.
2. To migrate you need to make snapshot, copy snapshot to different AZ, unseal snapshot.


**Use cases:**

- `gp2` and `gp3` (General Purpose `SSD`) balance price and performance. Are ideal for use cases such as
boot volumes, medium-size single instance databases, and development and test environments.
- `io1` and `io2` (Provisioned IOPS `SSD`) support up to 64,000 IOPS and 1,000 MiB/s of throughput. This
enables you to predictably scale to tens of thousands of IOPS per EC2 instance.
- `st1` (Throughput Optimized `HDD`) low-cost storage that defines performance in terms of throughput
rather than IOPS. These volumes are ideal for large, sequential workloads such as Amazon EMR, ETL, data
warehouses, and log processing.
- `sc1` (Cold HDD) provide low-cost magnetic storage that defines performance in terms of throughput rather
than IOPS. These volumes are ideal for large, sequential, cold-data workloads. If you require **infrequent
access** to your data and are looking to save costs, these volumes provide inexpensive block storage.

### EBS vs Instance store:

Instance store is physically attached to the machine, while EBS volume is a network drive.

`+` Instance store has better performance.
`+` Good for buffer/cache/scratch data/temporary content.
`+` Data survives reboot.

`-` On *stop* or *termination* instance store is lost.
`-` You can't resize the instance store.
`-` backups *must* be operated by user.

## File store

### EFS - Elastic file system

You can mount EFS volume to different AWS networks and devices, could be mounted to several AZ. Uses NFSv4 protocol.

Two storage classes:
- **Standard access** - for frequently accessed files.
- **Infrequent access (EFS IA)** - for long-lived but less ised files at lower cost.

**Key features:**

1. **Shared storage** - files could be access either from AWS or from on-premises. Could be accessed by thousands of EC2 instances with different technologies (from VPN, AWS Direct connect etc.) - good for hybrid solutions. Multiple AZ and Regions.
2. **Scalable performance** - EFS designed for low latency IOPS. As storage size grows - performance increases. 10GB/s at peek with 500k IOPS. EFS scales automatically.
3. **Secure and compliant** - EFS can be used with IAM roles as well as VPC security groups and allows you to define individual file permissions using POSIX. EFS has built-in compliance with common regulatory standards, including PCI DSS, HIPAA, and SOC with the ability to meet others if necessary.

**EFS Backup solutions:**

1. **AWS Backup** - backups could only be stored on AWS, requires you to manually pause the applications. But it is easy to implement, low cost, fully managed service, incremental, schedulable. This system is PCI and ISO compliant and HIPAA eligible, to ensure that your compliance needs are covered. It is possible to use AWS Backup whether your system has a cloud-native, hybrid, or on-premise configuration.
2. **There is no built-in EFS backup** - EFS also doesn't have native snapshot mechanism. Considered as EFS to EFS back ups. It is a special pipeline through a template in AWS CloudFormation.
3. **Backing up your data to S3** - you need programming abilities to do that and back ups are not incremental.

EFS is elastic and permanent. But there are no Windows instances and no system boot volumes. 

EFS is suitable for:

1. Web serving.
2. Enterprise app usage.
3. Media and entartainment.
4. Shared and home directories.
5. DB backups.
6. Big data analytics.

You pay for the storage used only.

### Amazon FSx

Amazon FSx for Windows File Server. Has features, performance and compatibility to easily lift and shift enterprise applications to the AWS cloud.

Supports Server Message Block (SMB) protocol. 

As a fully managed service, Amazon FSx for Windows File Server eliminates the administrative overhead of setting up and provisioning file servers and storage volumes. Additionally, Amazon FSx keeps Windows software up to date, detects and addresses hardware failures, and performs backups.

## Object stores
### Simple Storage Service (S3)

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



