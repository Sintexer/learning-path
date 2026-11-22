---
flashcards:
  q-a724: { nid: 1795356714016, hash: 2khur486, sync: c6isedpu }
  q-zs6f: { nid: 1795356714041, hash: 3t2dkdkq, sync: n4chtwx9 }
  q-35pu: { nid: 1795356714067, hash: dadudrrm, sync: j23xtkdh }
---

> ❯ `kubectl version`
> Client Version: v1.36.2
> Kustomize Version: v5.8.1
> Server Version: v1.35.0+k3s1  *(within ±1 minor — see [[K8S Version Skew]])*

I use containers via `colima` with built-in **K3s** (lightweight [[Kubernetes]]). Services do not auto-start, so I start them manually when needed and tear them down afterward to save RAM/battery.

## Bootstrap

```zsh
# First start after fresh installation
colima start --kubernetes --runtime containerd --cpu 4 --memory 6

# Subsequent starts
colima start
```

## Teardown

```zsh
colima stop
```

## Full Reset / Delete

```zsh
colima stop
colima delete -f
rm -rf ~/.colima
```

## Why `colima` on macOS?

> **Co**ntainer **Li**nux **Ma**chine

**Linux [[Container|containers]] cannot run natively on macOS.** Containers rely on Linux kernel primitives (**cgroups** and **namespaces**). macOS is Unix-based (Darwin/XNU), not Linux, so it lacks those kernel features.

**The Solution:** Run a lightweight Linux [[Virtual machine|VM]] in the background as the container host.

**What Colima does:** It spins up a minimal Linux VM (powered by `Lima` on Apple Virtualization / QEMU), runs [[containerd]] (or Docker) + K3s inside that VM, and forwards the container runtime socket and `kubeconfig` to macOS so local `docker` and `[[kubectl]]` commands transparently talk to the VM.

> [!tip] Production reality
> Local K3s/Colima/kind clusters are great for manifest validation and fast iteration, but beware differences from production: `local-path-provisioner` behaves differently from cloud CSI block storage, single-node clusters hide cross-node networking/anti-affinity bugs, and ARM64 (Apple Silicon) builds need `--platform linux/amd64` (or multi-arch builds) before pushing to x86 cloud nodes.

## Flashcards

Why can't Linux [[Container|containers]] run natively on macOS without a tool like `colima` or Docker Desktop?::Containers rely on Linux-specific kernel primitives (==cgroups and Linux namespaces==) that macOS's Darwin/XNU kernel does not have, so a lightweight Linux [[Virtual machine|VM]] is required.
^q-zs6f
^q-a724

What is a common pitfall when building container images on Apple Silicon (ARM64) laptops for cloud [[Kubernetes]] clusters?::Building an `arm64`-only image that fails with `exec format error` on `amd64` worker nodes; build multi-arch images (`docker buildx --platform linux/amd64,linux/arm64`) in CI.
^q-35pu

## Related
- [[Kubernetes]]
- [[K8S Core Containers]]
- [[K8S Version Skew]]
- [[containerd]]
- [[Container]]
- [[Virtual machine]]
- [[kubectl]]
