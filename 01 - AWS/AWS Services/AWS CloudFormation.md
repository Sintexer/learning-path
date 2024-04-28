The CloudFormation template specifies every detail of the [[AWS Lambda function]] and the environment required for the Lambda function to run. CloudFormation provides a common language and format that all parts of AWS can read and understand. CloudFormation is [[IaaC]].

The entire infrastructure needed for your Lambda function is written to a text file. This file (template) then deploys your desired stack. A stack is a collection of AWS resources that you can manage as a single unit. The template becomes the single source of truth for deploying identical stacks into any AWS account. Each time the Lambda function is invoked, it runs by using information provided in the CloudFormation template.

Like a blueprint for a house, the CloudFormation template can quickly become lengthy and difficult to manage. For example, if you have an application with a single API endpoint, the CloudFormation template must include mappings for [[IAM role|IAM roles]], [[AWS API Gateway]] endpoints, and connections to the Amazon DynamoDB table. Including these details can result in up to 100 lines of code in a simple template.

An easier option to work with CloudFormation is [[AWS SAM]].

> [!warning]
> Because serverless is hosted in the cloud, no option is available to check out a local copy of the application to do localized testing. You cannot recreate the environment specified in your CloudFormation template locally. Instead, you must have access to designated AWS accounts for testing. You can then perform realistic testing of your application in the cloud. With the appropriate access permissions, you can deploy and test your stack to any development, testing, staging, or production account in your organization.