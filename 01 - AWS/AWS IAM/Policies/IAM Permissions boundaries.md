Restricts permissions for the IAM entity attached to it

A permissions boundary sets the maximum permissions that an [[IAM Identity-based policy]] can grant to an IAM entity. The entity can perform only the actions that are **allowed by both** its identity-based policies and its permissions boundaries.

>[[IAM Resource-based policy|Resource-based policies]] that specify the user or [[IAM role|role]] as the principal are not limited by the permissions boundary.

> [!example]
> For example, assume that one of your IAM users should be allowed to manage only Amazon S3, Amazon CloudWatch, and Amazon EC2. To enforce this rule, you can use the customer-managed policy shown below to set the permissions boundary for the user. Then, add the condition block below to the IAM user's policy. The user can never perform operations in any other service, including IAM, even if it has a permissions policy that allows it.
> 
> ```json 
> {
>	"Version": "2012-10-17",
>     "Statement": [
> 		{
> 		    "Effect": "Allow",
> 			"Action": [
> 				"s3:*",
> 				"cloudwatch:*",
> 				"ec2:*"
> 			 ],
> 			"Resource": "*"
> 		}
> 	]
> }
> ```

 ```json
 "Condition": {
     "StringEquals":{
	     "iam:PermissionsBoundary":"arm:aws:iam::123456789012:policy/<policyname>"
	}
}
```