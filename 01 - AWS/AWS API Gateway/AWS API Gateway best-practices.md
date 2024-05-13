## Best-practices

***Use API Gateway stages with Lambda aliases***

Lambda and API Gateway are both designed to support flexible use of versions. You can do this by using aliases in Lambda and stages in API Gateway. When you couple that with stage variables, you don't have to hard-code components, which leads to having a smooth and safe deployment.

- In Lambda, enable **versioning** and use **aliases** to reference.
- In API Gateway, use **stages** for environments.
- Point API Gateway **stage variables** at the Lambda **aliases.**

***Use Canary deployments***

With Canary deployments, you can send a percentage of traffic to your **canary** while leaving the bulk of your traffic on a known good version of your API until the new version has been verified. API Gateway makes a base version available and updated versions of the API on the same stage. This way, you can introduce new features in the same environment for the base version. To set up a Canary deployment through the console, select a stage and then select the Canary tab for that stage.

***Use AWS SAM to simplify deployments***

One of the challenges of serverless deployments is the need to provide all the details of your deployment environment as part of your deployment package. The [[AWS SAM]] is an open-source framework that you can use to build serverless applications on AWS.

**SAM templates**: [[AWS SAM]] provides templates that help you define your serverless applications. These template specifications provide you with a straightforward and clean syntax to describe your functions, APIs, permissions, configurations, and events. You use an AWS SAM template file to operate on a single, deployable, versioned entity that makes up your serverless application. AWS SAM is an extension of [[AWS CloudFormation]], so it gives you the deployment capabilities and the full suite of resources available in CloudFormation. The AWS SAM template file closely follows the format of a CloudFormation template file.

**Use [[Swagger]] and [[OpenAPI]] for more complex APIs**: AWS SAM also supports OpenAPI to define more complex APIs. This can either be 2.0 for the Swagger specification, or one of the OpenAPI 3.0 versions, like 3.0.1. OpenAPI is an industry-standard way to document and design your APIs.  With SAM, you can document your API in an external OpenAPI or Swagger file, and then reference that in a SAM template.