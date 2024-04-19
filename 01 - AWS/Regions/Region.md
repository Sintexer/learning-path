What if entire AZ has collapsed? Like [[Data Center]], AWS also clusters [[Availability Zone]]s together and also connects them with redundant high speed and low latency links. A cluster of AZs is simply called a **Region**. 

Regions are usually called by location (Region - N. Virginia). Each AWS Region is associated with a geographical name and a Region code. Here are examples of Region codes: **us-east-1**, **ap-northeast-1**.

### Region considerations

Choosing the right region is based on 4 factors: *Compliance*,  *Latency*, *Pricing*, and *Service availability*.

- **Compliance** - some countries may require you to store and manage all the data within country boundaries
- **Latency** - we all are bound to the speed of light
- **Pricing** - it is cheaper to run in India than in London
- **Service Availability** - new AWS services are often bounded to specific Regions, this is called [[Service scope]].

Sometimes due to Compliance, we have to use Region far from our end users. How to ensure the data latency is minima for such clients (E.g. our Region is in England, but users from China would have a significant delay viewing our website). AWS covers this case by caching and [[Edge locations]].