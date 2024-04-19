## IAM user

An IAM user represents a person or service that interacts with AWS. You define the user in your AWS account. Any activity done by that user is billed to your account. When you create a user, that user can sign in to gain access to the AWS resources inside your account. One account can have many users, each with their own login credentials. First IAM user in the account must be created by [[Root User]].

You can provide user with two types of access: **Access to AWS Management Console** and **Programmatic Access to [[AWS CLI]] and [[AWS API]]**. IAM user credentials are considered permanent, until there's a forced rotation by admin.

When creating IAM user, you can grant permissions at the user level. It might be a good idea if there will be at most 3 or 4 users. However, as the number of users increases, keeping up with permissions can become more complicated. For example, if you have 3,000 users in your AWS account, administering access and getting a top-level view of who can perform what actions on which resources can be challenging.  Fortunately, you can group IAM users and attach permissions at the [[IAM#IAM group|group]] level.

[[IAM group]]