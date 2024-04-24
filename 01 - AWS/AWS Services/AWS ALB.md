> AKA **Application Load Balancer**

Used for [[HTTP]]/[[HTTPS]] traffic [[OSI 7 layer]]. One implementation of [[AWS ELB]].

**Target type:** IP, [[EC2]], [[AWS Lambda]].

To configure an application load balancing you must specify a **Listener** - checks for requests (e.g. port 443), a **Target group** - type of backend you want to direct the traffic to (EC2, Lambda, etc.) with health checks enabled, and **Rule** defines how request is routed (e.g. route requests to **/info** to the target group B).  

**Routes based on request data.** An Application Load Balancer makes routing decisions based on the HTTP and HTTPS protocol. For example, the ALB could use the URL path (/upload) and host, HTTP headers and method, or the source IP address of the client. This facilitates granular routing to target groups.

**Sends responses directly to clients**. An Application Load Balancer can reply directly to the client with a fixed response, such as a custom HTML page. It can also send a redirect to the client. This is useful when you must redirect to a specific website or redirect a request from HTTP to HTTPS. It removes that work from your backend servers.

**Uses TLS offloading**. An Application Load Balancer understands HTTPS traffic. To pass HTTPS traffic through an Application Load Balancer, an SSL certificate is provided one of the following ways:

- Importing a certificate by way of [[IAM]] or [[ACM]] services
- Creating a certificate for free using ACM

This ensures that the traffic between the client and Application Load Balancer is encrypted.

**Authenticates users**. An Application Load Balancer can authenticate users before they can pass through the load balancer. The Application Load Balancer uses the [[OIDC]] protocol and integrates with other AWS services to support protocols, such as the following:

- SAML
- Lightweight Directory Access Protocol (LDAP)
- Microsoft Active Directory
- Others

**Secures traffic**. To prevent traffic from reaching the load balancer, you configure a [[AWS Security groups|security group]] to specify the supported IP address ranges.

**Supports sticky sessions**. If requests must be sent to the same backend server because the application is stateful, use the sticky session feature. This feature uses an HTTP cookie to remember which server to send the traffic to across connections.