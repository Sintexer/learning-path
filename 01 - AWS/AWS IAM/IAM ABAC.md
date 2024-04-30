> AKA **Attribute-based access control** 

Attribute-based access control is an authorization strategy that defines permissions based on attributes. In AWS, these attributes are called [[Tag|tags]]. Tags can be attached to [[IAM policy principal|IAM principals]] (users or roles) and to AWS resources. IAM ABAC strategy relies on [[IAM policy condition|policy conditions]] mechanism.

You can create a single ABAC policy or small set of policies for your IAM principals. These ABAC policies can be designed to allow operations when the principal's tag matches the resource tag. ABAC is helpful in environments that are growing rapidly and helps with situations where policy management becomes cumbersome.

Pros:

- **Granularity:** ABAC allows for fine-grained access control, which is ideal for complex environments.
- **Flexibility:** ABAC can handle a wide range of access control scenarios, including context-aware and risk-adaptive access control.
- **Efficiency:** With ABAC, you can manage permissions for diverse and dynamic user groups without needing to create a multitude of roles.

Cons:

- **Complexity:** ABAC can be complex to set up and manage due to the number of attributes and conditions that can be used.
- **Performance:** Evaluating multiple attributes and conditions can potentially impact performance, especially in large-scale environments.