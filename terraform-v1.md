Task: Generate Complete Multi-Environment Terraform Infrastructure for RoboShop Microservices with Ansible Provisioning

Act as a Senior DevOps Engineer with extensive expertise in AWS, Terraform, Linux, Ansible, and Infrastructure as Code.

I want you to generate complete Terraform code to provision the AWS infrastructure for my RoboShop Microservices application.

You must actually create all required Terraform files in the current Cursor workspace.

Before writing any code, carefully inspect the GitHub repositories mentioned below. Understand the infrastructure requirements and follow the specified Terraform directory structure.

Do not generate sample code or explanations only. Implement the complete solution.

1. Reference GitHub Repositories
Terraform code structure

https://github.com/raghudevopsb90/terraform/tree/main/19-multi-env-code-structure

Study this repository carefully.

Follow its multi-environment Terraform code organization, naming conventions, and file organization.

Important: The Makefile in this repository must be copied exactly as it exists. Do not modify it.

RoboShop infrastructure architecture

https://github.com/raghudevopsb90/roboshop-microservices/blob/main/00-overview.md

Read this document carefully to understand all 13 servers, their component names, operating systems, instance types, and application architecture.

Ansible configuration repository

https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git

Inspect this repository to understand how the Ansible playbook works and how component names are passed to roboshop.yml.

Use these repositories as the source of truth.

2. Infrastructure Requirements

Provision exactly 13 AWS EC2 instances.

Use the following configuration for all instances:

Configuration	Value
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
Important AMI requirement

Use this exact AMI ID for all 13 EC2 instances:

ami-0eec78ea4cbaaf5bd

Do not use:

Dynamic AMI lookups.
aws_ami data sources.
AMI filters.
Any alternative AMI ID.

Verify that the AMI is available in the configured AWS region.

Required EC2 instances

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
3. Terraform Directory Structure

Follow the exact multi-environment structure from:

https://github.com/raghudevopsb90/terraform/tree/main/19-multi-env-code-structure

Important requirements:

Do not create Terraform modules.
Do not use module blocks.
Maintain environment separation.
Keep resource-specific Terraform files separate.
Do not put all resources in a single main.tf.
Follow clean Terraform coding practices.
Make the configuration understandable and maintainable.
Preserve the organizational conventions of the reference repository.
Resource-specific Terraform files

Create separate files for different resource types.

For example:

provider.tf
versions.tf
variables.tf
vpc.tf
security-groups.tf
ec2.tf
route53.tf
outputs.tf

Inspect the reference repository and use its actual naming and directory conventions wherever applicable.

Do not create modules under any circumstances.

Environment-specific input files

The environment-specific input files must be:

text
env-dev/
├── ec2.auto.tfvars
└── route53.auto.tfvars


These two files are mandatory.

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

Important: Loading both variable files

Terraform does not automatically load .auto.tfvars files from nested directories when executed from the project root.

Therefore, inspect the reference repository's execution approach and ensure both files are loaded correctly.

If required, document explicit commands such as:

bash
terraform plan \
  -var-file="env-dev/ec2.auto.tfvars" \
  -var-file="env-dev/route53.auto.tfvars"


Similarly, document the apply command.

Do not assume Terraform automatically discovers nested .auto.tfvars files.

4. Mandatory Makefile Requirement – 100% Exact create as per here

```
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
```

5. AWS Provider and Default VPC Configuration

Use the AWS Terraform provider.

Create appropriate provider and Terraform version configuration.

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

Keep VPC-related data sources in vpc.tf.

6. Security Group Configuration

Create a separate security group for every EC2 instance.

Exactly 13 security groups must be created.

Each security group must have this common SSH ingress rule:

Port	Protocol	Source
22	TCP	0.0.0.0/0

Additionally, allow the appropriate application port for each component.

Application port mapping
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

Requirements:

Allow each component's application port from 0.0.0.0/0.
Allow SSH port 22 from 0.0.0.0/0.
Associate each security group with its corresponding EC2 instance.
Do not create one shared security group.
Keep security group resources in security-groups.tf.
Use Terraform references to associate security groups with instances.

Do not silently change the specified ingress CIDRs or ports.

7. EC2 Instance Configuration

Create all 13 EC2 instances using Terraform.

Use this exact configuration:

hcl
ami           = "ami-0eec78ea4cbaaf5bd"
instance_type = "t3.small"


Requirements:

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
Required Name tags

Use names such as:

project = roboshop
env = dev
component = frontend<repalce for others>
Name = frontend-dev

Keep EC2 resources in ec2.tf.

Do not dynamically retrieve the AMI.

8. Route53 DNS Configuration

Create exactly 13 DNS records.

Use the following Route53 details.

Route53 configuration
Configuration	Value
Domain	rdevopsb90.online
Hosted Zone ID	Z05126561EI2V4JVVXVBY
Environment	dev
Record type	A
DNS target	EC2 private IP

Use these exact values as the development environment configuration.

Domain:

rdevopsb90.online

Hosted Zone ID:

Z05126561EI2V4JVVXVBY

Required DNS naming convention

Use:

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

DNS implementation requirements
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

Use the hosted zone ID directly through a variable populated by env-dev/route53.auto.tfvars.

Example:

hcl
domain_name    = "rdevopsb90.online"
hosted_zone_id = "Z05126561EI2V4JVVXVBY"
environment    = "dev"


Example record:

frontend-dev.rdevopsb90.online → Frontend EC2 private IP.

Make sure all 13 records reference the correct corresponding EC2 instances.

9. Ansible Installation and Provisioning Using Terraform Provisioners

This is one of the most important requirements.

After creating every EC2 instance, Terraform must connect to that instance using SSH and execute the Ansible configuration.

Ansible will NOT be preinstalled on the EC2 instances.

Terraform must install Python pip, install Ansible, and execute Ansible Pull independently on all 13 instances.

Step 1: SSH connection

Use Terraform remote-exec provisioners.

Use a proper Terraform connection block.

Requirements:

SSH username: ec2-user.
SSH password: DevOps321
Connect using the EC2 private IP.
Configure suitable SSH connection timeouts.


Step 2: Install Python pip

Execute:

bash
sudo dnf install python3.13-pip -y

Step 3: Install Ansible

Execute:

bash
sudo pip3.13 install ansible

Step 4: Execute Ansible Pull

After installing Ansible, execute:

bash
ansible-pull -i localhost, -e ansible_user=ec2-user -e ansible_password=DevOps321 -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git roboshop.yml -e component_name=<COMPONENT_NAME>


Replace <COMPONENT_NAME> with the appropriate component name for each instance.

Component-to-Ansible mapping
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
Example: Frontend

bash
ansible-pull -i localhost, -e ansible_user=ec2-user -e ansible_password=DevOps321 -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git roboshop.yml -e component_name=frontend

Example: MySQL
bash
ansible-pull -i localhost, -e ansible_user=ec2-user -e ansible_password=DevOps321 -U https://github.com/raghudevopsb90/roboshop-ansible-v4-with-opencode.git roboshop.yml -e component_name=mysql


Follow the same pattern for all 13 instances.

Provisioner implementation requirements
Use remote-exec provisioners.
Use Terraform connection blocks.
Use the instance private IP for SSH connectivity.
Install Python pip before installing Ansible.
Install Ansible before executing ansible-pull.
Execute Ansible Pull independently on all 13 instances.
Pass the correct component name to every instance.
Configure appropriate connection timeouts.
Ensure Terraform reports provisioning failures.
Detach provisioner with null_resource from aws_instance. This way ansible failure will not impact ec2 instance.
Do not install Ansible on only one instance.
Do not skip any component.
Do not use local-exec as a replacement for remote-exec.

Use appropriate Terraform resource references, such as self.public_ip, for SSH connections.

Credentials handling

The supplied Ansible password is:

DevOps321

10. Deployment Dependencies and Execution Order

Study the RoboShop architecture document and the Ansible repository before deciding the deployment order.

Some application components depend on databases, caching services, and message brokers.

Ensure infrastructure provisioning and Ansible configuration account for these dependencies.

Requirements:

Do not assume all 13 Ansible configurations can run successfully in parallel.
Provision required backend services before dependent application components when necessary.
Use Terraform dependencies where appropriate.
Avoid circular dependencies.
Keep the deployment order understandable.
Do not introduce unnecessary Terraform resources.

Follow the actual application architecture and Ansible playbook requirements rather than inventing dependencies.

11. Terraform Variables and Environment Inputs

Create a proper variables.tf.

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

The AMI ID must remain:

ami-0eec78ea4cbaaf5bd

The instance type must remain:

t3.small

Do not introduce dynamic AMI discovery.

Mandatory environment-specific input files

Create:

text
env-dev/
├── ec2.auto.tfvars
└── route53.auto.tfvars

env-dev/ec2.auto.tfvars

Keep EC2-related inputs here.

Example:

hcl
aws_region       = "YOUR_AWS_REGION"
environment      = "dev"
ami_id           = "ami-0eec78ea4cbaaf5bd"
instance_type    = "t3.small"



Use placeholders for AWS region.

env-dev/route53.auto.tfvars

Keep Route53-related inputs here.

Example:

hcl
domain_name    = "rdevopsb90.online"
hosted_zone_id = "Z05126561EI2V4JVVXVBY"
environment    = "dev"


Use the exact domain and hosted zone ID provided.

Do not put Route53 configuration into the EC2 input file.

Do not put EC2-specific inputs into the Route53 input file.

12. Terraform Outputs

Create a separate outputs.tf.

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

13. Terraform Coding Standards

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
14. Validation and Execution

After creating the complete Terraform implementation, perform the following checks.

Directory validation
Verify that the directory structure matches the reference repository.
Verify that environment separation is implemented correctly.
Verify env-dev/ec2.auto.tfvars exists.
Verify env-dev/route53.auto.tfvars exists.
Verify both input files are loaded correctly.
Verify resource-specific Terraform files exist.
Verify no Terraform modules have been created.
Verify the Makefile is present in the project root.
Verify the Makefile is 100% identical to the original.
Infrastructure validation
Verify exactly 13 EC2 instances.
Verify exactly 13 security groups.
Verify exactly 13 Route53 A records.
Verify all EC2 instances use the specified AMI ID.
Verify all EC2 instances use t3.small.
Verify all EC2 instances use the Default VPC.
Verify every DNS record references the correct private IP.
Verify the hosted zone ID.
Verify the domain name.
Verify application ports.
Verify SSH configurations.
Ansible validation
Verify Python pip installation commands.
Verify Ansible installation commands.
Verify the ansible-pull command.
Verify all 13 component names.
Verify every instance has its own remote-exec provisioning configuration.
Verify SSH connection settings.
Verify deployment dependencies.
Makefile validation
Compare the original reference Makefile with the copied Makefile.
Verify identical contents.
Do not modify the copied Makefile to accommodate any other requirement.
Report the comparison result.
Terraform commands

Run:

bash
terraform fmt -recursive


15. Expected Deliverables

Generate the complete implementation in the current Cursor workspace.

I expect:

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
Correct ansible-pull commands for all 13 components.
Terraform outputs.
Successful Terraform formatting and validation.
Documented Terraform execution commands.
16. Final Instructions to Cursor

Follow this implementation sequence:

Inspect the Terraform reference repository.
Inspect its exact directory structure.
Retrieve and inspect the original Makefile.
Copy the original Makefile verbatim into the project root.
Verify the copied Makefile is identical to the original.
Inspect the RoboShop architecture document.
Inspect the Ansible repository.
Understand the multi-environment directory structure.
Create the complete Terraform directory structure in the current workspace.
Create env-dev/ec2.auto.tfvars.
Create env-dev/route53.auto.tfvars.
Implement the AWS provider and Default VPC data sources.
Implement the 13 individual security groups.
Implement the 13 EC2 instances using the exact AMI ID.
Implement the 13 Route53 records using EC2 private IP addresses.
Implement Terraform remote-exec provisioning for all 13 instances.
Install Python pip and Ansible on every instance.
Execute the correct ansible-pull command for every component.
Implement Terraform outputs.
Verify all resource dependencies and references.
Run Terraform formatting, initialization, and validation.
Run Terraform plan using both environment-specific input files.
Verify the Makefile remains unchanged.
Fix any errors discovered during validation.
Report the final implementation details.

Do not stop after generating sample code or a partial implementation.

Actually create all required files.

Do not silently skip any requirements.

If a requirement conflicts with the reference repositories or AWS limitations, explain the conflict and propose a solution instead of making assumptions.

Final Objective

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
