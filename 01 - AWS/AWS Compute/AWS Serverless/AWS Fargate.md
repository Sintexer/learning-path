
Fargate abstracts the [[EC2|EC2 instance]] so that you’re not required to manage the underlying compute infrastructure. However, with **Fargate**, you can use all the same [[AWS Elastic Container Service|Amazon ECS]] concepts, APIs, and AWS integrations. It natively integrates with [[IAM|IAM]] and [[AWS VPC|Amazon Virtual Private Cloud (Amazon VPC)]]. With native integration with Amazon VPC, you can launch Fargate [[Container|containers]] inside your network and control connectivity to your applications.

AWS Fargate is a purpose-built [[Serverless|serverless]] compute **engine for containers**. AWS Fargate **scales** and **manages the infrastructure**, so developers can work on what they do best, application development. It achieves this by **allocating** the right amount of **compute**. This eliminates the need to choose and *manage EC2 instances*, *cluster capacity*, and *scaling*. Fargate supports both [[AWS Elastic Container Service|Amazon ECS]] and [[AWS Elastic Kubernetes Service|Amazon EKS]] architecture and provides workload isolation and **improved security** by design.

By migrating your application to Fargate, you become even more dependent on AWS and won't be able to switch cloud provider so easily.