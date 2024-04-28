One potential challenge to serverless deployments is that when the [[AWS Lambda function]]  is deployed, it becomes live immediately. This means that a function can potentially go live without testing it, which puts your working applications at risk. This risk is especially true if you move toward an automated CI/CD pipeline and need to easily promote new code or roll back if there's a problem. To mitigate this risk, you can version your Lambda functions and add aliases to ensure safe deployments.

## Versioning

You can use versions to manage the deployment of your functions. For example, you can publish a new version of a function for beta testing without affecting users of the stable production version. Lambda creates a new version of your function each time that you publish the function. The new version is a copy of the unpublished version of the function. 

When you create a Lambda function, only one version exists, which is identified by **$LATEST** at the end of the Amazon Resource Name (ARN).

`arn:aws:lambda:aws-region:acct-id:function:helloworld:$LATEST`

Publish makes a snapshot copy of $LATEST. 

Enable versioning to create immutable snapshots of your function every time you publish it. 

- Publish as many versions as you need. 
- Each version results in a new sequential version number. 
- Add the version number to the function ARN to reference it.
- The snapshot becomes the new version and is immutable.

## Aliases

A **Lambda alias** is like a pointer to a specific function version. You can access the function *version* using the alias ARN. Each alias has a unique ARN. An alias can point **only to a function version**, not to another alias. You can update an alias to point to a new version of the function.

### Alias routing

You can also use routing configuration on an alias to send a portion of traffic to a second function version. For example, you can reduce the risk of deploying a new version by configuring the alias to send most of the traffic to the existing version and only a small percentage of traffic to the new version.

You can point an alias to a maximum of two Lambda function versions. The versions must meet the following criteria:

- Both versions must have the same runtime role.
- Both versions must have the same dead-letter queue configuration, or no dead-letter queue configuration.
- Both versions must be published. The alias cannot point to $LATEST.