It is hard to [[Horizontal scaling|horizontally]] or [[Vertical scaling|vertically]] scale instances. With a traditional approach to [[Scaling|scaling]], you buy and provision enough servers to handle traffic at its peak. However, this means at nighttime, for example, you might have more capacity than traffic, which means you’re wasting money. Turning off your servers at night or at times when the traffic is lower only saves on electricity.

The cloud works differently with a pay-as-you-go model. You must turn off the unused services, especially [[EC2]] instances you pay for on-demand. You can manually add and remove servers at a predicted time. But with unusual spikes in traffic, this solution leads to a waste of resources with over-provisioning or a loss of customers because of under-provisioning.

The Amazon EC2 Auto Scaling service adds and removes capacity to keep a steady and predictable performance at the lowest possible cost. By adjusting the capacity to exactly what your application uses, you only pay for what your application needs. This means Amazon EC2 Auto Scaling helps scale your infrastructure and ensure high availability.

## Features

- Automatic scaling
- Scheduled scaling
- Fleet management - automatically replaces unhealthy instances
- Predictive scaling - uses ML to optimize scaling volumes
- Purchase options - includes multiple purchase models, [[EC2 instance type|instance types]] and [[Availability Zone]]

Additionally, the [[AWS ELB]] service integrates seamlessly with Amazon EC2 Auto Scaling. As soon as a new EC2 instance is added to or removed from the Amazon EC2 Auto Scaling group, ELB is notified. However, before ELB can send traffic to a new EC2 instance, it needs to validate that the application running on the EC2 instance is available.

## Components

There are three main components of Amazon EC2 Auto Scaling. Each of these components addresses one main question as follows:

- **Launch template or configuration:** Which resources should be automatically scaled?
- **Amazon EC2 Auto Scaling groups:** Where should the resources be deployed?
- **Scaling policies:** When should the resources be added or removed?

### Launch templates and configurations

Multiple parameters are required to create EC2 instances - [[AWS AMI]] ID, [[EC2 instance type]], [[AWS Security groups|security group]], additional [[AWS EBS|EBS volumes]], and more. All this information is also required by Amazon EC2 Auto Scaling to create the EC2 instance on your behalf when there is a need to scale. This information is stored in a **launch template**.  
  
You can use a launch template to manually launch an EC2 instance or for use with Amazon EC2 Auto Scaling. It also supports *versioning*, which can be used for quickly rolling back if there's an issue or a need to specify a default version of the template. This way, while iterating on a new version, other users can continue launching EC2 instances using the default version until you make the necessary changes.

### Amazon EC2 Auto Scaling groups

An Auto Scaling group helps you define where Amazon EC2 Auto Scaling deploys your resources. This is where you specify the [[AWS VPC]] and [[Subnet|subnets]] the EC2 instance should be launched in. Amazon EC2 Auto Scaling takes care of creating the EC2 instances across the subnets, so select at least two subnets that are across different [[Availability Zone|Availability Zones]].  
  
With Auto Scaling groups, you can specify the *type of purchase* for the EC2 instances. **You can use On-Demand Instances or Spot Instances.** You can also use a combination of the two, which means you can take advantage of Spot Instances with minimal administrative overhead.

To specify how many instances Amazon EC2 Auto Scaling should launch, you have three capacity settings to configure for the group size: **Min capacity**, **Desired capacity** and **Maximum capacity**.

### Scaling policies

By default, an Auto Scaling group will be kept to its initial desired capacity. While it’s possible to manually change the desired capacity, you can also use **scaling policies**.  
  
You use [[AWS CloudWatch]] metrics to keep information about different attributes of your EC2 instance, such as the CPU percentage. You use alarms to specify an action when a threshold is reached. Metrics and alarms are what scaling policies use to know when to act. For example, you can set up an alarm that states when the CPU utilization is above 70 percent across the entire fleet of EC2 instances. It will then invoke a scaling policy to add an EC2 instance.  
  
Three types of scaling policies are available: **simple**, **step**, and **target tracking** scaling.

**Simple scaling policy** does what said above, it also has a cool down after scaling performed. 

**Step scaling policy** respond to additional alarms even when a scaling activity or health check replacement is in progress. E.g. add 1 instance if instance group CPU utilization > 85% and 4 instances if CPU utilization > 95%.

**Target tracking policy** - If your application scales based on average CPU utilization, average network utilization (in or out), or request count, then this scaling policy type is the one to use. All you need to provide is the target value to track, and it automatically creates the required CloudWatch alarms.