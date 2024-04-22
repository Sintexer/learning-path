[[AWS CLI configuration#Command `aws configure`|Command aws configure]] can set up regular credentials. But [[IAM temporary credentials]] are packed with [[IAM session token]]. `aws configure` does not set the session token. You must do it manually by editing credentials file or instead you could use **environment variables** as they have precedence over config file for aws cli.

**Linux**

```bash
export AWS_ACCESS_KEY_ID=ASIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_SESSION_TOKEN=AQoDYXdzEJr...<remainder of session token>
aws ec2 describe-instances --region us-west-1
```

**Windows**

```cmd
SET AWS_ACCESS_KEY_ID=ASIAIOSFODNN7EXAMPLE
SET AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
SET AWS_SESSION_TOKEN=AQoDYXdzEJr...<remainder of token> 
aws ec2 describe-instances --region us-west-1
```

**PowerShell**

```PowerShell
$Env:AWS_ACCESS_KEY_ID = "ASIAIOSFODNN7EXAMPLE"
$Env:AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
$Env:AWS_SESSION_TOKEN = "AQoDYXdzEJr...<remainder of token>"
```

**Example for copy-pasting**

```PowerShell
$Env:AWS_ACCESS_KEY_ID = ""
$Env:AWS_SECRET_ACCESS_KEY = ""
$Env:AWS_SESSION_TOKEN = ""
```