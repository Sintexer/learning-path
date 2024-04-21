
[[EC2|Amazon EC2]] instances could be configurated with a plenty of storage options:

1. [[AWS EC2 Instance store]] - local storage attached directly to the EC2 instance. Data is lost on reboot. Really fast, usually smaller than EBS.
2. [[AWS EBS|AWS Elastic Block Store]] - persistent block storage volumes attached to EC2 instances. Scalable and with variety of types (SSDs, HDDs, cold HDDs).
3. [[AWS EFS|AWS Elastic File System]] - A scalable, elastic file system that provides shared storage for multiple EC2 instances.
4. [[AWS FSx]] - A managed file system service that provides Windows file server functionality for EC2 instances.

### EBS vs Instance store:

[[AWS EC2 Instance store|Instance store]] is physically attached to the machine, while [[AWS EBS|EBS]] volume is a network drive.

`+` Instance store has better performance.
`+` Good for buffer/cache/scratch data/temporary content.
`+` Data survives reboot.

`-` On *stop* or *termination* instance store is lost.
`-` You can't resize the instance store.
`-` backups *must* be operated by user.
`-` Instance store is of fixed size, while EC2 has one-to-many relationship with EBS.
