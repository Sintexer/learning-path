There are two types of IAM permissions for APIs:

1. `execute-api:*` - who can invoke the API.
2. `apigateway:*` - who can manage the API

## Invoke permissions

Invoke permissions are used when your API is configured to use [[AWS API Gateway IAM Authorization]].

For the `execute-api` permission, you need to create [[IAM policy|IAM policies]] that permit a specified API caller to invoke the desired API method. To apply this IAM policy on the API method, you need to configure the API method to use an authorization type of **AWS_IAM**. When you add IAM authorization type and attach a policy to an API method, API Gateway will expect an IAM Sig v4 request for any requests that come to that API method.

> [!info]
> API caller could be an [[IAM user]]; it could be an [[IAM group]] containing a set of IAM users; or it could be an [[IAM role]] assumed by a user, an [[EC2]] instance, or an application running inside AWS.

## Manage permissions

To allow an API developer to create and manage an API in [[AWS API Gateway]], you need [[IAM policy|IAM permission policies]] that allow a specified API developer to create, update, deploy, view, or delete required API entities. To do that, create a policy using the `apigateway:HTTP_VERB` format, associated with the specific resource using the verb that you want to permit or deny in the policy.

## Resource policies

Resource policies further refine access for APIs. While an [[IAM policy]] is used to grant permission to a user, group, or role, you can also apply policies **directly on API Gateway** using a resource policy. A resource policy is a JSON policy document that you attach to an API to **limit access by users** from a specified account, **IP address range**, **[[AWS VPC]]**, or VPC endpoint. You can make this as granular as you need, and resource policies can be used in coordination with IAM policies to restrict access.

> [!example]
> You could use resource policies to provide access to another AWS account, or to limit access to your API from a particular set of IP address ranges. You can also use resource policies to grant access to specific VPCs or VPC endpoints.

Resource policies are similar to [[#Invoke permissions]] and [[#Manage permissions]], but resource permissions must have **Principal** field. Use [[IAM policy condition|conditions]] to introduce any complex scenario like allowing only IP addresses from a specific range.

If the Principal in the policy is set to `"*"`, other authorization types can be used alongside the resource policy. However, if the Principal is set to `AWS`, authorization will fail for all resources not secured with `AWS_IAM` authorization, including unsecured resources.

Resource policy and authentication methods work together to grant access to your APIs. As illustrated below, methods for securing your APIs work in aggregate. There are some examples of different authentication approaches with the resource based policies:

1. **Resource-based policy only** - in this case any API call will be denied by default, and will be allowed only if there is an explicit allow by the resource-based policy for any criteria.
2. **IAM Authentication and resource-based policy** - If the user [[AWS API Gateway IAM Authorization|authenticates successfully with IAM]], policies attached to the IAM user and resource policy are evaluated together. If the caller and API owner are from separate accounts, both the IAM user policies and the resource policy explicitly allow the caller to proceed. If the caller and the API owner are in the same account, either user policies or the resource policy must explicitly allow the caller to proceed.
3. **Lambda authorizer and resource-based policy** - If the policy has explicit denials, the caller is denied access immediately. Otherwise, the [[AWS API Gateway Lambda Authorizer|Lambda Authorizer]] is called and returns a policy document that is evaluated with the resource policy.
4. **Cognito Authentication and resource-based policy** - If API Gateway [[AWS API Gateway Cognito Authorizer|authenticates the caller from Cognito]], the resource policy is evaluated independently. If there is an explicit allow, the caller proceeds. Otherwise, deny or neither allow nor deny will result in a deny.