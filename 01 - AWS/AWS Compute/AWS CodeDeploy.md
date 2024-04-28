[[AWS Lambda]] is integrated with AWS CodeDeploy for automated rollout with traffic shifting. CodeDeploy supports multiple traffic shifting methods, in addition to alarms and hooks. CodeDeploy supports the following traffic-shifting patterns:

- **Canary** – Traffic is shifted in two increments. If the first increment is successful, the second is completed based on the time specified in the deployment. 
- **Linear** – With linear traffic shifting, traffic is slowly shifted in a predetermined percentage every _X_ minutes based on how you have it configured. 
- **All-at-once** – Shifts all traffic from the original Lambda function to the updated Lambda function version at once.

Additionally, it supports the following testing options:

- **Alarms** – These instruct CloudWatch to monitor the deployment and trigger an alarm if any errors occurred during rollout. Any alarms would automatically roll back your deployment. 
- **Hooks** – Give you the option to run pre-traffic and post-traffic test functions that run sanity checks before traffic-shifting starts to the new version and after traffic-shifting completes. 

**Note:** When the alarms or hooks trigger a rollback, everything in the [[AWS CloudFormation]] template being deployed is rolled back. Best practice is to keep your [[AWS SAM]] templates and CloudFormation templates as concise in scope as possible. As a guideline, examine no fewer than one template per service that you are deploying.