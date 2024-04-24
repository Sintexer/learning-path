> AKA **Gateway Load Balancing**

Works on [[OSI 3 layer]] + [[OSI 4 layer]] - IP. One implementation of [[AWS ELB]].

**Target type**: IP, [[EC2]]

A Gateway Load Balancer helps you to deploy, scale, and manage your third-party appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems. It provides a gateway for distributing traffic across multiple virtual appliances while scaling them up and down based on demand.

It is **highly available**, could be monitored via [[AWS CloudWatch]] metrics. Connects internet gateways, [[AWS VPC|VPCs]], and other network resources over a private network.