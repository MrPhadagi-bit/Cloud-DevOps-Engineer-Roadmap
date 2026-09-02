<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Threat Detection with Amazon GuardDuty

**Project Link:** [View Project](http://nextwork.ai/projects/aws-security-guardduty)

**Author:** Phadagi Mannda Raven  
**Email:** ecommercesraven@gmail.com

---

![Project Overview](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_v1w2x3y4)

---

## Introduction

### Objective

This project demonstrates **threat detection and security monitoring** using **Amazon GuardDuty**, AWS's intelligent threat detection service. The workflow covers:

1. Deploying a deliberately vulnerable web application (**OWASP Juice Shop**) via CloudFormation to generate realistic attack surfaces.
2. Executing controlled offensive security techniques — SQL injection, command injection, and credential exfiltration — to simulate adversarial behavior.
3. Analyzing GuardDuty findings in near real-time to validate detection coverage for identity compromise, data exfiltration, and malware.
4. Enabling **GuardDuty Malware Protection for S3** and validating detection efficacy using the EICAR test vector.

### Tools and Concepts

| Component | Description |
|-----------|-------------|
| **Amazon GuardDuty** | Continuous security monitoring and threat detection service powered by ML and anomaly detection |
| **AWS CloudFormation** | Infrastructure-as-Code service for provisioning the vulnerable lab environment |
| **AWS CloudShell** | Browser-based shell environment for executing post-exploitation commands |
| **OWASP Juice Shop** | Intentionally insecure modern web application for security training |
| **SQL Injection (SQLi)** | Injection attack that manipulates backend database queries via unsanitized input |
| **Command Injection** | Attack that executes arbitrary OS commands on the host via unsanitized input |
| **Credential Exfiltration** | Unauthorized extraction and use of IAM credentials outside their intended scope |
| **EICAR Test File** | Standard anti-malware test file used to validate detection without causing harm |

### Project Metrics

- **Total Duration:** ~1.5 hours
- **GuardDuty Detection Latency:** ~16 minutes from attack execution to finding generation
- **Most Challenging Aspect:** Understanding the multi-step kill chain from web-app vulnerability → host compromise → credential exfiltration → S3 data access
- **Most Rewarding Aspect:** Observing GuardDuty correlate anomalous API calls with stolen EC2 instance credentials and surface a high-severity finding

---

## Lab Environment Setup

### Architecture

The lab environment was provisioned via CloudFormation and consists of three primary components:

| Component | Purpose |
|-----------|---------|
| **OWASP Juice Shop (EC2)** | Vulnerable web application serving as the attack target |
| **Amazon S3 Bucket** | Object storage target for post-compromise data exfiltration testing |
| **Amazon GuardDuty** | Threat detection and continuous monitoring across the AWS account |

### GuardDuty Fundamentals

Amazon GuardDuty is a **managed threat detection service** that analyzes CloudTrail event logs, VPC Flow Logs, and DNS logs using machine learning, anomaly detection, and integrated threat intelligence. It operates independently of user workloads — no agents or sensors are deployed on compute resources.

When GuardDuty identifies suspicious activity, it generates a **finding** — a structured JSON document containing:

- **Finding Type** — MITRE ATT&CK-aligned classification (e.g., `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS`)
- **Severity** — Score from 1.0 (low) to 8.9 (critical)
- **Resource** — Affected AWS resource (EC2 instance, IAM user, S3 bucket)
- **Actor** — IP address, ASN, and geolocation of the threat actor
- **Action** — API operation observed (e.g., `GetObject`, `ListBuckets`)

![Lab Setup](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_n1o2p3q4)

---

## Phase 1: SQL Injection

### Attack Vector

**SQL Injection (SQLi)** is an injection flaw that allows an attacker to interfere with the queries an application makes to its database. It is classified under **MITRE ATT&CK Technique T1190** (Exploit Public-Facing Application).

### Payload

The following payload was injected into the login form's email field:

```sql
' or 1=1;--
```

**Mechanism:** The payload terminates the original SQL query, appends a tautological condition (`1=1`), and comments out the remainder of the statement. This forces the authentication query to evaluate to `TRUE`, bypassing credential validation.

**Impact:** Unauthorized authentication, potential data exfiltration, and privilege escalation within the application context.

![SQL Injection](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_h1i2j3k4)

---

## Phase 2: Command Injection

### Attack Vector

**Command Injection** occurs when an application passes unsanitized user input to a system shell. It is classified under **MITRE ATT&CK Technique T1059** (Command and Scripting Interpreter).

### Exploitation

The Juice Shop application was vulnerable to command injection due to insufficient input validation on a server-side processing endpoint. Arbitrary system commands were executed on the underlying EC2 host, enabling:

- File system enumeration
- Process inspection
- Credential harvesting from instance metadata or configuration files

![Command Injection](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_t3u4v5w6)

---

## Phase 3: Credential Exfiltration

### Discovery

Following successful host compromise, a publicly exposed credentials file was discovered on the web server. The file contained **AWS access keys** belonging to the EC2 instance's attached IAM role, granting programmatic access to the developer's AWS environment.

**Risk:** These long-term or temporary credentials could be used by any external actor to assume the same level of access as the compromised instance.

![Exposed Credentials](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_x7y8z9a0)

### Post-Exploitation via AWS CloudShell

The exfiltrated credentials were operationalized using **AWS CloudShell** as the command-and-control medium:

1. **Download:** The credentials file was retrieved using `wget`.
2. **Inspection:** The JSON payload was parsed and formatted using `cat` and `jq` for readability.
3. **Profile Configuration:** A new AWS CLI profile was created to isolate the stolen credentials:
   ```bash
   aws configure --profile compromised
   # Access Key ID:     <exfiltrated_access_key>
   # Secret Access Key: <exfiltrated_secret_key>
   ```
4. **Lateral Movement:** The profile was used to invoke AWS APIs (e.g., `s3:GetObject`) from an unauthorized context.

![CloudShell Exfiltration](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_j9k0l1m2)

---

## GuardDuty Findings Analysis

### Detection Timeline

GuardDuty generated a finding approximately **16 minutes** after the credential exfiltration and anomalous API usage began.

### Finding: UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS

| Attribute | Value |
|-----------|-------|
| **Finding Type** | `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS` |
| **Detection Method** | Anomaly detection + machine learning |
| **Severity** | High (7.2 – 8.9) |
| **Resource** | EC2 instance whose credentials were compromised |
| **Action** | `GetObject` against an S3 bucket |
| **Actor** | External IP address and geolocation of the attacker |

**Interpretation:** GuardDuty detected that credentials associated with an EC2 instance were being used from an unusual network location and to access resources (S3) in a manner inconsistent with the instance's baseline behavior. This indicates credential compromise and potential data exfiltration.

![GuardDuty Finding](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_v1w2x3y4)

---

## Extension: Malware Protection for S3

### Overview

**GuardDuty Malware Protection for S3** automatically scans objects uploaded to S3 buckets for malicious content. It integrates with the GuardDuty detector and does not require additional infrastructure or agents.

### Test Methodology

The **EICAR (European Institute for Computer Antivirus Research) test file** was uploaded to a protected S3 bucket. The EICAR file is a standardized, non-malicious string designed to trigger antivirus and anti-malware engines:

```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

### Detection Result

GuardDuty instantaneously generated the following finding:

| Attribute | Value |
|-----------|-------|
| **Finding Type** | `Object:S3/MaliciousFile` |
| **Threat Type** | `EICAR-Test-File` |
| **Severity** | Low (indicates test file, not active malware) |
| **Resource** | S3 object key and bucket ARN |

**Conclusion:** Malware Protection for S3 successfully identified and flagged the test object, validating the detection pipeline.

![Malware Detection](http://nextwork.ai/gleeful_navy_kind_rabbit/uploads/aws-security-guardduty_sm42x3y4)

---

## Infrastructure as Code

### Overview

The following sections provide declarative infrastructure definitions for reproducing this GuardDuty lab using **AWS CloudFormation** and **HashiCorp Terraform**. Both templates provision:

- An Amazon GuardDuty detector (enabled by default)
- GuardDuty Malware Protection for S3
- An S3 bucket for exfiltration targets
- An IAM instance profile for the vulnerable EC2 application
- A VPC with public subnet for the OWASP Juice Shop deployment

> **Note:** The OWASP Juice Shop application itself is typically deployed via a pre-built CloudFormation template or container image. The snippets below provision the supporting AWS infrastructure and GuardDuty configuration.

---

### CloudFormation (YAML)

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  GuardDuty threat detection lab: VPC, S3 bucket, IAM instance profile,
  GuardDuty detector, and Malware Protection for S3.

Parameters:
  LabPrefix:
    Type: String
    Default: guardduty-lab
    Description: Prefix for all resource names.
  VpcCidr:
    Type: String
    Default: 10.0.0.0/16
  PublicSubnetCidr:
    Type: String
    Default: 10.0.1.0/24

Resources:
  # ------------------------------------------------------------------
  # Networking
  # ------------------------------------------------------------------
  LabVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub "${LabPrefix}-vpc"

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: !Ref PublicSubnetCidr
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs ""]
      Tags:
        - Key: Name
          Value: !Sub "${LabPrefix}-public-subnet"

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub "${LabPrefix}-igw"

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref LabVPC
      InternetGatewayId: !Ref InternetGateway

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref LabVPC
      Tags:
        - Key: Name
          Value: !Sub "${LabPrefix}-public-rt"

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # ------------------------------------------------------------------
  # S3 Bucket (Exfiltration Target)
  # ------------------------------------------------------------------
  ExfilBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${LabPrefix}-exfil-${AWS::AccountId}"
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Project
          Value: GuardDuty-Lab

  # ------------------------------------------------------------------
  # IAM Instance Profile for Vulnerable EC2
  # ------------------------------------------------------------------
  JuiceShopInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${LabPrefix}-juiceshop-role"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
      Tags:
        - Key: Project
          Value: GuardDuty-Lab

  JuiceShopInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: !Sub "${LabPrefix}-juiceshop-profile"
      Roles:
        - !Ref JuiceShopInstanceRole

  # ------------------------------------------------------------------
  # Security Group
  # ------------------------------------------------------------------
  JuiceShopSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: !Sub "${LabPrefix}-juiceshop-sg"
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref LabVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub "${LabPrefix}-juiceshop-sg"

  # ------------------------------------------------------------------
  # GuardDuty Detector
  # ------------------------------------------------------------------
  GuardDutyDetector:
    Type: AWS::GuardDuty::Detector
    Properties:
      Enable: true
      FindingPublishingFrequency: FIFTEEN_MINUTES
      DataSources:
        S3Logs:
          Enable: true
        Kubernetes:
          AuditLogs:
            Enable: false
        MalwareProtection:
          ScanEc2InstanceWithFindings:
            Enable: true

  # ------------------------------------------------------------------
  # GuardDuty Malware Protection for S3
  # ------------------------------------------------------------------
  GuardDutyMalwareProtectionPlan:
    Type: AWS::GuardDuty::MalwareProtectionPlan
    Properties:
      Role: !GetAtt GuardDutyMalwareProtectionRole.Arn
      ProtectedResource:
        S3Bucket:
          BucketName: !Ref ExfilBucket
      Actions:
        - Tagging
      Tags:
        - Key: Project
          Value: GuardDuty-Lab

  GuardDutyMalwareProtectionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${LabPrefix}-guardduty-malware-role"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: malware-protection-plan.guardduty.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonGuardDutyMalwareProtectionPolicy
      Tags:
        - Key: Project
          Value: GuardDuty-Lab

Outputs:
  VpcId:
    Description: Lab VPC ID
    Value: !Ref LabVPC
  PublicSubnetId:
    Description: Public subnet ID
    Value: !Ref PublicSubnet
  ExfilBucketName:
    Description: S3 bucket for exfiltration testing
    Value: !Ref ExfilBucket
  InstanceProfileArn:
    Description: IAM instance profile for Juice Shop EC2
    Value: !GetAtt JuiceShopInstanceProfile.Arn
  GuardDutyDetectorId:
    Description: GuardDuty detector ID
    Value: !Ref GuardDutyDetector
```

**Deployment:**

```bash
aws cloudformation deploy \
  --template-file guardduty-lab.yaml \
  --stack-name guardduty-lab-stack \
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

variable "lab_prefix" {
  description = "Prefix for all resource names"
  type        = string
  default     = "guardduty-lab"
}

variable "vpc_cidr" {
  description = "CIDR block for the lab VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidr" {
  description = "CIDR block for the public subnet"
  type        = string
  default     = "10.0.1.0/24"
}

# ------------------------------------------------------------------
# Data Sources
# ------------------------------------------------------------------
data "aws_caller_identity" "current" {}
data "aws_partition" "current" {}
data "aws_availability_zones" "available" {
  state = "available"
}

# ------------------------------------------------------------------
# Networking
# ------------------------------------------------------------------
resource "aws_vpc" "lab_vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.lab_prefix}-vpc"
  }
}

resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.lab_vpc.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[0]

  tags = {
    Name = "${var.lab_prefix}-public-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.lab_vpc.id

  tags = {
    Name = "${var.lab_prefix}-igw"
  }
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.lab_vpc.id

  tags = {
    Name = "${var.lab_prefix}-public-rt"
  }
}

resource "aws_route" "public_internet" {
  route_table_id         = aws_route_table.public_rt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

# ------------------------------------------------------------------
# S3 Bucket (Exfiltration Target)
# ------------------------------------------------------------------
resource "aws_s3_bucket" "exfil_bucket" {
  bucket = "${var.lab_prefix}-exfil-${data.aws_caller_identity.current.account_id}"

  tags = {
    Project = "GuardDuty-Lab"
  }
}

resource "aws_s3_bucket_public_access_block" "exfil_bucket_pab" {
  bucket = aws_s3_bucket.exfil_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "exfil_bucket_sse" {
  bucket = aws_s3_bucket.exfil_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_versioning" "exfil_bucket_versioning" {
  bucket = aws_s3_bucket.exfil_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

# ------------------------------------------------------------------
# IAM Instance Profile for Vulnerable EC2
# ------------------------------------------------------------------
resource "aws_iam_role" "juiceshop_role" {
  name = "${var.lab_prefix}-juiceshop-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  managed_policy_arns = [
    "arn:${data.aws_partition.current.partition}:iam::aws:policy/AmazonS3ReadOnlyAccess"
  ]

  tags = {
    Project = "GuardDuty-Lab"
  }
}

resource "aws_iam_instance_profile" "juiceshop_profile" {
  name = "${var.lab_prefix}-juiceshop-profile"
  role = aws_iam_role.juiceshop_role.name
}

# ------------------------------------------------------------------
# Security Group
# ------------------------------------------------------------------
resource "aws_security_group" "juiceshop_sg" {
  name        = "${var.lab_prefix}-juiceshop-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = aws_vpc.lab_vpc.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.lab_prefix}-juiceshop-sg"
  }
}

# ------------------------------------------------------------------
# GuardDuty Detector
# ------------------------------------------------------------------
resource "aws_guardduty_detector" "main" {
  enable                       = true
  finding_publishing_frequency = "FIFTEEN_MINUTES"

  datasources {
    s3_logs {
      enable = true
    }
    kubernetes {
      audit_logs {
        enable = false
      }
    }
    malware_protection {
      scan_ec2_instance_with_findings {
        enable = true
      }
    }
  }
}

# ------------------------------------------------------------------
# GuardDuty Malware Protection for S3
# ------------------------------------------------------------------
resource "aws_iam_role" "guardduty_malware_role" {
  name = "${var.lab_prefix}-guardduty-malware-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "malware-protection-plan.guardduty.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  managed_policy_arns = [
    "arn:${data.aws_partition.current.partition}:iam::aws:policy/AmazonGuardDutyMalwareProtectionPolicy"
  ]

  tags = {
    Project = "GuardDuty-Lab"
  }
}

resource "aws_guardduty_malware_protection_plan" "s3_protection" {
  role = aws_iam_role.guardduty_malware_role.arn

  protected_resource {
    s3_bucket {
      bucket_name = aws_s3_bucket.exfil_bucket.bucket
    }
  }

  actions {
    tagging {
      status = "ENABLED"
    }
  }
}

# ------------------------------------------------------------------
# Outputs
# ------------------------------------------------------------------
output "vpc_id" {
  description = "Lab VPC ID"
  value       = aws_vpc.lab_vpc.id
}

output "public_subnet_id" {
  description = "Public subnet ID"
  value       = aws_subnet.public_subnet.id
}

output "exfil_bucket_name" {
  description = "S3 bucket for exfiltration testing"
  value       = aws_s3_bucket.exfil_bucket.bucket
}

output "instance_profile_arn" {
  description = "IAM instance profile for Juice Shop EC2"
  value       = aws_iam_instance_profile.juiceshop_profile.arn
}

output "guardduty_detector_id" {
  description = "GuardDuty detector ID"
  value       = aws_guardduty_detector.main.id
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
| Create VPC | `AWS::EC2::VPC` | `aws_vpc` |
| Create public subnet | `AWS::EC2::Subnet` | `aws_subnet` |
| Create internet gateway | `AWS::EC2::InternetGateway` | `aws_internet_gateway` |
| Configure route table | `AWS::EC2::RouteTable` / `Route` | `aws_route_table` / `aws_route` |
| Create S3 bucket | `AWS::S3::Bucket` | `aws_s3_bucket` |
| Enable S3 SSE | `BucketEncryption` (nested) | `aws_s3_bucket_server_side_encryption_configuration` |
| Create IAM role for EC2 | `AWS::IAM::Role` | `aws_iam_role` |
| Create instance profile | `AWS::IAM::InstanceProfile` | `aws_iam_instance_profile` |
| Create security group | `AWS::EC2::SecurityGroup` | `aws_security_group` |
| Enable GuardDuty detector | `AWS::GuardDuty::Detector` | `aws_guardduty_detector` |
| Enable Malware Protection for S3 | `AWS::GuardDuty::MalwareProtectionPlan` | `aws_guardduty_malware_protection_plan` |
| Create IAM role for malware scanning | `AWS::IAM::Role` | `aws_iam_role` |

---

## Kill Chain Summary

| Phase | Tactic | Technique | GuardDuty Coverage |
|-------|--------|-----------|-------------------|
| Initial Access | Exploit Public-Facing App | T1190 — SQL Injection | Indirect (via CloudTrail anomaly) |
| Execution | Command and Scripting Interpreter | T1059 — Command Injection | Indirect (via CloudTrail anomaly) |
| Credential Access | Unsecured Credentials | T1552 — Credentials in Files | `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS` |
| Exfiltration | Exfiltration Over Web Service | T1567 — Exfiltration to Cloud Storage | `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS` |
| Impact | Data Encrypted for Impact | N/A (test vector) | `Object:S3/MaliciousFile` |

---

## Summary

| Phase | Key Action | Outcome |
|-------|-----------|---------|
| Lab Setup | CloudFormation deployment of vulnerable app + GuardDuty | Functional threat detection environment |
| Attack Execution | SQLi → Command Injection → Credential Exfiltration | Successful compromise and lateral movement |
| Detection | GuardDuty finding generation (~16 min) | High-severity credential exfiltration finding surfaced |
| Extension | Malware Protection for S3 + EICAR test | `Object:S3/MaliciousFile` finding validated |
| IaC | CloudFormation + Terraform templates | Reproducible, version-controlled lab infrastructure |

---
