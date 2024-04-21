> Elastic File System

**Amazon Elastic File System (Amazon EFS)** is a set-and-forget file system that *automatically grows* and *shrinks* as you add and remove files. There is no need for provisioning or managing storage capacity and performance. Amazon EFS can be used with [[AWS Compute]] services and on-premises resources. You can connect tens, hundreds, and even thousands of compute instances to an Amazon EFS file system at the same time, and Amazon EFS can provide consistent performance to each compute instance. It is a [[AWS File Storage|file storage]]. You can mount EFS volume to different AWS networks and devices, could be mounted to several [[Availability Zone|AZs]]. Uses NFSv4 protocol. You pay for the storage used only. EFS is **elastic and permanent**. But there are no Windows instances and no system boot volumes. 

**EFS is suitable for:**

1. Web serving.
2. Enterprise app usage.
3. Media and entertainment.
4. Shared and home directories.
5. DB backups.
6. Big data analytics.

There are **4 storage classes of EFS** for you to choose:
- **Standard** - for frequently accessed files.
- **Standard-Infrequent access (EFS Standard-IA)** - for long-lived, but less used files at lower cost.
- **EFS One Zone** - additional savings by saving your data in a single AZ.
- **EFS One Zone-Infrequent Access (EFS One Zone-IA**

**Key features:**

1. **Shared storage** - files could be access either from AWS or from on-premises. Could be accessed by thousands of [[EC2]] instances with different technologies (from VPN, AWS Direct connect etc.) - good for hybrid solutions. Multiple [[Availability Zone|AZs]] and [[Region|Regions]].
2. **Scalable performance** - EFS designed for low latency IOPS. As storage size grows - performance increases. 10GB/s at peek with 500k IOPS. **EFS scales automatically**.
3. **Secure and compliant** - EFS can be used with [[IAM role|IAM roles]] as well as [[AWS Security groups|VPC security groups]] and allows you to define individual file permissions using POSIX. EFS has built-in compliance with common regulatory standards, including PCI DSS, HIPAA, and SOC with the ability to meet others if necessary.

**EFS Backup solutions:**

1. **AWS Backup** - backups could only be stored on AWS, requires you to manually pause the applications. But it is easy to implement, low cost, fully managed service, incremental, schedulable. This system is PCI and ISO compliant and HIPAA eligible, to ensure that your compliance needs are covered. It is possible to use AWS Backup whether your system has a cloud-native, hybrid, or on-premise configuration.
2. **There is no built-in EFS backup** - EFS also doesn't have native snapshot mechanism. Considered as EFS to EFS back ups. It is a special pipeline through a template in AWS CloudFormation.
3. **Backing up your data to S3** - you need programming abilities to do that and back ups are not incremental.



