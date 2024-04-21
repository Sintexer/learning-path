Think of a **network access control list (network ACL)** as a *virtual firewall* at the *subnet level*. A network ACL lets you control what kind of *traffic is allowed to enter or leave your subnet*. You can configure this by setting up rules that define what you want to filter. Here is an example of a default ACL for a VPC that supports IPv4:

**Inbound:**

| **Rule #** | **Type**         | **Protocol** | **Port Range** | **Source** | **Allow or Deny** |
| ---------- | ---------------- | ------------ | -------------- | ---------- | ----------------- |
| 100        | All IPv4 traffic | All          | All            | 0.0.0.0/0  | ALLOW             |
| *          | All IPv4 traffic | All          | All            | 0.0.0.0/0  | DENY              |

**Outbound:**

| **Rule #**   | **Type**         | **Protocol** | **Port Range** | **Destination** | **Allow or Deny** |
| ------------ | ---------------- | ------------ | -------------- | --------------- | ----------------- |
| 100          | All IPv4 traffic | All          | All            | 0.0.0.0/0       | ALLOW             |
| *            | All IPv4 traffic | All          | All            | 0.0.0.0/0       | DENY              |

The default network ACL shown in the preceding table, allows all traffic in and out of the subnet. To allow data to flow freely to the subnet, this is a good starting place.

---

However, you might want to restrict data at the subnet level. For example, if you have a web application, you might restrict your network to allow HTTPS traffic and Remote Desktop Protocol (RDP) traffic to your web servers.

**Custom Inbound:**

| **Rule #** | **Source IP**    | **Protocol** | **Port** | **Allow or Deny** | **Comments**                                                                                                               |
| ---------- | ---------------- | ------------ | -------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 100        | All IPv4 traffic | TCP          | 443      | ALLOW             | Allows inbound HTTPS traffic from anywhere                                                                                 |
| 130        | 192.0.2.0/24     | TCP          | 3389     | ALLOW             | Allows inbound RDP traffic to the web servers from your home network’s public IP address range (over the internet gateway) |
| *          | All IPv4 traffic | All          | All      | DENY              | Denies all inbound traffic not already handled by a preceding rule (not modifiable)                                        |

**Custom Outbound:**

| **Rule #** | **Destination IP** | **Protocol** | **Port**   | **Allow or Deny** | **Comments**                                                                                                 |
| ---------- | ------------------ | ------------ | ---------- | ----------------- | ------------------------------------------------------------------------------------------------------------ |
| 120        | 0.0.0.0/0          | TCP          | 1025-65535 | ALLOW             | Allows outbound responses to clients on the internet (serving people visiting the web servers in the subnet) |
| *          | 0.0.0.0/0          | All          | All        | DENY              | Denies all outbound traffic not already handled by a preceding rule (not modifiable)                         |

Notice that in the custom network ACL in the preceding example, you allow inbound 443 and outbound range 1025–65535. That’s because HTTPS uses port 443 to initiate a connection and will **respond to an ephemeral port**. Network ACLs are considered stateless, so you need to include both the inbound and outbound ports used for the protocol. **If you don’t include the outbound range, your server would respond but the traffic would never leave the subnet.**  
  
Because network ACLs are configured by default to allow incoming and outgoing traffic, you don’t need to change their initial settings unless you need additional security layers.