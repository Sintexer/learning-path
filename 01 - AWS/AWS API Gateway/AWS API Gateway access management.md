API Gateway provides you with multiple, customizable options for:

- Authorizing an entity to access your APIs
- Providing more granular control
- Controlling the amount of access through throttling

there are three main ways to authorize API calls to your API Gateway endpoints:

1. [[AWS API Gateway IAM Authorization]]. Use IAM and Signature version 4 (also known as Sig v4) to authenticate and authorize entities to access your APIs.
2. [[AWS API Gateway Lambda Authorizer]]. Use Lambda Authorizers, which you can use to support bearer token authentication strategies such as OAuth or SAML. Also supports [[AWS Cognito]] user pools.
3. [[AWS API Gateway Cognito Authorizer]]. Use [[AWS Cognito]] with user pools.

Each of the authorizing options has advantages that should be matched to your application needs and organizational standards. For example, you need to consider who consumes the API. If it’s *external developers*, you probably want to consider using Lambda Authorizers or Cognito.