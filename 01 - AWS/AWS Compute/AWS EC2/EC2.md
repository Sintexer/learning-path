**Amazon EC2** is a web service that provides *secure*, *resizable* compute capacity in the cloud. With this service, you can provision virtual servers called **EC2 instances**.

**EC2 key features:**

- Provision and launch one or more EC2 instances **in minutes**.
- **Stop or shut down** EC2 instances when you finish running a workload.
- **Pay-as-you-go**: Pay by the hour or second for each instance type (minimum of 60 seconds).
- EC2 is **elastic**,  which means you can add or remove instances easily based on current demand.

EC2 instances have a [[EC2 lifecycle|defined lifecycle]], encompassing various states they can transition through over time.

## Orchestration

EC2 instances might be grouped into a cluster with an [[Containers orchestration|orchestration service]] to run containers in an elastic, highly-available environment. To do so it is required to install [[AWS ECS Container agent]] on all instances.
## EC2 instance creation

To **create** an EC2 instance, you must define the following:

- **Hardware specifications:** CPU, memory, network, and storage
- **Logical configurations:** Networking location, firewall rules, authentication, and the operating system of your choice

When launching an EC2 instance, the first setting you configure is which **operating system** you want by selecting an [[AWS AMI|Amazon Machine Image (AMI)]]. EC2 and AMI have strong [[AWS AMI#EC2 and AMI relation|functional relation]].

**Initialization script:** You can equip your EC2 instance with a custom initialization script that will automatically run after the instance boots up. This script allows you to configure and customize your instance according to your specific needs. For instance, in simple use cases such script could download your application and all it dependencies. But generally it should be used to install third-party Linux packages, vital for the application.

During creation you should also choose the [[EC2 instance type|instance type]], [[EC2 instance network|network]], and [[EC2 storage|storage]]**.