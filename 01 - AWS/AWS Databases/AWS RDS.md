> AKA **AWS Relational Database Service**

Amazon RDS is a [[AWS Managed Database|managed database service]] customers can use to create and manage [[Relational Database|relational databases]] in the cloud without the operational burden of traditional [[RDBMS|database management]]. For example, imagine you sell healthcare equipment, and your goal is to be the number-one seller on the West Coast of the United States. Building a database doesn’t directly help you achieve that goal. However, having a database is a necessary component to achieving that goal.

With Amazon RDS, you can offload some of the unrelated work of creating and managing a database. You can focus on the tasks that differentiate your application, instead of focusing on infrastructure-related tasks, like provisioning, patching, scaling, and restoring. RDS also provides you with *automated* and *manual* solutions for [[AWS RDS backups|backups]]. **Redundancy** is achievable by [[AWS RDS Multi-AZ redundancy|multi-AZ configuration]].

Amazon RDS supports most of the popular [[RDBMS|RDBMSs]], ranging from commercial options to open-source options and even a specific AWS option. Supported Amazon RDS engines include the following:

- **Commercial:** Oracle, SQL Server
- **Open source:** MySQL, PostgreSQL, MariaDB
- **Cloud native:** [[AWS Aurora]]

Amazon RDS is built from compute and storage. The compute portion is called the [[Database|database (DB)]] instance, which runs the DB engine. Depending on the engine selected, the instance will have different supported features and configurations. A DB instance can contain multiple databases with the same engine, and each DB can contain multiple tables.  
  
Underneath the DB instance is an [[EC2]] instance. However, this instance is managed through the **Amazon RDS console** instead of the Amazon EC2 console. When you create your DB instance, you choose the instance type and size. The DB **instance class** you choose affects how much processing power and memory it has:

- **Standard classes** - balance of memory, compute and network.
- **Memory optimized classes** - accelerate performance for workloads that processing large datasets in memory 
- **Burstable classes** - baseline level of CPU performance with the ability to burst above the baseline.
## Storage on Amazon RDS

The storage portion of DB instances for Amazon RDS use [[AWS EBS]] volumes for database and log storage. This includes MySQL, MariaDB, PostgreSQL, Oracle, and SQL Server. 

Amazon RDS provides three storage types: General Purpose SSD (also called gp2 and gp3), Provisioned IOPS SSD (also called io1), and Magnetic (also called standard). They differ in performance characteristics and price, which means you can tailor your storage performance and cost to the needs of your database workload.

## Network

When you create a DB instance, you select the [[AWS VPC]] your databases will live in. Then, you select the [[Subnet|subnets]] that will be designated for your DB. This is called a **DB subnet group**, and it has at least t**wo Availability Zones in its Region**. The subnets in a DB subnet group should be **private**, so they *don’t have a route to the internet gateway*. This ensures that your DB instance, and the data inside it, can be reached only by the application backend.  
  
Access to the DB instance can be restricted further by using [[AWS Network Access Control List]] and [[AWS Security groups]]. With these firewalls, you can control, at a granular level, the type of traffic you want to provide access into your database.

Using these controls provides layers of security for your infrastructure. It reinforces that only the backend instances have access to the database.

## Security

When it comes to security in Amazon RDS, you have control over managing access to your Amazon RDS resources, such as your databases on a DB instance. How you manage access will depend on the tasks you or other users need to perform in Amazon RDS. [[AWS Network Access Control List|Network ACLs]] and [[AWS Security groups|security groups]] help users dictate the flow of traffic. If you want to restrict the actions and resources others can access, you can use [[IAM policies]].

- **[[IAM policies]]** - You can use IAM policies to specify who can create, describe, modify, or delete DB instances, tag resources, or modify security groups. 
- **[[AWS Security groups]]** - control which IP addresses or [[EC2]] instances can connect to your DBs. When you first create a DB instance, all database access is prevented except through rules specified by an associated security group.
- **RDS encryption** - encrypt instance and snapshots to increase their security
- **SSL or TLS** - use SSL or TLS connections with DB instances. 