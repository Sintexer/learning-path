The desired state is the manifest files written in simple YAML and tell Kubernetes what an application should look like. It includes things such as which image to use, how many replicas to run, which network ports to listen on, and how to perform updates.

The easiest way to post the manifest to the API server is to use the [[kubectl]] cli utility. Once this is done, any required work tasks get [[K8S Control Plane#Scheduler|scheduled]] to cluster nodes where the [[K8S Worker nodes#Kubelet|kubelet]] co-ordinates the hard work of pulling images, starting containers, attaching to networks, and starting application processes.

Controllers run as background reconciliation loops that constantly monitor the state of things. 

In [[Kubernetes]], the declarative model works like this:

1. Declare the desired state of an application microservice in a manifest file.
2. Post it to the [[K8S Control Plane#API Server]]
3. Kubernetes stores it in the [[K8S Control Plane#Cluster store]] as the application's desired state.
4. Kubernetes implements the desired state on the cluster.
5. A controller makes sure the observed state of the application doesn't vary from the desired state.