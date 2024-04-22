[[EC2]] instance store provides temporary [[AWS Block Storage|block-level storage]] for an instance. This storage is located on disks that are *physically attached to the host computer*. This ties the **lifecycle of the data to the lifecycle of the EC2 instance**. Instance store cost is also already included into EC2 price, so you might save some many by using only Instance store without [[AWS EBS]] in some cases.

> **If you delete the instance, the instance store is also deleted. Because of this, instance store is considered ephemeral storage**.
  
Instance store is ideal if you host applications that replicate data to other EC2 instances, such as *Hadoop clusters*. For these cluster-based workloads, having the speed of locally attached volumes and the resiliency of replicated data helps you achieve data distribution at **high performance**. It’s also ideal for temporary storage of information that changes frequently, such as **buffers**, **caches**, **scratch data**, and other **temporary content**.

## Links:

- [[EC2 storage]]