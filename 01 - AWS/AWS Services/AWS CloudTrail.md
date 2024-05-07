AWS CloudTrail is a service that provides a record of actions taken by an [[IAM user]] or [[IAM role|role]]. CloudTrail captures all AWS service API calls as events, including calls from the console, AWS CLI, and API tools. CloudTrail is enabled on your AWS account when you create the account. If you create a trail, you can enable continuous delivery of CloudTrail events to an [[AWS S3 bucket]]. AWS services like [[AWS Athena]] can then be used to query the logs directly from the S3 bucket. If you don't configure a trail, you can still view the most recent events in the CloudTrail console in **Event history**.

AWS CloudTrail can be used to answer the following common questions:

- What actions did a user take over a given period of time?
- For a given resource, which AWS user has taken actions on it over a given time period?
- What is the source IP address of a given activity?
- Which user activities failed due to inadequate permissions?

If you are using AWS managed key for encryption at rest, usage of key is recorded in AWS CloudTrail. CloudTrail can tell who made the request, the services used, actions performed, parameters for the action, and response elements returned.

