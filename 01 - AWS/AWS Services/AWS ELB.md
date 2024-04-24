> AKA **Elastic Load Balancer**

ELB may seem a single point of failure, but it's a wrong impression. ELB is [[AWS Managed Service]] and it is highly available, [[Region|regional]] and automatically scalable by design.

There are three types of Load Balancers:

- **Application Load Balancer [[AWS ALB]]** - for [[HTTP]]/[[HTTPS]] traffic [[OSI 7 layer]]
- **Network Load Balancer [[AWS NLB]]** - for [[TCP]]/[[UDP]]/[[TLS]] - [[OSI 4 layer]]
- **Gateway Load Balancer [[AWS GLB]]** - [[OSI 3 layer]] + [[OSI 4 layer]] - IP

All ELB implementation do **preserve client source IP address**.

Each load balancer might be **Internet-facing** or **Internal-facing**. Internet-facing is four internet users traffic, while internal-facing is for private-to-private requests load balancing (e.g. between [[EC2]])

Monitoring is an important part of load balancers because they should route traffic to only healthy EC2 instances. That’s why ELB supports two types of health checks as follows:

- Establishing a connection to a backend EC2 instance using TCP and marking the instance as available if the connection is successful.
- Making an HTTP or HTTPS request to a webpage that you specify and validating that an HTTP response code is returned.

If Amazon [[EC2 Auto Scaling]] has a scaling policy that calls for a scale down action, it informs ELB that the EC2 instance will be terminated. ELB can prevent Amazon EC2 Auto Scaling from terminating an EC2 instance until all connections to the instance end. It also prevents any new connections. This feature is called **connection draining**.

## ELB components

The ELB service is made up of three main components: **rules**, **listeners**, and **target groups**.

**Rule** associates target group with a listener. Based on source IP of the user rule decides which target group to send the traffic to.

**Listener** is what a client connects to. Listener is defined by port and protocol, one load balancer might have many listeners.

**Target group** contains one or more backend servers. You define EC2 instances, Lambda functions or IP addresses to the target group. A health check must be defined to each target group.
[[AWS ELB]]