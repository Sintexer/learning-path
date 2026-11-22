> AKA **Virtual Private Network**

A **VPN** creates an encrypted tunnel over an untrusted public network (the internet), extending a private network (corporate [[LAN]] or cloud [[Virtual Private Cloud|VPC]]) securely to remote users or remote sites.

## Common VPN Architectures

- **Remote-Access VPN (Client-to-Site)**: an employee's laptop connects via a VPN client to a corporate gateway to access private internal services.
- **Site-to-Site VPN**: connects two whole networks (e.g., on-prem data center <-> AWS [[Virtual Private Cloud|VPC]]) persistently via routers/gateways.
- **Mesh VPN** (e.g., Tailscale / Headscale over WireGuard): direct encrypted peer-to-peer tunnels between nodes without backhauling traffic through a central concentrator.

## Protocols

- **IPsec** ([[OSI 3 layer]]): standard for enterprise site-to-site VPNs (IKEv2 key exchange + ESP encryption); complex configuration, often paired with BGP.
- **WireGuard**: modern, minimal-codebase, kernel-space [[UDP]] VPN with fast crypto (Noise protocol, ChaCha20) and seamless IP roaming.
- **OpenVPN**: runs over [[TLS]] ([[UDP]] or [[TCP]]); easy to traverse strict firewalls (TCP 443), slower than WireGuard.

> [!warning] Failure modes & trade-offs
> - **Hairpinning & bottlenecked concentrators**: full-tunnel VPNs route all user traffic through a central gateway, adding latency and creating a single bottleneck (mitigated by split-tunneling or mesh VPNs).
> - **TCP-over-TCP meltdown**: running a VPN over [[TCP]] (when carrying inner TCP traffic) causes competing congestion-control loops and severe stalls under packet loss—prefer [[UDP]]-based transports (WireGuard, IPsec, OpenVPN-UDP).
> - **Perimeter fallacy**: a traditional VPN grants broad network access once inside; modern architectures pair or replace VPNs with [[Zero-Trust Security]] (identity-aware proxies, per-request [[Authentication]]/[[Authorization]], mTLS).

## Flashcards

VPN:::Virtual Private Network — an encrypted network tunnel over an untrusted network that connects remote clients or sites into a private network.

Why should VPN tunnels prefer [[UDP]] over [[TCP]]?::Because tunneling TCP inside TCP causes ==TCP-over-TCP meltdown== (two nested retransmission/congestion-control loops fighting each other under packet loss).

### How does [[Zero-Trust Security]] change the role of a traditional corporate VPN? #card

A traditional VPN trusts anyone inside the network perimeter (broad lateral access). Zero-Trust authenticates and authorizes every user, device, and request per application (often via identity-aware proxies and mTLS), shrinking the blast radius even if a device is compromised.

## Related
- [[Virtual Private Cloud]]
- [[Forward proxy]]
- [[TLS]]
- [[UDP]]
- [[Zero-Trust Security]]
