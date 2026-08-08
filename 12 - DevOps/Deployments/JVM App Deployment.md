
To correctly build the robust and [[Zero-downtime deployment]] for a JVM application, one would need to consider correctly plug-in the bootstrap and shutdown semntics of the JVM world into the [[Kubernetes]]/[[Docker]] lifecycle.

## Readiness Probes

JVM apps take time to warmup JVM, load Spring context, and initialize DB pools. If a system begins routing traffic to a booting pod, we will cause failed requests and increased response wait times.

The solution is to set-up actuator, and point Kubernetes to an endpoint `/actuator/health/readiness`. It answers the question *Is the Pod ready to accept HTTP traffic*? Kubernetes won't send any traffic to a pod until it returns 200 OK.

## Liveness Probe

We can also benefit from `actuator/health/liveness` endpoint. Kubernetes might inspect it to answer the question *Is this pod alive or in a deadlock/OOM state?*

## Gracefull shutdown

If we run the upgrade procedure in our cluster, Kubernetes will eventually send a SIGTERM to our app. By default, Spring immediately shuts down. This means any running request will be terminated. To avoid that we can specify the `server.shutdown=graceful` in application.yaml. This way Spring server will wait for any running request to finish until terminating.

But be careful, this does not save your app from receiving new requests. Though this could easily be solved with a kubernetes: use PreStop hook to remove the pod from IP endpoints routing table before the Spring process starts shutting down.