A [[Kubernetes]] *control plane node* is a server running collection of system services that make up the control plane of the cluster. Sometimes we call them *Masters*, *Heads* or *Head nodes*.

The simplest setups run a single conrol plane node. However, prod environments run multiple control plane nodes configured for high availability (HA). 3 or 5 is recommended for HA. It is a good practice not to run user applications on control plane nodes.

## API Server

It is the Grand Central of Kubernetes. **All communication, between all components, must go through the API server**. 

It exposes a RESTful API that you POST YAML configuration files to over HTTPS. These YAML files, which sometimes call *manifests*, describe the *[[Pod Desired State|desired state]]* of an application.

All requests to the API server are subject to authentication and authorization checks. Once these are done, the config in the YAML file is validated, persisted to the cluster store, and work is scheduled to the cluster.

## Cluster store

The cluster store is the only *stateful* part of the control plane and persistently stores the entire configuration and state of the cluster. As such, it's a vital component of every Kubernetes cluster - no store, no cluster. 

Cluster store is currently based on [[etcd]]. Hence this is the single source of truth for a cluster, you should run between 3-5 etcd replicas for HA. A default installation of Kubernetes installs a replica of the cluster store on every control plane node with HA auto-configured.

## Controllers and Controllers Manager

Controller manager is a controller of controllers: it spawns all the independent controllers and monitors them. 

Some of the controllers include the Deployment controller, the StatefulSet controller, and the ReplicaSet controller.  Each one is responsible for a small subset of cluster intelligence and runs as a background wath-loop constantly watching the API Server for changes.

> The aim of a controller is to ensure the *observed state* of the cluster matches the *desired state*

The logic implemented by each controller is as follows, and is at the heart of Kubernetes and declarative design patters:
1. Obtain desired state
2. Observer current state
3. Determine differences
4. Reconcile differences

Each controller is also extremely specialized and only interested in its own little corner of the Kubernetes cluster. No attempt is made to over-complicate design by implementing awareness of other parts of the system - each controller takes care of its own business and leaves everything else alone. This is key to the distributed design of Kubernetes and adheres to the Unix philosophy of building complex systems from small specialized parts.

> Terms like *controller*, *control loop*, *watch loop* and *reconciliation loop* mean the same thing

## Scheduler

Scheduler watches the [[#API Server]] for new work tasks and assigns them to appropriate healthy worker nodes. He does this by filtering out invalid or unavailable nodes and ranking whats left with a sophisticated ranking algorithm.

If the scheduler doesn't find a suitable node, the task isn't scheduled and gets marked as pending.