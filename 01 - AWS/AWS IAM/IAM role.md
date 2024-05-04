IAM roles allow you to delegate access to users, applications, or services that normally don't have access to your organization's AWS resources. You can assume a role to obtain temporary security credentials that you can use to make AWS API calls. [[IAM policy]] could be attached to role. To define which actions a role can assume, it must have [[IAM role trust policy]].

All HTTP API requests to AWS must be authenticated. Users are using credentials or [[AWS Access keys|access keys]]. But what if one [[EC2]] instance must make a request to a AWS service? You don't want to hardcode access codes in your apps or create users for your services. **IAM roles** come handy in such cases.

IAM roles have **permissions** ([[IAM policy|policies]]) associated with it, but unlike users, roles doesn't have a username or password. Apps programmatically use roles to gain access keys to then use them to sign requests. Roles are designed for specific AWS services, like **EC2** or **AWS Lambda**.

Roles are also extremely useful in the scope of [[Identity federation]]. If you have enterprise users elsewhere, but you want them to have access to AWS services, should you create a separate user for each external user account? The answer is No, use IAM roles instead. AWS could assign role to a [[IAM Federated user]] when a sign in request is received. A special AWS service [[AWS Identity Center]] could handle such scenarios. See [[IAM with identity provider]]

When IAM entity wants to use a role, it must call [[AWS STS]] service in order to [[IAM role assume|assume a role]].