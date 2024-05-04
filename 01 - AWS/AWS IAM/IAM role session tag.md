Session tags are attributes passed in an [[IAM role]] **session** when you [[IAM role assume|assume a role]] or federate a user using the AWS CLI or AWS API. You can use **session tags** for access control in [[IAM policy|IAM policies]] and for monitoring. These tags are not stored in AWS and are valid only for the duration of the session. You define session tags just like tags in AWS, consisting of a customer-defined key and an optional value.

To be able to add session tags, you must have the `sts:TagSession` action allowed in your IAM policy. [[IAM role trust policy|Trust policy]] can specify set of required session tags using condition key `aws:RequestTag` (see [[IAM policy condition#Global Condition keys|global condition keys]].

Session tags consideration:

- Session tags are principal tags that you specify while requesting a session. 
- Session tags must follow the [rules for naming tags in IAM and AWS STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_tags.html#id_tags_rules_creating). This topic includes information about case sensitivity and restricted prefixes that apply to your session tags.
- New session tags override existing assumed role or federated user tags with the same tag key, regardless of case.
- You cannot pass session tags using the [[AWS Management Console]].
- Session tags are valid for only the current session.
- You can use session tags to control access to resources or to control the tags that can be passed into a subsequent session. 
- You can pass a maximum of 50 session tags.
- You can view the principal tags for your session, including its session tags, in the [[AWS CloudTrail]] logs.

## Transitive tag

Session tags support role chaining. Role chaining occurs when you use a role to assume a second role through the AWS CLI or API. You can assume one role and then use the temporary credentials to assume another role and continue from session to session. *By default, tags are not passed to subsequent role sessions*. However, you can set session tags as **transitive**. This ensures that those session tags pass to subsequent sessions in a role chain.

When you chain roles, you can ensure that tags from an earlier session persist to the later sessions. To do this using the assume-role CLI command, you must pass the tag as a session tag and set the tag as transitive. 

Role chaining is especially useful when you want to impose guardrails against yourself or an administrator in order to prevent something accidental. For example, maybe you assume a generic admin role on a weekly basis. Sometimes, all you need to perform are certain operations that are bounded by a more scoped-down role. You can allow the admin role to assume the other more scoped-down role when needed. This helps avoid management overhead by having roles individually assigned to single users.