Although running one instance is uncomplicated to manage, it lacks high availability and scalability. Most companies and organizations run many containers on many instances.
  
If you’re trying to manage your compute at a large scale, you should consider the following:
- How to place your containers on your instances
- What happens if your container fails
- What happens if your instance fails
- How to monitor deployments of your containers

This coordination is handled by a **container orchestration service**. Most popular choice for today is [[Kubernetes|K8S]]. Cloud platforms also offer their own orchestration services. AWS offers two container orchestration services: [[AWS Elastic Container Service|Amazon Elastic Container Service (Amazon ECS)]] and [[|Amazon Elastic Kubernetes Service (Amazon EKS)]].