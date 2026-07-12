Pod is a construct for running one or more containers. Pods themselves don't actually run applications - applications always run in containers, the Pod is just a sandbox to run one or more containers. 

Pods ring-fence an area of the host OS, build a network stack, create a bunch of kernel namespaces, and run one or more containers. Multiple containers in a pod share network stack, IP address, volumes, IPC namespace, shared memory, and more.

Usually each pod runs one container. However, advanced use-cases of multi-container Pods include:
- Service meshes
- Web containers supported by a helper container pulling updated content
- Container with a tightly coupled log scraper