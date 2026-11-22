---
flashcards:
  q-pgci: { nid: 1795356731518, hash: m297rted, sync: xhbcnjxi }
  q-ux7f: { nid: 1795356731542, hash: ugn5mtqc, sync: k5p5xcw8 }
---

# JVM args

## Memory and heap related

- `-Xss` - thread stack size (e.g. `512k`, `1m`)
- `-Xms` - initial heap size
- `-Xmx` - maximum possible heap size
- `-XX:MaxRAMPercentage=75.0` - container-aware heap cap as a percentage of cgroup memory limit (preferred over hardcoded `-Xmx` in [[Container|containers]] / [[Kubernetes]])
- `-XX:MinHeapFreeRatio` - Min heap space percentage for every heap zone. If free space in a zone is less than this percentage, additional space will be attached to this zone.
- `-XX:MaxHeapFreeRatio` - Max busy heap space percentage for every zone. If exceeded - additional space will be allocated for a zone.
- `-XX:NewRatio` - 1 new to x old space ratio. E.g. 3 means new space is 1/3 to the old gen space
- `-XX:NewSize` - New gen size.
- `-XX:MaxNewSize` - Max new gen size
- `-Xmn` - Equivalent to `-XX:NewSize=x`, `-XX:MaxNewSize=x`
- `-XX:SurvivorRatio` - Ratio of eden space to survivor. E.g. 6 means eden is 6 times larger than 1 survivor and eden is 1/8 of new gen.
- `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp` - capture heap dump automatically on OOM for post-mortem analysis.

## GC selection

- `-XX:+UseG1GC` - default balanced collector for most services.
- `-XX:+UseZGC -XX:+ZGenerational` - sub-millisecond pause collector for latency-sensitive services (default generational in Java 23+; see [[Java Evolution]]).

> [!warning] Failure modes & trade-offs
> - **Container OOMKill (`Exit 137`):** JVM total RSS = Heap + Metaspace + Thread stacks (`threads × -Xss`) + Direct/Off-heap buffers + CodeCache. Setting `-Xmx` or `MaxRAMPercentage` near 100% of the [[Pod]] memory limit causes the Linux OOM killer to terminate the container before Java throws `OutOfMemoryError`. Leave 20–30% headroom.
> - **Don't hardcode `-Xmn` with G1GC:** Pinning young generation size prevents G1GC from dynamically resizing regions to meet its pause-time target (`-XX:MaxGCPauseMillis`).

## Flashcards

Why does setting `-Xmx` equal to a [[Container]]'s memory limit cause OOMKill (`Exit 137`)?::JVM native memory (Metaspace, thread stacks, direct buffers, CodeCache, GC structures) lives outside the Java heap—leaving no headroom causes the kernel OOM killer to kill the container before the JVM throws `OutOfMemoryError`.
^q-pgci

In containerized [[Kubernetes]] deployments, prefer `-XX:MaxRAMPercentage` (typically 70–75%) over hardcoded `-Xmx` so heap scales cleanly with cgroup memory limits. #card-reminder
^q-ux7f

## Related
- [[JVM App Deployment]]
- [[jmxConfig]]
- [[Java Evolution]]
- [[Container]]
- [[Pod]]
