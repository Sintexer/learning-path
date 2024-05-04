Assuming a role in AWS means temporarily taking on a set of permissions different from your standard user permissions. For example, if you need to perform a task that requires extra permissions, you can assume an IAM role that has these permissions. When you're done with the task, you can then revert back to your original user permissions. The entity that assumes the role will lose its original privileges and gain the access associated with the role.

This concept is a core part of AWS's security best practices, as it allows for the [[POLP]] - users only have the permissions they need for a task while they're performing it, reducing the risk of unauthorized access or actions.

When you assume a **role**, IAM dynamically provides [[IAM temporary credentials|temporary credentials]] that expire after a defined period of time, between 15 minutes and 36 hours. Users, on the other hand, have long-term credentials in the form of user name and password combinations or a set of access keys.  [[STS Assume role request]] allows you to specify some additional info for the temporary credentials, like duration, [[IAM role session policy|inline session permissions]], session tags, or [[IAM role session name|a session name]].

## Service roles

IAM roles that can be assumed by an AWS service are called **service roles**. For example, a role that allows service to manage IAM roles. [[IAM policy condition]] might be useful in scope of service roles, to restrict access to this role from unwanted services.