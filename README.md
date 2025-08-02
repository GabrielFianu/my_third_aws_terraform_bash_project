Secure EC2 Auto Scaling Architecture with S3 Backup (Terraform)

Overview

This project provisions a secure EC2 Auto Scaling architecture with S3 backup using Terraform.

Files

- main.tf: Main Terraform configuration
- variables.tf: Input variables
- outputs.tf: Output values
- provider.tf: AWS provider configuration
- user-data.sh: EC2 instance user data script

Usage

1. Initialize Terraform: terraform init
2. Apply the Terraform configuration: terraform apply

Security Features

- Secure EC2 instances and S3 bucket configuration
- IAM roles for EC2 instances
- S3 bucket encryption

License

MIT License. See LICENSE for details
