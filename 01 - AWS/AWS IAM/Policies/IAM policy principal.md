[[IAM policy]] might specify [[AWS Principal]] that is the subject for a permission. Principal is specified inside policy statement and might refer any of AWS principal types.

## AWS accounts

When you use an AWS account identifier as the principal in a policy, you **delegate authority to the account**. Within that account, the permissions in the policy statement can be granted to **all identities**, including [[IAM user|IAM users]] and [[IAM role|roles]] in that account. When you specify an AWS account, you can use the [[ARN]] `arn:aws:iam::<AWS-account-ID>:root` or a shortened form that consists of the `aws:` prefix followed by the `account ID`.

```json
"Pricnipal": { "AWS": "arn:aws:iam::123456789012:root" }
```

## IAM users

You can specify an individual IAM user (or array of users) as the principal, as in the following examples. When you specify more than one principal in the element, you grant permissions to each principal. This is a logical OR and not a logical AND because you are authenticated as one principal at a time. In a Principal element, the user name is case-sensitive. When you specify users in a Principal element, you cannot use a wildcard `(*)` to mean "all users." Principals must always name a specific user or users.

```json
"Principal": { "AWS": [
	"arn:aws:iam::123456789012/username1"
]}
```

## Federated users

If you already manage user identities outside AWS, you can use IAM identity providers instead of creating IAM users in your AWS account. With an [[Identity provider]], you can manage your user identities outside AWS and give these external user identities permissions to use AWS resources in your account. IAM supports [[SAML]]-based IdPs and web identity providers, such as Login with Amazon, Amazon Cognito, Facebook, or Google.

```json
"Principal": { 
	"Federated": "www.amazon.com" 
}

"Principal": { 
	"Federated": "arn:aws:iam::123456789012:saml-provider/provider-name" 
}
```

## IAM roles

You can use [[IAM role|roles]] to delegate access to users, applications, or services that don't normally have access to your AWS resources. This is related to feature of [[IAM role#Assuming a role|role assumption]].

```json
"Principal": { "AWS": "arn:aws:iam::123456789012:role/<role-name>" }
```

## AWS services

[[IAM role#Service roles|Service roles]] must include a [[IAM role trust policy|trust policy]]. Some service roles have predefined trust policies. However, in some cases, you must specify the service principal in the trust policy. A service principal is an identifier that is used to grant permissions to a service.

The identifier includes the long version of a service name and is usually in the _long_service-name.amazonaws.com_ format. The following example shows a policy that can be attached to a service role. The policy enables two services—Amazon EMR and AWS Data Pipeline—to assume the role.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		"Effect": "Allow",
		"Principal": {
			"Service": [
				"elasticmapreduce.amazonaws.com",
				"datapipeline.amazonaws.com"
			]
		}
	]
}
```