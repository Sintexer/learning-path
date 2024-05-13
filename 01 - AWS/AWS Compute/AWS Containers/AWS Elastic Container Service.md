
> AKA [[AWS ECS]]

AWS ECS is an end-to-end [[Containers orchestration|Container orchestration service]]. This means that **Amazon ECS** handles the entire lifecycle of your [[Container|containerized]] applications, from initial deployment to ongoing management and scaling. With **Amazon ECS**, your containers are defined in a **task definition** that you use to run an individual task or a task within a service. You have the option to run your tasks and services on a *serverless* infrastructure that's managed by another AWS service called [[AWS Fargate]]. Alternatively, for more control over your infrastructure, you can run your tasks and services on a cluster of [[EC2|EC2 instances]] that you manage.

ECS brings four new terms that are important to understand:

- **Cluster** - it's a logical grouping of services, tasks, capacity providers in a region.
- **Service** - one or more identical tasks, checks and replaces unhealthy tasks.
- **Task** - one or more containers, specifies compute (EC2, EC2 spot, Fargate, Fargate Spot), networking, [[IAM]], config.

When the Amazon ECS container instances are up and running, you can perform actions that include, but are not limited to, the following:

- **Launching** and **stopping** containers
- Getting cluster **state**
- **Scaling** in and out
- **Scheduling** the placement of containers across your cluster
- Assigning **permissions**
- Meeting **availability** requirements
### Task definition

To prepare your application to run on **Amazon ECS**, you create a **task definition**. The task definition is a text file, in *JSON format*, that describes one or more [[Container|containers]]. A task definition is similar to *a blueprint* that describes the resources that you need to run a container, such as **CPU**, **memory,** **ports**, **images**, **storage**, and **networking information**.  
  
Here is a simple task definition that you can use for your corporate directory application. In this example, this runs on the Nginx web server.

```json
{
  "family": "webserver",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "nginx",
      "memory": "100",
      "cpu": "99"
    }
  ],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "networkMode": "awsvpc",
  "memory": "512",
  "cpu": "256"
}
```