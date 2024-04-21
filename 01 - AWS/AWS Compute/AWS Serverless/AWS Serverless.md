
If you run your code on [[EC2|Amazon EC2]], AWS is responsible for the **physical hardware**. You are still responsible for the **logical controls**, such as the *guest operating system*, *security* and *patching*, *networking*, and *scaling*.  
  
As covered in the previous Container Services lesson, you choose to have more control by running and managing your [[Container|containers]] on [[AWS Elastic Container Service|Amazon ECS]] and [[AWS Elastic Kubernetes Service|Amazon EKS]]. By doing so, AWS is still responsible for more of the container management, such as deploying containers across [[EC2|EC2 instances]] and managing the container cluster. However, when running Amazon ECS and Amazon EKS on Amazon EC2, you are still responsible for **maintaining the underlying EC2 instances**.  
  
If you want to deploy your workloads and applications without having to manage any EC2 instances, you can do that on AWS with ***serverless compute***.