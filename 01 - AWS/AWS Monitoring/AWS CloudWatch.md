AWS resources create data that you can monitor through metrics, logs, network traffic, events, and more. This data comes from components that are distributed in nature. This can lead to difficulty in collecting the data you need if you don’t have a *centralized* place to review it all. AWS has taken care of centralizing the data collection for you with a service called **CloudWatch**. CloudWatch is a [[AWS Managed Service]], so all you need to start with it is an AWS account.

> [!info] CloudWatch acts as a centralized place where metrics are gathered and analyzed.
> Collect -> Monitor -> Act -> Analyze

CloudWatch is a monitoring and observability service that **collects your resource data** and [[AWS CloudWatch Logs|logs]], and provides **actionable insights** into your applications. With CloudWatch, you can respond to system-wide performance changes, optimize resource usage, and get a unified view of operational health. You are not bound to using CloudWatch exclusively for all your visualization needs. You can use external or custom tools to ingest and analyze CloudWatch metrics using the `GetMetricData` API.

You can use CloudWatch to do the following:

- **Detect anomalous behavior** in your environments.
- Set **alarms** to alert you when something is not right.
- **Visualize logs and metrics** with the AWS Management Console.
- Take **automated actions** like scaling.
- **Troubleshoot** issues.
- Discover **insights** to keep your applications healthy.

## Pricing

Many AWS services automatically send metrics to CloudWatch for free at a rate of 1 data point per metric per 5-minute interval. This is called **basic monitoring**, and it gives you visibility into your systems without any extra cost. For many applications, basic monitoring is adequate. You can increase rate with **detailed monitoring**, but it *incurs a fee*.

## Metric

Metrics are the fundamental concept in CloudWatch. A **metric** represents a *time-ordered set of data points* that are published to CloudWatch. Think of a metric as a variable to monitor and the data points as representing the values of that variable over time. Every metric data point must be associated with a timestamp.

AWS services that send data to CloudWatch attach dimensions to each metric. A dimension is a name and value pair that is part of the metric’s identity. You can use dimensions to filter the results that CloudWatch returns. For example, many Amazon EC2 metrics publish **instanceId** as a dimension name and the actual instance ID as the value for that dimension.

### Custom metrics

With custom metrics, you can publish your own metrics to CloudWatch.  If you want to gain more granular visibility, you can use high-resolution custom metrics, which make it possible for you to collect custom metrics down to a 1-second resolution. This means you can send 1 data point per second per custom metric.

## CloudWatch dashboards  

Once you provision your AWS resources and they are sending metrics to CloudWatch, you can visualize and review that data using CloudWatch dashboards. Dashboards are customizable home pages you can configure for data visualization for one or more metrics through widgets, such as a graph or text.  
  
You can build many custom dashboards, each one focusing on a distinct view of your environment. You can even pull data from different AWS Regions into a single dashboard to create a global view of your architecture.

## CloudWatch alarms

You can create CloudWatch alarms to automatically initiate actions based on sustained state changes of your metrics. You configure when alarms are invoked and the action that is performed.

First, you must decide which metric you want to set up an alarm for, and then you define the threshold that will invoke the alarm. Next, you define the threshold's time period. For example, suppose you want to set up an alarm for an EC2 instance to invoke when the CPU utilization goes over a threshold of 80 percent. You also must specify the time period the CPU utilization is over the threshold. 

Alarm has 3 states:

 - **OK** - metric is within defined threshold.
 - **ALARM** - metric is outside defined threshold.
 - **INSUFFICIENT_DATA** - alarm just started, or metric not available, or not enough metric data.

You don’t want to invoke an alarm based on short, temporary spikes in the CPU. You only want to invoke an alarm if the CPU is elevated for a sustained amount of time. For example, if CPU utilization exceeds 80 percent for 5 minutes or longer, there might be a resource issue. To set up an alarm you need to choose the metric, threshold, and time period.

> [!info]
> An alarm can be invoked when it transitions from one state to another. After an alarm is invoked, it can initiate an action. Actions can be an [[EC2]] action, an [[EC2 Auto Scaling|automatic scaling action]], or a notification sent to [[AWS SNS]].

### Prevent and troubleshoot issues with CloudWatch alarms

[[AWS CloudWatch Logs]] uses metric filters to turn the log data into metrics that you can graph or set an alarm on. 

1. **Set up a metric filter** - Suppose you set up a metric filter for HTTP 500 error response codes.
2. **Define an alarm** - Continuing example, the alarm state is invoked if HTTP 500 error responses are sustained for a specified period of time.
3. **Define an action** - Send email or alert