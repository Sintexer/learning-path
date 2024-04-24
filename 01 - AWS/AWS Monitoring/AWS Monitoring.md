Monitoring provides a near real-time pulse on your system and helps answer the "why my website isn't working" questions. You can use the data you collect to watch for operational issues caused by events like overuse of resources, application flaws, resource misconfiguration, or security-related events. Think of the data collected through monitoring as outputs of the system, or metrics.

**CPU utilization** is one example of a metric. Other examples of metrics that [[EC2]] instances have are **network utilization**, **disk performance**, **memory utilization**, and the **logs** created by the applications running on top of EC2.

## Types of metrics

Different AWS resources provide different metrics:

- [[AWS S3]] - size of objects, number of objects in a bucket, number of HTTP requests per bucket.
- [[AWS RDS]] - DB connections, CPU utilization, disk consumption.
- [[EC2]] - CPU utilization, network utilization, disk performance, status checks.

## Monitoring benefits

1. The ability to respond proactively - if app fails you can see the epicenter of the problem via monitoring solutions.
2. Monitoring gives you the full picture to analyze performance in order to **optimize your system**.
3. Monitoring gives you a *baseline of a normal activity*, based on which you can spot **abnormal behavior** or **security threat**.
4. Make **data-driven decisions** - monitoring can give you a resulting popularity of your new service.
5. Create **cost-effective** solutions by cutting down excessive performance.