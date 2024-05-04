With the IAM policy simulator, you can test and troubleshoot [[IAM Identity-based policy|identity-based policies]], [[IAM Permissions boundary|IAM permissions boundaries]], [[AWS Organizations]] service control policies, and [[IAM Resource-based policy|resource-based policies]]. The policy simulator **does not make** an actual AWS service request, so you can *safely* test requests that might make unwanted changes to your live AWS environment. The only result returned is whether the requested action would be allowed or denied.

Here are some common use cases for the IAM policy simulator:

- Test policies that are attached to IAM users, groups, or roles in your AWS account. If more than one policy is attached, you can test all the policies or select individual policies. You can test which actions are allowed or denied by the selected policies for specific resources.
- Test and troubleshoot the effect of permissions boundaries on IAM entities one permissions boundary at a time.
- Test policies that are attached to AWS resources.
- Test the impact of SCPs on your IAM policies and resource policies if your AWS account is a member of an organization in AWS Organizations.
- Test new policies that are not yet attached to a user, group, or role by typing or copying them into the simulator. These are used only in the simulation and are not saved.
- Simulate real-world scenarios by providing context keys, such as an IP address or date, that are included in Condition elements in the policies being tested.
- Identify which specific statement in a policy results in allowing or denying access to a particular resource or action.

You can allow console or API users to test policies that are attached to IAM users, groups, or roles in your AWS account. To do so, you must provide permission to retrieve those policies. Users must have permission to retrieve the resource's policy in order to test resource-based policies.