To use a policy to control access in AWS, you must first understand how [[IAM]] grants access. When you use the [[AWS API]], the [[AWS CLI]], or the [[AWS Management Console]] to take an action, such as creating a [[IAM role|role]] or activating an [[AWS Access keys||IAM user access key]], you send a **request** for that **action**. 

Take a look at how IAM evaluates the first statement of this policy:

1. IAM checks that the user (the principal) is authenticated (signed in) to perform the specified action on the specified resource.
2. IAM confirms that the user is authorized (has the proper permissions) by checking all the [[IAM policies|policies]] attached to your user.
3. During authorization, IAM verifies that the requested actions are allowed by the policies.
4. IAM also checks any policies attached to the resource that the user is trying to access. These policies are known as resource-based policies. If the [[IAM policies#Identity-based policies|identity-based policy]] allows a certain action but the [[IAM policies#Resource-based policies|resource-based policy]] does not, the result will be a **deny**.
5. AWS authorizes the request only if each part of your request is allowed by the policies. By default, all requests are denied. An explicit allow overrides this default, and an explicit deny overrides any allows. After your request has been authenticated and authorized, AWS approves the actions in your request. Then, those actions can be performed on the related resources within your account.

> [!info]
> By default, all requests are denied. An explicit allow overrides this default, and an explicit deny overrides any allows.