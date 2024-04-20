
At a fundamental level, three types of compute options are available: ***virtual machines (VMs)***, ***container services***, and ***serverless***

A [[Virtual machine|virtual machine]] is often the easiest compute option to understand. In AWS, Amazon [[AWS EC2|Elastic Compute Cloud (Amazon EC2)]] is a web service that provides secure and resizable compute capacity in the cloud. You can provision virtual servers called **EC2** instances. Behind the scenes, AWS operates and manages the *host machines* and the *hypervisor layer*. AWS also installs the virtual machine *operating system*, called the **guest operating system**.  
  
Beneath the surface, some AWS compute services use Amazon EC2 or use virtualization concepts. You should understand this service before advancing to [[AWS Container services|container services]] and [[AWS Lambda|serverless compute]].