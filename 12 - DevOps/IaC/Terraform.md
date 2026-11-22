---
flashcards:
  q-h2ea: { nid: 1795356713618, hash: bn67au93, sync: 8msxnevg }
  q-8p33: { nid: 1795356713642, hash: 4f6x8nq8, sync: q7x83zsa }
  q-b278: { nid: 1795356713667, hash: phfvvfxi, sync: n8z6ahnp }
  q-5uwa: { nid: 1795356713694, hash: cv5cnihq, sync: 35trzt7v }
  q-h2sz: { nid: 1795356713733, hash: u9kcufhx, sync: 4mpckk5k }
---

**Terraform** is a declarative **Infrastructure as Code ([[IaaC|IaC]])** tool used primarily to provision and manage infrastructure lifecycle through provider APIs (with OpenTofu as its open-source fork).

It can manage resources in:

- AWS, Azure, and Google Cloud ([[IaaS]] / [[Virtual Private Cloud|VPCs]] / managed services)
- [[Kubernetes]]
- VMware, Cloudflare, GitHub, Datadog
- [[PostgreSQL]] and hundreds of other providers

## Key characteristics

- Uses declarative **HashiCorp Configuration Language (HCL)** to describe the desired end state
- Builds a **DAG (dependency graph)** to determine parallel creation and safe destruction order
- Maintains a **state file** (`terraform.tfstate`) mapping HCL resources to real-world cloud resource IDs
- Shows an exact execution diff (`terraform plan`) before mutating infrastructure

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

Typical workflow:

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

- `init`: Downloads providers/modules and initializes the backend.
- `plan`: Refreshes state and previews exact additions, in-place updates, and replacements (`-/+`).
- `apply`: Executes the planned DAG against provider APIs.
- `destroy`: Tears down all resources tracked in the state.

## Terraform Remote State & Locking

Terraform stores resource mappings and metadata in a state file. For teams, **never store state locally or commit it to Git** (it contains plaintext secrets and causes collisions). Instead, configure a **remote backend** (e.g., S3 with encryption + versioning and state locking via S3 native lockfile or DynamoDB, or Terraform Cloud):
- **State locking** prevents two engineers or CI pipelines from running `terraform apply` concurrently and corrupting state.
- **State segmentation**: split state by environment and blast radius (`network/`, `data/`, `app/`) so a bad app change cannot touch VPCs or databases, and `plan` stays fast.

## Infrastructure Drift

Drift occurs when live infrastructure diverges from the declared Terraform configuration—for example, when someone manually edits a security group in the cloud console ("ClickOps"). Running `terraform plan` compares HCL, state, and live APIs to surface drift; scheduled CI drift-detection runs catch it early.

> [!warning] Failure modes & trade-offs
> - **Accidental `destroy and recreate` (`-/+`)**: changing an immutable attribute (e.g., RDS engine/subnet group or VPC CIDR) forces replacement. Always inspect `plan` for `-/+` and add `lifecycle { prevent_destroy = true }` on stateful resources like databases and S3 buckets.
> - **Corrupted or leaked state**: state files store DB passwords and tokens in plaintext—encrypt the remote bucket with KMS, restrict IAM access ([[Least privilege principle]]), and enable bucket versioning for recovery.
> - **Monolithic state**: putting an entire company's infra in one state file makes `plan` take 15 minutes and turns any mistake into a company-wide blast radius.

## Flashcards

Terraform:::Declarative [[IaaC]] tool using HCL, a dependency DAG, provider APIs, and a tracked state file to provision and manage infrastructure lifecycle.
^q-h2ea

Why must team Terraform state live in a remote backend with state locking (never in Git)?::State contains ==plaintext secrets== and requires ==locking== so concurrent `terraform apply` runs don't race and corrupt infrastructure state.
^q-b278
^q-8p33

How do you protect critical stateful resources (like production databases) from accidental replacement in Terraform?::Add `lifecycle { prevent_destroy = true }` and isolate stateful resources into a separate state file with a smaller blast radius.
^q-5uwa

Always inspect `terraform plan` output for `-/+` (destroy-and-recreate) actions before applying changes to stateful infrastructure. #card-reminder
^q-h2sz

## Related
- [[Ansible]]
- [[Ansible vs Terraform]]
- [[IaaC]]
- [[IaaS]]
- [[Virtual Private Cloud]]
- [[Least privilege principle]]
