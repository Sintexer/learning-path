> AKA **Network Load Balancer**

Used for [[TCP]]/[[UDP]]/[[TLS]] - [[OSI 4 layer]]. One implementation of [[AWS ELB]].

**Target type:** IP, [[EC2]], [[AWS ALB]]

A Network Load Balancer is ideal for load balancing [[TCP]] and [[UDP]] traffic. It functions at Layer 4 of the OSI model, routing connections from a target in the target group based on IP protocol data.

Has **sticky sessions** - routes requests for the same targets. Offers **low latency**. Preserves the client-side **source IP address**. Automatically provides a static IP address per [[Availability Zone]] (subnet) (**Static IP support**). Lets users assign a custom, fixed IP address per Availability Zone (**Elastic IP support**). Uses [[AWS Route 53]] to direct traffic to load balancer nodes in other zones.