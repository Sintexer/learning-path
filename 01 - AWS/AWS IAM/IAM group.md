## IAM group

An IAM group is a collection of [[IAM#IAM user|users]] All users in the group inherit the permission assigned to the group. A JSON document could be assigned to group to define group permissions. This JSON is called [[IAM policy|IAM policy]]. It is usually a best practice to follow [[Least privilege principle]].

> A group cannot be added to another group. But one user can be added to many groups.


> [!example]
>- If you have an application that you’re trying to build and you have multiple users in one account working on the application, you might **organize the users by job function**. For example, you might organize your IAM groups by *developers*, *security*, and *admins*. You could then place all your IAM users into their respective groups.
> - A new developer joins your AWS account to help with your application. You create a new user and add them to the developer group, without thinking about which permissions they need.
> - A developer changes jobs and becomes a security engineer. Instead of editing the user’s permissions directly, you remove them from the old group and add them to the new group that already has the correct level of access.

This provides a way to see who has what permissions in your organization. It also helps you scale when new people join, leave, and change roles in your organization.

To keep in mid:
- Groups can have many users.
- Users can belong to many groups.
- Groups cannot belong to groups.

![[AWS Root User#IAM user best practice]]