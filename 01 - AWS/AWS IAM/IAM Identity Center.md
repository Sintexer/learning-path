> Successor to [[AWS Single Sign-On]]

AM Identity Center makes it easy to centrally manage federated access to **multiple AWS accounts** and business applications and provide users with single sign-on access to all their assigned accounts and applications from one place. You can use IAM Identity Center for identities in the Identity Center directory, your existing Microsoft Active Directory, or external [[IdP]].

IAM Identity Center provides the following benefits:

- Built-in integrations with business cloud applications, such as Salesforce, Box, GitHub, and Office 365.
- Built-in directory for user and group management to serve as an IdP to authenticate users to IAM Identity Center enabled applications, the AWS Management Console, and SAML 2.0 compatible cloud-based applications.
- Integration with AWS services, such as AWS Organizations.
- Log in CloudTrail of all sign-in and administrative activities for auditing. You can send these logs to SIEM solutions such as Splunk and Sumo Logic to analyze them.
- AWS Access portal for users to sign in with their existing corporate credentials and access all of their assigned accounts and applications from one place.
- Ability to use AWS CLI v2 to access AWS resources via IAM Identity Center. This has the benefit of providing short-term credentials to your users to more safely access your resources. You can automatically or manually configure a profile for the CLI to access resources in your AWS accounts.

IAM Identity Center integrates with Azure Active Directory, Okta Universal Directory, OneLogin, G Suite.

## Permission set

An IAM Identity Center permission set is a collection of administrator-defined [[IAM policy|policies]] that IAM Identity Center uses to determine a user's effective permissions to access a given AWS account. Permission sets can contain either [[IAM AWS managed policy|AWS-managed policies]] or custom policies. Permission sets are provisioned to the AWS account as [[IAM role|IAM roles]] and are presented to users as such. You can assign **more than one permission set to a user**. Users who have multiple permission sets must choose one of the roles when they sign in to the AWS access portal.

