🔹 Terraform Basics
1. What is Terraform?
Terraform is an Infrastructure as Code (IaC) tool by HashiCorp that lets you define, provision, and manage infrastructure using code—across cloud providers and on-prem environments.
Think of it as: “write code → Terraform creates/updates infrastructure”.
🔹 Why Terraform is used
• Automates infrastructure (no manual clicking in cloud consoles)
• Cloud-agnostic (AWS, Azure, GCP, Kubernetes, GitHub, Datadog, etc.)
• Declarative → you define what you want, Terraform figures out how
• Version-controlled → infra changes tracked in Git
• Repeatable & consistent across environments (dev/stage/prod)
🔹 How Terraform works (high level)
1. You write infrastructure definitions in HCL (HashiCorp Configuration Language)
2. Terraform:
    ◦ Reads your code
    ◦ Compares it with the current infrastructure state
    ◦ Creates an execution plan
3. Applies only the required changes
2. How is Terraform different from CloudFormation?
FeatureTerraformAWS CloudFormationVendorHashiCorpAWSCloud supportMulti-cloudAWS onlyLanguageHCL (human-readable)JSON / YAMLState managementTerraform state fileAWS manages stateRollbackManual / scriptedAutomatic rollbackModularityStrong (modules)Limited (nested stacks)EcosystemVery largeAWS-centricLearning curveModerateEasier for AWS-only users
3. What is Infrastructure as Code (IaC)?
Infrastructure as Code (IaC) is the practice of defining, provisioning, and managing infrastructure using code instead of manual processes.
In simple terms:

👉 You write code, and tools automatically create servers, networks, clusters, and cloud resources for you.
🔹 Why Infrastructure as Code is used
Traditional infrastructure:
• Manual setup in cloud consoles
• Error-prone
• Hard to reproduce
• No version history
IaC solves this by making infrastructure:
• Automated
• Repeatable
• Version-controlled
• Auditable
• Scalable
🔹 How IaC works
1. Infrastructure is written in code files
2. Code is stored in Git
3. An IaC tool:
    ◦ Reads the desired state
    ◦ Compares it with the current state
    ◦ Applies only the required changes

Code → Plan → Apply → Infrastructure

🔹 Types of IaC
1️⃣ Declarative IaC (most common)
You describe what you want, not how to do it.
Examples:
• Terraform
• AWS CloudFormation
• Azure ARM / Bicep

resource "aws_instance" "web" {
  instance_type = "t2.micro"
}

✔ Safer

✔ Easier to maintain
2️⃣ Imperative IaC
You define step-by-step instructions.
Examples:
• Scripts (Bash, PowerShell)
• Older configuration tools
❌ Harder to maintain

❌ Less predictable
🔹 Popular IaC toolsToolPurposeTerraformMulti-cloud infrastructure provisioningCloudFormationAWS infrastructureARM / BicepAzure infrastructurePulumiIaC using real programming languagesAnsibleConfiguration management (sometimes IaC)
🔹 Benefits of IaC
• Consistency – same infra every time
• Speed – infra created in minutes
• Version control – infra changes tracked in Git
• Rollback – revert to previous versions
• Collaboration – teams review infra like code
• Disaster recovery – rebuild infra quickly
4. What are Terraform providers?
Terraform providers are plugins that allow Terraform to interact with external platforms, services, and APIs to create and manage infrastructure.
In short:

👉 Providers tell Terraform how to talk to a specific cloud or service.
🔹 What a Terraform provider does
A provider:
• Authenticates with the target platform
• Understands available resources and data sources
• Translates Terraform code into API calls
• Manages the lifecycle: create, read, update, delete (CRUD)
Example:
• AWS provider → talks to AWS APIs
• Kubernetes provider → talks to K8s API server
• GitHub provider → manages repos, teams, webhooks
5. What is a Terraform resource?
A Terraform resource is the basic building block in Terraform that represents a real infrastructure object you want to create, update, or delete.
In simple terms:

👉 A resource is the actual thing Terraform manages.
🔹 What a Terraform resource represents
A resource can be:
• A VM / EC2 instance
• A Kubernetes Deployment
• A Load Balancer
• A Database
• A DNS record
• An IAM role or policy
• A VPC, subnet, or security group
Each resource maps directly to an API object in the provider.
6. What is a Terraform state file?
A Terraform state file is a file where Terraform stores the mapping between your configuration (code) and the real infrastructure it manages.
In simple words:

👉 The state file is Terraform’s “source of truth” about what it has already created.
🔹 What is stored in the Terraform state?
The state file (terraform.tfstate) contains:
• Resource IDs (e.g., EC2 instance ID, subnet ID)
• Resource attributes (IPs, ARNs, names, metadata)
• Resource dependencies
• Provider information
• Outputs
• Metadata about the last apply
⚠️ It may contain sensitive data (passwords, secrets).
🔹 Why Terraform needs a state file
Terraform uses the state file to:
✔ Track what exists

✔ Detect changes (diff between desired vs actual state)

✔ Plan safe updates

✔ Avoid recreating existing resources

✔ Support dependency management
Without state → Terraform would not know what it already created.
7. Why is Terraform declarative?
Terraform is declarative because you describe what the desired end state of your infrastructure should look like, not how to create it step by step.
👉 You define the outcome, Terraform figures out the actions.
🔹 Declarative vs Imperative (quick contrast)
Imperative (how)

create vpc
create subnet
create instance
attach subnet

You must manage order, logic, and retries.
Declarative (what) — Terraform

resource "aws_instance" "web" {
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id
}

Terraform:
• Figures out the order
• Handles dependencies
• Plans the changes
• Applies only what’s needed
🔹 Why Terraform is declarative
1️⃣ You define the desired state
Terraform config describes:
• What resources should exist
• Their configuration
• Relationships between resources
Not the execution steps.
2️⃣ Terraform computes the plan
Terraform compares:

Desired state (code)
Current state (state file)
Actual infrastructure

Then generates an execution plan automatically.
3️⃣ Automatic dependency management
Terraform builds a dependency graph:

subnet_id = aws_subnet.main.id

Terraform knows:
• Subnet must exist before instance
• Delete in reverse order
You don’t code the logic.
4️⃣ Idempotent behavior
Running Terraform multiple times:
• Makes no changes if infra matches code
• Only updates what’s different
This is a key declarative property.
5️⃣ Drift detection
If someone manually changes infra:
• Terraform detects the drift
• Brings infra back to declared state
8. What is terraform init?
terraform init is the first command you run in any Terraform project.

It initializes the working directory so Terraform can start managing infrastructure.
👉 Without terraform init, nothing else works.
🔹 What terraform init does
1️⃣ Downloads providers
Terraform reads your configuration and downloads the required provider plugins.
Example:

provider "aws" {
  region = "us-east-1"
}

Terraform fetches the AWS provider automatically.
2️⃣ Initializes the backend (state storage)
If you’re using a remote backend (S3, Terraform Cloud, etc.), terraform init:
• Configures the backend
• Connects to remote state
• Enables state locking

backend "s3" {
  bucket = "tf-state"
}

3️⃣ Initializes modules
If your config uses modules:

module "vpc" {
  source = "./modules/vpc"
}

Terraform downloads and prepares them.
4️⃣ Creates .terraform/ directory
This directory contains:
• Provider plugins
• Module code
• Backend metadata
🔹 Terraform Configuration & Language
9. What is HCL?
HCL (HashiCorp Configuration Language) is the declarative language used by Terraform (and other HashiCorp tools) to define infrastructure configuration in a human-readable way.

👉 Think of HCL as “JSON, but made for humans and infrastructure.”

🔹 Why HCL exists

HashiCorp created HCL to:

Be easy for humans to read and write

Be machine-parsable

Support declarative configuration

Avoid the verbosity of JSON/YAML

Terraform files (.tf) are written in HCL.
10. What are Terraform functions?
Terraform functions are built-in helper functions you use inside HCL to transform values, apply logic, and manipulate data in your infrastructure code.
👉 They make Terraform configs dynamic, flexible, and DRY.
🔹 What Terraform functions are used for
• String manipulation
• Math & numeric calculations
• Working with lists, maps, and objects
• Conditional logic
• Encoding/decoding data
• File and template handling
11. What is a dynamic block?
A dynamic block in Terraform is used to generate repeated nested blocks programmatically instead of writing them manually.
👉 It’s mainly for repeating nested blocks, not resources.
🔹 Why dynamic blocks are needed
Some Terraform resources have nested blocks, like:
• ingress rules in security groups
• listener blocks in load balancers
• rule blocks in firewalls
Without dynamic blocks → lots of copy-paste 😬

With dynamic blocks → clean, DRY code ✅
🔹 Basic syntax of a dynamic block

dynamic "ingress" {
  for_each = var.ingress_rules

  content {
    from_port   = ingress.value.from_port
    to_port     = ingress.value.to_port
    protocol    = ingress.value.protocol
    cidr_blocks = ingress.value.cidr_blocks
  }
}

🔹 Example: AWS Security Group
Without dynamic block (repetitive)

ingress {
  from_port   = 80
  to_port     = 80
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

ingress {
  from_port   = 443
  to_port     = 443
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

With dynamic block (clean & scalable)

variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
}

resource "aws_security_group" "web" {

  dynamic "ingress" {
    for_each = var.ingress_rules

    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

12. What are Terraform data sources?
Terraform data sources are used to read and fetch information about existing infrastructure that Terraform does not create, so you can reference it in your configuration.

👉 Think of them as read-only lookups.

🔹 Why data sources are needed

You often need:

An existing VPC

An existing AMI

An existing IAM role

A Kubernetes cluster already created

Instead of hardcoding values → use data sources.

🔹 Basic syntax
data "aws_vpc" "main" {
default = true
}


Usage:

vpc_id = data.aws_vpc.main.id

🔹 Example: Fetch latest AMI
data "aws_ami" "amazon_linux" {
most_recent = true

filter {
name   = "name"
values = ["amzn2-ami-hvm-*"]
}

owners = ["amazon"]
}

ami = data.aws_ami.amazon_linux.id

13. What is interpolation?
Interpolation in Terraform is the process of injecting dynamic values (variables, resource attributes, expressions, or function results) into strings or arguments so your configuration can adapt to different environments and inputs.
🔹 Terraform State Management
14. What is Terraform state?
Terraform state is a mapping file that records the real-world infrastructure Terraform manages and links it to the resources defined in your Terraform configuration.
👉 Think of it as Terraform’s source of truth for what actually exists.
🔹 What Terraform state does
Terraform state:
• Tracks resource IDs (EC2 IDs, subnet IDs, etc.)
• Stores resource attributes (IPs, ARNs, metadata)
• Maps configuration → real infrastructure
• Enables diffing (plan)
• Supports dependency management
• Allows safe updates and deletions
🔹 Where state is stored
Local state (default)

terraform.tfstate

Stored in the working directory.
Remote state (recommended)
Stored in:
• S3 + DynamoDB (AWS)
• Terraform Cloud / Enterprise
• Azure Blob Storage
• GCS
Benefits:

✔ Team collaboration

✔ State locking

✔ Safer backups
🔹 What’s inside the state file?
• Resource mappings
• Metadata
• Provider versions
• Outputs
• Dependencies
⚠️ State may contain secrets (passwords, tokens).
🔹 Why Terraform needs state
Without state, Terraform:

❌ Can’t tell what exists

❌ Can’t calculate changes

❌ Would recreate everything
State allows Terraform to be idempotent.
15. Why should state not be stored locally?
Storing Terraform state locally is generally not recommended for production or team environments because it creates several risks and limitations.
Here’s why:
1️⃣ Lack of collaboration
• Local state (terraform.tfstate) is stored on your machine.
• If multiple people run Terraform, they cannot see each other’s changes.
• This can lead to resource conflicts or accidental overwrites.
2️⃣ Risk of corruption
• Terraform tracks dependencies in the state file.
• If two people run terraform apply at the same time locally:
    ◦ The state file may get corrupted
    ◦ Terraform may try to recreate or delete resources incorrectly
3️⃣ No remote locking
• Remote backends support state locking, preventing simultaneous modifications.
• Local state cannot be locked, so concurrent updates are dangerous.
4️⃣ No backup/versioning
• Local state is typically one file on a machine.
• If the machine fails, is deleted, or the file is accidentally modified → state is lost.
• Terraform cannot track history automatically.
5️⃣ Harder to manage environments
• For multiple environments (dev/stage/prod):
    ◦ Local state requires manual separation
    ◦ Remote backends make this automatic and scalable
16. How does Terraform handle concurrent runs?
Terraform handles concurrent runs primarily through state locking to prevent multiple users or processes from modifying the same infrastructure at the same time. Without locking, simultaneous terraform apply commands could corrupt the state or create conflicting resources.
🔹 How concurrent runs work
1. Terraform reads the state file (local or remote).
2. Before applying changes, Terraform locks the state if the backend supports it.
3. While the state is locked:
    ◦ Other Terraform runs are blocked or fail.
4. Once the apply finishes, Terraform releases the lock.
This ensures only one process modifies the infrastructure at a time.
17. What is terraform refresh?
terraform refresh is a Terraform command that updates the Terraform state file to match the actual real-world infrastructure without making any changes to resources.
In other words:

👉 It synchronizes Terraform’s “known state” with what actually exists.
🔹 What terraform refresh does
1. Queries the provider APIs for all resources in your state.
2. Compares the current state in Terraform with the actual infrastructure.
3. Updates the state file to reflect any drift.
4. Does not modify any real resources.
🔹 Why use terraform refresh
• Detect drift caused by manual changes outside Terraform
• Ensure Terraform state is up-to-date before plan or apply
• Debug issues with resource attributes
• Validate data sources and outputs
18. How do you recover a corrupted state file?
Recovering a corrupted Terraform state file is critical because the state file is Terraform’s source of truth. If it’s corrupted or lost, Terraform may lose track of resources, potentially leading to resource recreation or accidental deletion.
Here’s how to handle it safely.
1️⃣ Identify corruption
Signs of a corrupted state file:
• JSON parse errors when running terraform plan or terraform apply
• Missing resources or outputs
• Terraform cannot find known resources
2️⃣ Use the backup state file
Terraform automatically creates a backup file for each state:

terraform.tfstate.backup

Steps:
1. Make a copy of the current corrupted file:

cp terraform.tfstate terraform.tfstate.corrupted

1. Restore from backup:

cp terraform.tfstate.backup terraform.tfstate

1. Run a plan to verify:

terraform plan
✅ Most common and easiest recovery method.
3️⃣ Use remote state versioning
If using remote backends (S3, Terraform Cloud, GCS):
• Most remote backends keep previous versions automatically.
• For S3:

aws s3 list-object-versions --bucket tf-state-bucket --prefix prod/terraform.tfstate

• Restore a previous version and copy it back as the current state.
• Lock the state while restoring to prevent conflicts.
4️⃣ Manual state reconstruction
If backup is not available:
1. Use terraform import to re-import resources into a new state file:

terraform import aws_instance.web i-1234567890abcdef0

1. Repeat for all existing resources.
2. Run terraform plan to ensure state matches configuration.⚠️ Tedious, but sometimes the only option if both local and remote states are lost.
🔹 Terraform Modules
19. What is a Terraform module?
A Terraform module is a reusable, self-contained package of Terraform configuration files that defines a set of resources and can be called from other configurations.
Think of it as a function in programming—you define it once and reuse it anywhere.
🔹 Why use modules
• Reusability → Avoid repeating the same resource definitions
• Maintainability → Make changes in one place
• Consistency → Standardize infrastructure across environments
• Encapsulation → Hide complexity of multiple resources
20. Difference between root module and child module?
Difference between root module and child module?
In Terraform, root module and child module are both modules, but they differ in how they are used and structured.
🔹 Root Module
• The root module is the module defined in the directory where you run terraform init.
• Every Terraform configuration has exactly one root module.
• It can call other modules (child modules).
• Typically contains:
    ◦ Main orchestration logic
    ◦ Module calls
    ◦ Provider configurations
Example:

project/
├── main.tf        ← root module
├── variables.tf
├── outputs.tf
└── modules/       ← child modules stored here


provider "aws" {
  region = "us-east-1"
}

module "vpc" {
  source = "./modules/vpc"
  vpc_cidr = "10.0.0.0/16"
}

• project/ is the root module
• Calls modules/vpc as a child module
🔹 Child Module
• A child module is any module called by another module.
• It cannot exist independently; it is invoked via module "name" from a parent module.
• Child modules are reusable and usually encapsulate specific infrastructure components.
Example:

modules/vpc/
├── main.tf
├── variables.tf
└── outputs.tf

• This folder is a child module.
• It defines resources like aws_vpc and aws_subnet, but doesn’t run standalone.
• Outputs from child modules are accessed in root module:

vpc_id = module.vpc.vpc_id

🔹 Key DifferencesFeatureRoot ModuleChild ModuleLocationDirectory where Terraform commands runCalled by root or another modulePurposeOrchestrates the infrastructureEncapsulates reusable componentsStandalone✅ Can be applied directly❌ Cannot be applied directlyProviderDefined here usuallyInherits provider from parent moduleExamplesmain.tf in project foldermodules/vpc, modules/iam
21. How do you version modules?
Versioning Terraform modules is important for stability, reproducibility, and safe upgrades. It ensures that your infrastructure code uses a specific known version of a module, avoiding unexpected changes.
Here’s a detailed breakdown:
1️⃣ Why version modules
• Ensures consistent infrastructure across environments
• Prevents breaking changes when a module is updated
• Makes it easier to track updates and roll back if needed
2️⃣ Ways to version modules
Terraform supports versioning with the module source, especially for remote modules like the Terraform Registry or Git.
A. Terraform Registry modules
Example:

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}

• source → registry identifier
• version → specific module version
• Terraform will only use 3.14.0, not newer versions
Version constraints can be used:

version = "~> 3.14"   # Compatible with 3.14.x
version = ">= 3.10, < 4.0"  # Between 3.10 and <4.0

B. Git modules
Example:

module "vpc" {
  source = "git::https://github.com/org/terraform-modules.git//vpc?ref=v1.2.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}

• ref → can be a tag, branch, or commit SHA
• Using a tag ensures stable versioning
• Using a commit SHA pins to a specific commit
C. Local modules
• Local modules are usually not versioned because they’re in your repo.
• Versioning is handled by git tags/branches if your repo evolves.
Example:

module "vpc" {
  source = "./modules/vpc"
}

• Can combine with git workflow if committed to version control
3️⃣ Best practices for module versioning
1. Always use pinned versions for registry or Git modules (version or ref)
2. Use semantic versioning for your own modules: MAJOR.MINOR.PATCH
3. Use ~> constraints for safe upgrades (patch/minor updates)
4. Avoid using latest — it can break infra unexpectedly
5. Test new module versions in dev/staging before production
4️⃣ Terraform commands for module version management
• terraform init -upgrade → upgrade modules to latest allowed by version constraints
• terraform get -update → refresh module sources
22. What is module reusability?
Module reusability in Terraform refers to the ability to use the same module multiple times in different parts of your infrastructure, across projects, or environments without rewriting the same resource definitions.
It’s one of the main advantages of using Terraform modules.
🔹 What makes modules reusable
1. Input variables – allow you to customize resources per environment or use case.
2. Outputs – expose relevant information back to the parent module.
3. Encapsulation – modules contain a specific set of resources with a clear interface.
4. Parameterization – you can pass different values for the same module, producing different infrastructure.
23. How do you structure Terraform code for large projects?
terraform/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   ├── rds/
│   └── security_group/
├── envs/
│   ├── dev/
│   │   ├── [main.tf](http://main.tf/)
│   │   ├── terraform.tfvars
│   └── prod/
│       ├── [main.tf](http://main.tf/)
│       └── terraform.tfvars
├── scripts/
└── [README.md](http://readme.md/)
🔹 Terraform with Cloud
24. How do you manage multiple regions?
In Terraform, managing multiple AWS regions is done by defining multiple provider blocks, each configured with a different region. You can either:
1. Use multiple provider aliases to deploy resources across regions in the same Terraform configuration.
2. Use workspaces or separate Terraform configurations if you want strict isolation between regions.
Example of provider aliasing:

provider "aws" {
  region = "us-east-1"
  alias  = "us_east"
}

provider "aws" {
  region = "us-west-2"
  alias  = "us_west"
}

resource "aws_s3_bucket" "east_bucket" {
  provider = aws.us_east
  bucket   = "my-east-bucket"
}

resource "aws_s3_bucket" "west_bucket" {
  provider = aws.us_west
  bucket   = "my-west-bucket"
}

This lets Terraform know exactly which region each resource should be created in.
25. How do you manage multiple AWS accounts?

Managing multiple AWS accounts in Terraform is typically done using a multi-account strategy with workspaces, provider aliasing, or separate Terraform configurations. Each AWS account is represented by a separate provider block, which allows Terraform to authenticate and deploy resources in that account.
🔹 Terraform Workspaces & Environments
26. What are Terraform workspaces?
Terraform workspaces are isolated environments within a single Terraform configuration that allow you to manage multiple instances of the same infrastructure. Each workspace has its own state file, which means the same Terraform code can be applied multiple times without conflicts.
By default, Terraform creates a workspace called default. You can create additional workspaces for different environments like dev, staging, or prod. Switching workspaces is done with terraform workspace select <name>.
Why it is done
Workspaces are used to:
• Separate environments without duplicating code. For example, dev and prod can share the same Terraform configuration but maintain separate states.
• Avoid state conflicts when deploying multiple instances of the same infrastructure.
• Simplify CI/CD pipelines by dynamically switching workspaces for different deployment targets.
They are not a replacement for multi-account setups but work well for isolating environments within the same account.
27. How do you avoid hardcoding values?
Hardcoding values means directly writing configuration details—like region, instance type, or credentials—inside your Terraform files. This is bad practice because it reduces flexibility and makes the code hard to maintain.
Terraform provides variables, locals, and external data sources to avoid hardcoding:
1. Variables – dynamic input values passed via terraform.tfvars, environment variables, or CLI arguments.
2. Locals – calculated or reusable values derived from other inputs.
3. Data sources / modules – fetch values from AWS, other providers, or reusable modules instead of hardcoding them.
28. How do you review Terraform changes?

Reviewing Terraform changes is the process of inspecting what Terraform plans to do before applying it. Terraform provides the terraform plan command, which generates an execution plan showing all actions it will take: resources to create, update, or destroy.

29. How do you implement CI/CD for Terraform?
CI/CD for Terraform automates infrastructure provisioning and updates in a consistent, repeatable way. The idea is to integrate Terraform into a pipeline so that every change to code is automatically validated, tested, and applied safely.
A typical workflow includes:
1. CI (Continuous Integration) – triggered by code changes (e.g., GitHub pull requests). It runs:
    ◦ terraform fmt → checks code formatting
    ◦ terraform validate → checks syntax and basic correctness
    ◦ terraform plan → generates an execution plan for review
2. CD (Continuous Deployment) – after plan approval:
    ◦ Apply changes automatically to non-prod environments
    ◦ For production, require manual approval or automated approval gates
Terraform integrates with tools like GitHub Actions, GitLab CI, Jenkins, or Terraform Cloud/Enterprise to implement this pipeline.
Why it is done
Implementing CI/CD for Terraform ensures:
• Consistency and repeatability – all environments are provisioned the same way.
• Error prevention – mistakes are caught early via automated validation and plan review.
• Collaboration – teams can safely propose changes in pull requests, ensuring peer review.
• Auditability – all changes are tracked in source control and logs, which is critical for compliance.
Example (GitHub Actions pipeline)

name: Terraform CI/CD

on:
  pull_request:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Format Check
        run: terraform fmt -check

      - name: Terraform Plan
        run: terraform plan -out=tfplan

Here, the plan is generated on pull request, allowing reviewers to inspect changes before applying. A manual approval step or a separate workflow applies the plan to production.
🔹 Terraform with CI/CD & Jenkins
30. How do you run Terraform from Jenkins?
Running Terraform from Jenkins allows you to automate infrastructure deployment as part of a CI/CD pipeline. Jenkins acts as the orchestrator: it checks out Terraform code, runs validation and planning steps, and optionally applies changes to AWS or other providers.
The typical approach is to create a pipeline job (Declarative or Scripted Pipeline) that runs Terraform commands (init, plan, apply) on a Jenkins agent that has Terraform installed. Remote backends (like S3 with DynamoDB for locking) are often used to share state across teams.
Why it is done
Using Jenkins with Terraform ensures:
• Automation and consistency – infrastructure is provisioned reliably without manual intervention.
• Safe change management – running terraform plan allows review before applying.
• Integration with CI/CD – Terraform runs can be triggered automatically on pull requests, merges, or schedule.
It also allows environment separation (dev/staging/prod) by switching workspaces or using different backend configurations.
Example (Declarative Pipeline)

pipeline {
    agent any
    environment {
        AWS_PROFILE = 'dev-account'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init -backend-config=backend.tfvars'
            }
        }
        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }
        stage('Terraform Apply') {
            steps {
                input message: "Approve Apply?"
                sh 'terraform apply tfplan'
            }
        }
    }
}

How it works:
1. Jenkins pulls code from Git.
2. Initializes Terraform with a remote backend (backend.tfvars).
3. Validates syntax and generates a plan.
4. Waits for manual approval before applying to the environment.
This approach ensures safe, automated deployments and integrates well with GitOps practices.
31. How do you prevent accidental terraform apply?
Preventing accidental terraform apply is about ensuring that infrastructure changes are intentional and controlled. Terraform provides multiple mechanisms to enforce safety before any changes are applied. These mechanisms work at both the command-line level and the CI/CD pipeline level.
Why it is done
Accidental terraform apply can:
• Destroy production resources unexpectedly.
• Cause downtime or data loss.
• Lead to unexpected costs if large infrastructure changes are applied unintentionally.
In interviews, demonstrating awareness of safety mechanisms shows production-readiness and risk awareness.
Common Methods to Prevent Accidental Apply
1. Use terraform plan and -out
    ◦ Generate a plan first:

terraform plan -out=tfplan

    ◦ Inspect the plan before applying.
    ◦ Apply using the saved plan only:

terraform apply tfplan

This ensures that no accidental changes happen without review.
2. Manual Approval in Pipelines
    ◦ In CI/CD pipelines (Jenkins, GitHub Actions, GitLab CI): require a human approval step before apply.
    ◦ Example in Jenkins:

input message: "Approve Apply to Production?"
sh 'terraform apply tfplan'

3. Enable -lock and Remote State
    ◦ Using remote backends (S3 + DynamoDB) prevents concurrent apply commands that could accidentally overwrite state.
4. Protect Production Workspaces / Accounts
    ◦ Use separate AWS accounts or Terraform workspaces for prod.
    ◦ Limit permissions via IAM roles so only approved users can apply changes.
5. Use terraform apply -auto-approve=false
    ◦ Never skip the approval prompt in interactive mode.
    ◦ Forces confirmation before changes are applied.
32. How do you add approval gates?
An approval gate is a controlled checkpoint in a CI/CD pipeline that requires human review before applying Terraform changes. It ensures that changes are intentional, reviewed, and safe, especially for sensitive environments like production.
In practice, the approval gate sits between the Terraform plan stage and the apply stage, preventing automatic execution until an authorized person approves the plan.
Why it is done
Approval gates are used to:
• Prevent accidental destruction or modification of critical resources.
• Ensure compliance and policy checks before production deployment.
• Support team collaboration by allowing peer review of planned changes.
It’s a key mechanism in governance and risk management, especially in enterprise environments.
How to implement approval gates
1. Jenkins Pipeline (Declarative)

stage('Terraform Apply') {
    steps {
        input message: "Approve Apply to Production?"
        sh 'terraform apply tfplan'
    }
}

• The pipeline pauses, waiting for a user to click “Proceed.”
• Only after approval will Terraform execute apply.
1. GitHub Actions / GitLab CI
• Use manual workflow approvals:
    ◦ GitHub Actions: workflow_dispatch with environment protection rules
    ◦ GitLab: when: manual in the job definition
• Example (GitHub Actions with environment protection):

jobs:
  apply:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Terraform Apply
        run: terraform apply tfplan

• Only users with permission to the production environment can approve the workflow.
1. Terraform Cloud / Enterprise
• Provides policy checks and approval gates out-of-the-box.
• Each workspace can require manual approval before apply, enforcing governance without custom scripting.
Example Flow

Terraform Plan → Manual Approval Gate → Terraform Apply

• The plan is inspected by a reviewer.
• If approved, the apply executes; otherwise, it stops.
33. How do you rollback Terraform changes?
Terraform does not have a native rollback command like application deployment tools. Instead, rollback means restoring your infrastructure to a previous known good state. This is typically achieved by leveraging:
1. Remote or versioned state files – Terraform stores the current infrastructure state in terraform.tfstate. If you maintain versioned state (S3, Terraform Cloud), you can roll back by reverting to a previous state.
2. Reverting code changes – Restore your .tf files to the previous commit or release and then run terraform apply to bring infrastructure back to its prior configuration.
Essentially, rollback is about aligning Terraform’s configuration and state with the desired previous state.
Why it is done
Rolling back Terraform changes is critical because:
• Infrastructure updates can fail or break production services.
• Certain resource modifications are destructive (e.g., deleting a database or S3 bucket).
• Maintaining uptime and reliability requires a fast and predictable way to revert changes.
Common Rollback Approaches
1. Using Versioned Remote State
• If your state is in S3 with versioning:
    ◦ Restore the previous state version
    ◦ Run terraform apply to revert resources
• Example:

aws s3 cp s3://tf-state-bucket/terraform.tfstate ./terraform.tfstate --version-id <old-version-id>
terraform apply

1. Reverting Code in Git
• Checkout the previous commit with the working configuration:

git checkout <previous-commit>
terraform plan -out=tfplan
terraform apply tfplan

• Ensures infrastructure matches the last stable configuration.
1. Using Terraform Cloud/Enterprise Rollback Features
• Terraform Cloud allows you to select previous workspace states and queue a new apply to restore that state.
Example Scenario
Suppose a production Terraform change accidentally deletes an S3 bucket. Rollback could be:
1. Restore the last versioned Terraform state where the bucket existed.
2. Run terraform apply to recreate the bucket and revert other changes.
🔹 Terraform Troubleshooting
34. Terraform apply failed — what do you do?
When terraform apply fails, Terraform stops the deployment and prints an error message, often with a resource name and cause. The infrastructure may be partially applied, meaning some resources were created or modified before the failure.
Handling a failed apply requires:
1. Identifying the root cause
2. Recovering the state
3. Retrying safely
Terraform tracks applied resources in the state file, so even partial failures are recoverable without breaking existing infrastructure.
Why it is done
Failing to properly handle an apply failure can lead to:
• Orphaned resources that incur cost
• Inconsistent state between Terraform and the real infrastructure
• Service downtime if critical resources were partially updated
Proper handling ensures infrastructure integrity, safety, and reliability, which is critical in production environments.
Steps to Handle a Terraform Apply Failure
1. Check the error message
    ◦ Terraform usually prints which resource caused the failure.
    ◦ Example:

Error: Error creating S3 bucket: BucketAlreadyExists

    ◦ This immediately identifies the conflict.
2. Inspect the current state
    ◦ Run:

terraform show
terraform state list

    ◦ Identify which resources were successfully applied and which were not.
3. Decide recovery action
    ◦ Fix the issue in the code (e.g., rename conflicting resource, adjust permissions).
    ◦ Manually fix infrastructure if necessary (rare, only if state is out-of-sync).
    ◦ Optionally use terraform state rm to remove resources in a bad state.
4. Re-run Terraform safely
    ◦ Generate a plan first:

terraform plan -out=tfplan
terraform apply tfplan

    ◦ Terraform will only apply the remaining changes and skip resources already applied.
5. Consider rollback if needed
    ◦ If the failure leaves infrastructure unusable, restore a previous versioned state or revert code and apply.
Example Scenario
Suppose terraform apply fails because an EC2 instance cannot be created due to a missing subnet:
1. Terraform prints the error and stops.
2. You run terraform state list to check which resources were already created.
3. Fix the subnet configuration in .tf files.
4. Run terraform plan to confirm the changes.
5. Apply again: Terraform only creates the EC2 instance; previously created resources are untouched.
35. Provider version conflict — how do you fix it?
A provider version conflict happens when Terraform detects that the required provider version specified in your configuration does not match the version installed or resolved. Terraform providers are plugins that interact with cloud platforms like AWS, Azure, or GCP.
Conflicts can occur due to:
• Multiple modules requiring different provider versions
• Terraform trying to upgrade a provider automatically when terraform init is run
• Local Terraform cache or lock files having a different version
Terraform enforces version constraints through the required_providers block in the configuration.
Why it is done / Why it matters
Fixing provider version conflicts is critical because:
• Different provider versions can break modules due to API changes.
• Using the wrong version can cause apply failures, unexpected behavior, or resource drift.
• Ensures stability and reproducibility of your infrastructure across environments and team members.
How to Fix a Provider Version Conflict
1. Check the conflict message
    ◦ Terraform error messages specify which provider and version constraints are conflicting:

Error: Failed to query available provider packages
Provider registry.terraform.io/hashicorp/aws: version constraints = ">= 4.0, < 5.0"

2. Update required_providers
    ◦ Explicitly specify the compatible version in your Terraform configuration:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.50"
    }
  }
}

3. Run terraform init -upgrade
    ◦ Re-initializes the workspace and upgrades providers to satisfy constraints:

terraform init -upgrade

4. Check modules for conflicting provider versions
    ◦ If using modules, ensure each module is compatible with the same provider version.
    ◦ Sometimes you may need to override provider versions or update modules to a compatible release.
5. Use .terraform.lock.hcl
    ◦ The lock file ensures consistent provider versions across runs.
    ◦ Update it carefully if upgrading providers:

rm .terraform.lock.hcl
terraform init

36. Terraform created resources twice — why?
Terraform creating resources twice usually happens due to state misalignment. Terraform relies on its state file (terraform.tfstate) to track what resources exist. If the state file does not match the actual infrastructure, Terraform may attempt to create resources that already exist.
Common scenarios include:
1. State not shared or lost – working locally without a remote backend can cause Terraform to “forget” existing resources.
2. Import not performed – if existing resources were created outside Terraform and not imported using terraform import.
3. Changes in resource identifiers – changing name, id, or provider attributes can make Terraform think a new resource is needed.
4. Module duplication or provider misconfiguration – using multiple modules or providers incorrectly may lead to duplicate creation.
Why it is done / Why it happens
Terraform creates resources twice because it thinks the resource doesn’t exist in its state file. This is usually a state management or configuration issue, not a Terraform bug.
• Terraform relies entirely on its state file.
• Without accurate state, Terraform cannot detect that a resource already exists.
• This often happens in teams without remote backends, or when state files are manually modified or deleted.
How to Fix / Prevent It
1. Use a remote state backend
    ◦ Store state in S3 + DynamoDB, Terraform Cloud, or similar.
    ◦ Ensures all team members and CI/CD pipelines see the same state.
2. Import existing resources
    ◦ Use terraform import to bring manually created or out-of-band resources into state:

terraform import aws_s3_bucket.my_bucket my-bucket-name

3. Check resource identifiers
    ◦ Avoid changing attributes like name or id that Terraform uses to detect existing resources.
4. Enable locking
    ◦ Use dynamodb or terraform cloud locking to avoid concurrent applies causing duplication.
5. Use workspaces correctly
    ◦ Ensure different environments (dev/staging/prod) don’t accidentally share the same state file.
Example Scenario
• You created an S3 bucket manually in AWS: my-bucket.
• Terraform configuration has:

resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
}

• Running terraform apply without importing the bucket: Terraform doesn’t know it exists → tries to create it → fails or duplicates resources.
Fix:

terraform import aws_s3_bucket.example my-bucket
terraform plan   # confirms no changes needed
terraform apply

37. How do you debug Terraform errors?

1. Read the error message carefully
    ◦ Terraform usually provides the resource name and reason for the failure.
    ◦ Example:

Error: creating S3 bucket: BucketAlreadyExists

    ◦ Indicates a resource conflict rather than a syntax error.
2. Enable detailed logs
    ◦ Use Terraform’s debug mode to get detailed output:

TF_LOG=DEBUG terraform apply
TF_LOG_PATH=./tf_debug.log

    ◦ Logs contain API calls, provider responses, and internal Terraform actions.
3. Validate configuration and formatting
    ◦ Run:

terraform validate
terraform fmt -check

    ◦ Catches syntax or misconfigured arguments before applying.
4. Inspect state and plan
    ◦ Check which resources Terraform thinks exist:

terraform state list
terraform show
terraform plan

    ◦ Identify discrepancies between state and real infrastructure.
5. Check provider version and API limits
    ◦ Ensure provider plugins are compatible: terraform providers
    ◦ Confirm cloud account permissions and resource limits.
6. Break down complex code
    ◦ Comment out modules or resources to isolate the failing part.
    ◦ Apply in smaller increments to identify which resource triggers the error.
7. Use community tools and documentation
    ◦ Terraform docs, GitHub issues, and community forums often provide solutions for common errors.
Example Scenario

terraform apply
# Error: creating aws_instance.web: InvalidSubnetID.NotFound

Debugging steps:
1. Verify the subnet exists in AWS console.
2. Check if the subnet_id variable is correct.
3. Run terraform plan to confirm Terraform recognizes the subnet.
4. Apply again after correcting the variable.
🔹 Terraform Scenario-Based
38. How do you design Terraform for multi-account AWS?
Designing Terraform for multiple AWS accounts involves creating a scalable, modular, and secure infrastructure setup where each AWS account (e.g., dev, staging, prod) is isolated, but the code is reusable and manageable.
Key elements include:
1. Provider Aliasing – Define multiple AWS providers with aliases for different accounts:

provider "aws" {
  alias   = "dev"
  region  = "us-east-1"
  profile = "dev-account"
}

provider "aws" {
  alias   = "prod"
  region  = "us-east-1"
  profile = "prod-account"
}

1. Modules – Use reusable modules for common resources (VPCs, S3, EC2) so you don’t duplicate code across accounts.
2. State Management – Use remote backends with locking (S3 + DynamoDB, or Terraform Cloud) per account or environment to prevent conflicts.
3. Workspaces / Directory Structure – Organize environments and accounts using directories or Terraform workspaces:

terraform/
 ├─ modules/
 │   └─ vpc/
 ├─ envs/
 │   ├─ dev/
 │   ├─ staging/
 │   └─ prod/

1. IAM Roles / Cross-Account Access – Use assume-role patterns for cross-account deployments to avoid hardcoding credentials.
39. How do you manage Terraform at scale?
Managing Terraform at scale means designing your infrastructure-as-code workflows so that large, complex, multi-team, multi-account deployments remain reliable, maintainable, and auditable.
Key strategies include:
1. Modularization – Break your Terraform code into reusable modules for resources like VPCs, EC2, S3, IAM, etc. This avoids duplication and improves maintainability.
2. Remote State Management – Use remote backends (S3 + DynamoDB for AWS, Terraform Cloud, or Terraform Enterprise) to store state centrally. This allows:
    ◦ Locking to prevent concurrent updates
    ◦ State versioning to enable rollbacks
    ◦ Shared visibility across teams
3. Environment Separation – Use workspaces, directories, or separate state files for dev, staging, and prod to isolate environments.
4. Version Control & CI/CD – Store Terraform code in Git and integrate with CI/CD pipelines (Jenkins, GitHub Actions, GitLab CI, Terraform Cloud) to automate:
    ◦ Formatting (terraform fmt)
    ◦ Validation (terraform validate)
    ◦ Planning (terraform plan)
    ◦ Apply with approval gates
5. Policy Enforcement / Governance – Use Sentinel (Terraform Enterprise), pre-commit hooks, or static code analysis to enforce rules (tags, resource types, naming conventions, cost controls).
Why it is done
Managing Terraform at scale ensures:
• Reliability – fewer errors, reproducible environments
• Collaboration – multiple teams can work without overwriting each other’s changes
• Security & Compliance – policies and access control are enforced
• Auditability – all infrastructure changes are tracked in version control and state history
• Scalability – easier to add new accounts, regions, or resources without rewriting code
Example Scaled Setup
Directory structure for multi-team, multi-account setup:

terraform/
 ├─ modules/
 │   ├─ network/
 │   ├─ compute/
 │   └─ storage/
 ├─ envs/
 │   ├─ dev/
 │   ├─ staging/
 │   └─ prod/
 ├─ scripts/
 └─ ci-cd/

Remote backend for prod environment:

terraform {
  backend "s3" {
    bucket         = "tf-state-prod"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-lock-prod"
  }
}

Pipeline stages (CI/CD):

Checkout → Terraform fmt → Terraform validate → Terraform init
→ Terraform plan → Manual approval → Terraform apply → Post-deployment tests

This setup:
• Supports team collaboration
• Reduces manual errors
• Maintains state integrity and auditability
40. How do you handle drift detection?
How do you handle drift detection?What is lifecycle block?What is create_before_destroy?Terraform vs Pulumi?How do you destroy only specific resources?What happens if someone changes infra manually?
