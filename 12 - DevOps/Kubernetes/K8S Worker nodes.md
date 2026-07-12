Nodes are serves that are the workers of a [[Kubernetes]] cluster.

1. Watch the API server for new work assignments
2. Execute work assignments
3. Report back to the [[K8S Control Plane]].

## Kubelet

The kubelet is main Kubernetes agent and runs on every cluster node. In fact, it's common to use the terms *node* and *kubelet* interchangeably.

When you join a node to a cluster, the process installs the kubelet, which is then responsible for registering it with the cluster. This process registers the node's CPU, memory, and storage into the wider cluster pool.

One of the main jobs of the kubelet is to watch the [[K8S Control Plane#API Server|API Server]] for new work tasks. If a kubelet can't run a task, it reports back to the control plane and let's the control plane decide what actions to take.

## Container runtime

The [[#Kubelet]] needs a container runtime to perform container-related tasks - things like pulling images and starting and stopping containers. K8S uses [[containerd]] over [[Container Runtime Interface|CRI]].

## Kube-proxy

Kube-proxy runs on every node and is responsible for local cluster networking. It ensures each node gets its own unique IP address, and it implements local iptables or IPVS rules to handle routing and load-balancing of traffic on the Pod network.