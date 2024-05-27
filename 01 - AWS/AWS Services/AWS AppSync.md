AWS AppSync is a [[AWS Managed Service|fully managed service]] that enables developers to create scalable applications on a range of data sources with real-time and offline capabilities.

## Key Features

- **Real-Time Data**: AppSync supports [[GraphQL]] subscriptions to perform real-time operations.
- **Offline Data Access**: AppSync automatically manages offline data access and synchronizes data when the client is back online.
- **Data Security**: AppSync provides fine-grained access control to data using [[IAM]] and [[AWS Cognito]].
- **Multiple Data Sources**: AppSync supports multiple data sources including [[AWS DynamoDB]], [[AWS Lambda]], and any HTTP data source.

## Architectural Patterns

- AppSync is used in [[AWS Serverless patterns#WebSockets pattern with AppSync|serverless WebSocket pattern]].
- **Real-Time Collaboration**: AppSync can be used to build real-time collaborative apps. For example, a chat application where messages are updated in real-time for all users.
- **Offline Data Synchronization**: AppSync is ideal for mobile applications that require offline data access and synchronization when the device reconnects.
- **Microservices Integration**: AppSync can integrate with microservices via AWS Lambda, allowing you to aggregate data from multiple sources.

## Integration with AWS Services

- **Amazon DynamoDB**: AppSync integrates with DynamoDB to provide a scalable NoSQL database for your applications.
- **AWS Lambda**: AppSync can trigger Lambda functions, allowing you to execute custom logic and aggregate data from multiple sources.
- **Amazon Cognito**: AppSync integrates with Cognito to provide user sign-up, sign-in, and access control to your applications.