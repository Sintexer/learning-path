Condition element in a [[IAM policy|policy]] lets you indicate the circumstances for when a policy is in effect. You can use the Condition element to compare keys in the request context with key values that you specify in your policy. This gives you granular control over when your JSON policy statements match or don't match an incoming request. This approach is used in [[IAM ABAC]].

The **condition key** that you specify can be a service-specific or a global condition key. Service-specific condition keys have the service's prefix. For example, [[EC2]] lets you write a condition using the `ec2:InstanceType` key, which is unique to that service. This lesson covers the different service-specific keys for [[IAM]].

 In the condition element, you build expressions in which you use condition operators (equal, less than, etc.) to match the condition keys and values in the policy against keys and values in the request. Condition structure:
 
 ```json
"Condition" : { 
	"{condition-operator}" : { "{condition-key}" : "{condition-value}" }
}
```

## IAM Condition keys

Service keys are prefixed with service name; IAM uses `iam` prefix. There are many IAM condition keys, here are some of them:

- `iam:AWSServiceName` - might be used in scope of [[IAM role#Service roles|service roles]] when roles need to be **attached** to only specific services.
- `iam:OrganizationPolicyId`
- `iam:PermissionsBoundary` - checks that the specified policy is attached as a [[IAM Permissions boundary|permissions boundary]] on the IAM principal resource.
- `iam:PolicyARN` - This condition key checks the [[ARN]] of a policy in requests that involve that same managed policy. You use this key for actions like `iam:AttachGroupPolicy` to control how users can apply AWS managed and customer managed policies. For example, you might create a policy that allows users to attach only the `IAMUserChangePassword` AWS managed policy to a new IAM user, group, or role.
- `iam:ResourceTag` - checks that the tag attached to the identity resource. For example, you can add this condition for `iam:DeleteUser` action to allow removal of only inactive users by applying condition to the **status** resource tag.
- `iam:PrincipalTag` - checks that the tag attached to the [[AWS Principal]] - author of the request.
- `iam:PassedToService` - extremely useful to control [[IAM role assume|role assumption]], specifically the action `iam:PassRole`. Using this key in your conditions you might restrict the role **passing** action to only specific AWS services.
- `iam:AssociatedResourceArn` - also used to control [[IAM role assume|role assumption]]. Can be added to `iam:PassRole` related policy to check the [[ARN]] of the service with which the role is assumed.

## Global Condition keys

Global condition keys start with the `aws:` prefix. AWS services can support global condition keys or provide service-specific keys that include their service prefix, as [[#IAM Condition keys]]. Not all AWS services support all of the available global condition keys.

### CalledVia

`aws:CalledVia` - request context might contain this key if some service was called by [[AWS Principal]], and then this service made a subsequent request to other service (and it has no role associated). Like service used your token to make a request. This condition key includes information in the form of an ordered list of each service in the chain that made requests on the principal’s behalf.

Imagine example: User 1 makes a request to AWS CloudFormation, which calls DynamoDB, which calls AWS KMS. These are three separate requests. In this case, the `aws:CalledVia` key in the request context includes **cloudformation.amazonaws.com** and **dynamodb.amazonaws.com** in that order. If you care only that the call was made via DynamoDB somewhere in the chain of requests, you can use this condition key in your policy. For instance, you might allow using specific [[AWS KMS]] key only if call to it was made via dynamodb.

>[!tip]
>If you want to enforce which service makes the first or last call in the `aws:CalledVia` context key, you can use the `aws:CalledViaFirst` and `aws:CalledViaLast` keys.

### Time keys

- `aws:CurrentTime` - restrict an action by a certain period of time.
- `aws:EpochTime` - Unix timestamp.
- `aws:TokenIssueTime` - restrict security credentials issued before a specific period of time.

### Security keys

- `aws:MultiFactorAuthAge` - compares number of seconds since principal's MFA authentication.
- `aws:MultiFactorAuthPresent` - checks if MFA was used to validate temp credentials. 
- `aws:SecureTransport` - was SSL used for request or not.

### Request source keys

- `aws:SpurceAccount` - restrict access to a specific 
- `aws:SourceArn` - restrict access source to a specific [[ARN]]
- `aws:SourceIp`
- `aws:SourceVpc`
- `aws:SourceVpce` - restrict by a specific VPC endpoint
- `aws:VpcSourceIp`

### Principal keys

- `aws:PrincipalAccount`
- `aws:PrincipalArn`
- `aws:PrincipalOrgId`
- `aws:PrincipalOrgPaths`
- `aws:PrincipalType` - see [[AWS Principal]]

### Tag keys

- `aws:PrincipalTag`
- `aws:RequestTag` - for tags passed in request, like when user attaches a tag to a resource.
- `aws:ResourceTag`
- `aws:TagKeys` - to specify which tags are allowed for a resource, useful for [[IAM ABAC]]

### Other keys

- `aws:Referer` - for API requests, checks referrer header
- `aws:RequestedRegion` - restrict access to a specific service in unwanted regions (e.g., allow EC2 only in cheap regions).
- `aws:UserAgent` - checks HTTP header for user app type
- `aws:userid` - restrict access to only specific users and roles
- `aws:username` - restricts access to only specific IAM users.
- `aws:ViaAWSService` - check whether call was made via AWS. If user calls directly, this key value is false.

>[!tip]
>If a condition key is missing from a request context, the policy can fail the evaluation. If you use condition keys that are available only in some circumstances, you can use the `IfExists` versions of the condition operators.