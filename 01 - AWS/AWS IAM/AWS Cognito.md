AWS Cognito is the preferred way to use [[AWS Identity federation#Web-Based federation|web identity federation]] for mobile applications. You can define roles and map users to different roles so that your app can access only the resources that are authorized for each user. Amazon Cognito scales to millions of users and supports sign-in with social [[Identity provider|identity providers]], such as Apple, Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0. User sign-in can also be done directly via Amazon Cognito.

The authentication workflow with Amazon Cognito starts by having the user log in to a web IdP. Then, the user device sends a `GetId` API call to establish a new identity in Amazon Cognito or return the identity already associated with that particular device. 

The device will send the token received by the web IdP to Amazon Cognito to validate with the provider and ensure that the token: 

- Is valid and from the configured provider.
- Is not expired.
- Matches the application identifier created with that provider.
- Matches the user identifier.

Then, after you establish an Amazon Cognito identity ID, the `GetCredentialsForIdentity` API can be called. Amazon Cognito will once again validate the web IdP token before making the `AssumeRoleWithWebIdentity` API call to [[AWS STS]] for the [[IAM temporary credentials|temporary security credentials]].