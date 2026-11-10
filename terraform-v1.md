Task: Generate Complete Multi-Environment Terraform Infrastructure for RoboShop Microservices with Ansible Provisioning
1. Role and Objective

Act as a Senior DevOps Engineer with extensive expertise in:

AWS
Terraform
Linux (RHEL)
Ansible
Infrastructure as Code (IaC)
Microservices architecture
Multi-environment infrastructure management

Generate a complete, production-quality Terraform implementation to provision the AWS infrastructure for my RoboShop Microservices application.

Important: You must actually create all required Terraform files in the current Cursor workspace. Do not provide sample code or explanations only.

Before implementing anything, carefully inspect all the GitHub repositories referenced below. Understand their architecture, Terraform conventions, directory structure, and Ansible provisioning workflow.

Follow the specified repository structures and use the referenced repositories as the source of truth.

2. Reference GitHub Repositories
2.1 Terraform Multi-Environment Structure

Repository:

https://github.com/raghudevopsb90/terraform/tree/main/19-multi-env-code-structure

Requirements:

Study the repository carefully.
Understand its multi-environment Terraform organization.
Follow its naming conventions.
Follow its directory and file organization.
Inspect its Makefile.
Copy the original Makefile exactly as it exists.
Do not modify the Makefile under any circumstances.
2.2 RoboShop Infrastructure Architecture

Repository:

https://github.com/raghudevopsb90/roboshop-microservices/blob/main/00-overview.md

Requirements:

Read the architecture document completely.
Understand all 13 servers.
Identify component names.
Identify operating systems.
Identify instance types.
Understand application architecture and dependencies.
Use this information to determine appropriate Ansible deployment dependencies.
2.3 Ansible Configuration Repository

Repository:

https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git

Requirements:

Inspect the repository.
Understand how the roboshop.yml playbook works.
Understand how component_name is passed to the playbook.
Identify dependencies between application components.
Use the correct component names during provisioning.
3. AWS Infrastructure Requirements

Provision exactly 13 AWS EC2 instances.

Configuration	Required Value
Cloud provider	AWS
AMI ID	ami-0eec78ea4cbaaf5bd
Operating system	RHEL 10
Instance type	t3.small
VPC	Existing Default VPC
Security groups	Separate security group per EC2
SSH port	22
SSH source	0.0.0.0/0
Application ports	Component-specific
Environment	dev
Domain	rdevopsb90.online
3.1 Mandatory AMI Requirement

Use this exact AMI ID for all 13 instances:

hcl
ami = "ami-0eec78ea4cbaaf5bd"


Do not use:

Dynamic AMI lookups.
aws_ami data sources.
AMI filters.
Alternative AMI IDs.

Verify that this AMI is available in the configured AWS region.

3.2 Required EC2 Instances

Create exactly these 13 instances:

No.	Component	Instance Type	AMI
1	frontend	t3.small	ami-0eec78ea4cbaaf5bd
2	mysql	t3.small	ami-0eec78ea4cbaaf5bd
3	catalogue	t3.small	ami-0eec78ea4cbaaf5bd
4	mongodb	t3.small	ami-0eec78ea4cbaaf5bd
5	user	t3.small	ami-0eec78ea4cbaaf5bd
6	valkey	t3.small	ami-0eec78ea4cbaaf5bd
7	cart	t3.small	ami-0eec78ea4cbaaf5bd
8	shipping	t3.small	ami-0eec78ea4cbaaf5bd
9	rabbitmq	t3.small	ami-0eec78ea4cbaaf5bd
10	payment	t3.small	ami-0eec78ea4cbaaf5bd
11	notification	t3.small	ami-0eec78ea4cbaaf5bd
12	orders	t3.small	ami-0eec78ea4cbaaf5bd
13	ratings	t3.small	ami-0eec78ea4cbaaf5bd
4. Terraform Directory Structure

Follow the exact multi-environment structure from:

https://github.com/raghudevopsb90/terraform/tree/main/19-multi-env-code-structure

4.1 Directory Structure Requirements
Do not create Terraform modules.
Do not use module blocks.
Maintain environment separation.
Keep resource-specific Terraform files separate.
Do not put all resources into one main.tf.
Follow clean Terraform coding practices.
Preserve the reference repository's organizational conventions.

Create separate files for different resource types, including:

text
provider.tf
versions.tf
variables.tf
vpc.tf
security-groups.tf
ec2.tf
route53.tf
outputs.tf


Inspect the reference repository and follow its actual naming and directory conventions wherever applicable.

4.2 Environment-Specific Input Files

The following files are mandatory:

text
env-dev/
├── ec2.auto.tfvars
└── route53.auto.tfvars

File 1: env-dev/ec2.auto.tfvars

This file must contain EC2-related environment inputs:

AWS region.
Environment name.
AMI ID.
EC2 instance type.
Other EC2-related configuration.
File 2: env-dev/route53.auto.tfvars

This file must contain Route53-related inputs:

Domain name.
Hosted zone ID.
Environment DNS suffix.
Other Route53-specific configuration.

Keep the inputs logically separated.

Do not combine all environment inputs into one terraform.tfvars file.

4.3 Loading Both Variable Files

Terraform does not automatically load .auto.tfvars files located inside nested directories when executed from the project root.

Inspect the reference repository's execution approach and ensure both files are loaded correctly.

If necessary, document explicit commands:

Terraform Plan:

bash
terraform plan \
  -var-file="env-dev/ec2.auto.tfvars" \
  -var-file="env-dev/route53.auto.tfvars"


Terraform Apply:

bash
terraform apply \
  -var-file="env-dev/ec2.auto.tfvars" \
  -var-file="env-dev/route53.auto.tfvars"


Do not assume Terraform automatically discovers nested variable files.

5. Mandatory Makefile Requirement

The Makefile must be copied exactly as it exists in the reference repository.

Reference:

https://github.com/raghudevopsb90/terraform/tree/main/19-multi-env-code-structure

Copy the following Makefile without changing its contents:

makefile
# Usage: make apply env=dev   |   make destroy env=prod

init:
	rm -rf .terraform *.auto.tfvars
	terraform init -backend-config=env-$(env)/state.tfvars
	# This only command saves a lot and lot of TF code and avoid module for a single project mono repo.
	cp env-$(env)/*.auto.tfvars .

plan: init
	terraform plan -var-file=env-$(env)/main.tfvars

apply: init
	terraform apply -auto-approve

destroy: init
	terraform destroy -auto-approve

Mandatory Rules
Copy the original Makefile from the reference repository.
Do not modify its contents.
Do not modify its commands.
Do not modify its comments.
Do not modify its formatting.
Do not modify it to accommodate other Terraform requirements.
Verify that the copied file is 100% identical to the original.
Report the comparison result.

If the Makefile conflicts with the required environment variable file structure, explain the conflict and provide a separate execution approach without modifying the Makefile.

6. AWS Provider and Default VPC Configuration

Use the AWS Terraform provider.

Create appropriate:

Provider configuration.
Terraform version constraints.
AWS provider version constraints.
VPC Requirements

Use the existing AWS Default VPC.

Use Terraform data sources to retrieve:

Default VPC ID.
Available subnets belonging to the Default VPC.

Requirements:

Do not create a new VPC.
Do not create new subnets.
Do not create an Internet Gateway.
Do not create a NAT Gateway.
Do not create new route tables.
Reuse existing Default VPC infrastructure.
Select an appropriate available subnet dynamically.

Keep all VPC-related data sources in:

text
vpc.tf

7. Security Group Configuration

Create exactly 13 separate security groups, one for each EC2 instance.

7.1 Common SSH Ingress Rule

Every security group must include:

Port	Protocol	Source
22	TCP	0.0.0.0/0
7.2 Application Port Mapping
Component	Application Port
frontend	3000
mysql	3306
catalogue	8002
mongodb	27017
user	8001
valkey	6379
cart	8003
shipping	8004
rabbitmq	5672
payment	8005
notification	8008
orders	8007
ratings	8006
7.3 Security Group Requirements
Allow each component's application port from 0.0.0.0/0.
Allow SSH port 22 from 0.0.0.0/0.
Associate each security group with its corresponding EC2 instance.
Do not create one shared security group.
Keep security group resources in security-groups.tf.
Use Terraform references to associate security groups with instances.
Do not silently change specified ingress CIDRs or ports.
8. EC2 Instance Configuration

Create all 13 EC2 instances using Terraform.

Use this exact configuration:

hcl
ami           = "ami-0eec78ea4cbaaf5bd"
instance_type = "t3.small"

8.1 Requirements
Use the exact AMI ID.
Use t3.small for every instance.
Use the existing Default VPC.
Select an available subnet dynamically.
Create a separate security group for every instance.
Associate the correct security group with each instance.
Enable public IP assignment for SSH-based Terraform provisioning.
Capture each instance's private IP.
Capture each instance's public IP.
Assign meaningful Name tags.

Keep EC2 resources in:

text
ec2.tf


Do not dynamically retrieve the AMI.

8.2 Required Tags

Use meaningful tags such as:

hcl
project   = "roboshop"
env       = "dev"
component = "frontend"
Name      = "frontend-dev"


Replace the component name appropriately for all 13 instances.

9. Route53 DNS Configuration

Create exactly 13 Route53 DNS A records.

9.1 Route53 Configuration
Configuration	Value
Domain	rdevopsb90.online
Hosted Zone ID	Z05126561EI2V4JVVXVBY
Environment	dev
Record type	A
DNS target	EC2 private IP

Use these exact development environment values.

Domain:

text
rdevopsb90.online


Hosted Zone ID:

text
Z05126561EI2V4JVVXVBY

9.2 DNS Naming Convention

Use:

text
<component>-dev.rdevopsb90.online


Create these records:

No.	DNS Record
1	frontend-dev.rdevopsb90.online
2	mysql-dev.rdevopsb90.online
3	catalogue-dev.rdevopsb90.online
4	mongodb-dev.rdevopsb90.online
5	user-dev.rdevopsb90.online
6	valkey-dev.rdevopsb90.online
7	cart-dev.rdevopsb90.online
8	shipping-dev.rdevopsb90.online
9	rabbitmq-dev.rdevopsb90.online
10	payment-dev.rdevopsb90.online
11	notification-dev.rdevopsb90.online
12	orders-dev.rdevopsb90.online
13	ratings-dev.rdevopsb90.online
9.3 DNS Implementation Requirements
Create exactly 13 Route53 A records.
Every record must point to the corresponding EC2 instance's PRIVATE IP address.
Never use public IP addresses for DNS records.
Never hardcode EC2 private IP addresses.
Reference the private IP attributes of the corresponding EC2 resources.
Use the supplied hosted zone ID.
Do not create a new hosted zone.
Do not modify existing hosted zone configuration.
Keep all Route53 resources in route53.tf.
Use variables for domain name, hosted zone ID, and environment suffix.
Use meaningful Terraform resource names.

Use the hosted zone ID through a variable populated by:

text
env-dev/route53.auto.tfvars


Example:

hcl
domain_name    = "rdevopsb90.online"
hosted_zone_id = "Z05126561EI2V4JVVXVBY"
environment    = "dev"


Example DNS association:

text
frontend-dev.rdevopsb90.online → Frontend EC2 private IP


Ensure all 13 records reference the correct corresponding EC2 instances.

10. Ansible Installation and Provisioning Using Terraform Provisioners

This is one of the most important requirements.

After creating every EC2 instance, Terraform must connect to that instance using SSH and execute its Ansible configuration.

Ansible will NOT be preinstalled on the EC2 instances.

Terraform must independently install Python pip, install Ansible, and execute Ansible Pull on all 13 instances.

10.1 SSH Connection

Use Terraform remote-exec provisioners.

Configure a proper Terraform connection block.

Required connection settings:

Configuration	Value
SSH username	ec2-user
SSH password	DevOps321
Connection target	EC2 private IP
Provisioner	remote-exec
Connection timeout	Appropriate SSH timeout
10.2 Step 1: Install Python pip

Execute:

bash
sudo dnf install python3.13-pip -y

10.3 Step 2: Install Ansible

Execute:

bash
sudo pip3.13 install ansible

10.4 Step 3: Execute Ansible Pull

After installing Ansible, execute:

bash
ansible-pull -i localhost, \
  -e ansible_user=ec2-user \
  -e ansible_password=DevOps321 \
  -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git \
  roboshop.yml \
  -e component_name=<COMPONENT_NAME>


Replace <COMPONENT_NAME> with the appropriate component name for each instance.

10.5 Component-to-Ansible Mapping
EC2 Instance	component_name
Frontend	frontend
MySQL	mysql
Catalogue	catalogue
MongoDB	mongodb
User	user
Valkey	valkey
Cart	cart
Shipping	shipping
RabbitMQ	rabbitmq
Payment	payment
Notification	notification
Orders	orders
Ratings	ratings
10.6 Ansible Pull Examples

Frontend:

bash
ansible-pull -i localhost, \
  -e ansible_user=ec2-user \
  -e ansible_password=DevOps321 \
  -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git \
  roboshop.yml \
  -e component_name=frontend


MySQL:

bash
ansible-pull -i localhost, \
  -e ansible_user=ec2-user \
  -e ansible_password=DevOps321 \
  -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git \
  roboshop.yml \
  -e component_name=mysql


Follow the same pattern for all 13 instances.

10.7 Provisioner Implementation Requirements
Use remote-exec provisioners.
Use Terraform connection blocks.
Use the instance private IP for SSH connectivity.
Install Python pip before installing Ansible.
Install Ansible before executing Ansible Pull.
Execute Ansible Pull independently on all 13 instances.
Pass the correct component name to every instance.
Configure appropriate connection timeouts.
Ensure Terraform reports provisioning failures.
Detach provisioners from aws_instance using null_resource, so Ansible failures do not impact the EC2 instance lifecycle.
Do not install Ansible on only one instance.
Do not skip any component.
Do not use local-exec as a replacement for remote-exec.
Use appropriate Terraform resource references, such as self.public_ip, for SSH connections where applicable.
10.8 Credentials Handling

The supplied Ansible password is:

text
DevOps321


Use sensitive Terraform variables for password handling.

Do not unnecessarily expose credentials in Terraform outputs or documentation.

11. Deployment Dependencies and Execution Order

Study the RoboShop architecture document and Ansible repository before deciding the deployment order.

Some application components depend on:

Databases.
Caching services.
Message brokers.
Other application components.

Ensure infrastructure provisioning and Ansible configuration account for these dependencies.

Requirements
Do not assume all 13 Ansible configurations can run successfully in parallel.
Provision required backend services before dependent application components when necessary.
Use Terraform dependencies where appropriate.
Avoid circular dependencies.
Keep deployment ordering understandable.
Do not introduce unnecessary Terraform resources.
Follow the actual application architecture and Ansible playbook requirements rather than inventing dependencies.
12. Terraform Variables and Environment Inputs

Create a proper:

text
variables.tf


Include variables for:

Variable	Purpose
aws_region	AWS deployment region
environment	Deployment environment
ami_id	Fixed AMI ID
instance_type	EC2 instance type
domain_name	Route53 domain
hosted_zone_id	Route53 hosted zone ID
ansible_password	Ansible authentication password

Use a sensitive declaration for password variables.

Mandatory values:

hcl
ami_id       = "ami-0eec78ea4cbaaf5bd"
instance_type = "t3.small"


Do not introduce dynamic AMI discovery.

12.1 Mandatory Environment-Specific Files

Create:

text
env-dev/
├── ec2.auto.tfvars
└── route53.auto.tfvars

12.2 env-dev/ec2.auto.tfvars

Keep EC2-related inputs here.

Example:

hcl
aws_region    = "YOUR_AWS_REGION"
environment   = "dev"
ami_id        = "ami-0eec78ea4cbaaf5bd"
instance_type = "t3.small"


Use a placeholder for the AWS region.

12.3 env-dev/route53.auto.tfvars

Keep Route53-related inputs here.

Example:

hcl
domain_name    = "rdevopsb90.online"
hosted_zone_id = "Z05126561EI2V4JVVXVBY"
environment    = "dev"

Important Separation Rules
Do not put Route53 configuration into the EC2 input file.
Do not put EC2-specific inputs into the Route53 input file.
Do not combine both files into one terraform.tfvars.
13. Terraform Outputs

Create a separate:

text
outputs.tf


Provide the following outputs:

All 13 EC2 instance IDs.
All 13 EC2 private IP addresses.
All 13 EC2 public IP addresses.
All 13 DNS record names.
DNS name and corresponding private IP mapping.
Default VPC ID.
Deployment environment.

Use meaningful output names.

All output values must reference actual Terraform resources or data sources.

Do not hardcode output values.

14. Terraform Coding Standards

Follow these rules throughout the implementation:

No Terraform modules.
No module blocks.
Exactly 13 EC2 instances.
Exactly 13 separate security groups.
Exactly 13 Route53 A records.
All EC2 instances must use AMI ami-0eec78ea4cbaaf5bd.
All EC2 instances must use t3.small.
Use the existing Default VPC.
DNS records must resolve to private IP addresses.
SSH must be enabled on port 22.
Application-specific ports must be opened as specified.
Ansible must be installed independently on all 13 instances.
The correct Ansible component name must be passed to every instance.
Follow the reference multi-environment directory structure.
Keep resource-specific Terraform files separate.
Keep EC2 and Route53 input files separate.
Copy the original Makefile without changing anything.
Use Terraform references instead of hardcoded resource attributes.
Avoid unnecessary duplication.
Use meaningful resource names and tags.
Keep code properly formatted.
Do not create unnecessary AWS infrastructure.
Do not create a new Route53 hosted zone.
15. Validation and Execution

After creating the complete Terraform implementation, perform the following checks.

15.1 Directory Validation

Verify that:

Directory structure matches the reference repository.
Environment separation is implemented correctly.
env-dev/ec2.auto.tfvars exists.
env-dev/route53.auto.tfvars exists.
Both input files are loaded correctly.
Resource-specific Terraform files exist.
No Terraform modules have been created.
Makefile is present in the project root.
Makefile is 100% identical to the original.
15.2 Infrastructure Validation

Verify that:

Exactly 13 EC2 instances are configured.
Exactly 13 security groups are configured.
Exactly 13 Route53 A records are configured.
All EC2 instances use the specified AMI ID.
All EC2 instances use t3.small.
All EC2 instances use the Default VPC.
Every DNS record references the correct private IP.
Hosted zone ID is correct.
Domain name is correct.
Application ports are correct.
SSH configurations are correct.
15.3 Ansible Validation

Verify that:

Python pip installation commands are present.
Ansible installation commands are present.
Ansible Pull command is correct.
All 13 component names are configured.
Every instance has its own remote-exec provisioning configuration.
SSH connection settings are correct.
Deployment dependencies are configured appropriately.
15.4 Makefile Validation
Compare the original reference Makefile with the copied Makefile.
Verify identical contents.
Do not modify the copied Makefile.
Report the comparison result.
15.5 Terraform Formatting

Run:

bash
terraform fmt -recursive


Also run Terraform initialization and validation where the environment permits.

15.6 Terraform Plan

Run:

bash
terraform plan \
  -var-file="env-dev/ec2.auto.tfvars" \
  -var-file="env-dev/route53.auto.tfvars"


Verify the planned resource counts and configuration.

Do not apply infrastructure unless explicitly authorized.

16. Expected Deliverables

Generate the complete implementation in the current Cursor workspace.

Expected deliverables:

Complete multi-environment Terraform directory structure.
All resource-specific .tf files.
AWS provider configuration.
Terraform version constraints.
Variables.
env-dev/ec2.auto.tfvars.
env-dev/route53.auto.tfvars.
Original Makefile copied exactly from the reference repository.
Default VPC data source configuration.
Exactly 13 EC2 instances.
Exactly 13 individual security groups.
Exactly 13 Route53 DNS records.
Correct private-IP-based DNS associations.
Terraform SSH connection configurations.
Remote Ansible installation on all 13 instances.
Correct Ansible Pull commands for all 13 components.
Terraform outputs.
Successful Terraform formatting and validation, where execution prerequisites permit.
Documented Terraform execution commands.
17. Final Instructions to Cursor

Follow this implementation sequence.

Phase 1: Repository Inspection
Inspect the Terraform reference repository.
Inspect its exact directory structure.
Retrieve and inspect the original Makefile.
Copy the original Makefile verbatim into the project root.
Verify the copied Makefile is identical to the original.
Inspect the RoboShop architecture document.
Inspect the Ansible repository.
Understand the multi-environment directory structure.
Identify application dependencies and Ansible provisioning requirements.
Phase 2: Terraform Implementation
Create the complete Terraform directory structure in the current workspace.
Create env-dev/ec2.auto.tfvars.
Create env-dev/route53.auto.tfvars.
Implement AWS provider and Default VPC data sources.
Implement the 13 individual security groups.
Implement the 13 EC2 instances using the exact AMI ID.
Implement the 13 Route53 records using EC2 private IP addresses.
Implement Terraform remote-exec provisioning for all 13 instances.
Install Python pip and Ansible on every instance.
Execute the correct Ansible Pull command for every component.
Implement Terraform outputs.
Verify all resource dependencies and references.
Phase 3: Validation
Run Terraform formatting.
Run Terraform initialization.
Run Terraform validation.
Run Terraform plan using both environment-specific input files.
Verify the Makefile remains unchanged.
Fix any errors discovered during validation.
Report final implementation details.

Do not stop after generating sample code or a partial implementation.

Actually create all required files.

Do not silently skip any requirements.

If a requirement conflicts with the reference repositories or AWS limitations, explain the conflict and propose a solution instead of making assumptions.

18. Final Objective

Generate a complete, working, multi-environment Terraform implementation that:

Provisions exactly 13 RoboShop EC2 instances.
Uses AMI ami-0eec78ea4cbaaf5bd.
Uses t3.small for every instance.
Reuses the existing Default VPC.
Creates 13 individual security groups.
Creates 13 Route53 A records using private IP addresses.
Uses Route53 domain rdevopsb90.online.
Uses hosted zone ID Z05126561EI2V4JVVXVBY.
Separates EC2 and Route53 inputs into env-dev/ec2.auto.tfvars and env-dev/route53.auto.tfvars.
Copies the original Makefile from the reference repository with 100% identical content.
Does not modify the Makefile in any way.
Installs Ansible independently on all 13 instances.
Executes the correct Ansible Pull command for every component through Terraform remote-exec.
Follows the reference multi-environment directory structure.
Does not create Terraform modules.
Passes Terraform formatting and validation.

Start by inspecting the reference repositories, copying the original Makefile exactly, and then implementing the complete Terraform solution directly in the current Cursor workspace.
