> Amazon Virtual Private Cloud

A **virtual private cloud (VPC)** is an isolated network that you create in the AWS Cloud, similar to a traditional network in a data center. VPC is bound to a [[Region]], and Amazon automatically creates a **default VPC** per [[Region]].

When working with networks in the AWS Cloud, you choose your network size by using [[CIDR Notation]]. In AWS, the smallest IP range you can have is **/28**, which provides 16 IP addresses. The largest IP range you can have is a /16, which provides 65,536 IP addresses.

### Creating a VPC

To create a basic VPC you must define a **VPC name**, a [[Region]], and an **address block** in [[CIDR Notation]]. After creating a VPC, you must divide it into smaller [[Subnet|subnets]]. And all computation resources, like [[EC2|EC2 instance]] will be running in these **subnets**. 

In AWS **subnets** are bound to a **VPC**, an [[Availability Zone]], and have its **own IP range**, which is a sub-range of the VPC's CIDR range. When you launch an [[EC2]] instance, you launch it inside a subnet, which will be located **inside the Availability Zone** that you choose.

### Subnet best practices

- Use a public subnet for resources that must be connected to the internet and a private subnet for resources that won't be connected to the internet.
- When using a **VPC** for your application, consider [[High availability with VPC]].