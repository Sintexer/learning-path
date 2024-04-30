A trust policy is a [[IAM Resource-based policy|resource-based policy]], that defines what actions and what principal can assume the [[IAM role]]. The trust policy allows [[AWS Lambda]] and other services to use the role's permissions by giving the [[IAM policy principal|service principal]] (like lambda.amazonaws.com) permission to call the [[AWS STS]] `AssumeRole` action.

This example illustrates that the principal `"Service":"lambda.amazonaws.com"` can take the `"Action":"sts:AssumeRole"` allowing Lambda to assume the role and invoke the function on your behalf.

![[IAM role trust policy example.png]]