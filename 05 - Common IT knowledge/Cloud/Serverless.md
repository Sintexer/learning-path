With **serverless compute**, you can spend time on the things that differentiate your application, rather than spending time on ensuring *[[Availability|availability]]*, *[[Scaling|scaling]]*, and *managing servers*. **Serverless** is a *cloud computing model* where you don't have to manage *servers* or *infrastructure*. Instead, you simply write and deploy your code, and the cloud **provider takes care** of everything else. Serverless also decreases [[Time-to-Market]]. Every definition of serverless mentions the following four aspects:

- There are **no servers** to provision or manage.
- It **scales** with usage (including scale-to-zero).
- You **never pay for idle** resources.
- **[[Availability]]** and **fault tolerance** are built in.

> Imagine a restaurant where you only pay for the food you eat, not the kitchen staff, equipment, or utilities. That's essentially the concept of serverless computing.

* **You write your code:** Focus on your application logic without worrying about server setup or maintenance.
* **Cloud provider manages servers:** The cloud provider dynamically allocates and scales servers based on your code's needs.
* **You pay per execution:** You only pay for the time your code is actually running, not for idle servers.

Compared to the classic [[IaaS]] application, serverless takes care of: **configuring an instance**, **updating OS**, **installation of application platform**, **configuring auto-scaling and [[Load Balancing|load balancing]]**, **securing and monitoring instances**. Though you are still required to *build and deploy the app* and *monitor and maintain the apps*.

## Benefits of serverless

* **Cost-effective:** You only pay for what you use, reducing infrastructure costs for spiky or low-utilization workloads.
* **Scalable:** The cloud provider automatically scales your application based on demand.
* **Faster development:** You can focus on coding without server management distractions.
* **Increased reliability:** The cloud provider handles server maintenance and updates.

## When to use serverless?

Serverless is ideal for applications that:

* **Are [[Event-driven architecture|event-driven]]:** Respond to events like user actions, queue messages, or data changes.
* **Have unpredictable or bursty traffic:** Scale up and down automatically based on demand.
* **Need fast development cycles:** Focus on coding without server management overhead.

## Examples of serverless services

* [[AWS Lambda]]: Run code (functions) without provisioning or managing servers.
* [[AWS Fargate]]: Run [[Container|containers]] without managing servers.
* **Google Cloud Functions / Cloud Run:** Deploy functions or containers that automatically scale based on events/requests.
* **Azure Functions:** Build and run event-driven applications without managing infrastructure.

**Remember:**

* Serverless doesn't mean "no servers." The cloud provider still manages them, but you don't have to.
* It's not a one-size-fits-all solution. Consider your application's needs before choosing serverless.

> [!warning] Failure modes & trade-offs
> - **Cold starts**: spinning up a fresh execution environment (especially for JVM/VPC-attached functions) adds tail latency; mitigate with provisioned concurrency, SnapStart, or lean runtimes.
> - **Steady high-traffic cost inversion**: at constant high utilization, serverless compute costs significantly more than reserved [[IaaS]]/[[Kubernetes]] instances.
> - **Connection exhaustion**: 1,000 concurrent Lambda invocations each opening a DB connection can overwhelm an [[RDBMS]] unless fronted by a connection proxy (e.g., RDS Proxy) or [[Backpressure]]/concurrency limits.
> - **Execution limits & vendor lock-in**: bounded execution duration, statelessness, and tight coupling to provider event triggers.

## Flashcards

Serverless computing::A cloud execution model where the provider manages provisioning, scaling (including to zero), and availability, and billing is based on actual execution rather than idle servers.

Two major production traps with serverless functions (e.g., AWS Lambda) are ==cold-start tail latency== and ==database connection exhaustion== under high concurrency.

Always set concurrency limits and use a DB connection proxy when connecting serverless functions to a relational database. #card-reminder

### When is serverless a poor fit compared to containers or VMs? #card

For steady, high-utilization workloads (cost inversion vs reserved compute), strict low-latency p99 requirements sensitive to cold starts, long-running stateful connections (e.g., [[WebSocket]] servers), or workloads needing custom OS/hardware tuning.

## Related
- [[IaaS]]
- [[Cloud-Native App]]
- [[Event-driven architecture]]
- [[Scaling]]
- [[Container]]
- [[Backpressure]]
