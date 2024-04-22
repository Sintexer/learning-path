
## [[AWS EC2 Instance store]]

Instance store is ephemeral block storage. This is preconfigured storage that exists on the same physical server that hosts the EC2 instance and cannot be detached from Amazon EC2. You can think of it as a built-in drive for your EC2 instance.

Instance store is generally well suited for temporary storage of information that is constantly changing, such as buffers, caches, and scratch data. It is not meant for data that is persistent or long lasting. If you need persistent long-term block storage that can be detached from Amazon EC2 and provide you more management flexibility, such as increasing volume size or creating snapshots, you should use Amazon EBS.

## [[AWS EBS]]

Amazon EBS is meant for data that changes frequently and must persist through instance stops, terminations, or hardware failures. Amazon EBS has two types of volumes: SSD-backed volumes and HDD-backed volumes.

The performance of SSD-backed volumes depends on the IOPs and is ideal for transactional workloads, such as databases and boot volumes. 

The performance of HDD-backed volumes depends on megabytes per second (MBps) and is ideal for throughput-intensive workloads, such as big data, data warehouses, log processing, and sequential data I/O.

Here are a few important features of Amazon EBS that you need to know when comparing it to other services.

- It is block storage.
- You pay for what you provision (you have to provision storage in advance).
- EBS volumes are replicated across multiple servers in a single Availability Zone.
- Most EBS volumes can only be attached to a single EC2 instance at a time.

## [[AWS S3]]

If your data doesn’t change often, Amazon S3 might be a cost-effective and scalable storage solution for you. Amazon S3 is ideal for storing static web content and media, backups and archiving, and data for analytics. It can also host entire static websites with custom domain names.

Here are a few important features of Amazon S3 to know about when comparing it to other services:

- It is object storage.
- You pay for what you use (you don’t have to provision storage in advance).
- Amazon S3 replicates your objects across multiple Availability Zones in a Region.
- Amazon S3 is not storage attached to compute.

## [[AWS EFS]]

Amazon EFS provides highly optimized file storage for a broad range of workloads and applications. It is the only cloud-native shared file system with fully automatic lifecycle management. Amazon EFS file systems can automatically scale from gigabytes to petabytes of data without needing to provision storage. Tens, hundreds, or even thousands of compute instances can access an Amazon EFS file system at the same time.

Amazon EFS Standard storage classes are ideal for workloads that require the highest levels of durability and availability. EFS One Zone storage classes are ideal for workloads such as development, build, and staging environments.

Here are a few important features of Amazon EFS to know about when comparing it to other services:

- It is file storage.
- Amazon EFS is elastic, and automatically scales up or down as you add or remove files. And you pay only for what you use.
- Amazon EFS is highly available and designed to be highly durable. All files and directories are redundantly stored within and across multiple Availability Zones. 
- Amazon EFS offers native lifecycle management of your files and a range of storage classes to choose from.

## [[AWS FSx]]

Amazon FSx provides native compatibility with third-party file systems. You can choose from NetApp ONTAP, OpenZFS, Windows File Server, and Lustre. With Amazon FSx, you don't need to worry about managing file servers and storage. This is because Amazon FSx automates time consuming administration task such as hardware provisioning, software configuration, patching, and backups. This frees you up to focus on your applications, end users, and business.

Amazon FSx file systems offer feature sets, performance profiles, and data management capabilities that support a wide variety of use cases and workloads. Examples include machine learning, analytics, high performance computing (HPC) applications, and media and entertainment.