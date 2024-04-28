A [[IAM user|user]] in AWS consists of a name, a password to sign in to the [[AWS Management Console]], and up to two access keys, which can be used with the API or CLI.

## Username and password

A **password policy** is a set of rules that define the type of password an [[IAM user]] can set. You should define a password policy for all of your IAM users to enforce strong passwords and to require your users to regularly change their passwords. Password requirements are similar to those found in most secure online environments.

## Multi-factor authentication

Multi-factor authentication (MFA) is an additional layer of security for accessing AWS services. With this authentication method, more than one authentication factor is checked before access is granted, which consists of a user name, a password, and the single-use code from the MFA device. [[AWS CLI]] also supports MFA.

## User access keys

Users need their own access keys to make programmatic calls to AWS using the [[AWS CLI]] or the AWS SDKs, or direct HTTPS calls using the APIs for individual AWS services. Access keys are used to digitally sign API calls made to AWS services. Each access key credential consists of an **access key ID** and a **secret key**. Each user can have **two active access keys**, which is useful when you need to rotate the user's access keys or revoke permissions.