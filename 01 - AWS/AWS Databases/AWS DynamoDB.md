DynamoDB is a fully managed [[NoSQL database]] that provides fast, consistent performance at any scale. It has a flexible billing model, tight integration with [[IaaC]], and a hands-off operational model. DynamoDB has become the database of choice for two categories of applications: **high-scale applications** and **serverless applications**. Although DynamoDB is the database of choice for high-scale and serverless applications, it can work for nearly all [[OLTP]] application workloads. We will explore DynamoDB more in the next lesson.

With DynamoDB, you can do the following:

- Create database tables that can store and retrieve any amount of data and serve any level of request traffic. 
- Scale up or scale down your tables' throughput capacity without downtime or performance degradation. 
- Monitor resource usage and performance metrics using the [[AWS Management Console]].

DynamoDB **automatically** spreads the data and traffic for your tables over a sufficient number of servers to handle your throughput and storage requirements. It does this while maintaining consistent, fast performance. All your data is stored on SSDs and is **automatically replicated** across multiple [[Availability Zone|Availability Zones]] in a [[Region]], providing **built-in high availability and data durability**.

## Core concepts

In DynamoDB, **tables**, **items**, and **attributes** are the core components that you work with. 

- **table** is a *collection* of items. For example, table **Person** stores 'persons' items.
- **item** is a collection of **attributes**. Item is uniquely identifiable.
- **attribute** is a fundamental data element that does not need to be divided any further (like `lastName`, `firstName`, `departmentId`, etc.).

DynamoDB uses primary keys to uniquely identify each item in a table and secondary indexes to provide more querying flexibility.

## Use cases

DynamoDB is a fully [[AWS Managed Database|managed service]] that handles the operations work. You can offload the administrative burdens of operating and scaling distributed databases to AWS.

You might want to consider using DynamoDB in the following circumstances:

- You are experiencing scalability problems with other traditional database systems.
- You are actively engaged in developing an application or service.
- You are working with an [[OLTP]] workload.
- You care deploying a mission-critical application that must be highly available at all times without manual intervention.
- You require a high level of data durability, regardless of your backup-and-restore strategy.

DynamoDB is used in a wide range of workloads because of its simplicity, from low-scale operations to ultrahigh-scale operations, such as those demanded by Amazon.com.

## Security

DynamoDB provides a number of security features to consider as you develop and implement your own security policies. They include the following:

- DynamoDB provides a highly durable storage infrastructure designed for mission-critical and primary data storage. Data is **redundantly stored** on multiple devices across multiple facilities in a DynamoDB Region.  
- All user data stored in DynamoDB is fully **encrypted** at rest. DynamoDB encryption at rest provides enhanced security by encrypting all your data at rest using encryption keys stored in [[AWS KMS]].
- [[IAM]] administrators control who can be authenticated and authorized to use DynamoDB resources. You can use IAM to manage access permissions and implement [[IAM policy|security policies]].
- As a managed service, DynamoDB is protected by the **AWS global network security procedures**
- You can use [[AWS CloudTrail]] to view different security actions logs.
- Use [[IAM role]] for applications, users and services that access DynamoDB.

