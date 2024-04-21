Errors happen. One error is not backing up data and then inevitably losing it. To prevent this from happening to you, always back up your data, even in AWS. Because your [[AWS EBS|EBS volumes]] consist of the data from your [[EC2]] instance, you should make **backups** of these volumes, called **snapshots**.

EBS snapshots are **incremental** backups that only save the blocks on the volume that have changed after your most recent snapshot. For example, if you have 10 GB of data on a volume and only 2 GB of data have been modified since your last snapshot, only the 2 GB that have been changed are written to [[AWS S3]].

When you take a snapshot of any of your EBS volumes, the backups are **stored redundantly** in multiple [[Availability Zone|Availability Zones]] using **Amazon S3**. This aspect of storing the backup in Amazon S3 is handled by AWS, so you won’t need to interact with Amazon S3 to work with your EBS snapshots. You manage them in the Amazon EBS console, which is part of the Amazon EC2 console.  
  
EBS snapshots can be used to create multiple new volumes, whether they’re in the same Availability Zone or a different one. When you create a new volume from a snapshot, it’s an **exact copy of the original volume** at the time the snapshot was taken.

**EBS snapshots best practices:**

1. **Incremental** (only changed should be snapped).
2. Could be done during instance running, but **better not to do it during high load**.
3. Max **100\_000** snapshots.
4. **Can be duplicated** across AZ.
5. Should be warmed up after unsealing.
6. Could be automated using AWS Lifecycle management.
7. You can make AMI from snapshot.
8. Snapshots are stored in S3.