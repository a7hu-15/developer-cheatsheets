# 🏗️ Terraform & Infrastructure as Code (IaC) Reference Cheatsheet

A production reference guide for Terraform, HCL (HashiCorp Configuration Language), State Management, Remote Backends, Terraform Lifecycle Commands, Modules, and Infrastructure Security Best Practices.

---

## 📋 Core HCL Syntax & Structure

### Provider & Resource Definitions

```hcl
# Provider configuration
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Data source for existing resources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# Managed Infrastructure Resource
resource "aws_instance" "web_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = {
    Name        = "${var.environment}-web-server"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

### Variables, Locals & Outputs

```hcl
# Input Variables with Validation
variable "environment" {
  type        = string
  description = "Deployment target environment"
  default     = "production"

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be one of: dev, staging, production."
  }
}

# Locals for transformed/derived values
locals {
  common_tags = {
    Project     = "CloudInfrastructure"
    Environment = var.environment
  }
  name_prefix = "${var.environment}-app"
}

# Output values
output "web_server_public_ip" {
  description = "Public IP address of the EC2 web server"
  value       = aws_instance.web_server.public_ip
}
```

---

## 🗄️ Remote State Management & Locking

### S3 Backend with DynamoDB State Locking

```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-tf-state-prod"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

### Essential State CLI Commands

```bash
# List all resources managed in the current state
terraform state list

# Show detailed attributes of a specific resource in state
terraform state show aws_instance.web_server

# Rename or move a resource in state (refactoring without destruction!)
terraform state mv aws_instance.old_name aws_instance.new_name

# Remove a resource from state tracking without deleting cloud resource
terraform state rm aws_instance.orphaned_server

# Pull remote state to local stdout
terraform state pull > state_backup.json
```

---

## ⚡ Terraform CLI Lifecycle & Command Workflow

| Command | Purpose | Key Flags |
|---|---|---|
| `terraform init` | Initialize backend, download providers & modules | `-upgrade`, `-reconfigure` |
| `terraform fmt` | Format HCL files according to canonical style | `-recursive`, `-check` |
| `terraform validate` | Validate structural syntax and parameter types | `-json` |
| `terraform plan` | Generate & show an execution plan | `-out=tfplan`, `-var-file=prod.tfvars`, `-destroy` |
| `terraform apply` | Apply changes to reach desired state | `tfplan`, `-auto-approve` |
| `terraform destroy` | Tear down all infrastructure managed by current config | `-target=aws_instance.web_server` |

---

## 🧩 Advanced HCL Patterns & Iteration

### `for_each` vs `count`

```hcl
# Preferred: for_each with map of objects (Key stability over index manipulation!)
variable "subnets" {
  type = map(string)
  default = {
    "us-east-1a" = "10.0.1.0/24"
    "us-east-1b" = "10.0.2.0/24"
  }
}

resource "aws_subnet" "public" {
  for_each          = var.subnets
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value
  availability_zone = each.key

  tags = {
    Name = "subnet-${each.key}"
  }
}
```

### Dynamic Blocks

```hcl
# Dynamically generate repeating nested security group rules
resource "aws_security_group" "web_sg" {
  name   = "web-ingress-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

---

## 🔒 Security Best Practices & Drift Detection

- 🔐 **Never hardcode secrets** in `.tf` files! Use AWS Secrets Manager, HashiCorp Vault, or environment variables (`TF_VAR_db_password`).
- 🛡️ Add `.tfstate`, `.tfstate.backup`, and `.tfvars` containing secrets to `.gitignore`.
- 🔎 **Drift Detection**: Run `terraform plan -refresh-only` to detect manual out-of-band changes made via Cloud Console.
- 🧪 **Security Scanners**: Run `tfsec` or `checkov` in CI/CD pipeline to catch overly permissive IAM roles or open S3 buckets prior to deployment.
