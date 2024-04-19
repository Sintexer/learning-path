AWS **Identity and Access Management** (**IAM**) is an *AWS service* that helps you **manage access** to your AWS account and resources. It also provides a ==centralized view of [[Authentication and Authorization#Authentication|Authentication]] and [[Authentication and Authorization#Authorization|Authorization]]==. With IAM, you can share access to an AWS account and resources *without sharing your set of access keys or password*. You can also provide ==granular access== to those working in your account, so people and services only have permissions to the resources that they need. 

> For example, to provide a user of your AWS account with read-only access to a particular AWS service, you can granularly select which actions and which resources in that service that they can access.

IAM is global and not specific to one [[Region]]. It integrates with many AWS services by default. IAM enables permissions sharing without actually giving your password and key. IAM supports [[MFA]] for whole account and for individual users. IAM also supports [[Identity federation]].

IAM is completely free.

### What next

- [[IAM user]]
- [[IAM group]]
- [[IAM policies]]