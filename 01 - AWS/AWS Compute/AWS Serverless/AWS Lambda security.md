With Lambda functions, there are two sides that define the necessary scope of permissions – **permission to invoke the function**, and **permission of the Lambda function itself** to act upon other services. Because Lambda is fully integrated with [[IAM]], you can control the exact actions of each side of the [[AWS Lambda function]].

For ease of policy management, you can use authoring tools such as the [[AWS SAM]] to help manage your policies. For a Lambda function, AWS SAM scopes the permissions of your Lambda functions to the resources used by your application. You can add IAM policies as part of the AWS SAM template. The policies property can be the name of AWS managed policies, inline IAM policy documents, or AWS SAM policy templates. AWS SAM is discussed in more detail later in this course.

## Lambda resource-based policy

Permissions to invoke the function are controlled using an [[IAM policy#Resource-based policies|resource-based policy]]. A resource policy tells the Lambda service which principals have permission to invoke the Lambda function. An AWS principal may be a user, role, another AWS service, or another AWS account.

The resource-based policy is an easier option and you can see and modify it via the Lambda console. A consideration with cross-account permissions is that a resource policy does have a size limit. If you have many different accounts that need to invoke the function and you have to add permissions for each account via the resource policy, you might reach the policy size limit. In that case, you would need to use IAM roles instead of resource policies. 

Lambda resource-based policy:

- Associated with a "push" event source such as Amazon API Gateway
- Created when you add a trigger to a Lambda function
- Allows the event source to take the `lambda:InvokeFunction` action

## Execution role

An IAM execution role defines the permissions that control what the function is allowed to do when interacting with other AWS services. The [[IAM policy|policy]] for this role defines the actions the role is allowed to take — for example, writing to a DynamoDB table. The role must include a [[IAM role trust policy]] that allows Lambda to `AssumeRole` so that it can take that action for another service. You can write the role or use the managed roles (with predefined permissions) provided by Lambda to simplify the process of creating an execution role. You can add or remove permissions from a function's execution role at any time, or configure your function to use a different role.

> [!tip]
> Remember to use the principle of least privilege when creating IAM policies and roles. Always start with the most restrictive set of permissions and only grant further permissions as required for the function to run. Using the principle of least privilege ensures security in depth and eliminates the need to remember to 'go back and fix it' once the function is in production. You can also use [[IAM Access Analyzer]].

Execution role:

- Role selected or created when you create a Lambda function
- IAM policy includes actions you can take with the resource
- Trust policy that allows Lambda to `AssumeRole`
- Creator must have permission for `iam:PassRole`

### Accessing resources in a VPC

Enabling your Lambda function to access resources inside your [[AWS VPC]] requires additional VPC-specific configuration information, such as **VPC subnet IDs** and **security group IDs** ([[AWS Security groups]]). This functionality allows Lambda to access resources in the VPC. It does not change how the function is secured. You also need an execution role with permissions to create, describe, and delete elastic network interfaces. Lambda provides a permissions policy for this purpose named `"AWSLambdaVPCAccessExecutionRole"`.

To establish a private connection between your VPC and Lambda, create an interface VPC endpoint. Interface endpoints are powered by [[01 - AWS/AWS Networking/AWS PrivateLink|AWS PrivateLink]], which enables you to privately access Lambda APIs without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. 

Instances in your VPC don't need public IP addresses to communicate with Lambda APIs. Traffic between your VPC and Lambda does not leave the AWS network.