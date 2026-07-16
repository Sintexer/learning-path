Namespaces are a native way to divide a single [[Kubernetes]] cluster into multiple *virtual clusters*. They are designed as an easy way to apply *quotas* and *policies* to groups of objects. They are not designed for strong workload isolation.

Most Kubernetes objects are deployed to a namespace. These objects are said to be *namespaced* and include common objects like [[Pod|Pods]], Services and Deployments. Other objects exist outside of Namespaces and include nodes and PodSecurityPolicies. If an object is deployed without explicitly defining the target Namespace, it will be deployed to the *default* Namespace.

