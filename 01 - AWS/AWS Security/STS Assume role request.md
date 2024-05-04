Assume role request are sent by [[IAM identity|identities]] to the [[AWS STS]].

The AssumeRole request allows you to add several optional parameters to help further secure the role session when [[IAM role assume|assuming a role]]:

- **DurationSeconds** – By default, the temporary security credentials created by AssumeRole last for 1 hour. However, you can use this parameter to specify the duration of your session and further control the session. You can provide a value from 900 seconds (15 minutes) up to 12 hours.
- **Policy** – This parameter includes [[IAM policy]] that you want to use as an [[IAM role session policy|inline session policy]]. The resulting session's permissions are the intersection of the role's [[IAM Identity-based policy|identity-based policy]] and the session policies. 
- **PolicyArns.member.N** – This parameter includes the [[ARN|ARNs]] of the **managed policies** that you want to use as managed session policies. The policies must exist in the same account as the role. You can provide up to 10 managed policy ARNs.
- **Tags.member.N** – This parameter lists the session tags that you want to pass with the role. Each session tag consists of a key name and an associated value. An upcoming lesson will cover session tags in more detail.
- **SerialNumber** and **TokenCode** – You can include MFA information when you call AssumeRole with these parameters. This is useful for cross-account scenarios to ensure that the user who assumes the role has been authenticated with an AWS MFA device. In that scenario, the trust policy of the role being assumed includes a condition that tests for MFA.

The Assume role response contains the [[ARN]] of the assumed role and assumed role id, credentials and packed size, which indicates what percentage of max temp credentials size was taken by additional policies and tags provided with the assume role request. 

