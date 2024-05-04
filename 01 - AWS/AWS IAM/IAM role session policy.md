> Session policies allows you to **narrow the scope of permissions** granted when assuming a role. When you are using session policy, your *effective* session permissions are the intersection of the permissions on the role and the session policy.

A session policy is an inline permissions policy that users pass in the session when they assume the role. You can pass the policy yourself, or you can configure your broker to insert the policy when your identities federate in to AWS.

Session policies allows administrators to:

- Reduce the number of roles they need to create because multiple users can assume the same role yet have unique session permissions. 
- Set permissions for users to perform only those specific actions for that session. If users don’t require all the permissions associated with the role to perform a specific action in a given session, you can configure the identity broker to pass a session policy to reduce the scope of session permissions when users assume the role.

When you pass session policies, the resulting  permissions are the *intersection* of the IAM entity's [[IAM Identity-based policy]] and the **session policies**. You can use AWS managed or customer managed policies as session policies and also apply the same session permissions for multiple sessions. When an [[IAM Resource-based policy]] is being used, you can specify the [[ARN]] of the user or role as a principal. In that case, the permissions from the resource-based policy are added to the role or user's identity-based policy before the session is created. *The session policy limits the total permissions granted by the resource-based policy and the identity-based policy*. The resulting permissions are the intersection of the session policies and either the resource-based policy or the identity-based policy.

![[IAM role session policy scheme.png]]