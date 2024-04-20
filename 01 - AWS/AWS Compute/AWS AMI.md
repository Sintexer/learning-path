AMI - Amazon Machine Image. It usually specifies the **Root volume templates (usually an OS and application that will be installed at boot**, **Launch permissions**, and **Block device mapping**. An AMI includes the *operating system*, *storage mapping*, *architecture type*, *launch permissions*, and any additional *preinstalled software applications*.

AMIs are used to create multiple [[AWS EC2|instances]] with same configuration, which makes these services pretty identical. 

Some AMIs are provided and managed by AWS, other are created and managed by community. You are also allowed to create your own AMI.

## EC2 and AMI relation

[[AWS EC2]] instances are live instantiations (or versions) of what is defined in an **AMI**, as a cake is a live instantiation of a cake recipe. If you are familiar with software development, you can also see this kind of relationship between a class and an object. In this case, the **AMI** is how you *model and define your instance*. The EC2 instance is *the entity you interact with*, where you can install your web server and serve your content to users.

When you launch a new instance, AWS allocates a virtual machine that runs on a hypervisor. Then the AMI that you selected is copied to the root device volume, which contains the image that is used to boot the volume. In the end, you get a server that you can connect to and install packages and additional software on. In the example, you install a web server along with the properly configured source code of your employee directory application.

If you want to create another EC2 instance with the same configurations, you could create and configure a new EC2 instance to match the first instance. Or you could **create an AMI from your running instance** and use the AMI to start *a new instance*. That way, your new instance would have the same configurations as your current instance because the configurations set in the AMIs are the same.

## AMI ID

Each AMI in the AWS Management Console has an **AMI ID**, which is prefixed by `ami-`, followed by a random hash of numbers and letters. The IDs are unique to each [[Region|AWS Region]].