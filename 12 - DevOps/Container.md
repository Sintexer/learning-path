
**What is a container?** Imagine a container as a lightweight, isolated environment that encapsulates an **application** and **all its dependencies** (libraries, configuration files, etc.). It's like a **self-contained package** that allows the application to *run consistently across different environments*, regardless of the underlying infrastructure. It is important to [[Difference between VMs and containers|differentiate VMs and containers]]. Containers share the *same operating system and kernel* as the host that they exist on. But [[Virtual machine|virtual machines]] contain their *own operating system*.

**Key characteristics:**

* **Isolation:** Each container runs in its own isolated space, preventing conflicts with other applications or the host system.
* **Portability:** Containers are designed to be portable, meaning they can be easily moved between different machines without requiring any modifications.
* **Lightweight:** Compared to virtual machines, containers have a much smaller footprint, making them more efficient in terms of resource utilization.
* **Scalability:** Containers can be easily scaled up or down to meet changing application demands.

In the real world it is hard to manage containers manually. Various [[Containers orchestration|orchestration services]] like [[AWS Elastic Container Service|Amazon ECS]] or [[Kubernetes|K8S]] are designed to help with hundreds of containers running simultaneously, spin up and shut down additional instances on demand, and even enable [[Service discovery|service discovery]].

### Use cases for containers:

* **Microservices architecture:** Containers are ideal for building and deploying microservices, which are small, independent services that communicate with each other.
* **Continuous integration/continuous delivery (CI/CD):** Containers streamline the CI/CD process by providing a consistent and portable environment for building, testing, and deploying applications.
* **Cloud-native applications:** Containers are a perfect fit for cloud-native applications, which are designed to be deployed and managed in cloud environments.

### How do containers work?

Containers leverage a technology called **containerization**, which allows multiple isolated user spaces to run on a single operating system (OS) kernel. This is achieved through technologies like **cgroups** (control groups) and **namespaces**, which provide resource isolation and process isolation, respectively.

### Benefits of using containers

* **Faster deployment:** Containers boot up quickly, enabling faster application deployment and updates.
* **Improved resource utilization:** Due to their lightweight nature, containers consume fewer resources compared to virtual machines.
* **Increased consistency:** Containers ensure consistent application behavior across different environments, promoting reliability and predictability.
* **Enhanced scalability:** Containers can be easily scaled up or down to meet fluctuating application demands.