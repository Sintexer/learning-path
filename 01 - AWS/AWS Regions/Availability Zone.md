> AZ stands for **Availability Zone**.

AWS cluster Data Centers around the world into **Availability Zones (AZ)**. AWS would have a second data center connected to the first through redundant high speed and low latency links. If the first [[Data Center]] goes down, the second Data Center is still up and running. AZ is a cluster of Data Centers.

**Availability Zones** have code names. Because they are located inside Regions, they can be addressed by appending a letter to the end of the Region code name. Here are examples of Availability Zone codes:

- **us-east-1a** is an Availability Zone in us-east-1 (N. Virginia Region).
- **sa-east-1b** is an Availability Zone in sa-east-1 (São Paulo Region).

Therefore, if you see that a resource exists in us-east-1c, you can infer that the resource is located in Availability Zone c of the us-east-1 [[Region]].