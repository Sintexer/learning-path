> AKA **Infrastructure as Code (IaC)**

**Infrastructure as Code (IaC)** is the practice of defining, provisioning, and managing infrastructure (networks, [[Virtual Private Cloud|VPCs]], [[Virtual machine|VMs]], [[Kubernetes]] clusters, databases, IAM policies) through version-controlled, machine-readable files instead of manual UI clicks or ad-hoc scripts.

## Declarative vs Imperative IaC

- **Declarative** (e.g., [[Terraform]], CloudFormation, Kubernetes manifests): you describe the *desired end state*, and the tool computes the diff/plan to reach it. Easier to reason about idempotency and drift.
- **Imperative** (e.g., shell scripts, procedural SDK calls): you specify the exact *sequence of commands* to execute. More flexible for custom workflows, but harder to keep idempotent across retries.

## Mutable vs Immutable Infrastructure

- **Mutable** (often [[Ansible]]): update existing servers in place. Risk of configuration drift over time.
- **Immutable** (often [[Terraform]] + Packer/containers): replace instances with newly built images on every change, making rollbacks predictable.

> [!warning] Failure modes & trade-offs
> - **Configuration drift ("ClickOps")**: manual console edits diverge from code; the next `apply` can unexpectedly overwrite or destroy hotfixes.
> - **Blast radius**: a single misconfigured IaC change (e.g., replacing a VPC or RDS resource) can wipe production infrastructure if plans aren't reviewed and state isn't segmented by environment/blast radius.
> - **Secrets in state**: IaC state files (like [[Terraform]] state) often store plaintext secrets and must be encrypted at rest with strict [[Least privilege principle|least-privilege]] access.

## Flashcards

IaC (Infrastructure as Code):::Managing and provisioning infrastructure through version-controlled code rather than manual configuration.

Declarative IaC specifies the ==desired end state== (tool computes the diff), whereas imperative IaC specifies the ==exact sequence of steps== to execute.

Split IaC state by blast radius (network, stateful data, stateless apps) so an app deployment can never accidentally destroy core networking or databases. #card-reminder

### What is configuration drift in IaC and how do you prevent it? #card

Drift happens when live infrastructure diverges from the IaC definitions due to manual console changes or out-of-band scripts. Prevent it with least-privilege read-only console access in production, immutable artifacts, and scheduled drift-detection plans in CI.

## Related
- [[Terraform]]
- [[Ansible]]
- [[Ansible vs Terraform]]
- [[IaaS]]
- [[Cloud-Native App]]
- [[Least privilege principle]]
