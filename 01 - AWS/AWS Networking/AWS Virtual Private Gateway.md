A **virtual private gateway** connects your [[AWS VPC|VPC]] to another **private network**. When you create and attach a virtual private gateway to a VPC, the gateway acts as **anchor on the AWS side** of the connection. On the other side of the connection, you will need to connect a **customer gateway** to the other private network. A customer gateway device is a physical device or software application on your side of the connection. When you have both gateways, you can then establish an encrypted [[VPN]] connection between the two sides.