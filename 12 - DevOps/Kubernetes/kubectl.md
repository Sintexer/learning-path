
TBD

## Context

`kubectl` context is a bunch of configs that tell `kubectl` which cluster to send commands to, and which credentials to authenticate with.

## Commands

 - `kubectl get <type> <name>`  - get resources by type and (optionally) name. 
 - `kubectl describe <type> <name>` - get details for the specific resource
 - `kubectl scale deployment <name> --replicas <number>` scale the deployment vi replica set
 - `kubectl rollout <action> <type> <name>` - control the reconciliation