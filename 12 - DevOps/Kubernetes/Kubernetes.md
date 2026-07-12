---
aliases:
  - K8S
---
> v1.0 Since 2015, donated to CNCF in 2014, pronounced as *koo-ber-net-eez* or K8s as *kates*

Kubernetes is a service that provides containers orchestration.

Kubernetes deploys and manages (orchestrates) applications that are packaged and run as containers (containerized) and that are built in ways (cloud-native microservices) that allow them to scale, self-heal, and be updated in-line with modern cloud-like requirements.

Kubernetes and [[Docker]] are complementary technologies.

Kubernetes performs high-level orchestration tasks as self-healing, scaling and rolling updates, but it needs a tool like Docker to perform low-level tasks such as staring and stopping containers.

Kubernetes might work with different container runtimes, not limited to Docker (e.g. containerd, kata) through [[Container Runtime Interface|CRI]]

> Docker was a Kubernetes runtime. But it is deprecated since 1.20 for removal in favor of [[containerd]].

Docker Swarm and Mesosphere DCOS are former K8S competitors, while swarm is still under active development.

Kubernetes abstracts the cloud and hence the way we distribute resources between containers. It is like modern programming languages abstract developers from the memory addresses and cpu cores. 

---

## Kubernetes as a cluster

Kubernetes is like any other cluster - a bunch of machines to host applications. We call these machines **nodes**, and they can be physical servers, virtual machines, cloud instances, Raspberry Pis, and more.

Cluster is made of *control plane* and *nodes*.

## Kubernetes as an orchestrator

Orchestrator is a system that takes care of deploying and managing applications.

The K8s cluster is made of one or more *control plane* nodes and a bunch of *worker nodes*. Control plane nodes implement the cluster intelligence, worker nodes are where user applications run.

## Packaging apps for Kubernetes

An application needs to tick a few boxes to run on a Kubernetes cluster:

1. Packaged as a container
2. Wrapped in a Pod
3. Deployed via a declarative manifest file

While it's possible to run static Pods like this on a Kubernetes cluster, the preferred model is to deploy all Pods via higher-level controllers. The most common controller is the *Deployment*. It offers scalability, self-healing, and replicas to deploy and how to perform updates. 