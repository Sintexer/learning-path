AWS SAM is an open-source framework for building [[Serverless|serverless]] applications. It provides shorthand syntax to express **functions**, **APIs**, **databases**, and [[AWS Lambda invocation models#Event source mapping|event source mappings]]. With just a few lines per resource, you can define the application you want and model it using YAML. You provide AWS SAM with simplified instructions for your environment and during deployment AWS SAM transforms and expands the AWS SAM syntax into [[AWS CloudFormation]] syntax (a fully detailed CloudFormation template). All CloudFormation options are still available within AWS SAM. AWS SAM just makes it easier to set up the resources commonly needed for serverless applications. 

## AWS SAM CLI

AWS SAM CLI launches a Docker container that you can interact with to test and debug your Lambda functions. With AWS SAM CLI for testing, you can do the following:

- Invoke functions and run automated tests locally.
- Generate sample event source payloads.
- Run API Gateway locally.
- Debug code.
- Review Lambda function logs.
- Validate AWS SAM templates.