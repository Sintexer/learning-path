
> AKA **Amazon EKS**


**Amazon EKS** is a managed service that you can use to run [[Kubernetes]] on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes. **Amazon EKS** is conceptually similar to [[AWS Elastic Container Service|Amazon ECS]], but with the following differences:

- In **Amazon ECS**, the machine that runs the containers is an **EC2 instance** that has an [[AWS ECS Container agent|ECS agent]] installed and configured to run and manage your containers. This instance is called a **container instance**. In Amazon EKS, the machine that runs the containers is called a **worker node** or **Kubernetes node**. 
- An ECS container is called a **task**. An EKS container is called a **pod**.
- Amazon ECS runs on AWS *native technology*. Amazon EKS runs on **Kubernetes**. 

If you have containers running on *Kubernetes* and want an advanced [[Containers orchestration|orchestration solution]] that can provide *simplicity*, *high availability*, and fine-grained *control* over your infrastructure, **Amazon EKS** could be the tool for you.