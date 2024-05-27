> AKA **Simple Notification Service**

**SNS provides automatic retries** for deliveries to HTTP/S endpoints. If the initial attempt to deliver a message fails, SNS will retry several times with delays between attempts. For critical workflows, you can even set up a dead-letter queue (DLQ) to capture the messages that couldn't be delivered after all retry attempts, allowing you to analyze and handle them accordingly.

SNS is used for [[AWS Serverless patterns#Webhook pattern with SNS|AWS Webhook pattern]] in the serverless architecture.