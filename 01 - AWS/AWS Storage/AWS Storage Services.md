Data stored in AWS is safe. AWS ensures redundancy of your data to insure natural disaster or any other catastrophic event won't make you lose your data. Your data might be replicated across [[Region|Regions]], [[Availability Zone|Availability Zones]], or [[Data Center|Data Centers]], or not even replicated at all. This is based on the service you are using and service settings. Some services require you to configure data redundancy, some handle this automatically. If a service is **Region-scoped**, it handles redundancy across AZs automatically.

AWS storage services are grouped into three categories: [[AWS File Storage|file storage]], [[AWS Block Storage|block storage]], and [[AWS Object Storage|object storage]]. 

- In **file storage**, data is stored as files in a hierarchy. 
- In **block storage**, data is stored in fixed-size blocks. 
- In **object storage**, data is stored as objects in buckets.

When configuring AWS resources, like [[EC2]] instance, you can set them up with [[AWS EFS]], [[AWS FSx]], [[AWS EBS]], and integrate it with [[AWS S3]]. Options for a **EC2** storage are [[EC2 storage|explained here]].