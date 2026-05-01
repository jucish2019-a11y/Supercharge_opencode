---
name: devops-iac
description: Infrastructure as Code — Terraform, Pulumi, environment provisioning, secrets management, and CI/CD for infrastructure
---

## What I do

I implement Infrastructure as Code (IaC) practices:

- **Terraform** — Resource provisioning, state management, modules
- **Pulumi** — TypeScript/Python-based infrastructure definitions
- **Environment management** — Dev, staging, production parity
- **Secrets management** — Vault, AWS Secrets Manager, Azure Key Vault
- **CI/CD for IaC** — Automated plan/apply, drift detection
- **Cost management** — Resource tagging, cost estimation, cleanup

## When to use me

Use this skill when:
- Provisioning cloud infrastructure (AWS, GCP, Azure)
- Setting up environments for a new project
- Managing secrets and configuration across environments
- Automating infrastructure deployment via CI/CD
- Detecting and remediating infrastructure drift
- Implementing disaster recovery and backup strategies

## Terraform patterns

### Project structure

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── database/
│   └── compute/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── production/
└── backend.tf
```

### Module example

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.project_name}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.cidr_block, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name        = "${var.project_name}-public-${count.index + 1}"
    Environment = var.environment
    Type        = "public"
  }
}

# modules/vpc/variables.tf
variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}
```

### Environment configuration

```hcl
# environments/production/main.tf
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "terraform"
    }
  }
}

module "vpc" {
  source = "../../modules/vpc"

  project_name       = var.project_name
  environment        = var.environment
  cidr_block         = var.vpc_cidr
  availability_zones = var.availability_zones
}

module "database" {
  source = "../../modules/database"

  project_name = var.project_name
  environment  = var.environment
  vpc_id       = module.vpc.vpc_id
  subnet_ids   = module.vpc.private_subnet_ids
}
```

## Secrets management

### AWS Secrets Manager

```hcl
resource "aws_secretsmanager_secret" "db_password" {
  name        = "${var.project_name}/${var.environment}/db-password"
  description = "Database password for ${var.environment}"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}

resource "random_password" "db_password" {
  length  = 32
  special = false
}
```

### HashiCorp Vault

```hcl
resource "vault_mount" "secrets" {
  path        = "secret"
  type        = "kv-v2"
  description = "Key-Value secrets store"
}

resource "vault_kv_secret_v2" "database" {
  mount = vault_mount.secrets.path
  name  = "${var.project_name}/${var.environment}/database"

  data_json = jsonencode({
    host     = module.database.endpoint
    username = module.database.username
    password = module.database.password
    database = var.project_name
  })
}
```

## CI/CD for IaC

### GitHub Actions workflow

```yaml
name: Terraform

on:
  push:
    branches: [main]
    paths: ['terraform/**']
  pull_request:
    paths: ['terraform/**']

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "1.7.0"

      - name: Terraform Format
        run: terraform fmt -check -recursive

      - name: Terraform Init
        working-directory: terraform/environments/production
        run: terraform init
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Terraform Plan
        working-directory: terraform/environments/production
        run: terraform plan -out=tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Upload Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: terraform/environments/production/tfplan

  apply:
    needs: plan
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3

      - name: Download Plan
        uses: actions/download-artifact@v4
        with:
          name: tfplan
          path: terraform/environments/production

      - name: Terraform Apply
        working-directory: terraform/environments/production
        run: terraform apply -auto-approve tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Quality checklist

- [ ] Terraform state stored remotely (S3, GCS, Azure Storage) with locking
- [ ] Modules are reusable and well-documented
- [ ] Resources tagged for cost allocation
- [ ] Secrets never committed to repository — use Vault or cloud secret managers
- [ ] Terraform plan reviewed before apply in production
- [ ] Drift detection automated (terraform plan in CI)
- [ ] Backend configuration separate from resources
- [ ] Environment isolation (separate state files)
- [ ] Cost estimation run before major changes
- [ ] Rollback plan documented for each environment

## Anti-patterns I avoid

- Committing Terraform state files to version control
- Hardcoding secrets in Terraform files
- Running terraform apply locally on production
- Not using remote state with locking
- Giant monolithic Terraform configurations
- Not tagging resources for cost tracking
- Missing environment isolation
- Not running terraform plan before apply
- Using `terraform apply -auto-approve` without CI review
- Not versioning Terraform provider and module dependencies