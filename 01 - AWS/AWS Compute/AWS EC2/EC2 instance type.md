
**EC2 instances** are a combination of *virtual processors* (vCPUs), *memory*, *network*, and, in some cases, instance *storage* and *graphics processing units* (GPUs). When you create an EC2 instance, you need to choose how much you need of each of these components.

To get an overview of the capacity details for a particular instance, you should look at the **instance type**. Instance types consist of a *prefix* identifying the type of workloads they’re optimized for, followed by a *size*. For example, the instance type `c5n.xlarge` can be broken down as follows:

- **First position –** The first position, **c**, indicates the **[[EC2 Instance family|instance family]]**. This indicates that this instance belongs to the compute optimized family.
- **Second position –** The second position, **5**, indicates the **generation** of the instance. This instance belongs to the fifth generation of instances.
- **Remaining letters before the period –** In this case, **n** indicates **additional attributes**, such as local NVMe storage.
- **After the period –** After the period, **xlarge** indicates the **[[EC2 instance sizing|instance size]]**.