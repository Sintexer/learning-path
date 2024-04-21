> AKA **Elastic Block Store**

As the name implies, **Amazon Elastic Block Store (Amazon EBS)** is [[AWS Block Storage|block-level storage]] that you can attach to an [[EC2]] instance. You can compare this to how you much attach an external drive to your laptop. This attachable storage is called an **EBS volume**. EBS volumes act similarly to external drives in more than one way. EBS are also [[AWS EBS scaling|scalable]] and EC2 has **one-to-many** relationship with EBS, so you can also scale the number of EBS attached to one EC2.

- **Detachable:** You can detach an EBS volume from one EC2 instance and attach it to another EC2 instance in the same [[Availability Zone]] to access the data on it.
- **Distinct:** The external drive is separate from the computer. That means that if an accident occurs and the computer goes down, you still have your data on your external drive. The same is true for EBS volumes.
- **Size-limited:** You’re limited to the size of the external drive, because it has a fixed limit to how scalable it can be. For example, you might have a 2 TB external drive, which means you can only have 2 TB of content on it. This also relates to Amazon EBS, because a volume also has a max limitation of how much content you can store on it.
- **1-to-1 connection:** Most EBS volumes can only be connected with one computer at a time. Most EBS volumes have a one-to-one relationship with EC2 instances, so they cannot be shared by or attached to multiple instances at one time. **(But Amazon has announced the Amazon EBS multi-attach)**

[[AWS EBS volume types|EBS offers user a choice of storage types]].

## Use cases

Amazon EBS is useful when you must retrieve data quickly and have data persist long term. Volumes are commonly used in the following scenarios:

- **OS** - Boot and root volumes can be used to store an operating system. The root device for an instance launched from an [[AWS AMI|AMI]] is typically an EBS volume. These are commonly referred to as EBS-backed AMIs.
- **Databases** - because it can scale with your performance needs and provide consistent and low-latency performance
- **Enterprise applications** - because of high availability and high durability.
- **Big data analytics engine** - because of persistence and scaling possibility.

## EBS benefits

- **High availability** - When you create an EBS volume, it is automatically replicated in its Availability Zone to prevent data loss from single points of failure.
- **Data persistence**.
- **Data encryption** - When activated by the user, all EBS volumes support encryption.
- **Flexibility** - EBS volumes support on-the-fly changes. Modify volume type, volume size, and input/output operations per second (IOPS) capacity without stopping your instance.
- **Backups** - Amazon EBS provides the ability to create backups of any EBS volume.

---



**EBS Encryption:**

1. Everything is encrypted (volume, data transferred between volume and instance, snapshots, volumes from snapshots).
2. Encryption is transparent, without your efforts.
3. Low impact on latency.
4. KMS keys could be used.
5. Snapshots of encrypted volumes are encrypted.

**EBS Migration:**

1. EBS volumes are locked to specific AZ.
2. To migrate you need to make snapshot, copy snapshot to different AZ, unseal snapshot.
