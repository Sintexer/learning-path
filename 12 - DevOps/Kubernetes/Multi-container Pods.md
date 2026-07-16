It is a powerful [[Pod]] pattern, because at the very high-level, each container should have a single clearly defined responsibility. When there is an idea to tightly couple two or more functions, we can benefit from multi-container pods. They might implement several well-known patterns:

1. Sidecar pattern
2. Adapter pattern
3. Ambassador pattern
4. Init pattern

## Sidecar multi-container Pods

Probably the most popular pattern. It has a main application container, and a sidecar container. For example, one could serve a web page, and the other (sidecar) sync container could pull the newest page version in background.

## Adapter multi-container Pods

This is a special variation of the sidecar pattern where one container acts as a input-output mapper for the main container. It might take non-standardized output from the main container and route it to a target in a converted format.

## Ambassador multi-container Pods

Another variation of sidecar. This time, the helper container brokers connectivity to an external system. For example, the main application container can just dump its output to a port the ambassador container is listening on and sit back while the ambassador container does the hard work of getting it to external system.

## Init multi-container Pods

It runs a special *init container* that's guaranteed to start and complete **before** your main application container. It's also guaranteed to only run once.