To manage access and provide **permissions** to AWS services and resources, you create **IAM policies** and attach them to an [[IAM identity]]. Whenever an **IAM identity** makes a request, AWS evaluates the policies associated with them. For example, if you have a developer inside the developers group who makes a request to an AWS service, AWS evaluates any **policies** attached to the developers [[IAM group|group]] and any policies attached to the developer user to determine if the request should be *allowed or denied* (controlled by policy [[IAM policy#^8c66de|Effect]]). Policy could also be applied to [[IAM role]]

IAM policies are often referred as **permissions**.

Each policy is defined from three pieces of logic:

1. **Principal** - User, role, external user, or application that sent the request and the policies associated with that principal.
2. **Action** - What the principal is attempting to do
3. **Resource** - AWS resource object upon which the actions or operations are performed

Policies offer great control via [[IAM policy condition|conditions]] and [[IAM policy advanced statements]].

## Policies types

All policies belong to one of:

- [[IAM AWS managed policy]] - created and managed by AWS
- [[IAM Customer managed policy]] -- created by user
- [[IAM Inline policy]] - inlined to a principal or a resource

Policies are used in many places across all AWS features. By their duty they are separated to:

- [[IAM Identity-based policy]] (grants)
- [[IAM Resource-based policy]] (grants)
- [[IAM ACL]] (grants)
- [[IAM Permissions boundary]] (restricts)
- [[IAM AWS Organizations SCPs]] (restricts)
- [[IAM Session policy]] (restricts)

Some policies are used to *restrict permissions* while others are used to *grant access*. Using a combination of different policy types not only improves your overall security posture but also minimizes your blast radius in case an incident occurs. In the list above they are marked as ***grants***
(grants permissions) or ***restrict*** (means guardrails).

A request results in an explicit deny if an applicable policy includes a Deny statement. If policies that apply to a request include an Allow statement and a Deny statement, the Deny statement trumps the Allow statement. The request is explicitly denied. An **implicit denial** occurs when there is no applicable Deny statement but also no applicable Allow statement. If the requested resource has a [[IAM Resource-based policy|resource-based policy]] that allows the requested action, then AWS returns a final decision of allow. If there is no resource-based policy or if the policy does not include an Allow statement, then the evaluation continues.

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
- ***Id*** is policy identifier, optional.
- ***Statement*** element is the main element for a policy. It can contain a single statement or an array of individual statements.
- ***Sid*** element is optional and provides a brief description of the policy statement.
- The _**Effect**_ element specifies whether the policy will *allow or deny access*. In this policy, the Effect is **"Allow"**, which means you’re providing access to a particular resource. ^8c66de
- ***Principal*** element is an account/group/user/role to which this policy is applied to. See [[IAM policy principal]].
- The _**Action**_ element describes the type of action that should be allowed or denied. In the example policy, the action is `"*"`. This is called a wildcard, and it is used to symbolize every action inside your AWS account.
- The _**Resource**_ element specifies the object or objects that the policy statement covers. In the policy example, the resource is the wildcard `"*"`. This represents all resources inside your AWS console.
- **Condition** element is optional and lets you specify conditions for when a policy is in effect. There are plenty of features to use for [[IAM policy condition|conditions]].

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

