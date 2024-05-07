As an administrator, you might grant permissions to [[IAM user|IAM users]] or [[IAM role|roles]] beyond what they require. Because of this, [[IAM]] provides last accessed information to help you identify unused permissions, refine your [[IAM policy|policies]], and allow access to only the services and actions that your IAM entities use. This information helps you to better adhere to the best practice of [[POLP|least privilege]]. You can view last accessed information for entities or policies that exist in IAM or [[AWS Organizations]].

You can view **two types** of last accessed information for IAM entities using the [[AWS Management Console]], [[AWS CLI]], or [[AWS API]]. The two types are **allowed AWS service information** and **allowed action information**. The information includes the date and time when the attempt was made. Action last accessed information is available only when [[AWS S3]] management actions, such as creation, deletion, and modification actions, are used.

> You can view two types of last accessed information:
> 1. Allowed AWS service information
> 2. Allowed action information (only for S3)

You can view information for each IAM resource below. In each case, the information includes allowed services for the given reporting period:

- **User** – View the last time that the user tried to access each allowed service.
- **Group** – View information about the last time that a group member attempted to access each allowed service. This report also includes the total number of members that attempted access.
- **Role** – View the last time that someone used the role in an attempt to access each allowed service.
- **Policy** – View information about the last time that a user or role attempted to access each allowed service. This report also includes the total number of entities that attempted access.

## Things to consider when using Access history

1. **Tracking period** - Recent activity usually appears in the IAM console within 4 hours. The tracking period for service information is the last 400 days.
2. **PassRole** - The `iam:PassRole` action is not tracked and is not included in IAM service last accessed information.
3. **Report owner** - Only the principal that generates a report can view the report details. If you use the AWS CLI or AWS API to get report details, your credentials must match the credentials of the principal that generated the report. If you use temporary credentials for a role or federated user, you must generate and retrieve the report during the same session.
4. **IAM entities** - The information for IAM includes IAM entities users or roles in your account. Information for Organizations includes IAM users, IAM roles, or the AWS account root user in the specified Organizations entity. The information does not include unauthenticated attempts.
5. **IAM policy types** - The information for IAM includes services that are allowed by an IAM entity's policies. These are policies either attached to a role or attached to a user directly or through a group. Access allowed by other policy types is not included in your report. The excluded policy types include [[IAM Resource-based policy|resource-based policies]], [[AWS Network Access Control List|access control lists]], AWS Organizations SCPs, [[IAM Permissions boundary|IAM permissions boundaries]], and [[IAM Session policy|session policies]]. Permissions that are provided by service-linked roles are defined by the service that they are linked to and can't be modified in IAM. 
6. **Required permissions** - To use the IAM console to view the last accessed information for an IAM user, role, or policy, you must have a policy that includes the following actions:
	1. `iam:GenerateServiceLastAccessedDetails`
	2. `iam:Get*`
	3. `iam:List*`
	To use the AWS CLI or AWS API to view last accessed information for IAM, you must have permissions that match the operation you want to use:
	- `iam:GenerateServiceLastAccessedDetails`
    - `iam:GetServiceLastAccessedDetails`
    - `iam:GetServiceLastAccessedDetailsWithEntities`
    - `iam:ListPoliciesGrantingServiceAccess`