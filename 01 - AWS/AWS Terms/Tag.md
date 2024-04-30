In AWS, tags are labels that you assign to AWS resources. Each tag consists of a key and a value, both of which you define. They allow you to manage, search for, and filter resources.

Tags are also useful in the scope of security as they are included to the [[IAM Authorization context]] and might be used to build [[IAM ABAC|attribute-based access control strategy]].

> [!example]
> You can use tags to categorize resources by purpose, owner, environment, or other criteria. You might tag an Amazon EC2 instance with "Environment: Test" to denote that it's part of your test environment, or "Owner: John" to indicate that John is the person responsible for managing it.

Tags are a useful tool for managing resources at scale, helping you track cost, enforce compliance protocols, control access, and organize resources effectively.