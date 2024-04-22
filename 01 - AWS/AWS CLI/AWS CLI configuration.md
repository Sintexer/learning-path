> [!info] **AWS requires date/time stamp for requests**
> AWS requires that all incoming requests are cryptographically signed. The AWS CLI does this for you. The "signature" includes a date/time stamp. Therefore, you must ensure that your computer's date and time are set correctly. If you don't, and the date/time in the signature is too far off of the date/time recognized by the AWS service, AWS rejects the request.

When you run [[AWS CLI]] commands, the AWS CLI looks for credentials in a specific order—first in environment variables and then in the configuration file. Therefore, after you've put the temporary credentials into environment variables, the AWS CLI uses those credentials by default.

## Command `aws configure`

For general use, the `aws configure` command is the fastest way to set up your [[AWS CLI]] installation. This configure wizard prompts you for each piece of information you need to get started. Unless otherwise specified by using the `--profile` option, the AWS CLI stores this information in the `default` profile.

### AWS CLI config files

Command `aws configure` help you filling the AWS CLI configuration file. Configuration is usually located at `~/.aws`, and consists of two files: **config** and **credentials**. **config** contains per-profile configuration of default region and output format. **credentials** stores per-profile access keys and session tokens. This command does not suitable for [[IAM temporary credentials|temporary credentials]], instead see [[AWS CLI with temporary credentials]].

**Credentials file:**

```
[default]
aws_access_key_id=ASIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
aws_session_token = IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE

[user1]
aws_access_key_id=ASIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
aws_session_token = fcZib3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE
```

**Config file:**

```
[default]
region=us-west-2
output=json

[profile user1]
region=us-east-1
output=text
```

## Configuration and credentials precedence

Credentials and configuration settings are located in multiple places, such as the system or user environment variables, local AWS configuration files, or explicitly declared on the command line as a parameter. Certain locations take precedence over others. The AWS CLI credentials and configuration settings take precedence in the following order:

1. **[Command line options](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html)** – Overrides settings in any other location, such as the `--region`, `--output`, and `--profile` parameters.
2. **[Environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)** – You can store values in your system's environment variables.
3. **[Assume role](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html)** – Assume the permissions of an IAM role through configuration or the [`aws sts assume-role`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html) command.
- **[Assume role with web identity](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html)** – Assume the permissions of an IAM role using web identity through configuration or the [`aws sts assume-role`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html) command.
- **[AWS IAM Identity Center](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)** – The IAM Identity Center configuration settings stored in the `config` file are updated when you run the `aws configure sso` command. Credentials are then authenticated when you run the `aws sso login` command. The `config` file is located at `~/.aws/config` on Linux or macOS, or at ``C:\Users\`USERNAME`\.aws\config`` on Windows.
- **[[#AWS CLI config files|Credentials file]]** – The `credentials` and `config` file are updated when you run the command `aws configure`. The `credentials` file is located at `~/.aws/credentials` on Linux or macOS, or at ``C:\Users\`USERNAME`\.aws\credentials`` on Windows.
- **[Custom process](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html)** – Get your credentials from an external source.
- **[Configuration file](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)** – The `credentials` and `config` file are updated when you run the command `aws configure`. The `config` file is located at `~/.aws/config` on Linux or macOS, or at ``C:\Users\`USERNAME`\.aws\config`` on Windows.
- **[Container credentials](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)** – You can associate an IAM role with each of your Amazon Elastic Container Service (Amazon ECS) task definitions. Temporary credentials for that role are then available to that task's containers. For more information, see [IAM Roles for Tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) in the _Amazon Elastic Container Service Developer Guide_.
- **[Amazon EC2 instance profile credentials](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)** – You can associate an IAM role with each of your Amazon Elastic Compute Cloud (Amazon EC2) instances. Temporary credentials for that role are then available to code running in the instance. The credentials are delivered through the Amazon EC2 metadata service. For more information, see [IAM Roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) in the _Amazon EC2 User Guide for Linux Instances_ and [Using Instance Profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) in the _IAM User Guide_.