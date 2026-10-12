# Terraform AWS Provisioning

## Overview

This repository contains my hands-on practice with Terraform and AWS Infrastructure as Code (IaC). It documents the process of provisioning AWS resources, configuring Terraform providers, managing infrastructure state, and troubleshooting common Terraform and AWS CLI issues.

## Projects

### 1. AWS EC2 Provisioning with Local State

**Technologies:** Terraform, AWS EC2, AWS CLI, HCL, Linux, Git

- Created a Terraform configuration to define an AWS EC2 instance.
- Configured the AWS provider, region, AMI, instance type, and resource tags.
- Used Terraform commands to initialize the working directory, validate configuration, review infrastructure changes, and manage the resource lifecycle.
- Practiced AWS CLI authentication and troubleshooting provider credential errors.
- Worked with Terraform state stored locally.

**Key file:** `aws/local_state/main.tf`

### 2. Terraform Remote State with S3 and DynamoDB

**Technologies:** Terraform, AWS S3, DynamoDB, AWS CLI

- Practiced configuring AWS resources for Terraform remote state management.
- Worked on S3 bucket and DynamoDB table configuration for the backend exercise.
- Explored how remote state supports centralized state storage and how state locking helps prevent concurrent Terraform operations.
- Documented the configuration and workflow in the remote-state project files.

**Key files:** `aws/remote_state/main.tf`, `aws/remote_state/outputs.tf`, `aws/remote_state/README.md`

*Implementation status should be updated after verifying the actual resources and backend configuration.*

## Tools and Skills Practiced

- Terraform HCL and provider configuration
- AWS EC2, S3, and DynamoDB
- Terraform local and remote state concepts
- Variables, outputs, resource configuration, and provider version constraints
- AWS CLI authentication and troubleshooting
- Terraform `init`, `validate`, `plan`, `apply`, and `destroy`
- Git and GitHub version control

## Repository Structure

```text
terraform-aws-learning-projects/
├── README.md
├── .gitignore
├── images/
└── aws/
    ├── local_state/
    │   └── main.tf
    └── remote_state/
        ├── main.tf
        ├── outputs.tf
        ├── README.md
        └── s3.md
```

## Security Practices

- Do not commit AWS access keys, secret keys, private keys, or credentials.
- Exclude Terraform state files and sensitive variable files from Git.
- Review Terraform plans before applying infrastructure changes.
- Verify AWS resource costs and destroy temporary resources when they are no longer needed.

## Learning Outcome

This project helped me practice Infrastructure as Code fundamentals, AWS resource provisioning, Terraform state concepts, AWS CLI authentication, and troubleshooting common configuration issues.
