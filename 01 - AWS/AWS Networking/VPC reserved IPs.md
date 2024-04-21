For AWS to configure your [[AWS VPC|VPC]] appropriately, AWS **reserves five IP addresses** in **each subnet**. These IP addresses are used for **routing**, [[DNS]], and **network management**.  
  
For example, consider a **VPC** with the IP range `10.0.0.0/22`. The VPC includes **1,024** total IP addresses. This is then divided into four equal-sized subnets, each with a /24 IP range with **256** IP addresses. Out of each of those IP ranges, there are **only 251 IP addresses** that **can be used** because AWS reserves five.

> if `10.0.0.0/24` is our subnet, then `10.0.0.1` is reserved for the **VPC local router**, `10.0.0.2` is reserved for **DNS server**, `10.0.0.3` is reserved for **future use**, and `10.0.0.255` is **Network broadcast address**