Identity that has access to all AWS services and resources is called **root user**. It is accessed by signing in with email and password that were used to create the account. 

For programmatic access ([[AWS CLI]], [[AWS API]]) to the root user account, it has **access keys**. Access keys consist of *key id* and *secret*. Both of them are required to access the account - similar to the *email + password* combination.

> Access keys consist of two parts:
> 
> - **Access key ID:** for example, _**A2lAl5EXAMPLE**_
> - **Secret access key:** for example, _**wJalrFE/KbEKxE

For greater security of user account, it should enable [[MFA]]