You can scale [[AWS EBS|EBS]] volumes in two ways:

**Increase the volume size** only if it doesn’t increase above the maximum size limit. Depending on the volume selected, **Amazon EBS** currently supports a maximum volume size of 64 tebibytes (TiB). For example, if you provision a 5-TiB io2 Block Express volume, you can choose to increase the size of your volume until you get to 64 TiB.

**Attach multiple volumes** to a single [[EC2]] instance. Amazon EC2 has a **one-to-many relationship with EBS volumes**. You can add these additional volumes during or after EC2 instance creation to provide more storage capacity for your hosts.