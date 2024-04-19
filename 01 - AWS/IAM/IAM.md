AWS **Identity and Access Management** (**IAM**) is an *AWS service* that helps you **manage access** to your AWS account and resources. It also provides a ==centralized view of [[Authentication and Authorization#Authentication|Authentication]] and [[Authentication and Authorization#Authorization|Authorization]]==. With IAM, you can share access to an AWS account and resources *without sharing your set of [[AWS Access keys]] or password*. You can also provide ==granular access== to those working in your account, so people and services only have permissions to the resources that they need. 

> For example, to provide a user of your AWS account with read-only access to a particular AWS service, you can granularly select which actions and which resources in that service that they can access.

IAM is global and not specific to one [[Region]]. It integrates with many AWS services by default. IAM enables permissions sharing without actually giving your password and key. IAM supports [[MFA]] for whole account and for individual users. IAM also supports [[Identity federation]].

IAM is completely free.

## IAM best practices

### 1. Lock down [[Root User]]

Root user has too much power. To lock down your root user you should **prevent credentials sharing**, **delete root user [[AWS Access keys|access keys]]**, and **activate [[MFA]] on the root account**.

### 2. Follow the principle of least privilege

Grant only the necessary permissions to do a particular job and nothing more. To implement least privilege for access control, start with the minimum set of permissions in an IAM policy and then grant additional permissions as necessary for a [[IAM user|user]], [[IAM group|group]], or [[IAM role|role]].

### 3. Use IAM properly

IAM is used to secure access to your AWS account and resources. It provides a way to create and manage  [[IAM user|users]], [[IAM group|groups]], or [[IAM role|roles]]. to access resources in a single AWS account. 

IAM **is not** used for website [[Authentication and Authorization]], such as providing users of a website with sign-in and sign-up functionality. IAM also **does not** support security controls for protecting operating systems and networks.

### 4. Use IAM roles when possible

Maintaining roles is more efficient than maintaining users. When you assume a role, IAM dynamically provides temporary credentials that expire after a defined period of time, between 15 minutes and 36 hours. Users, on the other hand, have long-term credentials in the form of user name and password combinations or a set of access keys.

User access keys only expire when you or the account admin rotates the keys. User login credentials expire if you applied a password policy to your account that forces users to rotate their passwords.

### 5. Consider using an identity provider

If you decide to make your cat photo application into a business and begin to have more than a handful of people working on it, consider managing employee identity information through an identity provider (IdP). Using an IdP, whether it's with an AWS service such as AWS IAM Identity Center (successor to AWS Single Sign-On) or a third-party identity provider, provides a single source of truth for all identities in your organization.

You no longer have to create separate IAM users in AWS. You can instead use IAM roles to provide permissions to identities that are _federated_ from your IdP. Being federated is a process that allows for the transfer of identity and authentication information across a set of networked systems. 

> For example, your employee Martha has access to multiple AWS accounts. Instead of creating and managing multiple IAM users named Martha in each of those AWS accounts, you could manage Martha in your company’s IdP. If Martha moves in the company or leaves the company, Martha can be updated in the IdP rather than in every AWS account in the company.

### 6. Regularly review and clean up unused users, groups, permissions and roles

You might have IAM users, roles, permissions, policies, or credentials that you are no longer using in your account. IAM provides last accessed information to help you identify irrelevant credentials that you no longer need so that you can remove them. This helps you reduce the number of users, roles, permissions, policies, and credentials that you have to monitor.

### What next

- [[IAM user]]
- [[IAM group]]
- [[IAM policies]]