> AKA **AWS Security Token Service**

You may want to grant temporary access and limit access to users. You can use AWS STS to create and provide trusted [[IAM user|IAM users]] or users that you authenticate ([[IAM Federated user|federated users]]) with [[IAM temporary credentials|temporary security credentials]] that can control access to your AWS resources. 

AWS STS is a global service. All AWS STS API requests go to a single endpoint at [https://sts.amazonaws.com](https://sts.amazonaws.com). When a user or application requires temporary security credentials to access AWS resources, they make the AssumeRole API request. These temporary credentials consist of an access key ID, a secret access key, and a security token. Each time a role is assumed and a set of temporary security credentials is generated, an IAM role session is created.

