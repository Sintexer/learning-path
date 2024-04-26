A trust policy defines what actions your role can assume. The trust policy allows Lambda to use the role's permissions by giving the service principal lambda.amazonaws.com permission to call the [[AWS STS]] `AssumeRole` action.

This example illustrates that the principal `"Service":"lambda.amazonaws.com"` can take the `"Action":"sts:AssumeRole"` allowing Lambda to assume the role and invoke the function on your behalf.

![[IAM Trust policy example.png]]