> ❯ `kubectl version` 
> Client Version: v1.36.2 
> Kustomize Version: v5.8.1 
> Server Version: v1.35.0+k3s1

I use docker via `colima`. My services do not auto-start, so I manually start them whenever needed, and then tear down to save resources.

## Bootstrap

```zsh
# For the first start after fresh installation
colima start --kubernetes --runtime containerd --cpu 4 --memory 6
# or after first start
colima start
```

I use k3s built-in into colima 

## Teardown

```zsh
colima stop
```

## Delete

```zsh
colima stoop
colima delete -f
rm -fr ~/.colima
```

## Why `colima`?

> **Co**ntainer **Li**nux **Ma**chine

**Docker cannot run natively on macOS.** Docker containers rely on Linux-specific kernel features (like cgroups and namespaces). macOS is Unix-based, but it is _not_ Linux. It doesn't have those features.

**The Solution:** You need a tiny Linux Virtual Machine (VM) running in the background to act as the "host" for your containers.

**What Colima does:** It automatically spins up a lightweight, invisible Linux VM (using a tool called `Lima`) and sets up the Docker daemon inside it. It then maps the `docker.sock` from that VM to your Mac, so when you type `docker ps` on your Mac terminal, it talks to the Docker engine running inside the Colima VM.