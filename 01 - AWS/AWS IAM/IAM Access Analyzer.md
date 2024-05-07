**IAM Access Analyzer** continuously monitors [[IAM policy|policies]] for changes where you no longer need to rely on intermittent manual checks in order to catch issues as policies are added or updated. Using IAM Access Analyzer, you can proactively address any resource policies that violate their security and governance best practices around resource sharing and protect their resources from unintended access. IAM Access Analyzer delivers comprehensive, detailed findings through the [[IAM]], [[AWS S3]], and [[AWS Security Hub]] consoles and also through its APIs. **Findings** can also be exported as a report for auditing purposes. 

IAM Access Analyzer findings provide definitive answers of who has *public* and *cross-account* access to AWS resources from outside an account.

You can use IAM Access Analyzer to help identify the required permissions for the [[IAM role]]. **IAM Access Analyzer** reviews your [[AWS CloudTrail]] logs over the date range that you specify and generates a policy template with only the permissions that the [[AWS Lambda function|function]] used during that time. For more information on IAM Access Analyzer, see [Using IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html) in the _AWS Identity and Access Management User Guide_.

IAM Access Analyzer supports the following resource types:

- [[IAM role|IAM roles]] – [[IAM role trust policy|Trust policies]] are analyzed and findings are generated for roles within the zone of trust. These findings are accessible by an external entity that is outside your zone of trust.
- AWS S3 buckets – Findings are generated when an AWS S3 bucket policy, ACL, or access point applied to a bucket grants access to an external entity.
- [[AWS KMS]] keys – Findings are generated if a key policy or grant allows an external entity to access the key.
- [[AWS Lambda function|AWS Lambda functions]] – Findings are generated for policies that grant access to the function to an external entity.
- [[AWS SQS]] queues – Findings are generated for policies that allow an external entity to access a queue.

To successfully configure and use Access Analyzer, the account you use must be granted the required permissions. To access and use all Access Analyzer features, you can apply the `IAMAccessAnalyzerFullAccess` managed policy to the account.

