> AKA **Storage tier**

When you upload an object to [[AWS S3]] and you don’t specify the storage class, you upload it to the **default storage class**, often referred to as *standard storage*.

Amazon S3 storage classes let you change your storage tier when your data characteristics change. For example, if you are accessing your old photos infrequently, you might want to change the storage class for the photos to save costs.

Sometimes you find yourself moving objects from tier to tier based on their lifetime or other principles. In such cases you can automate this process by using [[AWS S3 lifecycle]].

## S3 Standard

This is considered general-purpose storage for cloud applications, dynamic websites, content distribution, mobile and gaming applications, and big data analytics.

## S3 Intelligent-Tiering

This tier is useful if your data has unknown or changing access patters. **S3 Intelligent-Tiering** stores objects in three tiers: 

1. frequent access tier
2. infrequent access tier
3. archive instance access tier. 

S3 monitors access patterns of your data and **automatically moves** your data to the most cost-effective storage tier based on frequency of access.

## S3 Standard-Infrequent Access (S3 Standard-IA)

This tier is for data that is accessed less frequently but requires **rapid access when needed**. S3 Standard-IA offers the high durability, high throughput, and low latency of S3 Standard, with a low per-GB storage price and per-GB retrieval fee. This storage tier is ideal if you want to store long-term backups, disaster recovery files, and so on.

## S3 One Zone-Infrequent Access (S3 One Zone-IA)

Unlike other S3 storage classes that store data in a minimum of three [[Availability Zone|Availability Zones]], **S3 One Zone-IA** stores data in a **single Availability Zone**, which makes it less expensive than S3 Standard-IA. S3 One Zone-IA is ideal for customers who want a lower-cost option for infrequently accessed data, but do not require the availability and resilience of S3 Standard or S3 Standard-IA. It's a good choice for storing secondary backup copies of on-premises data or easily recreatable data.

## S3 Glacier Instant Retrieval

Use S3 Glacier Instant Retrieval for archiving data that is *rarely accessed* and requires *millisecond retrieval*. Data stored in this storage class offers a cost savings of up to 68 percent compared to the S3 Standard-IA storage class, with the same latency and throughput performance.

## S3 Glacier Flexible Retrieval

S3 Glacier Flexible Retrieval offers low-cost storage for archived data that is accessed **1–2 times per year**. With S3 Glacier Flexible Retrieval, your data can be accessed in as little as 1–5 minutes using an expedited retrieval. You can also request free bulk retrievals in up to 5–12 hours. It is an ideal solution for backup, disaster recovery, offsite data storage needs, and for when some data occasionally must be retrieved in minutes.

## S3 Glacier Deep Archive

S3 Glacier Deep Archive is the lowest-cost Amazon S3 storage class. It supports long-term retention and digital preservation for data that might be accessed once or twice a year. Data stored in the S3 Glacier Deep Archive storage class has a default retrieval time of 12 hours. It is designed for customers that retain data sets for 7–10 years or longer, to meet regulatory compliance requirements. Examples include those in highly regulated industries, such as the financial services, healthcare, and public sectors.

## S3 on Outposts

Amazon S3 on Outposts delivers object storage to your **on-premises AWS Outposts** environment using S3 API's and features. For workloads that require satisfying local data residency requirements or need to keep data close to on premises applications for performance reasons, the S3 Outposts storage class is the ideal option.