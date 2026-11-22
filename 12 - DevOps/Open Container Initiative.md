---
aliases:
  - OCI
---

The **Open Container Initiative (OCI)** is a Linux Foundation governance body (founded in 2015 by Docker, CoreOS, and others) that defines open, vendor-neutral industry standards for [[Container|container]] formats and runtimes.

## The Three Core OCI Specifications

1. **Image Specification (`image-spec`)**: defines how a container image is packaged—a manifest, JSON config (entrypoint, env, user), and content-addressable filesystem layers (tarballs). Because of this spec, an image built with Docker, Buildah, or Kaniko runs seamlessly on [[containerd]], CRI-O, or Podman.
2. **Runtime Specification (`runtime-spec`)**: defines how to unpack a filesystem bundle and run a container process on the OS (implemented by low-level runtimes like **`runc`**, `crun`, or sandboxed runtimes like `gVisor` / Kata).
3. **Distribution Specification (`distribution-spec`)**: defines the HTTP API for pushing and pulling container images (and arbitrary OCI artifacts like Helm charts or Wasm modules) to/from registries (Docker Hub, ECR, GHCR, Harbor).

## How OCI Fits into [[Kubernetes]]

```text
kubelet --(CRI)--> containerd / CRI-O --(OCI runtime-spec)--> runc --> Linux container process
```

[[Container Runtime Interface|CRI]] standardizes how `kubelet` talks to a high-level runtime ([[containerd]]), while **OCI** standardizes the image format, registry protocol, and low-level container execution (`runc`).

> [!tip] Production reality
> Because of OCI standards, you are never locked into Docker Desktop or the Docker Engine in production—[[Kubernetes]] dropped `dockershim` in v1.24 while continuing to run standard "Docker" images because they are actually **OCI images**.

## Flashcards

OCI (Open Container Initiative):::Linux Foundation project defining open standards for [[Container]] images (`image-spec`), low-level execution (`runtime-spec`), and registry APIs (`distribution-spec`).

What are the three Open Container Initiative (OCI) specifications?::==Image Spec== (manifest + layers), ==Runtime Spec== (how `runc` spawns a container), and ==Distribution Spec== (registry push/pull API).

### How do CRI and OCI differ in the Kubernetes container stack? #card

[[Container Runtime Interface|CRI]] is the Kubernetes-specific gRPC API between `kubelet` and a high-level runtime ([[containerd]], CRI-O). **OCI** is the lower-level, orchestrator-agnostic standard for image formats, registries, and spawning the container process via `runc`.

## Related
- [[Container]]
- [[Container Runtime Interface]]
- [[containerd]]
- [[Kubernetes]]
- [[K8S Worker nodes]]
