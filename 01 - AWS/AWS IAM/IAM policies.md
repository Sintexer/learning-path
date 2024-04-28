To manage access and provide **permissions** to AWS services and resources, you create **IAM policies** and attach them to an [[IAM identity]]. Whenever an **IAM identity** makes a request, AWS evaluates the policies associated with them. For example, if you have a developer inside the developers group who makes a request to an AWS service, AWS evaluates any **policies** attached to the developers [[IAM group|group]] and any policies attached to the developer user to determine if the request should be *allowed or denied* (controlled by policy [[IAM policies#^8c66de|Effect]]). Policy could also be applied to [[IAM role]]

IAM policies are often referred as **permissions**.

Each policy is defined from three pieces of logic:

1. **Principal** - User, role, external user, or application that sent the request and the policies associated with that principal.
2. **Action** - What the principal is attempting to do
3. **Resource** - AWS resource object upon which the actions or operations are performed

## Access policies

When IAM policies are attached to your resources (e.g. [[AWS S3 Bucket|buckets and objects]]) or [[IAM user|IAM users]], [[IAM group|groups]], and [[IAM role|roles]], the policies define which actions they can perform. **Access policies** that you attach to your resources are referred to as _resource-based policies_ and access policies attached to users in your account are called _identity-based policies_.

### Resource-based policies

**Access policies** that you attach to your resources (e.g. [[AWS S3 Bucket|buckets and objects]]) are referred to as _resource-based policies_.

### Identity-based policies

An identity-based policy is an object in AWS that, when associated with an IAM identity, defines their permissions. AWS evaluates these policies when a principal entity (IAM user or role) makes a request. Permissions in the policies determine whether the request is allowed or denied.

There are three types of identity-based policies:

1. **AWS managed** -AWS manages and creates these types of policies. They can be attached to multiple users, groups, and roles.
2. **Customer managed** - This type of policy provides more precise control than AWS managed policies and can also be attached to multiple users, groups, and roles. They are created and managed by users.
3. **Inline** - Inline policies are embedded directly into a single user, group, or role. In most cases, AWS doesn't recommend using inline policies. This type of policy might be useful to maintain strict one-to-one relationship.

## Policy elements

```json
{
	"Version": "2012-10-17",
	"Statement": [{
		"Sid": "AllActionsAllResources"
		"Effect": "Allow",
		"Action": "*",
        "Resource": "*"
    }]
}
```

This policy has four major JSON elements: _**Version**_, _**Statement**_, _**Sid**_,  _**Effect**_, _**Action**_, and _**Resource**_.

- The _**Version**_ element defines the version of the policy language. It specifies the language syntax rules that are needed by AWS to process a policy. To use all the available policy features, include `"Version": "2012-10-17"` before the **"Statement"** element in your policies.
- ***Statement*** element is the main element for a policy. It can contain a single statement or an array of individual statements.
- ***Sid*** element is optional and provides a brief description of the policy statement.
- The _**Effect**_ element specifies whether the policy will *allow or deny access*. In this policy, the Effect is **"Allow"**, which means you’re providing access to a particular resource. ^8c66de
- The _**Action**_ element describes the type of action that should be allowed or denied. In the example policy, the action is `"*"`. This is called a wildcard, and it is used to symbolize every action inside your AWS account.
- The _**Resource**_ element specifies the object or objects that the policy statement covers. In the policy example, the resource is the wildcard `"*"`. This represents all resources inside your AWS console.
- **Condition** element is optional and lets you specify conditions for when a policy is in effect. In the condition element, you build expressions in which you use condition operators (equal, less than, etc.) to match the condition keys and values in the policy against keys and values in the request. Condition structure:
	- `"Condition" : { "{condition-operator}" : { "{condition-key}" : "{condition-value}" }}`

## Policy example

The next example shows a more granular IAM policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3AccessOutsideMyBoundary",
      "Effect": "Deny",
      "Action": [
        "s3:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:ResourceAccount": [
            "222222222222"
          ]
        }
      }
    }
  ]
}
```

This policy uses a _Deny_ effect to block access to Amazon S3 actions, unless the Amazon S3 resource that's being accessed is in account _222222222222_. This ensures that any Amazon S3 principals are accessing only the resources that are inside of a trusted AWS account.

