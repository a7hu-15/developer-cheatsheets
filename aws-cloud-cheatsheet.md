# AWS Cloud Architecture & CLI Reference Cheatsheet

A practical reference guide for Amazon Web Services (AWS) core concepts, architecture design patterns, IAM policies, and AWS CLI commands.

---

## 📋 Table of Contents
1. [Core Services Overview](#1-core-services-overview)
2. [Identity & Access Management (IAM)](#2-identity--access-management-iam)
3. [Networking & VPC Architecture](#3-networking--vpc-architecture)
4. [Storage & Database Operations](#4-storage--database-operations)
5. [AWS CLI Essential Commands](#5-aws-cli-essential-commands)
6. [Serverless & Container Architecture](#6-serverless--container-architecture)
7. [Security & Compliance Best Practices](#7-security--compliance-best-practices)

---

## 1. Core Services Overview

| Category | Service | Common Use Case |
|---|---|---|
| **Compute** | EC2, Lambda, ECS, EKS | Virtual servers, serverless functions, container orchestration |
| **Storage** | S3, EBS, EFS | Object storage, block storage for EC2, shared network storage |
| **Database** | RDS, DynamoDB, ElastiCache | Relational SQL, NoSQL key-value document store, Redis caching |
| **Networking** | VPC, CloudFront, Route 53 | Virtual isolated network, CDN edge distribution, DNS routing |
| **Security** | IAM, KMS, Secrets Manager | Authentication/RBAC, key management, credential storage |
| **Observability**| CloudWatch, X-Ray, CloudTrail | Metrics/logging, distributed tracing, API activity auditing |

---

## 2. Identity & Access Management (IAM)

### IAM Policy Structure (JSON)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadWriteLimited",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-data-bucket",
        "arn:aws:s3:::my-app-data-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "true"
        }
      }
    }
  ]
}
```

### Principle of Least Privilege Checklist
- Avoid using `Resource: "*"` with wildcard actions (`s3:*`).
- Assign permissions to **IAM Roles** and enforce role assumption instead of IAM Users.
- Enforce Multi-Factor Authentication (MFA) for high-privilege API operations.

---

## 3. Networking & VPC Architecture

### Standard Multi-AZ VPC Design
- **Public Subnet**: Internet Gateway (IGW) attached; contains ALB, NAT Gateway.
- **Private App Subnet**: Outbound internet traffic routed via NAT Gateway; contains EC2 / ECS / Lambda tasks.
- **Private Data Subnet**: Completely isolated; contains RDS databases & ElastiCache.

```
Internet -> Internet Gateway -> Public Subnet (ALB) -> Private Subnet (App Servers) -> Isolated Data Subnet (DB)
```

### Security Group vs Network ACL (NACL)
| Feature | Security Group (SG) | Network ACL (NACL) |
|---|---|---|
| **Level** | Instance / Interface level | Subnet level |
| **Stateful / Stateless**| Stateful (return traffic auto-allowed) | Stateless (inbound & outbound rules explicit) |
| **Evaluation** | All rules evaluated before decision | Rules evaluated in numerical order |
| **Deny Rules** | Allow rules only | Supports explicit Allow and Deny rules |

---

## 4. Storage & Database Operations

### S3 Storage Classes Quick Reference
- **S3 Standard**: High availability, low latency, general data storage.
- **S3 Intelligent-Tiering**: Automatic cost savings for data with unknown access patterns.
- **S3 Glacier Flexible / Deep Archive**: Long-term compliance archive ($0.00099/GB/month).

### DynamoDB Key Concepts
- **Partition Key (PK)**: Determines hash partition distribution.
- **Sort Key (SK)**: Enables range queries within a partition (`BeginsWith`, `Between`).
- **Global Secondary Index (GSI)**: Secondary index with alternative PK and SK for flexible queries.

---

## 5. AWS CLI Essential Commands

### Configuration & Identity
```bash
# Configure CLI credentials
aws configure

# Verify current authenticated identity
aws sts get-caller-identity

# Assume an IAM Role for session credentials
aws sts assume-role \
  --role-arn "arn:aws:iam::123456789012:role/DevAdminRole" \
  --role-session-name "CLI-Session"
```

### S3 Operations
```bash
# List buckets
aws s3 ls

# Sync local directory to S3 bucket recursively
aws s3 sync ./dist s3://my-web-app-bucket --delete

# Download file with server-side encryption
aws s3 cp s3://my-data-bucket/export.json ./export.json
```

### EC2 & VPC Operations
```bash
# List running instances with Name tag and IP address
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,Tags[?Key=='Name'].Value|[0]]" \
  --output table
```

---

## 6. Serverless & Container Architecture

### AWS Lambda Function Best Practices
- **Execution Context Reuse**: Declare database client connections and HTTP clients outside the handler function.
- **Environment Variables**: Store non-sensitive configuration; use AWS Secrets Manager for DB passwords.
- **Memory & Power Tuning**: Allocating higher memory increases CPU quota proportionally.

### ECS Task vs Service
- **Task Definition**: Blueprint defining container image, CPU/RAM limits, port mappings, log group.
- **ECS Service**: Maintains desired count of running tasks, handles auto-healing & ALB target registration.

---

## 7. Security & Compliance Best Practices

1. **Encryption at Rest & in Transit**: Use AWS KMS Customer Managed Keys (CMK) and enforce HTTPS TLS 1.3.
2. **Secrets Management**: Never commit hardcoded API keys; fetch dynamically from Secrets Manager / SSM Parameter Store.
3. **CloudTrail Auditing**: Enable multi-region CloudTrail logging to S3 with object lock enabled for immutability.
