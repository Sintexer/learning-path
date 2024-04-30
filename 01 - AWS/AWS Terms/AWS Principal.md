A principal in AWS generally refers to an entity that is allowed to interact with AWS resources. This entity can be 

1. [[IAM user]]
2. [[IAM role]]
3. [[IAM Federated user]]
4. AWS service
5. Anonymous user
6. AWS account.

 **Principal** is a person, [[IAM role]], or application that can make a request for an action or operation on an *AWS resource*. The principal is authenticated as the [[AWS Root User]] or an IAM entity to make requests to AWS. Principal are parsed and included into [[IAM Authorization context]] during action authorization.
 
AWS principals are usually used inside policies, see [[IAM policy principal]]: