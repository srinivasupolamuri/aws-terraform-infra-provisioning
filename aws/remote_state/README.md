# Terraform Remote State Backend on AWS (S3 + DynamoDB)

## Overview

This project explores how Terraform state can be stored remotely in AWS instead of being kept only on a developer's machine. The configuration is organized around an S3 bucket for state storage and a DynamoDB table for state locking, following the backend approach used in the original learning exercise.

Terraform state tracks the resources managed by a Terraform configuration. When state exists only on a local workstation, sharing work with another engineer or running Terraform from a different machine becomes difficult. A remote backend provides centralized storage and helps coordinate infrastructure changes.

> **Implementation note:** This README describes the S3 + DynamoDB backend pattern used in this learning exercise. DynamoDB-based locking for the Terraform S3 backend is deprecated in current Terraform documentation. For new implementations, prefer native S3 lock-file support (`use_lockfile = true`) unless you need to maintain a legacy setup.

## Project goals

- Understand the purpose of Terraform state and why remote state is useful.
- Configure AWS provider access for the backend setup.
- Create an S3 bucket to hold Terraform state.
- Configure a DynamoDB table for state-lock coordination in the legacy approach.
- Learn how to initialize a backend and migrate existing local state safely.
- Practice reviewing Terraform plans and protecting backend resources.

## Architecture

```text
Developer / CI runner
        |
        | AWS credentials or IAM role
        v
    Terraform CLI
        |
        +--------------------------+
        |                          |
        v                          v
    Amazon S3                 DynamoDB
  State file storage       Legacy state lock
        |
        v
 Terraform state for the managed infrastructure
```

S3 stores the state object. In the legacy S3 backend configuration, DynamoDB provides a lock so two Terraform operations do not modify the same state at the same time.

## Repository files

The original learning folder contains:

```text
aws/remote_state/
├── main.tf
├── outputs.tf
├── README.md
└── s3.md
```

- `main.tf` — backend-related AWS resource and configuration definitions.
- `outputs.tf` — outputs useful information about created resources.
- `s3.md` — supporting notes about S3.
- `README.md` — project explanation and usage notes.

Review the existing `main.tf` before running commands. Resource names, bucket names, DynamoDB key schema, and backend settings must agree with one another.

## Prerequisites

- An AWS account.
- Terraform CLI installed.
- AWS CLI installed and configured, or another supported AWS credential method.
- IAM permissions to create and manage the S3 bucket and DynamoDB table.
- A globally unique S3 bucket name.
- A selected AWS region.

Check AWS authentication:

```bash
aws sts get-caller-identity
aws configure list
```

Do not put AWS access keys or secret keys directly into Terraform files. Prefer an AWS CLI profile, environment credentials, or an IAM role where available.

## Step 1 — Review the configuration

```bash
cd ~/devops_project/downloads/write_your_first_terraform_project/aws/remote_state
ls -la
cat main.tf
cat outputs.tf
```

Make sure the S3 bucket name is globally unique and that the configured AWS region is intentional. Do not use a bucket name that belongs to another account.

## Step 2 — Initialize and validate

```bash
terraform fmt
terraform init
terraform validate
```

`terraform init` downloads required providers and initializes the working directory. `terraform validate` checks configuration syntax and internal consistency; it does not prove that AWS permissions or every remote setting will succeed.

## Step 3 — Review the plan

```bash
terraform plan
```

Confirm which S3 and DynamoDB resources Terraform proposes to create or modify. Do not apply a plan that proposes unexpected deletion or replacement.

## Step 4 — Create the backend resources

If the plan is correct:

```bash
terraform apply
```

Review the plan and enter `yes` only if you intend to create the resources. Keep the state for this bootstrap configuration safe: it may be needed to manage the S3 bucket and DynamoDB table later.

## Step 5 — Configure an S3 backend for another Terraform project

Creating an S3 bucket and DynamoDB table does not automatically move another project's state. The application configuration needs a matching backend block.

A legacy S3 backend using DynamoDB locking looks like this:

```hcl
terraform {
  backend "s3" {
    bucket         = "YOUR_EXISTING_STATE_BUCKET"
    key            = "my-project/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "YOUR_EXISTING_LOCK_TABLE"
  }
}
```

Replace the example bucket, key, region, and table names with actual values. The S3 bucket and DynamoDB table must exist before initializing the project with this backend.

From the directory of the Terraform project whose state should be moved, add the backend configuration and run:

```bash
terraform init -migrate-state
```

Read Terraform's migration prompt and confirm only if you intend to migrate existing local state. After migration, verify Terraform can read the state and that `terraform plan` does not show unexpected resource recreation.

## Legacy DynamoDB locking and current recommendation

The original exercise uses DynamoDB for state locking. In current Terraform documentation, DynamoDB-based locking for the S3 backend is deprecated. New projects should generally use native S3 lock-file support:

```hcl
terraform {
  backend "s3" {
    bucket       = "YOUR_EXISTING_STATE_BUCKET"
    key          = "my-project/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

This requires a compatible Terraform version and appropriate S3 permissions for the state object and lock file. Do not configure both approaches without understanding migration and compatibility implications. Official documentation: https://developer.hashicorp.com/terraform/language/backend/s3

## Security and operational practices

- Enable S3 bucket versioning so earlier state object versions can be recovered.
- Enable server-side encryption and block all public access to the state bucket.
- Apply least-privilege IAM permissions for the bucket, state key, and (for legacy locking) DynamoDB table.
- Treat Terraform state as sensitive; it can contain resource attributes and sometimes secret values.
- Never commit `terraform.tfstate`, state backups, plan files, credentials, private keys, or real `.tfvars` files.
- Avoid manually editing or deleting state objects or lock records.
- Keep the bootstrap configuration's own state protected; do not delete the backend bucket while it stores state you still need.
- Review plans before applying changes and avoid concurrent Terraform operations against the same state.

## Common issues

### AWS credentials are not found

```bash
aws sts get-caller-identity
```

If it fails, fix the credential/profile setup before retrying Terraform.

### Bucket name is already taken

S3 bucket names are globally unique. Choose another name and update the configuration before applying.

### Backend bucket does not exist

The backend bucket must be created before running `terraform init` for the project that uses it.

### Backend configuration changed

After changing backend settings, run:

```bash
terraform init -reconfigure
```

Use `-reconfigure` to use new backend settings without migrating existing state. If you intend to move existing local state, use `terraform init -migrate-state` instead. Read the prompts carefully.

### Lock acquisition fails

Check whether another Terraform operation is still running and confirm the locking mechanism and IAM permissions. Do not delete a lock or lock record while another operation may be active.

## Cleanup

Backend resources should not be destroyed casually. First confirm no Terraform project still depends on the bucket or lock table and that any state you need has been backed up or migrated.

If this is a temporary lab, inspect the plan before destroying resources. The S3 bucket may contain state objects, and deleting it can make infrastructure harder to manage safely.

## What I learned

This exercise helped me understand why Terraform state matters, how remote state supports team workflows, how S3 stores state, how locking reduces the risk of concurrent changes, and why backend setup and state migration must be handled carefully.

## Status

Update this checklist to reflect what has actually been tested:

- [ ] Terraform configuration formatted and validated.
- [ ] S3 bucket created and verified.
- [ ] DynamoDB lock table created and verified (legacy exercise).
- [ ] Application backend configured.
- [ ] State migration completed and verified.
- [ ] Remote `terraform plan` reviewed successfully.
