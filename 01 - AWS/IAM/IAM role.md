IAM role grants temporary access to AWS account resources. [[IAM policies]] could be attached to role.

All HTTP API requests to AWS must be authenticated. Users are using credentials or [[AWS Access keys|access keys]]. But what if one [[EC2]] instance must make a request to a AWS service? You don't want to hardcode access codes in your apps or create users for your services. **IAM roles** come handy in such cases.

IAM roles have **permissions** ([[IAM policies|policies]]) associated with it, but unlike users, roles doesn't have a username or password. Apps programmatically use roles to gain access keys to then use them to sign requests. Roles are designed for specific AWS services, like **EC2** or **AWS Lambda**.

Roles are also extremely useful in the scope of [[Identity federation]]. If you have enterprise users elsewhere, but you want them to have access to AWS services, should you create a separate user for each external user account? The answer is No, use IAM roles instead. AWS could assign role to a [[Federated user]] when a sign in request is received. A special AWS service [[AWS Identity Center]] could handle such scenarios. 