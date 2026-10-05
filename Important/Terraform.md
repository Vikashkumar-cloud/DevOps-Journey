# SECTION A: BASICS - 10 Qs

## 1. What is Terraform and why use it over CloudFormation?

Terraform is an IaC tool which is used to create, manage and delete infrastructure on the cloud like AWS, Azure, GCP.
Why use it over CloudFormation?
Because CloudFormation works only on AWS, but Terraform works on all clouds with same language. So we can manage everything from one tool.

## 2. What is Provider, Resource, Data Source?

Provider is a plugin which is used to communicate with cloud like AWS, Azure.
Resource is what we create by Terraform like EC2, S3, VPC.
Data Source is already created resource which we use as reference. Like if VPC is already there, we fetch it by data source to use in our code.

## 3. Explain main.tf, variables.tf, outputs.tf, terraform.tfvars

main.tf -> we have resources code
variables.tf -> is used to declare the variable
terraform.tfvars -> is used to store the variable value
outputs.tf -> is used to display output like IP address after resource created

## 4. What is terraform state file? What if you delete it?

Terraform state file is the heart of Terraform. It stores the information of resources which are created and managed by Terraform. 
So next time when we run terraform apply, it will first check the state file.If we delete it, we will first check if any backup is available. 
If backup is not available and state is local, we need to import the resources again with terraform import. Otherwise Terraform will try to 
create resources again and give "already exists" error. That's why we store state file in S3 remote backend.

## 5. What is terraform init, plan, apply, destroy?

terraform init is used to initialize the directory and install the provider plugins to communicate with cloud like AWS.
terraform plan is like a blueprint, it will show what is going to create, modify or delete.
terraform apply will create, modify or delete the resources which we saw in plan.
terraform destroy will delete all the resources managed by Terraform.

## 6. What is State Locking and why it is needed? How S3 + DynamoDB does it?

State locking is used to lock the state file when multiple users work on same folder, so we don't get conflict error during apply.
Previously we used S3 for backend and DynamoDB for locking. Now Terraform has inbuilt feature, we use S3 as remote backend with 
use_lockfile = true to lock the state file.

## 7. Difference between count and for_each?

Both are used to create multiple resources at a time.

count is used when we want to create resources with index number like app-0, app-1.
for_each is used when we want to create resources with key-value names like app, db.

## 8. What are Terraform Modules? How you create reusable module?

Terraform module is a reusable code so we don't need to write the same infra again and again. For example, if we want to create same 
VPC in dev and prod, we can call the module.
We have 2 types of modules: root and child. Child module is where the actual source code is present, and root module is the caller who 
calls the child module.

## 9. What is variable types - string, list, map, object?

Variable is a reusable value. If we have to change same value at multiple places, we create a variable and just change the variable value.

string - for single text value like "t2.micro"
list - for multiple values like ["a", "b", "c"]
map - for key-value pair like { dev = "t2.micro", prod = "t3.large" }
object - for combination of different types in one variable.

## 10. How to handle secrets in Terraform?

We never hardcode secrets in Terraform code.

1. Environment Variables: We set secret as TF_VAR_ environment variable, like export TF_VAR_db_password="secret". 
Terraform automatically reads it. It's good for local development.

2. AWS SSM Parameter Store / Secrets Manager: This is most common in AWS projects. We store the secret in SSM and fetch it in Terraform 
using a data source.

3. HashiCorp Vault: For enterprise or multi-cloud setup, we use Vault. We fetch secrets using vault_generic_secret data source.

Also, we mark variable as sensitive = true so it doesn't show in logs or console output.

# SECTION B: INTERMEDIATE - 10 Qs

## 11. What is Remote State? How to store state in S3 backend?

By default Terraform stores state file locally as terraform.tfstate.
This is risky in a team because others cannot access it and if it gets deleted we lose infra tracking.To solve this we use Remote State,
where we store the state file remotely in a backend like S3.
 

## 12. What is Workspace? When to use it?

Terraform Workspace allows us to use the same code for multiple environments with separate state files.
By default we are in default workspace. We can create new workspaces like: terraform workspace new dev
When we create workspaces, Terraform creates separate state files for each:

When to use it?
We use it when we have same infrastructure for dev, staging, prod but want isolated state.

## 13. What is terraform import? Have you used it?

Terraform import is used when a resource is manually created in console and we want to bring it under Terraform management without recreating it.

Steps:
1. Create empty resource block:
resource "aws_instance" "my_ec2" {
}
2. Run import command:
terraform import aws_instance.my_ec2 i-0abcd1234efgh5678
This will import the current state into terraform.tfstate.
3. Write configuration:
Then we run terraform plan to see the difference and we add required arguments like ami, instance_type, subnet_id, tags to match the real infrastructure.


## 14. What is taint / terraform state replace?

This is used when a resource is degraded or corrupted and we want to force recreate it.
In older versions we used:
terraform taint aws_instance.web
It would mark the resource as tainted in the statefile, so on next terraform apply it would be destroyed and recreated.
But taint is deprecated now. From Terraform v0.15.2+, we use: terraform apply -replace="aws_instance.web"

## 15. What is lifecycle block - create_before_destroy, prevent_destroy?

Lifecycle block controls how Terraform creates, updates and destroys a resource.

1. create_before_destroy = true
Normally Terraform destroys old first then creates new. With this, it creates new first then destroys old.
Use case: Zero downtime. Like when updating Launch Template or EC2.

2. prevent_destroy = true

It prevents accidental deletion. If someone tries to destroy, Terraform will give error.
Use case: For critical resources like S3 bucket with statefile, RDS, Production DB.

## 16. What is provisioner? Why we should avoid it?

Provisioners are used to execute scripts or actions on local or remote machine after resource is created.
Types:

1. file provisioner - to copy file from local to remote EC2
2. local-exec - runs command on local machine where Terraform is running
3. remote-exec - runs command on remote EC2 via SSH, like installing nginx

## 17. Terraform vs Ansible - when to use what?

Terraform is for Infrastructure Provisioning and Ansible is for Configuration Management.

We use Terraform first to provision the EC2 server, and then we use Ansible to configure that server, like installing packages and deploying app.

In modern setup, we combine both: Terraform + Ansible, or Terraform + Packer + User Data.


## 18. How to do version locking of provider and terraform?

We do version locking in terraform block to avoid breaking changes in production.

1. Terraform Version Locking:

terraform {
  required_version = ">= 1.5.0, < 2.0.0"
}

2. Provider Version Locking:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # means >=5.0 and <6.0
    }
  }
}


## 19. How you manage 3 environments - dev, staging, prod - in Terraform?

I use a module-based, folder-per-environment approach. I don't use workspaces for this.
I have one reusable module, and three environment folders - dev, staging, prod. Each environment has its own backend and state file for isolation.


## 20. What is drift and how to handle it?

Drift is when our actual infrastructure on AWS is different from our desired infrastructure defined in Terraform code. 
It happens when someone makes manual changes from the AWS console.

For example, in Terraform code we have t2.micro, but someone manually changed it to t2.medium from console. When we run terraform plan, 
Terraform will detect this mismatch and show drift.

How to handle it:

First, I run terraform plan to identify which resource has drifted.
Then I check with the team if this manual change was intentional.
If it was intentional, I update my Terraform code to match the new state, i.e., change t2.micro to t2.medium in code.
If it was not intentional, I run terraform apply to revert the infrastructure back to the desired state defined in code.

"To prevent drift, we use driftctl in our pipeline to detect drift early, and we restrict manual changes by giving 
read-only IAM access and enforcing changes only via Terraform pipeline."

# SECTION C: ADVANCED + EKS - 5 Qs - MOST IMPORTANT FOR YOU

## 21. How to create EKS cluster with Terraform? Which modules you used?

I used the terraform-aws-modules/eks/aws module to create the EKS cluster.
I also used the terraform-aws-modules/vpc/aws module to create the VPC, and passed its private subnets to the EKS module.

## 22. What is locals in Terraform? Difference between locals and variables?

In Terraform, locals is used to define local values or expressions inside the code itself.
Its value cannot be changed from outside, like from tfvars or command line.

technically everything can be done with variables, but there are main technical reasons for locals:

1. Variables cannot reference other variables, It will throw an error. In locals you can do this. So locals is needed to combine variables.

 You cannot write
 
variable "bucket_name" {
  default = "prod-${var.env}-bucket" // YE ERROR DEGA
}
 
locals {
  bucket_name = "prod-${var.env}-bucket" // YE CHALEGA
}
 
## 23. Tell me about your EKS + Terraform folder structure?

I follow an environment-based structure. I have a modules/ folder which contains reusable code for VPC and EKS.
Then I have an envs/ folder with separate folders for dev and prod.
Each env folder has its own main.tf which calls the VPC and EKS modules, plus a .tfvars file for env-specific values.
This keeps our code DRY and environment isolated.

modules/vpc/
modules/eks/
envs/dev/main.tf
envs/prod/main.tf

## 24. How to do zero-downtime infra update with Terraform?

"To achieve zero-downtime infra updates with Terraform, we use the
 lifecycle { 
 create_before_destroy = true
 } block. 
This ensures Terraform creates the new resource before destroying the old one. 
For example, in case of ASG or Launch Template changes, we combine it with AWS instance_refresh and ALB, 
so new instances are up and healthy before the old ones are terminated, resulting in zero downtime at the infra level."

## 25. How you handle error when terraform apply fails in middle?

"When terraform apply fails in the middle, first I check the error logs. 
Then I run terraform state list to see what resources are already tracked in state and compare it with AWS console to find any drift.
If a resource is created in AWS but not in state due to a state save failure, I use terraform import to bring it into state.
Otherwise I fix the root cause in code and re-run terraform apply. Terraform is idempotent, so it will only create the pending resources."

---------------------------------------------------------------------------------------------------------------------------------------------

# Terraform Interview Questions & Answers

> **Level:** Working Knowledge
> **Cloud:** AWS
> **Focus:** Easy to Medium Interview Questions

---

# What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool which is used to provision and manage infrastructure on cloud platforms like AWS, Azure, and GCP.

The main advantage of Terraform is reusable code. Once we write the code, we can use the same code again to create the same infrastructure whenever required.

It saves time, reduces manual work, and minimizes human errors.

---

# Why do we use Terraform when we can create resources from the AWS Console?

Yes, we can create resources from the AWS Console, but if we create infrastructure manually, it takes more time and there are more chances of human errors.

With Terraform, we write the code only once. After that, we can reuse the same code to create the same infrastructure whenever required.

It saves time, reduces manual work, and minimizes human errors.

---

# If Terraform is so good, why do people still use the AWS Console?

We use Terraform for automation and reusable infrastructure.

We use the AWS Console for:

- Quick changes
- Learning
- Testing
- Troubleshooting

Both have their own use cases.

---

# Provider

## What is a Provider in Terraform?

A provider is a plugin that helps Terraform connect to cloud platforms like AWS, Azure, or GCP so that it can create and manage resources.

### Example

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

---

## Can Terraform work without a provider?

No. Terraform cannot work without a provider because the provider is required to communicate with the cloud platform and create or manage resources.

---

## How does Terraform know which provider to use?

We define the provider in our Terraform code.

For example, if I want to create resources in AWS, I use the AWS provider block.

When I run **terraform init**, Terraform automatically downloads the AWS provider and uses it to communicate with AWS.

### Command

```bash
terraform init
```

---

# Terraform Init

## Why do we run terraform init?

Terraform init is used to download the required provider.

After downloading the provider, Terraform uses it to communicate with the cloud platform like AWS.

### Command

```bash
terraform init
```

---

## Do we run terraform init every time?

No.

We run terraform init the first time to download the required provider.

If we create a new project or change the provider, we run it again.

---

# Terraform Workflow

## terraform init

Download the required provider to communicate with the cloud platform.

```bash
terraform init
```

---

## terraform plan

Terraform plan is like a blueprint.

It shows what resources Terraform is going to create, modify, or destroy before making any changes.

### Command

```bash
terraform plan
```

Save the execution plan.

```bash
terraform plan -out=tfplan
```

---

## terraform apply

Terraform apply creates the resources defined in the Terraform code.

### Command

```bash
terraform apply
```

Apply saved plan.

```bash
terraform apply tfplan
```

Skip confirmation.

```bash
terraform apply --auto-approve
```

---

## Can we run terraform apply directly without running terraform plan?

Yes, we can run terraform apply directly.

Terraform automatically checks the changes before applying them.

If we use **--auto-approve**, it skips the confirmation only.

But in production, I prefer to run terraform plan first because it shows exactly what is going to be created, modified, or destroyed.

---

# Terraform State File

## What is Terraform State File or terraform.tfstate?

Terraform state file is the heart of Terraform. It stores all resources created and managed by Terraform. When we run Terraform again, it uses the state file to check what changes are required.

### Useful Commands

```bash
terraform state list
```

Show a specific resource.

```bash
terraform state show aws_instance.web
```

Show complete state.

```bash
terraform show
```

---

## What will happen if you delete the terraform.tfstate file?

If the terraform.tfstate file is deleted, Terraform loses the information about the infrastructure it was managing. First, I will check if we have a backup or remote state and restore it. If there is no backup, I will recover the state by importing the existing resources into Terraform state. Until the state is recovered, Terraform may consider the resources as new and may try to create them, so I will not directly run terraform apply.

---

# Terraform Drift

## What is Drift in Terraform?

Terraform Drift means difference between desired code and current infrastructure.

For example,

In the Terraform code instance type is **t2.micro**, but someone changed it to **t3.micro** from the AWS Console.

When we run terraform plan, it shows the changes.

### Command

```bash
terraform plan
```

---

## How can you fix Terraform Drift?

We have two options.

First, we can update the Terraform code according to the new infrastructure.

Second, we can revert the manual changes from the AWS Console so the infrastructure matches the Terraform code again.

---

# Terraform Validate

## What is terraform validate?

Terraform validate is used to validate the Terraform code.

If there is any error in the code, it shows the error before creating the infrastructure.

### Command

```bash
terraform validate
```

---

# Terraform fmt

## What is terraform fmt?

Terraform fmt is used to format the Terraform code.

It automatically arranges the code in a proper and readable format.

### Command

```bash
terraform fmt
```

Format all Terraform files.

```bash
terraform fmt -recursive
```

---

# Terraform Destroy

## What is terraform destroy?

Terraform destroy is used to destroy all the resources created by Terraform.

It uses the Terraform state file to identify the resources and destroy them.

### Command

```bash
terraform destroy
```

Skip confirmation.

```bash
terraform destroy --auto-approve
```

---

## Can we destroy only one Terraform resource instead of all resources?

Yes, we can destroy only one resource instead of all resources.

We use the target option and specify the resource name.

### Command

```bash
terraform destroy -target=aws_instance.web
```
# Variables

## What are Variables in Terraform?

Terraform variables are used to make the code reusable.

They help us avoid hardcoding values.

For example,

If I have a 1000-line Terraform file and I use the same value many times, I don't need to change it everywhere.

I just update the variable value once, and it is updated everywhere it is used.

### Variable Example

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

Using Variable

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = var.instance_type
}
```

---

# terraform.tfvars

## What is terraform.tfvars?

Terraform.tfvars is used to store the values of the variables.

This helps us avoid hardcoding values in the Terraform code.

### Example

```hcl
instance_type = "t2.micro"
environment   = "dev"
```

---

## What is the difference between variables.tf and terraform.tfvars?

**variables.tf**

variables.tf is used to define the variables.

Example

```hcl
variable "instance_type" {}
```

---

**terraform.tfvars**

terraform.tfvars is used to store the values of those variables.

Example

```hcl
instance_type = "t2.micro"
```

---

## Can we define variable values without using terraform.tfvars?

Yes, we can define variable values without using terraform.tfvars.

We can pass the variable values while running the Terraform command.

### Command

```bash
terraform apply -var="instance_type=t2.micro"
```

or

```bash
terraform plan -var="instance_type=t2.micro"
```

Using another variable file

```bash
terraform apply -var-file="dev.tfvars"
```

---

# Output

## What is Output in Terraform?

Output is a block where we define the output values.

For example,

After Terraform creates an EC2 instance, it can display the public IP on the terminal.

So, we don't need to log in to the AWS Console to check the public IP.

### Example

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

### Command

Show all outputs

```bash
terraform output
```

Show specific output

```bash
terraform output public_ip
```

---

# Locals

## What are Locals in Terraform?

Locals are similar to variables, but they are defined in the local block.

Local values are fixed inside the code, and we cannot change them while running terraform apply.

### Example

```hcl
locals {
  environment = "dev"
}
```

Using Local

```hcl
tags = {
  Environment = local.environment
}
```

---

## Variable vs Locals

### Variables

Variables can be changed at runtime.

Example

```bash
terraform apply -var="instance_type=t3.micro"
```

---

### Locals

Locals are fixed values inside the Terraform code and cannot be changed at runtime.

---

# Module

## What is Module?

A module is a reusable Terraform code.

Instead of writing the same code again and again, we can use a module to create the infrastructure.

We can use our own module or download a module from the Terraform Registry.

### Example

```hcl
module "ec2" {
  source = "./modules/ec2"
}
```

Terraform Registry

https://registry.terraform.io/

Initialize module

```bash
terraform init
```

---

## Advantages of Module

- Reusable code
- Avoid duplicate code
- Easy to maintain
- Easy to manage
- Standardized infrastructure

# Workspace

## What is Workspace?

Terraform Workspace is used to deploy the same infrastructure in different environments like Development, Testing, and Production.

Each workspace maintains its own Terraform state file.

### Workspace Commands

List all workspaces

```bash
terraform workspace list
```

Show current workspace

```bash
terraform workspace show
```

Create a new workspace

```bash
terraform workspace new dev
```

Switch workspace

```bash
terraform workspace select dev
```

Delete workspace

```bash
terraform workspace delete dev
```

---

# Terraform Import

## What is Terraform Import?

Terraform Import is used when a resource is already created manually in AWS, and we want Terraform to manage that resource.

Instead of creating a new resource, we import the existing resource into Terraform.

### Step 1

Write the resource block.

```hcl
resource "aws_instance" "web" {

}
```

### Step 2

Import the existing resource.

```bash
terraform import aws_instance.web i-0123456789abcdef0
```

### Step 3

Verify everything.

```bash
terraform plan
```

---

## After importing the resource, is the work completed?

No.

Import is not enough.

After importing the resource, we also need to write the Terraform configuration in the `.tf` file.

Then we run terraform plan to verify everything is in sync.

---

## Multiple engineers are working on the same Terraform project. If two engineers run terraform apply at the same time, what problem can happen and how will you handle it?

If multiple engineers run terraform apply at the same time, it can create a state file conflict.

Terraform uses the state file to track infrastructure, so simultaneous changes can make the state inconsistent.

To avoid this issue, we use remote backend with state locking.

In AWS, we use:

S3 Backend → Store Terraform state file
DynamoDB → Lock the state file during changes

This allows only one person to modify infrastructure at a time.

## Why do we use remote backend in production instead of local terraform.tfstate?

Local state is not suitable for team environments because every engineer may have a different state file.

In production, we use remote backend like S3 because:

State is stored centrally
Multiple team members can access it
State locking can be implemented
Backup and recovery is easier

## You have created one EC2 module. How will you use the same module for Dev, Test and Prod?

I will create one reusable EC2 module containing the resource code.

Then I will create separate root modules for Dev, Test and Prod.

All environments will call the same child module but pass different variable values.

Example:

Dev:
 ```
instance_type = t2.micro
```

Test:
```
instance_type = t2.micro
```

Prod:
```
instance_type = t3.large
```

## Production is using module version v1.0.0. Developer released v1.1.0. Will production automatically use the new version?

No, production will not automatically update.

The caller uses the version mentioned in the module source.

To upgrade, we update the module version:

Before:
```
ref=v1.0.0
```

After:
```
ref=v1.1.0
```

Then run:
```
terraform init -upgrade
terraform plan
terraform apply
```

## An EC2 instance was created manually from AWS Console. Now you want Terraform to manage it. What will you do?
First, I will create the resource block in Terraform.

Then I will import the existing resource:
```
terraform import aws_instance.web instance-id
```
After import, I will run:
```
terraform plan
```
to verify Terraform code and AWS resource are matching.

## Someone changed EC2 instance type manually from AWS Console. What will happen?

This is called Terraform drift.

The actual AWS infrastructure and Terraform code are different.

I will run:
```
terraform plan
```

Terraform will show the difference.

Then I will either:

Update Terraform code if the change is required
Revert the manual AWS change if it is not required

## How will Terraform deployment work through Jenkins?

Flow:

```
Developer Commit Code

↓

Jenkins Trigger

↓

terraform init

↓

terraform validate

↓

terraform plan

↓

Approval

↓

terraform apply
```

## Terraform state file is deleted but AWS resources are still running. What will you do?

First, I will check if state backup is available.

If using S3 backend, I will check S3 versioning and restore the previous state file.

If backup is not available, I will import existing resources using:

```
terraform import
```


After that I will run:
```
terraform plan
```
# Terraform count vs for_each

I use count when I need multiple similar resources using numeric indexes. I use for_each when I need to create resources based on unique keys or values, because it gives better control over individual resources.
```
count

resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-xxxx"
  instance_type = "t3.micro"
}

for_each

resource "aws_instance" "web" {
  for_each = {
    dev  = "t3.micro"
    prod = "t3.medium"
  }

  ami           = "ami-xxxx"
  instance_type = each.value
}
```
# Quick Commands Revision

## Initialize

```bash
terraform init
```

---

## Validate Code

```bash
terraform validate
```

---

## Format Code

```bash
terraform fmt
```

---

## Create Execution Plan

```bash
terraform plan
```

Save Plan

```bash
terraform plan -out=tfplan
```

---

## Apply Changes

```bash
terraform apply
```

Apply Saved Plan

```bash
terraform apply tfplan
```

Skip Confirmation

```bash
terraform apply --auto-approve
```

---

## Destroy Resources

```bash
terraform destroy
```

Destroy Specific Resource

```bash
terraform destroy -target=aws_instance.web
```

---

## State Commands

List Resources

```bash
terraform state list
```

Show Resource

```bash
terraform state show aws_instance.web
```

Show Current State

```bash
terraform show
```

---

## Output Commands

Show All Outputs

```bash
terraform output
```

Show Specific Output

```bash
terraform output public_ip
```

---

## Workspace Commands

```bash
terraform workspace list
terraform workspace show
terraform workspace new dev
terraform workspace select dev
terraform workspace delete dev
```

---

## Variable Commands

Using Command Line Variable

```bash
terraform apply -var="instance_type=t2.micro"
```

Using tfvars File

```bash
terraform apply -var-file="dev.tfvars"
```

---

## Import Command

```bash
terraform import aws_instance.web i-0123456789abcdef0
```

---

# Most Important Interview Questions ⭐⭐⭐⭐⭐

- What is Terraform?
- Why Terraform instead of AWS Console?
- What is Provider?
- terraform init
- terraform plan
- terraform apply
- terraform validate
- terraform fmt
- terraform destroy
- Terraform State File
- Terraform Drift
- Variables
- terraform.tfvars
- Variable vs Local
- Output
- Module
- Workspace
- Terraform Import

---

# Self Lab Statement (Interview)

> I have working knowledge of Terraform and I am doing self-lab on my laptop.
>
> I have created AWS resources like EC2, Security Groups and VPC resources using Terraform.
>
> I understand Terraform workflow, Provider, Variables, State File, Drift, Output, Module, Workspace and Import.

---

# Interview Tips

- Always run `terraform plan` before `terraform apply` in production.
- Keep the `terraform.tfstate` file safe because Terraform tracks infrastructure using it.
- Avoid manual changes from the AWS Console to prevent Terraform Drift.
- Reuse variables and modules instead of writing duplicate code.
- Practice Import and Workspace commands because they are common interview questions.

---

# End of Terraform Interview Notes

**Level Covered**

- ✅ Beginner
- ✅ Easy Interview
- ✅ Medium Interview
- ✅ Working Knowledge
- ✅ Self Lab Ready
---
