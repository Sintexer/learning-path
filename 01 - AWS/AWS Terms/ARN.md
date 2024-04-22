> Amazon Resource Name

**Amazon ARN** is a unique address for any resource within the vast **AWS** ecosystem. Just like your home address helps identify your location, an **ARN** helps identify a specific resource in AWS, such as an [[EC2|EC2 instance]], an [[AWS S3 Bucket]], or an [[IAM user|IAM user]].

An ARN follows a specific format:

```
arn:<partition>:<service>:<region>:<account-id>:<resource-type>:<resource-id>
```

* **partition:** Identifies the AWS partition (e.g., aws, aws-cn, aws-us-gov).
* **service:** Identifies the AWS service (e.g., ec2, s3, iam).
* **region:** Identifies the [[Region|AWS region]] (e.g., us-east-1, eu-west-2).
* **account-id:** Identifies your unique [[Root User|AWS account]] ID.
* **resource-type:** Identifies the type of resource (e.g., instance, bucket, user).
* **resource-id:** Identifies the specific instance of the resource (e.g., instance ID, bucket name, user name).

**Why are ARNs important?**

* **Uniquely identify resources:** ARNs provide a unique identifier for every resource in AWS, regardless of its location or type.
* **Simplify resource management:** ARNs allow you to easily reference and manage resources across different AWS services.
* **Enhance security:** ARNs are used in [[IAM policies]] to control access to AWS resources.

**Examples of ARNs:**

* `arn:aws:ec2:us-east-1:123456789012:instance/i-01234567890abcdef` (identifies an EC2 instance)
* `arn:aws:s3:::my-bucket` (identifies an S3 bucket)
* `arn:aws:iam::123456789012:user/Alice` (identifies an IAM user)

**Remember:**

* ARNs are case-sensitive.
* You can use the AWS Management Console or CLI to find the ARN of any resource.
* Understanding ARNs is essential for effectively managing and securing your AWS resources.
