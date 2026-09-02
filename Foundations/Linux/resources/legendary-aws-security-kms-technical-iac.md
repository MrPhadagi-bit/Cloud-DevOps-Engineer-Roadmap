<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Encrypt Data with AWS KMS

**Project Link:** [View Project](http://nextwork.ai/projects/aws-security-kms)

**Author:** Phadagi Mannda Raven  
**Email:** ecommercesraven@gmail.com

---

![Project Overview](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_w0x1y2z3)

---

## Introduction

### Objective

This project demonstrates how to secure data at rest using **AWS Key Management Service (KMS)** in conjunction with **Amazon DynamoDB** and **AWS IAM**. The workflow covers:

1. Provisioning a customer-managed symmetric encryption key in AWS KMS.
2. Configuring a DynamoDB table to use the CMK for server-side encryption (SSE).
3. Validating access control boundaries via IAM — distinguishing between resource-level permissions (DynamoDB) and data-level permissions (KMS decrypt).
4. Implementing defense-in-depth by layering encryption with identity-based access controls.

### Tools and Concepts

| Component | Description |
|-----------|-------------|
| **AWS KMS** | Managed service for creating and controlling encryption keys |
| **Amazon DynamoDB** | Fully managed NoSQL database with native SSE support |
| **AWS IAM** | Identity and access management for users, roles, and policies |
| **Customer-Managed Key (CMK)** | User-created key with full control over key policy and rotation |
| **Transparent Data Encryption (TDE)** | Automatic encryption/decryption handled by the service layer |
| **Defense in Depth** | Security strategy combining multiple independent control layers |

### Project Metrics

- **Total Duration:** ~1.5 hours (including demonstration)
- **Most Challenging Aspect:** Understanding the distinction between key visibility and cryptographic action permissions
- **Most Rewarding Aspect:** Observing the test user regain data access after KMS key policy modification

---

## Encryption Fundamentals

### What Is Encryption?

**Encryption** is the process of transforming plaintext data into ciphertext using a cryptographic algorithm and a key. It ensures data confidentiality by rendering content unreadable to unauthorized parties.

**Encryption Keys** are secret values that parameterize the encryption algorithm. Key management is critical: unauthorized access to a key effectively equals unauthorized access to all data encrypted under that key.

### AWS KMS

AWS KMS is a managed service that provides centralized key lifecycle management, including creation, rotation, disabling, and deletion. KMS integrates natively with over 100 AWS services.

### Key Types

| Type | Use Case | Characteristics |
|------|----------|-----------------|
| **Symmetric** | Single key for encrypt and decrypt operations | AES-256; default for most AWS services |
| **Asymmetric** | Separate public/private key pair | RSA or ECC; used for signing and encryption |

For this project, a **symmetric CMK** was selected because DynamoDB SSE requires a single key for both encryption and decryption operations.

![KMS Console](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_a2b3c4d5)

---

## DynamoDB Server-Side Encryption

### Encryption Options

DynamoDB supports three server-side encryption configurations:

| Option | Key Ownership | Management Overhead | Use Case |
|--------|--------------|---------------------|----------|
| **DynamoDB Owned** | AWS (service account) | None | Default; no customer control |
| **AWS Managed Key** | AWS (`aws/dynamodb`) | Low | Customer visibility; AWS manages rotation |
| **Customer-Managed Key (CMK)** | Customer | High | Full control over key policy, rotation, and audit |

**Selection:** Customer-Managed Key (CMK) to demonstrate explicit key policy governance and granular access control.

### Transparent Data Encryption (TDE)

When a DynamoDB table is encrypted with a CMK, the service handles all cryptographic operations transparently:

- **Write path:** DynamoDB encrypts data using the CMK before persisting to disk.
- **Read path:** DynamoDB decrypts data automatically when an authorized request is received.
- **Client impact:** Zero application-level code changes required.

![DynamoDB Encryption](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_q8r9s0t1)

---

## Data Visibility and Access Control

### KMS Permission Model

KMS enforces access through **key policies** and **IAM policies** that authorize specific **cryptographic actions**, not merely key discovery. The following actions are relevant:

- `kms:Encrypt`
- `kms:Decrypt`
- `kms:ReEncrypt*`
- `kms:GenerateDataKey`
- `kms:DescribeKey`

**Key Insight:** Granting a principal `kms:DescribeKey` (visibility) without `kms:Decrypt` does not confer access to encrypted data. The cryptographic action permission is the control gate.

### Authorized User Behavior

Despite the DynamoDB table being encrypted with a CMK, the root/admin user retained full data visibility because:

1. The user held `kms:Decrypt` on the CMK via the key policy.
2. DynamoDB invoked KMS on the user's behalf during read operations.
3. TDE abstracted the encryption layer entirely from the user experience.

![Data Visibility](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_c0d1e2f3)

---

## Denying Access: Validation

### Test User Configuration

An IAM test user (`nextwork-kms-user`) was provisioned with the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:*",
      "Resource": "*"
    }
  ]
}
```

**Explicitly omitted:** Any KMS permissions, including `kms:Decrypt`.

### Observed Behavior

When the test user attempted to read items from the encrypted DynamoDB table, the request failed with:

> **AccessDeniedException:** User is not authorized to perform `kms:Decrypt` on resource `arn:aws:kms:...`

**Root Cause:** DynamoDB's TDE architecture requires KMS decryption on every read. Without `kms:Decrypt`, the service layer cannot return plaintext to the caller, even though DynamoDB-level permissions are unrestricted.

![Access Denied](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_w0x1y2z3)

---

## Granting Access: Key Policy Modification

### Key User Authorization

To restore access, the test user was added as a **Key User** in the KMS console. This operation updated the CMK's key policy to include:

```json
{
  "Sid": "Allow test user decrypt",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT_ID:user/nextwork-kms-user"
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

### Validation

After policy propagation, the test user successfully retrieved plaintext items from DynamoDB. This confirmed that:

1. Encryption controls data access independently of resource-level IAM.
2. KMS key policies are the authoritative gate for cryptographic operations.

![Access Granted](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-kms_feffb2fb8)

---

## Defense in Depth

Encryption operates at the **data layer**, complementing — not replacing — resource-level controls such as:

- **Security Groups** (network layer)
- **IAM Policies** (identity layer)
- **VPC Endpoints** (network segmentation)
- **KMS Key Policies** (cryptographic layer)

By combining these controls, a multi-layered security posture is achieved: an attacker must compromise not only the resource access controls but also the encryption key to exfiltrate meaningful data.

---

## Infrastructure as Code

### Overview

The following sections provide declarative infrastructure definitions for reproducing this project using **AWS CloudFormation** and **HashiCorp Terraform**. Both templates provision:

- A symmetric customer-managed KMS key with an explicit key policy
- A DynamoDB table encrypted with the CMK
- An IAM test user with DynamoDB full access but no default KMS permissions
- A key policy update to demonstrate access restoration

---

### CloudFormation (YAML)

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  DynamoDB table encrypted with a customer-managed KMS key,
  plus an IAM test user to validate KMS access control boundaries.

Parameters:
  TableName:
    Type: String
    Default: EncryptedDemoTable
  TestUserName:
    Type: String
    Default: nextwork-kms-user

Resources:
  # ------------------------------------------------------------------
  # KMS Customer-Managed Symmetric Key
  # ------------------------------------------------------------------
  DynamoDBEncryptionKey:
    Type: AWS::KMS::Key
    Properties:
      Description: CMK for DynamoDB server-side encryption
      EnableKeyRotation: true
      KeyPolicy:
        Version: "2012-10-17"
        Statement:
          # Allow the account root full control (required)
          - Sid: Enable IAM User Permissions
            Effect: Allow
            Principal:
              AWS: !Sub "arn:aws:iam::${AWS::AccountId}:root"
            Action: "kms:*"
            Resource: "*"
          # Allow the test user cryptographic actions
          - Sid: AllowTestUserCryptographicActions
            Effect: Allow
            Principal:
              AWS: !GetAtt TestUser.Arn
            Action:
              - kms:Encrypt
              - kms:Decrypt
              - kms:ReEncrypt*
              - kms:GenerateDataKey*
              - kms:DescribeKey
            Resource: "*"
      Tags:
        - Key: Project
          Value: KMS-DynamoDB-Encryption

  DynamoDBEncryptionKeyAlias:
    Type: AWS::KMS::Alias
    Properties:
      AliasName: !Sub "alias/${TableName}-key"
      TargetKeyId: !Ref DynamoDBEncryptionKey

  # ------------------------------------------------------------------
  # DynamoDB Table with SSE using CMK
  # ------------------------------------------------------------------
  EncryptedTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Ref TableName
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: PK
          AttributeType: S
      KeySchema:
        - AttributeName: PK
          KeyType: HASH
      SSESpecification:
        SSEEnabled: true
        SSEType: KMS
        KMSMasterKeyId: !Ref DynamoDBEncryptionKey
      Tags:
        - Key: Project
          Value: KMS-DynamoDB-Encryption

  # ------------------------------------------------------------------
  # IAM Test User (no KMS permissions by default)
  # ------------------------------------------------------------------
  TestUser:
    Type: AWS::IAM::User
    Properties:
      UserName: !Ref TestUserName
      Tags:
        - Key: Project
          Value: KMS-DynamoDB-Encryption

  TestUserDynamoDBPolicy:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: DynamoDBFullAccessPolicy
      Users:
        - !Ref TestUser
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: DynamoDBFullAccess
            Effect: Allow
            Action: "dynamodb:*"
            Resource: "*"

Outputs:
  KeyArn:
    Description: ARN of the customer-managed KMS key
    Value: !GetAtt DynamoDBEncryptionKey.Arn
  TableArn:
    Description: ARN of the encrypted DynamoDB table
    Value: !GetAtt EncryptedTable.Arn
  TestUserArn:
    Description: ARN of the IAM test user
    Value: !GetAtt TestUser.Arn
```

**Deployment:**

```bash
aws cloudformation deploy \
  --template-file kms-dynamodb.yaml \
  --stack-name kms-dynamodb-stack \
  --capabilities CAPABILITY_NAMED_IAM
```

---

### Terraform (HCL)

```hcl
terraform {
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

# ------------------------------------------------------------------
# Variables
# ------------------------------------------------------------------
variable "aws_region" {
  description = "AWS region for resource deployment"
  type        = string
  default     = "us-east-1"
}

variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
  default     = "EncryptedDemoTable"
}

variable "test_user_name" {
  description = "Name of the IAM test user"
  type        = string
  default     = "nextwork-kms-user"
}

# ------------------------------------------------------------------
# Data Sources
# ------------------------------------------------------------------
data "aws_caller_identity" "current" {}
data "aws_partition" "current" {}

# ------------------------------------------------------------------
# KMS Customer-Managed Symmetric Key
# ------------------------------------------------------------------
resource "aws_kms_key" "dynamodb_encryption_key" {
  description             = "CMK for DynamoDB server-side encryption"
  deletion_window_in_days = 30
  enable_key_rotation     = true

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "Enable IAM User Permissions"
        Effect = "Allow"
        Principal = {
          AWS = "arn:${data.aws_partition.current.partition}:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action   = "kms:*"
        Resource = "*"
      },
      {
        Sid    = "AllowTestUserCryptographicActions"
        Effect = "Allow"
        Principal = {
          AWS = aws_iam_user.test_user.arn
        }
        Action = [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:ReEncrypt*",
          "kms:GenerateDataKey*",
          "kms:DescribeKey"
        ]
        Resource = "*"
      }
    ]
  })

  tags = {
    Project = "KMS-DynamoDB-Encryption"
  }
}

resource "aws_kms_alias" "dynamodb_encryption_key_alias" {
  name          = "alias/${var.table_name}-key"
  target_key_id = aws_kms_key.dynamodb_encryption_key.key_id
}

# ------------------------------------------------------------------
# DynamoDB Table with SSE using CMK
# ------------------------------------------------------------------
resource "aws_dynamodb_table" "encrypted_table" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "PK"

  attribute {
    name = "PK"
    type = "S"
  }

  server_side_encryption {
    enabled     = true
    kms_key_arn = aws_kms_key.dynamodb_encryption_key.arn
  }

  tags = {
    Project = "KMS-DynamoDB-Encryption"
  }
}

# ------------------------------------------------------------------
# IAM Test User
# ------------------------------------------------------------------
resource "aws_iam_user" "test_user" {
  name = var.test_user_name
  tags = {
    Project = "KMS-DynamoDB-Encryption"
  }
}

resource "aws_iam_user_policy" "test_user_dynamodb_policy" {
  name = "DynamoDBFullAccessPolicy"
  user = aws_iam_user.test_user.name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid      = "DynamoDBFullAccess"
        Effect   = "Allow"
        Action   = "dynamodb:*"
        Resource = "*"
      }
    ]
  })
}

# ------------------------------------------------------------------
# Outputs
# ------------------------------------------------------------------
output "key_arn" {
  description = "ARN of the customer-managed KMS key"
  value       = aws_kms_key.dynamodb_encryption_key.arn
}

output "table_arn" {
  description = "ARN of the encrypted DynamoDB table"
  value       = aws_dynamodb_table.encrypted_table.arn
}

output "test_user_arn" {
  description = "ARN of the IAM test user"
  value       = aws_iam_user.test_user.arn
}
```

**Deployment:**

```bash
terraform init
terraform plan
terraform apply
```

---

## IaC Resource Mapping

| Manual Console Step | CloudFormation Resource | Terraform Resource |
|---------------------|------------------------|--------------------|
| Create symmetric CMK | `AWS::KMS::Key` | `aws_kms_key` |
| Create key alias | `AWS::KMS::Alias` | `aws_kms_alias` |
| Create DynamoDB table with SSE | `AWS::DynamoDB::Table` | `aws_dynamodb_table` |
| Create IAM test user | `AWS::IAM::User` | `aws_iam_user` |
| Attach DynamoDB policy | `AWS::IAM::Policy` | `aws_iam_user_policy` |
| Update key policy for test user | Nested in `KeyPolicy` | Nested in `policy` argument |

---

## Summary

| Phase | Key Action | Outcome |
|-------|-----------|---------|
| Key Provisioning | Symmetric CMK created in KMS | Customer-controlled encryption key ready |
| Table Encryption | DynamoDB table configured with CMK SSE | Data encrypted at rest transparently |
| Access Denial | Test user with DynamoDB-only access | `AccessDeniedException` on KMS decrypt |
| Access Restoration | Test user added to CMK key policy | Successful plaintext data retrieval |
| IaC | CloudFormation + Terraform templates | Reproducible, version-controlled infrastructure |

---
