Compared to [[Virtual machine|virtual machines (VMs)]], [[Container|containers]] share the *same operating system and kernel* as the host that they are deployed on.

**Containers** share the *same operating system and kernel* as the host that they exist on. But **virtual machines** contain their *own operating system*. Each virtual machine must maintain a copy of an operating system, which results in a degree of wasted resources.  
  
A container is more lightweight. Containers **spin up quicker**, almost instantly. This difference in startup time becomes instrumental when designing applications that must scale quickly during I/O bursts.

Containers can provide speed, but virtual machines offer the full strength of an operating system and more resources, like package installation, dedicated kernel, and more.

![[Difference between VMs and containers visual.png]]