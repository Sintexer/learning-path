Everything in [[AWS S3]] is **private by default**. This means that all S3 resources, such as [[AWS S3 Bucket|buckets]] and [[AWS S3 Bucket#Object|objects]], can only be viewed by the user or AWS account that **created** that resource.
  
If you decide that you want everyone on the internet to see your photos, you can choose to make your buckets and objects *public*. A **public** resource means that everyone on the internet can see it. Most of the time, you don’t want your permissions to be all or nothing. Typically, you want to be more granular about the way that you provide access to your resources.

To be more specific about who can do what with your S3 resources, S3 provides several security management features: 

- [[IAM policy]], 
- [[#S3 bucket policies]]
- [[#AWS S3 Encryption|S3 encryption]] to develop and implement your own security policies.

You should **use IAM policies** for private buckets **in the following two scenarios**:

- You have many buckets with different permission requirements. Instead of defining many different S3 bucket policies, you can use IAM policies.
- You want all policies to be in a centralized location. By using IAM policies, you can manage all policy information in one location.

## S3 bucket policies

Like IAM policies, **S3 bucket policies** are defined in a JSON format. Unlike [[IAM policy]] which are attached to resources and users, S3 bucket policies can only be attached to S3 buckets. The policy that is placed on the bucket **applies to every object in that bucket**. S3 bucket policies specify what actions are allowed or denied on the bucket.

You should use S3 bucket policies in the following scenarios:

- You need a simple way to do cross-account access to S3, *without using [[IAM role|IAM roles]]*.
- Your IAM policies bump up against the defined size limit. **S3 bucket policies have a larger size limit than IAM policy**.

## AWS S3 Encryption

S3 reinforces *encryption* in transit (as it travels to and from S3) and at rest. To protect data, S3 automatically encrypts all objects on upload and applies **server-side encryption** with S3-managed keys as the base level of encryption for every bucket in Amazon S3 at no additional cost.

**Encryption solutions:**

1. **SSE-S3** - S3 [[SSE]] Uses keys handled and managed by AWS. Uses [[AES256]] encryption type. Must set header `"x-amz-server-side-encryption":"AES256"`.
2. **SSE-KMS** - Uses KMS. Gives user control over who has an access. Plus audit trial. Must set header `"x-amz-server-side-encryption":"aws:kms"`.
3. **SSE-C** - Amazon doesn't store client key, client must handle that. Client must use [[HTTPS]]. Key must be provided in HTTP headers for every HTTP request.
4. **Client side encryption** - Client must encrypt data before sending it to AWS and decrypt after retrieving it.