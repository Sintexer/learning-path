**`kubectl`** is the official command-line client for interacting with the [[Kubernetes]] [[K8S Control Plane#API Server|API Server]] over [[HTTPS]]. Its supported version skew is **±1 minor version** relative to `kube-apiserver` (see [[K8S Version Skew]]).

## Context & `kubeconfig`

`kubectl` reads `~/.kube/config` (or `$KUBECONFIG`), which bundles three sections:
- **clusters**: API server URLs and CA certificates.
- **users**: credentials (client certs, OIDC/[[JWT]] tokens, exec plugins like `aws eks get-token`).
- **contexts**: named tuples combining `(cluster, user, default namespace)`.

```bash
kubectl config get-contexts
kubectl config use-context prod-eks
kubectl config set-context --current --namespace=payments
```

## Essential Production Commands

- **Inspect & debug**:
  - `kubectl get <type> <name> -o wide|yaml` — list or inspect resources.
  - `kubectl describe <type> <name>` — detailed status + recent **Events** (first place to check for `Pending`, `ImagePullBackOff`, or `OOMKilled`).
  - `kubectl logs <pod> -c <container> --previous` — view logs from the *previous crashed* container instance.
  - `kubectl exec -it <pod> -- /bin/sh` — interactive shell (or `kubectl debug` for distroless containers lacking a shell!).
  - `kubectl top pods|nodes` — live CPU/RAM usage (requires `metrics-server`, see [[K8S Core Containers]]).
- **Apply & rollouts**:
  - `kubectl apply -f manifest.yaml` — declarative create/update (preferred over imperative `create`/`edit`).
  - `kubectl diff -f manifest.yaml` — dry-run server-side diff before applying.
  - `kubectl rollout status|history|undo deployment/<name>` — monitor or roll back a [[K8S Deployment]].
  - `kubectl scale deployment/<name> --replicas=<n>` — adjust replica count.
- **Node maintenance**:
  - `kubectl cordon <node>` — mark node unschedulable.
  - `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` — gracefully evict Pods respecting `PodDisruptionBudget`.

> [!warning] Failure modes & trade-offs
> - **Wrong context disaster**: running `kubectl delete` or `apply` against `prod` while thinking you're on `dev` is a classic outage—display active context/namespace in your shell prompt (`kubectx`/`kubens`, Starship) and restrict prod `kubeconfig` to read-only by default (use GitOps / CI for mutations).
> - **Distroless containers & `kubectl exec`**: hardened production images have no `/bin/sh` or `curl`; use **`kubectl debug -it <pod> --image=nicolaka/netshoot --target=<container>`** to attach an ephemeral debug container sharing the Pod's namespaces.

## Flashcards

What three components are combined inside a `kubectl` **context** in `kubeconfig`?::A ==cluster== (API server URL + CA), a ==user== (credentials), and a default ==namespace==.

Which `kubectl` command lets you inspect logs from a container that already crashed and restarted (`CrashLoopBackOff`)?::`kubectl logs <pod> --previous`

How do you debug a running distroless [[Pod]] that has no `/bin/sh`, `curl`, or diagnostic tools installed?::Attach an ephemeral debug container sharing the Pod's network/process namespace with ==`kubectl debug -it <pod> --image=... --target=<container>`==.

Always run `kubectl diff -f` and verify your active context before applying manual changes to a production cluster. #card-reminder

## Related
- [[Kubernetes]]
- [[K8S Control Plane]]
- [[K8S Version Skew]]
- [[K8S Deployment]]
- [[Pod]]
- [[Running Kubernetes Cluster Locally]]
