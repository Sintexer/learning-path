> AKA **AWS Simple Storage Service**

[[AWS Object Storage|Object-level storage]] of any data type. Doesn't store files in a [[AWS File Storage|file system]] and doesn't tied to [[AWS Compute]]. S3 offers a wide range of [[AWS S3 storage classes|storage options]] to optimize costs.

Object stored in S3 along with some metadata:

- **System-Defined** - used to maintain things such as created date, size and last modified.
- **User-Defined** - allows user to assign key-value pairs to the data.

In S3 your objects are stored redundantly inside a [[Region]] across multiple [[Availability Zone|Availability Zones]].

> [!success]
> Amazon S3 provides SLA 99.999999999% (11 9's). This is possible due to high rate of data redundancy across at least 3 AZs in an S3 Region.

In S3 you store objects in containers, called **[[AWS S3 Bucket|buckets]]**. You can’t upload an object, not even a single photo, to Amazon S3 without creating a bucket first. 

After creating a bucket, all data is private by default according to the [[AWS S3 security]].

## S3 Use cases

Amazon S3 is a widely used storage service, with far more use cases than described below:

1. **Backup and storage** - S3 is a natural place to back up files because it is highly redundant. As mentioned in the last lesson, AWS stores your [[AWS EBS snapshots|EBS snapshots]] in S3 to take advantage of its high availability.
2. **Media hosting** - Because you can store unlimited objects, and each individual object can be up to 5 TB, S3 is an ideal location to host video, photo, and music uploads.
3. **Software delivery** - You can use Amazon S3 to host your software applications that customers can download.
4. **Data lakes** - S3 is an optimal foundation for a data lake because of its virtually unlimited scalability. You can increase storage from gigabytes to petabytes of content, paying only for what you use.
5. **Static websites** - You can configure your S3 bucket to host a static website of HTML, CSS, and client-side scripts. To reduce the latency across multiple [[Region|Regions]] one can use [[AWS CloudFront]].
6. **Static content** - Because of the limitless scaling, the support for large files, and the fact that you can access any object over the web at any time, Amazon S3 is the perfect place to store static content.