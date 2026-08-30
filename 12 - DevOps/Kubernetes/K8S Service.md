[[K8S Deployment]] runs [[Pod]]s in replica sets, perform scaling, healing. But the app client cannot talk directly to pods - pods are ephemeral and could go up and down and hence change ip. There is when services come in play.

Service is a stable network abstraction point that provides TCP and UDP load-balancing across a dynamic set of pods.

Kubernetes Service is a fully-fledged object in the Kubernetes API - like [[Pod]]s and [[K8S Deployment]]s. They have front-end consisting of a stable DNS name, ip address, and port. On the back-end they load-balance traffic across a dynamic set of Pods. As Pods come and go, the Service observes this, automatically updates itself, and continues to provide that stable networking endpoint. The same applies if you scale the number of Pods up or down.

As Services operate at the TCP and UDP layer, they don't posses *application intelligence*. This means they cannot provide application-layer host and path routing. For that, you need a *[[K8S Gateway]]*, which understands HTTP and provides host and path-based routing.