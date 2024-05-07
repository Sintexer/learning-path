AWS **Identity and Access Management** (**IAM**) is a global *AWS service* that helps you **manage access** to your AWS account and resources. It also provides a centralized view of [[Authentication]] and [[Authorization]]. With IAM, you can share access to an AWS account and resources *without sharing your set of [[AWS Access keys]] or password*. You can also provide granular access to those working in your account, so people and services only have permissions to the resources that they need. 

Best way to work with IAM is to follow the [[Least privilege principle]]. When user makes a request, AWS checks all associated permissions according to [[IAM Access granting]] to understand whether ALLOW or DENY an action.

> For example, to provide a user of your AWS account with read-only access to a particular AWS service, you can granularly select which actions and which resources in that service that they can access.

IAM is global and not specific to one [[Region]]. It integrates with many AWS services by default. IAM enables permissions sharing without actually giving your password and key. IAM supports [[MFA]] for whole account and for individual users. IAM also supports [[Identity federation]].

IAM is completely free.

## IAM best practices

### 1. Lock down [[AWS Root User]]

Root user has too much power. To lock down your root user you should **prevent credentials sharing**, **delete root user [[AWS Access keys|access keys]]**, and **activate [[MFA]] on the root account**.

### 2. Follow the [[Least privilege principle]]

Grant only the necessary permissions to do a particular job and nothing more. To implement least privilege for access control, start with the minimum set of permissions in an IAM policy and then grant additional permissions as necessary for a [[IAM user|user]], [[IAM group|group]], or [[IAM role|role]]. Use secondary services [[IAM Access Analyzer]] and [[IAM policy simulator]] to better control the permissions across your AWS accounts. [[IAM Access history]] might also be useful to tackle down unused permissions to enforce the [[POLP]].

### 3. Use IAM properly

IAM is used to secure access to your AWS account and resources. It provides a way to create and manage  [[IAM user|users]], [[IAM group|groups]], or [[IAM role|roles]]. to access resources in a single AWS account. 

IAM **is not** used for website [[Authentication and Authorization]], such as providing users of a website with sign-in and sign-up functionality. IAM also **does not** support security controls for protecting operating systems and networks.

### 4. Use IAM roles when possible

Maintaining [[IAM role|roles]] is more efficient than maintaining users. User access keys only expire when you or the account admin rotates the keys. User login credentials expire if you applied a password policy to your account that forces users to rotate their passwords.

### 5. Consider using an identity provider

It is much simpler to manage one user in [[Identity provider]] with assigned IAM roles, than managing several IAM users attached to one person. If this person leaves company, you can simply remove IAM role from the federated account. See [[IAM with identity provider]]

### 6. Regularly review and clean up unused users, groups, permissions and roles

You might have IAM users, roles, permissions, policies, or credentials that you are no longer using in your account. IAM provides last accessed information to help you identify irrelevant credentials that you no longer need so that you can remove them. This helps you reduce the number of users, roles, permissions, policies, and credentials that you have to monitor.

### What next

- [[IAM user]]
- [[IAM group]]
- [[IAM policy]]
- [[IAM identity]]
- [[IAM role]]