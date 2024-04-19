Identity that has access to all AWS services and resources is called **root user**. It is accessed by signing in with *email and password* that were used to create the account. It is a bad practice to use root user as it has all the permissions. If you accidentally lose the access to this account, criminal would have access to all your billing and banking information, as well as it will have power to affect all the resources linked to the account.

## IAM user best practice

It is a **best practice** to create [[IAM#IAM user|IAM user]] instead **root user**, and assign only *needed* permissions to it. The **root user** can perform all actions on all resources inside an AWS account by default. This is in contrast to creating new [[IAM#IAM user|IAM users]], new [[IAM#IAM group|IAM groups]], or new roles. To allow an IAM identity to perform specific actions in AWS, such as implement resources, you must grant the IAM user the necessary permissions.  
  
The way you grant permissions in IAM is by using [[IAM#IAM policies|policies]].

## Root user access credentials

For programmatic access ([[AWS CLI]], [[AWS API]]) to the root user account, it has **access keys**. Access keys consist of *key id* and *secret*. Both of them are required to access the account - similar to the *email + password* combination.

> Access keys consist of two parts:
> 
> - **Access key ID:** for example, _**A2lAl5EXAMPLE**_
> - **Secret access key:** for example, _**wJalrFE/KbEKxE

For greater security of user account, it should enable [[MFA]]