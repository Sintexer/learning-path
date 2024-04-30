IAM roles allow you to delegate access to users, applications, or services that normally don't have access to your organization's AWS resources. You can assume a role to obtain temporary security credentials that you can use to make AWS API calls. [[IAM policy]] could be attached to role. To define which actions a role can assume, it must have [[IAM role trust policy]].

All HTTP API requests to AWS must be authenticated. Users are using credentials or [[AWS Access keys|access keys]]. But what if one [[EC2]] instance must make a request to a AWS service? You don't want to hardcode access codes in your apps or create users for your services. **IAM roles** come handy in such cases.

IAM roles have **permissions** ([[IAM policy|policies]]) associated with it, but unlike users, roles doesn't have a username or password. Apps programmatically use roles to gain access keys to then use them to sign requests. Roles are designed for specific AWS services, like **EC2** or **AWS Lambda**.

Roles are also extremely useful in the scope of [[Identity federation]]. If you have enterprise users elsewhere, but you want them to have access to AWS services, should you create a separate user for each external user account? The answer is No, use IAM roles instead. AWS could assign role to a [[IAM Federated user]] when a sign in request is received. A special AWS service [[AWS Identity Center]] could handle such scenarios. See [[IAM with identity provider]]

## Assuming a role

Assuming a role in AWS means temporarily taking on a set of permissions different from your standard user permissions. For example, if you need to perform a task that requires extra permissions, you can assume an IAM role that has these permissions. When you're done with the task, you can then revert back to your original user permissions. The entity that assumes the role will lose its original privileges and gain the access associated with the role.

This concept is a core part of AWS's security best practices, as it allows for the [[POLP]] - users only have the permissions they need for a task while they're performing it, reducing the risk of unauthorized access or actions.

When you assume a **role**, IAM dynamically provides *temporary credentials* that expire after a defined period of time, between 15 minutes and 36 hours. Users, on the other hand, have long-term credentials in the form of user name and password combinations or a set of access keys. 

### Service roles

IAM roles that can be assumed by an AWS service are called **service roles**. For example, a role that allows service to manage IAM roles. [[IAM policy condition]] might be useful in scope of service roles, to restrict access to this role from unwanted services.