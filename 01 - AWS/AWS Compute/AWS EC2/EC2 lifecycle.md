
![[EC2 lifecycle scheme.png]]

### Lifecycle:

1. When you launch an instance, it enters the **pending** state. When an instance is pending, billing has not started. At this stage, the instance is preparing to enter the running state. Pending is where AWS performs all actions needed to set up an instance, such as copying the AMI content to the root device and allocating the necessary networking components.
2. When your instance is **running**, it's ready to use. This is also the stage where *billing begins*. As soon as an instance is running, you can take other actions on the instance, such as **reboot**, **terminate**, **stop**, and **stop-hibernate**. [[EC2 lifecycle#Difference between stop and stop-hibernate|The difference between stop and stop-hibernate]]
3. When you reboot an instance, it’s different than performing a stop action and then a start action. **Rebooting** an instance is equivalent to rebooting an operating system. The instance keeps its public DNS name (IPv4) and private and public IPv4 addresses. An IPv6 address (if applicable) remains on the same host computer and maintains its public and private IP address, in addition to any data on its instance store volumes.
4. When you stop your instance, it enters the **stopping** and then **stopped** state. This is similar to when you shut down your laptop. You can stop and start an instance if it **has an [[Storage and S3#EBS - Elastic block store|Amazon Elastic Block Store (Amazon EBS)]]** volume as its root device. When you stop and start an instance, your instance can be placed on a *new underlying physical server*. Your instance retains its private IPv4 addresses and if your instance has an IPv6 address, it retains its IPv6 address. When you put the instance into stop-hibernate, the instance enters the stopped state, but saves the last information or content into memory, so that the start process is faster.
5. When you **terminate** an instance, the **instance stores are erased**, and you lose both the public IP address and private IP address of the machine. Termination of an instance means that you can no longer access the machine. As soon as the status of an instance changes to **shutting down** or **terminated,** you stop incurring charges for that instance.

### Difference between stop and stop-hibernate

When you **stop** an instance, it enters the **stopping** state until it reaches the **stopped** state. AWS *does not charge* usage or data transfer fees for your instance *after you stop* it. But **storage for any Amazon EBS volumes is still charged**. 

While your instance is in the stopped state, you can modify some attributes, like the instance type. **When you stop your instance, the data from the instance memory (RAM) is lost.**  
  
When you **stop-hibernate** an instance, Amazon EC2 signals the operating system to perform hibernation (suspend-to-disk), which **saves** the contents from the **instance memory (RAM)** to the EBS root volume. You can hibernate an instance only if hibernation is turned on and the instance meets the *hibernation prerequisites*.

## EC2 instance termination

When you **terminate** an instance, the **instance stores are erased**, and you lose both the public IP address and private IP address of the machine. Termination of an instance means that you can no longer access the machine. As soon as the status of an instance changes to **shutting down** or **terminated,** you stop incurring charges for that instance.