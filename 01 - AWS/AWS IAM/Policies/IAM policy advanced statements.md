
## NotPrincipal

The _NotPrincipal_ element lets you specify an exception to a list of principals. For example, you can use this element to allow all AWS accounts except a specific account to access a resource. Conversely, you can deny access to all principals except the one named in the `NotPrincipal` element.

**Usage with Allow** - ⚠️ It is strongly recommend that you do not use NotPrincipal in the same policy statement as "Effect": "Allow". Doing so allows all principals except the one named in the NotPrincipal element access to your resources. By doing this, you might grant access to anonymous (unauthenticated) users.

**Usage with Deny** - When you use NotPrincipal in the same policy statement as "Effect": "Deny", the actions specified in the policy statement are explicitly denied to all principals except for the ones specified in the policy. You can use this method to allow access to only a certain principal while denying the rest. When you use NotPrincipal with Deny, you must also specify the account [[ARN]] of the not-denied principal. Otherwise, the policy might deny access to the entire account, including the principal.

**You cannot use the NotPrincipal element in an IAM identity-based policy. You can use it in the trust policies for IAM roles and in resource-based policies.**


## NotAction

The _NotAction_ element explicitly matches everything except the specified list of actions. Using _NotAction_ can result in a shorter policy by listing only a few actions that should not match rather than including a long list of actions that will match. This, in turn, means that all of the applicable actions or services that are not listed are allowed if you use the Allow effect. In addition, such unlisted actions or services are denied if you use the Deny effect. When you use _NotAction_ with the Resource element, you provide scope for the policy. This is how AWS determines which actions or services are applicable.

**Usage with Allow** - ⚠️ As with using the _NotPrincipal_ element with _"Allow"_, it is also strongly recommend that you do not use NotAction in the same policy statement as "Effect": "Allow". Doing so allows all actions except the one named in the NotAction element.

**Usage with Deny** - You can use the _NotAction_ element in a statement with _"Effect": "Deny"_ to deny access to all of the listed resources except for the actions specified in the _NotAction_ element. This combination does not allow the listed items but instead explicitly denies the actions not listed. You must still allow actions that you want to allow.

Example: create an admin role and forbid anyone except admin to change it, but allow only role listing and permissions reading from it.

## NotResource

_NotResource_ is an advanced policy element that explicitly matches every resource except those specified. Using _NotResource_ can result in a shorter policy by listing only a few resources that should not match rather than including a long list of resources that will match. This is particularly useful for policies that apply within a single AWS service.

**Usage with Allow** - ⚠️ Be careful using the _NotResource_ element and _"Effect": "Allow"_ in the same statement or in a different statement within a policy. _NotResource_ allows all services and resources that are not explicitly listed in the policy and could result in granting users more permissions than you intended.

**Usage with Deny** - When using _NotResource_, you should keep in mind that resources specified in this element are the only resources that are not limited. This, in turn, limits all of the resources that would apply to the action. Using the _NotResource_ element and _"Effect": "Deny"_ in the same statement denies services and resources that are not explicitly listed in the policy.

Example: As an example, imagine you have a group named _HRPayroll_. Members of _HRPayroll_ should not be allowed to access any Amazon S3 resources except the Payroll folder in the _HRBucket_ bucket. So you create a policy that explicitly denies access to all Amazon S3 resources other than the listed resources.

> [!failure]  You should never use the _NotResource_ element with the Allow and Action:\*
>This statement is very dangerous because it allows all actions in AWS on all resources except the resource specified in the policy. This would even allow the user to add a policy to themselves that allows them to access the resource specified by the _NotResource_ element.