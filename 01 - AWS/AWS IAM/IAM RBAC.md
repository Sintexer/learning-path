> AKA **Role-based access control**

RBAC is a strategy where permissions are assigned to specific roles, and then users are assigned these roles. In scope of AWS, RBAC roles are [[IAM policy]] attached to [[IAM user|users]], [[IAM group|groups]], or other [[AWS Principal|principals]]. For example, a role might be "Database Admin" and anyone assigned this role would have permissions related to database management. This approach is straightforward and works well in environments where job functions are clearly defined.

Pros:

- **Simplicity:** RBAC's role-based model is straightforward to understand and manage.
- **Scalability:** As your organization grows, you can easily create new roles and assign them to users.
- **Auditability:** Since permissions are tied to roles, it's easy to audit access by reviewing the roles.

Cons:

- **Lack of Granularity:** RBAC can be too coarse when fine-grained access control is needed. It's not suitable for complex scenarios where access depends on context.
- **Role Explosion:** In large organizations, the number of roles can become unwieldy, leading to difficulty in managing and understanding them.