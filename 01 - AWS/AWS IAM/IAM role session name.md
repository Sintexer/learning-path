Each [[IAM role]] session is uniquely identified by a **role session name**. [[AWS STS]] provides a condition key called `sts:RoleSessionName` that controls how IAM principals and applications name their role sessions when they assume an IAM role. Administrators can rely on the role session name to track user actions when viewing [[AWS CloudTrail]] logs.

For example, the following [[IAM role trust policy]] requires that IAM users in account 111122223333 provide their IAM user name as the session name when they assume the role. This requirement is enforced using the `aws:username` condition variable in the condition key. This policy allows IAM users to [[IAM role assume|assume the role]] to which the policy is attached. This policy does not allow anyone using temporary credentials to assume the role because the username variable is present for only IAM users.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RoleTrustPolicyRequireUsernameForSessionName",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
            "Condition": {
                "StringLike": {"sts:RoleSessionName": "${aws:username}"} }
        }
    ]
}
```

There are different ways to name a role session, and it depends on the method used to assume the IAM role:

- **Service**: In some cases, AWS sets the role session name on your behalf. For example, for Amazon EC2 instance profiles, AWS sets the role session name to the instance profile ID. Lambda uses function name.
- **SAML-based**: When you use the `AssumeRolewithSAML` API to assume an IAM role, AWS sets the role session name value to the attribute provided by the identity provider, which your administrator defined. For scenarios in which corporate identities outside AWS need to access AWS resources, the corporate SAML-based IdP will provide the role session name.
- **User-defined**: In other cases, you provide the role session name when assuming the IAM role. For example, when assuming an IAM role with APIs such as `AssumeRole` or `AssumeRoleWithWebIdentity`, the role session name is a required input parameter that you set when making the API request.

You can enforce the requirement to pass the **session name** during role assumption. To do this, you need to add an [[IAM role trust policy]], that allows role to be assumed only when `"sts:RoleSessionName": [${list of allowed session names}]`.