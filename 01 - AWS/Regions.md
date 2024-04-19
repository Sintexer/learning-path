Data stored in AWS is safe. AWS ensures redundancy of your data to insure natural disaster or any other catastrophic event won't make you lose your data.

---
## Data Center 

#DataCenter

Data is stored in a **Data Center**. Data center is an isolated unit with redundant power system and data storage and computing cluster. But what if natural disaster destroys whole Data Center?

---
## Availability Zone

#AZ 

Luckily AWS cluster Data Centers around the world into **Availability Zones (AZ)**. So here AWS would have a second data center connected to the first through redundant high speed and low latency links. If the first Data Center goes down, the second Data Center is still up and running. AZ is a cluster of Data Centers.

Availability Zones have code names. Because they are located inside Regions, they can be addressed by appending a letter to the end of the Region code name. Here are examples of Availability Zone codes:

- **us-east-1a** is an Availability Zone in us-east-1 (N. Virginia Region).
- **sa-east-1b** is an Availability Zone in sa-east-1 (São Paulo Region).

Therefore, if you see that a resource exists in us-east-1c, you can infer that the resource is located in Availability Zone c of the us-east-1 Region.

---
## Region 

#Region

What if entire AZ has collapsed? Like data centers, AWS also clusters AZs together and also connects them with redundant high speed and low latency links. A cluster of AZs is simply called a **Region**. 

Regions are usually called by location (Region - N. Virginia). Each AWS Region is associated with a geographical name and a Region code. Here are examples of Region codes: **us-east-1**, **ap-northeast-1**.

### Region considerations

Choosing the right region is based on 4 factors: *Compliance*,  *Latency*, *Pricing*, and *Service availability*.

- **Compliance** - some countries may require you to store and manage all the data within country boundaries
- **Latency** - we all are bound to the speed of light
- **Pricing** - it is cheaper to run in India than in London
- **Service Availability** - new AWS services are often bounded to specific Regions.

---
## Edge locations

AWS enables **caching** for users. If AZ is located in Ohio, users might download content from regional **edge caches** without the need to download it directly from Ohio Data Center. AWS service **Amazon CloudFront** might be used to cache content using the edge locations.

---

## Scope of AWS services

Depending on the AWS service that you use, your resources are either deployed at the Availability Zone, Region, or Global level. Each service is different, so you must understand how the scope of a service might affect your application architecture.  
  
When you operate a **Region-scoped** service, you only need to select the Region that you want to use. If you are not asked to specify an individual Availability Zone to deploy the service in, this is an indicator that the service operates on a Region-scope level. For Region-scoped services, AWS automatically performs actions to increase data durability and availability.  
  
On the other hand, some services ask you to specify an **Availability Zone**. With these services, you are often responsible for increasing the data durability and high availability of these resources.

To keep your application available, you must maintain **high availability** and **resiliency**. A well-known best practice for cloud architecture is to use Region-scoped, managed services. These services come with availability and resiliency built in. When that is not possible, make sure your workload is **replicated across multiple Availability Zones**. At a minimum, you should use two Availability Zones. That way, if an Availability Zone fails, your application will have infrastructure up and running in a second Availability Zone to take over the traffic.