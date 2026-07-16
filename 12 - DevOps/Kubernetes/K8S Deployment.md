A Deployment is a higher-level [[Kubernetes]] object that wraps around a [[Pod]] and adds features such as self-healing, scaling, zero-downtime rollouts, and versioned rollback.

Most of the time you'll deploy Pods indirectly via higher-level controllers. Examples of higher-level controllers include *Deployments*, *DaemonSets*, *StatefulSets*. Behind the scenes, they are implemented as [[K8S Control Plane#Controllers and Controllers Manager|Controllers]] that run as watch loops constantly observing the cluster making sure observed state matches desired state.

There are two major pieces to Deployments:
1. The spec
2. The controller

The Deployment spec is a declarative YAML object where you describe the desired state of a stateless app. Deployment controller implements and manages it. The controller aspect is highly-available and operates as a background loop reconciling observed state with desired state.

A Deployment object only manages a single Pod template. For example, an application with a front-end web service and a back-end catalog will have a different Pod for each (two Pod templates). As a result, it’ll need two Deployment objects – one managing front-end Pods, the other managing back-end Pods. However, a Deployments can manage multiple [[K8S ReplicaSet|replicas]] of the same Pod. For example, the front-end Deployment might be managing 5 identical front-end Pod replicas.

## Rolling Update

A **Rolling Update** is the default deployment strategy in Kubernetes, designed to update an application’s version without causing service interruptions. Instead of shutting down the entire application at once, Kubernetes performs a gradual transition. It incrementally replaces instances of the old version (the "old ReplicaSet") with instances of the new version (the "new ReplicaSet") until the entire deployment is running the updated code.

The process is governed by two key parameters: 
- `maxUnavailable` - defines how many pods can be taken offline simultaneously during the update
- `maxSurge`-defines how many extra pods can be created above the desired capacity. 

By adjusting these, you can control the speed of the rollout and ensure that your cluster always has enough capacity to handle incoming traffic.

During the update, Kubernetes monitors the health of the new pods using **readiness probes**. A new pod is only considered "ready" once it passes these checks, at which point the load balancer begins routing traffic to it. Only after the new pod is confirmed healthy does Kubernetes terminate one of the old pods. This "ready-check-then-terminate" loop continues until all old pods are replaced.

If something goes wrong during the rollout - such as the new version failing to start or crashing - Kubernetes automatically halts the process. Because the old pods are still running until their replacements are verified, the system remains stable. Furthermore, if the deployment fails, Kubernetes allows you to easily "roll back" to the previous stable state, making rolling updates a safe and reliable way to manage continuous delivery in production environments.