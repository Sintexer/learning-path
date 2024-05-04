> AKA **Attribute-based access control** 

Attribute-based access control is an authorization strategy that defines permissions based on attributes. In AWS, these attributes are called [[Tag|tags]]. Tags can be attached to [[IAM policy principal|IAM principals]] (users or roles), to AWS resources, or even to [[IAM role session tag|IAM role session]]. IAM ABAC strategy relies on [[IAM policy condition|policy conditions]] mechanism.

You can create a single ABAC policy or small set of policies for your IAM principals. These ABAC policies can be designed to allow operations when the principal's tag matches the resource tag. ABAC is helpful in environments that are growing rapidly and helps with situations where policy management becomes cumbersome.

Pros:

- **Granularity:** ABAC allows for fine-grained access control, which is ideal for complex environments.
- **Flexibility:** ABAC can handle a wide range of access control scenarios, including context-aware and risk-adaptive access control.
- **Efficiency:** With ABAC, you can manage permissions for diverse and dynamic user groups without needing to create a multitude of roles.

Cons:

- **Complexity:** ABAC can be complex to set up and manage due to the number of attributes and conditions that can be used.
- **Performance:** Evaluating multiple attributes and conditions can potentially impact performance, especially in large-scale environments.

## Implementation details

Users that are [[AWS Identity federation|federated into AWS]] can also use ABAC. User attributes can be passed as [[IAM role session tag|session tags]] using standards-based SAML. You can use attributes defined in external identity systems as part of attributes-based access control decisions within AWS. Administrators of the [[IdP]] manage user attributes and define attributes to pass in during federation.

> [!example]
> Your systems engineer configures your IdP to include `"CostCenter"` as a session tag when users federate into AWS using an [[IAM role]]. All federated users assume the same role but are granted access only to AWS resources belonging to their cost center. To setup the solution, first you need to *tag all project resources* with their respective tags and *configure the IdP* to include the `"CostCenter"` tag in the session. The IAM role for this scenario would then grant access to project resources based on the `"CostCenter"` tag with the `ec2:ResourceTag/CostCenter` condition key. Now, whenever users federate into AWS using this role, they get access to only the resources belonging to their cost center based on the `"CostCenter"` tag included in the federated session. If a user switches cost centers or is added to a specific cost center, your system administrator will only have to update the IdP, and the permissions in AWS will automatically apply to grant access to the proper cost center's AWS resources without requiring a permissions update in AWS