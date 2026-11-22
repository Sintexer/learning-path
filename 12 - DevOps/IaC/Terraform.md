**Terraform** is an **Infrastructure as Code ([[IaaC|IaC]])** tool used primarily to create and manage infrastructure through provider APIs.

It can manage resources in:

- AWS, Azure, and Google Cloud
- [[Kubernetes]]
- VMware
- GitHub
- Datadog
- [[PostgreSQL]] and many other systems

## Key characteristics

- Uses the declarative **HashiCorp Configuration Language**, or HCL
- Describes the desired infrastructure state
- Builds a dependency graph to determine resource order
- Maintains a **state file** mapping configuration to real infrastructure
- Uses providers to communicate with external APIs
- Shows planned changes before applying them

## Example

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "database_server" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"

  tags = {
    Name = "postgres-server"
  }
}
```

Typical commands:

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

- `init`: Downloads providers and initializes the project.
- `plan`: Previews proposed changes.
- `apply`: Creates or modifies infrastructure.
- `destroy`: Removes managed infrastructure.

## Terraform state

Terraform stores information about managed resources in a state file. For team environments, state is normally kept in a remote backend with access control and locking.

The state may contain sensitive information, so it must not be casually committed to Git.

## Infrastructure drift

Drift occurs when real infrastructure differs from the declared Terraform configuration—for example, when someone manually modifies a security group in the cloud console.

`terraform plan` compares the configuration, state, and current infrastructure to identify required changes.

## Related

- [[Ansible]]
- [[Ansible vs Terraform]]
- [[IaaC]]
