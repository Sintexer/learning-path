AWS offers different solutions for [[Identity federation|federating]] your employees, contractors, and partners (workforce) to AWS accounts and business applications, and for adding federation support to your customer-facing web and mobile applications. AWS supports commonly used open identity standards, including Security Assertion Markup Language 2.0 (SAML 2.0), Open ID Connect ([[OIDC]]), and OAuth 2.0.

AWS provides single sign-on to AWS accounts and central managed access to resources through [[IAM Identity Center]]. Fine grained access to AWS could be achieved by [[IAM]]. Access to web and mobile apps is configured through [[AWS Cognito]].

## SAML-Based federation

You can enable federated access to AWS accounts using [[IAM]] and [[AWS STS]], which allows you to enable a separate SAML 2.0-based [[Identity provider]] for each AWS account and use federated user attributes for access control. With IAM, you can **pass user attributes**, such as cost center or job role, from your [[IdP|IdPs]] to AWS, and implement fine-grained access permissions based on these attributes. IAM helps you define permissions once and then grant, revoke, or modify AWS access by simply changing the attributes in the IdP.

AWS allows to configure [[IAM ABAC]] with SAML-Based federation. All you need is to configure tags for the resources and create a special role.

Before your application can call `AssumeRoleWithSAML`, you must configure your SAML IdP to issue the claims that AWS requires. Additionally, you must use IAM to create a SAML provider entity in your AWS account that represents your identity provider. You must also create an [[IAM role]] that specifies this SAML provider in its [[IAM role trust policy|trust policy]].

`AssumeRoleWithSAML` request can include additional attributes, such as `DurationSeconds` - role session duration, `Policy` - [[IAM role session policy|a session policy]], and `PolicyArns.member.N` - [[IAM AWS managed policy|IAM managed policies]] that you want to use as managed session policies. 

The `AssumeRoleWithSAML` call returns a set of temporary security credentials for users who have been authenticated via a SAML authentication response. This operation provides a mechanism for tying an enterprise identity store or directory to role-based AWS access without user-specific credentials or configuration.

## Web-Based federation

[[Identity federation]] is also available for your AWS customer-facing web and mobile applications via a web identity provider. Examples of web identity providers supported by AWS include [[AWS Cognito]], Login with Amazon, Facebook, Google, or any [[OIDC|OpenID Connect]]-compatible identity provider.

Before your application can call `AssumeRoleWithWebIdentity`, you must have an identity token from a supported identity provider and create a role that the application can assume. The role that your application assumes must trust the identity provider that is associated with the identity token. In other words, the identity provider must be specified in the role's [[IAM role trust policy|trust policy]].

Calling `AssumeRoleWithWebIdentity` does not require the use of AWS security credentials. Therefore, you can distribute an application (for example, on mobile devices) that requests temporary security credentials without including long-term AWS credentials in the application. You also don't need to deploy server-based proxy services that use long-term AWS credentials. Instead, the identity of the caller is validated by using a token from the web identity provider.

`AssumeRoleWithWebIdentity` request can include additional attributes, such as `DurationSeconds` - role session duration, `Policy` - [[IAM role session policy|a session policy]], and `PolicyArns.member.N` - [[IAM AWS managed policy|IAM managed policies]] that you want to use as managed session policies. 

The temporary security credentials returned by `AssumeRoleWithWebIdentity` API consist of [[IAM temporary credentials]]. Applications can use these temporary security credentials to sign calls to AWS service API operations.