When a principal tries to use the [[AWS Management Console]], the [[AWS API]], or the [[AWS CLI]], that principal sends a request to AWS. AWS gathers the request information into a request context, which is used to evaluate and authorize the request. During authorization, AWS uses values from the request context to check for policies that apply to the request. It then uses the policies to determine whether to allow or deny the request.

Authorization context is used in [[IAM Access granting]].

Authorization context consists of:

1. Principal id
2. Action - like `iam:GetUser`
3. Resource - `arn:aws:iam::123456789012:user/Carol`
4. Context - list of different metadata associated with the authentication process and with the resource, like aws namespace, user id, principal id, org id, current time, resource [[Tag|tags]], principal tags, etc.

When a call is authenticated, IAM gathers context about the resource, [[AWS Principal]] and the call itself. The following are examples of the context that can be used from principal in the authentication process:

- The **principal** – `arn:aws:iam::<AWS-account-ID>:user/<username>`
- Principal **tags** – dept=123, project=blue
- **Condition keys** – `aws:UserID`, `aws:SourceIP`, `aws:PrincipalArn`, `aws:PrincipalAccount`, etc.
