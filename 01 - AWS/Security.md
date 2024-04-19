---
sr-due: 2024-04-21
sr-interval: 2
sr-ease: 250
tags:
  - flashcards
  - review
---

# Shared responsibility

AWS manages security **OF** the cloud, while users must manage security **IN** the cloud. This is called **shared responsibility** between AWS and you. 

AWS is responsible for security of the cloud. This means that AWS protects and secures the infrastructure that runs the services offered in the AWS Cloud. It includes protecting and securing AWS [[Regions#Region|Regions]], AZs, and data centers, down to the physical security of the buildings for low-level services (EC2), and managing hardware and software and encryption for services with higher levels of abstraction ([[Storage and S3#Simple Storage Service (S3)|S3]]).

The customers' level of responsibility depends on the AWS service. Some services require the customer to perform all the necessary security configuration and management tasks. Other more abstracted services require customers to only manage the data and control access to their resources.

# User

Identity that has access to all AWS services and resources is called **root user**. It is accessed by signing in with email and password that were used to create the account. 

For programmatic access ([[AWS CLI]], AWS API) to the root user account, it has **access keys**. Access keys consist of *key id* and *secret*. Both of them are required to access the account - similar to the *email + password* combination.

> Access keys consist of two parts:
> 
> - **Access key ID:** for example, _**A2lAl5EXAMPLE**_
> - **Secret access key:** for example, _**wJalrFE/KbEKxE**_

## MFA (Multi-Factor Authentication)

MFA requires two or more authentication methods to verify an identity. MFA usually consists of:

1. **Something you know** - Something you know, such as a user name and password or pin number
2. **Something you have** - Something you have, such as a one-time passcode from a hardware device or mobile app
3. **Something you are** - Something you are, such as a fingerprint or face scanning technology

With a combination of this information, systems can provide a layered approach to account access. So even if the first method of authentication, like Bob’s password, is cracked by a malicious actor, the second method of authentication, such as a fingerprint, provides another level of security. This extra layer of security can help protect your most important accounts, which is why you should activate MFA on your AWS root user. 

There are plenty of MFA devices. AWS supports 3: **Virtual MFA (DUO)** - mobile app with short-lived codes, **Hardware TOTP** - hardware device that generates a one-time password, **FIDO security key** - physical USB that could be plugged into USB port.

# IAM (Identity and Access Management)

> **Authentication answers the question, "Are you who you say you are?" Authorization answers the question, "What actions can you perform?"**

