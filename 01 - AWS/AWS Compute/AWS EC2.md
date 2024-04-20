**Amazon EC2** is a web service that provides *secure*, *resizable* compute capacity in the cloud. With this service, you can provision virtual servers called **EC2 instances**.

**EC2 key features:**

- Provision and launch one or more EC2 instances in minutes.
- Stop or shut down EC2 instances when you finish running a workload.
- Pay by the hour or second for each instance type (minimum of 60 seconds).

## EC2 instance creation

To **create** an EC2 instance, you must define the following:

- **Hardware specifications:** CPU, memory, network, and storage
- **Logical configurations:** Networking location, firewall rules, authentication, and the operating system of your choice

When launching an EC2 instance, the first setting you configure is which **operating system** you want by selecting an [[AWS AMI|Amazon Machine Image (AMI)]]. EC2 and AMI have strong [[AWS AMI#EC2 and AMI relation|functional relation]].

During creation you should also choose the [[AWS EC2#Instance types|instance type]], [[AWS EC2#Network (instance location)|network]], and [[Storage and S3|storage]]**.

## Instance types

**EC2 instances** are a combination of *virtual processors* (vCPUs), *memory*, *network*, and, in some cases, instance *storage* and *graphics processing units* (GPUs). When you create an EC2 instance, you need to choose how much you need of each of these components.

To get an overview of the capacity details for a particular instance, you should look at the **instance type**. Instance types consist of a *prefix* identifying the type of workloads they’re optimized for, followed by a *size*. For example, the instance type `c5n.xlarge` can be broken down as follows:

- **First position –** The first position, **c**, indicates the **[[AWS EC2#Instance family|instance family]]**. This indicates that this instance belongs to the compute optimized family.
- **Second position –** The second position, **5**, indicates the **generation** of the instance. This instance belongs to the fifth generation of instances.
- **Remaining letters before the period –** In this case, **n** indicates **additional attributes**, such as local NVMe storage.
- **After the period –** After the period, **xlarge** indicates the **[[AWS EC2#Instance sizing|instance size]]**.

### Instance sizing

Your instance sizing will depend on both the demands of your application and the anticipated size of your user base. Forecasting server capacity for an [[Cloud#On-premises|on-premises]] application requires difficult decisions involving significant upfront capital spending. In contrast, changes to the allocation of your [[Cloud#Cloud|cloud-based]] services can be made with a simple API call. Because of the AWS pay-as-you-go model, you can match your infrastructure capacity to your application’s demand, instead of the other way around.

### Instance family

Each instance family is optimized to fit different use cases. AWS has:

- **General purpose** - for apps that use resources in equal proportions.
- **Compute optimized** - high-performance processors.
- **Memory optimized** - for apps with demand to top RAM speed, like caches, high-performance DBs, etc.
- **Accelerated computing** - Accelerated computing instances use hardware accelerators or co-processors to perform functions such as floating-point number calculations, graphics processing, or data pattern matching more efficiently, used for Machine learning, etc.
- **Storage optimized** - high sequential read and write access to large datasets on local storage, for NoSQL DBs, in-memory DBs, Elasticsearch, etc.
- **HPC Optimized** - High performance computing (HPC) instances are purpose built to offer the best price performance for running HPC workloads at scale on AWS. For complex simulations and deep learning workloads.

## Network (instance location)

Unless otherwise specified, when you launch EC2 instances, they are placed in a default [[Virtual Private Cloud|virtual private cloud (VPC)]]. The **default VPC** is suitable for getting started quickly and launching public EC2 instances without having to create and configure your own VPC.

Any resource that you put inside the **default VPC** will be public and accessible by the internet, so **you shouldn’t place** any customer data or private information in it.

## EC2 high availability

When architecting any application for **high availability**, consider using at least two EC2 instances in two separate [[Availability Zone|Availability Zones]]. Although EC2 instances are typically reliable, two are better than one, and three are better than two. Specifying the [[AWS EC2#Instance sizing|instance size]] gives you an advantage when designing your architecture because you can use more smaller instances rather than a few larger ones.

