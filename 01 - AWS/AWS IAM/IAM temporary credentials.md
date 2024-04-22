You can use **temporary security credentials** to make programmatic requests for AWS resources using the [[AWS CLI]] or AWS API

When you make a call using temporary security credentials, the call must include a [[IAM session token|session token]], which is returned along with those temporary credentials. AWS uses the **session token** to validate the temporary security credentials.