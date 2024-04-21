## EC2 high availability

When architecting any application for **high availability**, consider using at least two EC2 instances in two separate [[Availability Zone|Availability Zones]]. Although EC2 instances are typically reliable, two are better than one, and three are better than two. Specifying the [[AWS EC2#Instance sizing|instance size]] gives you an advantage when designing your architecture because you can use more smaller instances rather than a few larger ones.

