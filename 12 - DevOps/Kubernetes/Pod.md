Pod is a construct for running one or more containers. Pods themselves don't actually run applications - applications always run in containers, the Pod is just a sandbox to run one or more containers. 

Pods ring-fence an area of the host OS, build a network stack, create a bunch of kernel namespaces, and run one or [[Multi-container Pods|more containers]]. Multiple containers in a pod share network stack, IP address, volumes, IPC (Inter-process communication) namespace, shared memory, and more. Two containers in the same Pod can talk to each other using the Pod's `localhost`.

> [!tip] Scaling
> If you need to scale an app, you add or remove Pods. You **do not** scale by adding more containers to existing pods.

Usually each pod runs one container. However, advanced use-cases of [[Multi-container Pods]] include:
- Service meshes
- Web containers supported by a helper container pulling updated content
- Container with a tightly coupled log scraper

The deployment of a Pod is an atomic operation. This means a Pod is only ready for service when all its containers are up and running. The entire Pod either comes up and is put into service, or it doesn't, and it fails.

A single Pod can only be scheduled a to a single node. - you cannot schedule a single Pod across multiple nodes. This is also true of multip-container Pods - all containers in the same Pod run on the same node.

Pods are created, they live an, and they die. A new one is started in place of a dead one. This has implications on how you design your applications. Pods might change their IP and ID.

Pods are **immutable** - this means you don't change them once they're running. Once a pod is running, you never change its configuration. Any update or replacement operation is done by deleting the old one and replacing it with a new one.

Pods augment containers, pods assist scheduling, pods enable resource sharing. 

Pods add: 
- labels and annotations
- restart policies
- probes
- affinity and anti-affinity rules
- termination control
- security policies
- resource policies and limits

Pods introduce a **shared execution environment**: all containers within a pod share net namespace (ip address, port range, routing table), pid namespace (isolated process tree), mnt namespace (filesystems and volumes), UTS namespace (hostname), icp namespace (unix domain sockets and shared memory).

> [!tip] Fun fact
> Pod is a special type of container called a "pause container", if you are using docker or [[containerd]]

## Pods and shared networking

Each pod has its own network namespace. Pos has its own IP, range of TCP and UDP ports, and a single routing table.

Each pod has a unique IP in the *pod network*. Network is flat, meaning every Pod can talk to every other Pod without the need for complex port mappings. In a default out-of-the box cluster, The Pod network is wide open from a security perspective.

## Pod Lifecycle

1. `pending` phase - pod is declared through the yaml and waiting for scheduling
2. `running` phase - all containers are pulled and running
3. `succeeded` phase - for a short running pods, they end up successfully

## Static pod vs controllers

Pods deployed directly from manifest are called *static pods*. They do not offer self-healing, scaling and other features that controllers bring.