---
flashcards:
  q-ap85: { nid: 1795356731568, hash: fh6pmwzj, sync: tfkrqycn }
---

# JMX java opts config for app in [[Container|Docker]] to attach VisualVM

```bash
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=9010
-Dcom.sun.management.jmxremote.local.only=false
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.rmi.port=9010
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=localhost
```

> [!warning] Failure modes & trade-offs
> - **Never expose unauthenticated JMX in production:** `authenticate=false` and `ssl=false` allow remote code execution / MBean manipulation on any reachable network. Restrict to local dev or SSH-tunneled loopback.
> - **Two-port RMI trap:** Without `-Dcom.sun.management.jmxremote.rmi.port=9010`, JMX opens a random ephemeral RMI port that Docker port-forwarding and firewalls silently block.
> - **Production observability:** Prefer low-overhead Java Flight Recorder (JFR), Prometheus/Micrometer metrics, and [[OpenTelemetry]] over exposing remote JMX ports.

## Flashcards

Why must `-Dcom.sun.management.jmxremote.rmi.port` be pinned alongside `jmxremote.port` when profiling a JVM inside a [[Container]]?::Without pinning `rmi.port`, JMX opens a random ephemeral RMI port that container port mappings and firewalls block, causing VisualVM/JConsole connections to hang.
^q-ap85

## Related
- [[jvmArgs]]
- [[JVM App Deployment]]
- [[Container]]
- [[OpenTelemetry]]
