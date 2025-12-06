# AWS Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [AWS CLI Setup](#aws-cli-setup)
- [IAM (Identity and Access Management)](#iam-identity-and-access-management)
- [EC2 (Elastic Compute Cloud)](#ec2-elastic-compute-cloud)
- [VPC (Virtual Private Cloud)](#vpc-virtual-private-cloud)
- [S3 (Simple Storage Service)](#s3-simple-storage-service)
- [RDS (Relational Database Service)](#rds-relational-database-service)
- [Lambda](#lambda)
- [ECS & Fargate](#ecs--fargate)
- [EKS (Elastic Kubernetes Service)](#eks-elastic-kubernetes-service)
- [CloudFormation](#cloudformation)
- [CloudWatch](#cloudwatch)
- [Route 53](#route-53)
- [Load Balancers](#load-balancers)
- [Auto Scaling](#auto-scaling)
- [SNS & SQS](#sns--sqs)
- [DynamoDB](#dynamodb)
- [ElastiCache](#elasticache)
- [CloudFront](#cloudfront)
- [API Gateway](#api-gateway)
- [Systems Manager (SSM)](#systems-manager-ssm)
- [Secrets Manager](#secrets-manager)
- [ECR (Elastic Container Registry)](#ecr-elastic-container-registry)
- [Cost Management](#cost-management)
- [Security Best Practices](#security-best-practices)
- [Multi-Account Strategy](#multi-account-strategy)

---

## Introduction

Amazon Web Services (AWS) is the world's most comprehensive and broadly adopted cloud platform, offering over 200 fully featured services from data centers globally.

### Core Services Overview

| Category | Services |
|----------|----------|
| Compute | EC2, Lambda, ECS, EKS, Fargate, Batch |
| Storage | S3, EBS, EFS, FSx, Storage Gateway |
| Database | RDS, DynamoDB, ElastiCache, Redshift, Aurora |
| Networking | VPC, Route 53, CloudFront, API Gateway, Direct Connect |
| Security | IAM, KMS, Secrets Manager, WAF, Shield, GuardDuty |
| Management | CloudFormation, CloudWatch, Systems Manager, Config |
| Analytics | Athena, EMR, Kinesis, Glue, QuickSight |
| DevOps | CodePipeline, CodeBuild, CodeDeploy, CodeCommit |

---

## AWS CLI Setup

### Installation

#### Linux
```bash
# Download and install
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

#### macOS
```bash
# Using Homebrew
brew install awscli

# Or download installer
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Verify
aws --version
```

#### Windows
```powershell
# Download and run MSI installer from:
# https://awscli.amazonaws.com/AWSCLIV2.msi

# Or using Chocolatey
choco install awscli

# Verify
aws --version
```

### Configuration

```bash
# Configure AWS CLI
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region name: us-east-1
# Default output format: json

# Configure named profile
aws configure --profile production
aws configure --profile staging

# List profiles
cat ~/.aws/config
cat ~/.aws/credentials

# Set default profile
export AWS_PROFILE=production

# Configure with SSO
aws configure sso
aws sso login --profile my-sso-profile

# Set specific configurations
aws configure set region us-west-2
aws configure set output json
aws configure set aws_access_key_id YOUR_KEY
aws configure set aws_secret_access_key YOUR_SECRET

# Use environment variables
export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
export AWS_DEFAULT_REGION=us-east-1
export AWS_DEFAULT_OUTPUT=json
```

### AWS CLI Common Commands

```bash
# Get caller identity
aws sts get-caller-identity

# List all regions
aws ec2 describe-regions --output table

# Get account information
aws iam get-user
aws organizations describe-organization

# Set output format
aws ec2 describe-instances --output json
aws ec2 describe-instances --output yaml
aws ec2 describe-instances --output text
aws ec2 describe-instances --output table

# Query and filter
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType]' \
  --output table

# Use JQ for JSON processing
aws ec2 describe-instances | jq '.Reservations[].Instances[].InstanceId'

# Dry run (test without executing)
aws ec2 run-instances --dry-run ...

# Generate CLI skeleton
aws ec2 run-instances --generate-cli-skeleton > template.json

# Use CLI skeleton
aws ec2 run-instances --cli-input-json file://template.json

# Pagination
aws s3api list-objects --bucket mybucket --max-items 100
aws s3api list-objects --bucket mybucket --page-size 100

# Debug mode
aws ec2 describe-instances --debug
```

---

## IAM (Identity and Access Management)

### Users

```bash
# Create user
aws iam create-user --user-name john

# List users
aws iam list-users

# Get user details
aws iam get-user --user-name john

# Delete user
aws iam delete-user --user-name john

# Create access key
aws iam create-access-key --user-name john

# List access keys
aws iam list-access-keys --user-name john

# Delete access key
aws iam delete-access-key --user-name john --access-key-id AKIAIOSFODNN7EXAMPLE

# Update access key
aws iam update-access-key --user-name john --access-key-id AKIAIOSFODNN7EXAMPLE --status Inactive

# Create login profile (console password)
aws iam create-login-profile --user-name john --password MyPassword123!

# Update login profile
aws iam update-login-profile --user-name john --password NewPassword123!

# Add user to group
aws iam add-user-to-group --user-name john --group-name Developers
```

### Groups

```bash
# Create group
aws iam create-group --group-name Developers

# List groups
aws iam list-groups

# Get group details
aws iam get-group --group-name Developers

# Delete group
aws iam delete-group --group-name Developers

# List users in group
aws iam get-group --group-name Developers

# Remove user from group
aws iam remove-user-from-group --user-name john --group-name Developers
```

### Roles

```bash
# Create role with trust policy
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role --role-name EC2-S3-Role --assume-role-policy-document file://trust-policy.json

# List roles
aws iam list-roles

# Get role details
aws iam get-role --role-name EC2-S3-Role

# Delete role
aws iam delete-role --role-name EC2-S3-Role

# Create instance profile
aws iam create-instance-profile --instance-profile-name EC2-S3-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile --instance-profile-name EC2-S3-Profile --role-name EC2-S3-Role

# Associate instance profile with EC2
aws ec2 associate-iam-instance-profile --instance-id i-1234567890abcdef0 --iam-instance-profile Name=EC2-S3-Profile
```

### Policies

```bash
# Create custom policy
cat > s3-read-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/*",
        "arn:aws:s3:::my-bucket"
      ]
    }
  ]
}
EOF

aws iam create-policy --policy-name S3ReadPolicy --policy-document file://s3-read-policy.json

# List policies
aws iam list-policies --scope Local
aws iam list-policies --scope AWS

# Get policy details
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy

# Get policy version
aws iam get-policy-version --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy --version-id v1

# Attach policy to user
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy

# Attach policy to group
aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy

# Attach policy to role
aws iam attach-role-policy --role-name EC2-S3-Role --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Detach policy
aws iam detach-user-policy --user-name john --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy

# List attached policies
aws iam list-attached-user-policies --user-name john
aws iam list-attached-group-policies --group-name Developers
aws iam list-attached-role-policies --role-name EC2-S3-Role

# Delete policy
aws iam delete-policy --policy-arn arn:aws:iam::123456789012:policy/S3ReadPolicy
```

### MFA

```bash
# Enable MFA for user
aws iam enable-mfa-device \
  --user-name john \
  --serial-number arn:aws:iam::123456789012:mfa/john \
  --authentication-code1 123456 \
  --authentication-code2 789012

# List MFA devices
aws iam list-mfa-devices --user-name john

# Deactivate MFA
aws iam deactivate-mfa-device --user-name john --serial-number arn:aws:iam::123456789012:mfa/john
```

---

## EC2 (Elastic Compute Cloud)

### Instance Management

```bash
# List instances
aws ec2 describe-instances

# List instances with filters
aws ec2 describe-instances \
  --filters "Name=instance-type,Values=t2.micro" "Name=instance-state-name,Values=running"

# List instances with specific output
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]' \
  --user-data file://user-data.sh

# Launch instance with block device mapping
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":30,"VolumeType":"gp3","DeleteOnTermination":true}}]'

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Get instance console output
aws ec2 get-console-output --instance-id i-1234567890abcdef0

# Modify instance attribute
aws ec2 modify-instance-attribute --instance-id i-1234567890abcdef0 --instance-type "{\"Value\":\"t2.small\"}"

# Enable termination protection
aws ec2 modify-instance-attribute --instance-id i-1234567890abcdef0 --disable-api-termination

# Disable termination protection
aws ec2 modify-instance-attribute --instance-id i-1234567890abcdef0 --no-disable-api-termination
```

### AMI Management

```bash
# List AMIs
aws ec2 describe-images --owners self

# List public Ubuntu AMIs
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*" \
  --query 'Images[*].[ImageId,Name,CreationDate]' \
  --output table | sort -k3 | tail -n 5

# Create AMI from instance
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "MyAMI-$(date +%Y%m%d-%H%M%S)" \
  --description "My custom AMI"

# Copy AMI to another region
aws ec2 copy-image \
  --source-region us-east-1 \
  --source-image-id ami-0c55b159cbfafe1f0 \
  --name "MyAMI-Copy" \
  --region us-west-2

# Deregister AMI
aws ec2 deregister-image --image-id ami-0c55b159cbfafe1f0

# Create snapshot
aws ec2 create-snapshot --volume-id vol-0123456789abcdef0 --description "My snapshot"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Delete snapshot
aws ec2 delete-snapshot --snapshot-id snap-0123456789abcdef0
```

### Key Pairs

```bash
# Create key pair
aws ec2 create-key-pair --key-name my-key-pair --query 'KeyMaterial' --output text > my-key-pair.pem
chmod 400 my-key-pair.pem

# List key pairs
aws ec2 describe-key-pairs

# Import key pair
aws ec2 import-key-pair --key-name my-imported-key --public-key-material fileb://~/.ssh/id_rsa.pub

# Delete key pair
aws ec2 delete-key-pair --key-name my-key-pair
```

### Security Groups

```bash
# Create security group
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --vpc-id vpc-0123456789abcdef0

# List security groups
aws ec2 describe-security-groups

# Add ingress rule (SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Add ingress rule (HTTP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Add ingress rule (HTTPS)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Add ingress rule from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-9876543210abcdef0

# Remove ingress rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Add egress rule
aws ec2 authorize-security-group-egress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Delete security group
aws ec2 delete-security-group --group-id sg-0123456789abcdef0
```

### Elastic IPs

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# List Elastic IPs
aws ec2 describe-addresses

# Associate Elastic IP with instance
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-0123456789abcdef0

# Disassociate Elastic IP
aws ec2 disassociate-address --association-id eipassoc-0123456789abcdef0

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-0123456789abcdef0
```

### EBS Volumes

```bash
# Create volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3 \
  --iops 3000 \
  --throughput 125

# List volumes
aws ec2 describe-volumes

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Detach volume
aws ec2 detach-volume --volume-id vol-0123456789abcdef0

# Delete volume
aws ec2 delete-volume --volume-id vol-0123456789abcdef0

# Modify volume
aws ec2 modify-volume --volume-id vol-0123456789abcdef0 --size 200 --volume-type gp3

# Create snapshot from volume
aws ec2 create-snapshot --volume-id vol-0123456789abcdef0 --description "My backup"

# Create volume from snapshot
aws ec2 create-volume \
  --snapshot-id snap-0123456789abcdef0 \
  --availability-zone us-east-1a \
  --volume-type gp3
```

---

## VPC (Virtual Private Cloud)

### VPC Management

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# List VPCs
aws ec2 describe-vpcs

# Modify VPC attributes
aws ec2 modify-vpc-attribute --vpc-id vpc-0123456789abcdef0 --enable-dns-hostnames

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-0123456789abcdef0

# Create tags
aws ec2 create-tags --resources vpc-0123456789abcdef0 --tags Key=Name,Value=MyVPC
```

### Subnets

```bash
# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# List subnets
aws ec2 describe-subnets

# Modify subnet to auto-assign public IP
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-0123456789abcdef0 \
  --map-public-ip-on-launch

# Delete subnet
aws ec2 delete-subnet --subnet-id subnet-0123456789abcdef0
```

### Internet Gateway

```bash
# Create internet gateway
aws ec2 create-internet-gateway

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0123456789abcdef0 \
  --vpc-id vpc-0123456789abcdef0

# Detach from VPC
aws ec2 detach-internet-gateway \
  --internet-gateway-id igw-0123456789abcdef0 \
  --vpc-id vpc-0123456789abcdef0

# Delete internet gateway
aws ec2 delete-internet-gateway --internet-gateway-id igw-0123456789abcdef0
```

### NAT Gateway

```bash
# Create NAT Gateway
aws ec2 create-nat-gateway \
  --subnet-id subnet-0123456789abcdef0 \
  --allocation-id eipalloc-0123456789abcdef0

# List NAT Gateways
aws ec2 describe-nat-gateways

# Delete NAT Gateway
aws ec2 delete-nat-gateway --nat-gateway-id nat-0123456789abcdef0
```

### Route Tables

```bash
# Create route table
aws ec2 create-route-table --vpc-id vpc-0123456789abcdef0

# List route tables
aws ec2 describe-route-tables

# Create route to internet gateway
aws ec2 create-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0123456789abcdef0

# Create route to NAT gateway
aws ec2 create-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-0123456789abcdef0

# Associate route table with subnet
aws ec2 associate-route-table \
  --route-table-id rtb-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0

# Delete route
aws ec2 delete-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0

# Delete route table
aws ec2 delete-route-table --route-table-id rtb-0123456789abcdef0
```

### VPC Peering

```bash
# Create VPC peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-0123456789abcdef0 \
  --peer-vpc-id vpc-9876543210abcdef0

# Accept VPC peering connection
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0123456789abcdef0

# List VPC peering connections
aws ec2 describe-vpc-peering-connections

# Delete VPC peering connection
aws ec2 delete-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0123456789abcdef0
```

### VPC Endpoints

```bash
# Create VPC endpoint for S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-0123456789abcdef0 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-0123456789abcdef0

# Create interface endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-0123456789abcdef0 \
  --service-name com.amazonaws.us-east-1.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-0123456789abcdef0 \
  --security-group-ids sg-0123456789abcdef0

# List VPC endpoints
aws ec2 describe-vpc-endpoints

# Delete VPC endpoint
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids vpce-0123456789abcdef0
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl --vpc-id vpc-0123456789abcdef0

# List network ACLs
aws ec2 describe-network-acls

# Create network ACL entry (allow inbound HTTP)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Delete network ACL entry
aws ec2 delete-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --rule-number 100 \
  --egress false

# Associate network ACL with subnet
aws ec2 replace-network-acl-association \
  --association-id aclassoc-0123456789abcdef0 \
  --network-acl-id acl-0123456789abcdef0

# Delete network ACL
aws ec2 delete-network-acl --network-acl-id acl-0123456789abcdef0
```

---

## S3 (Simple Storage Service)

### Bucket Management

```bash
# Create bucket
aws s3 mb s3://my-bucket-name

# Create bucket in specific region
aws s3 mb s3://my-bucket-name --region us-west-2

# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://my-bucket-name
aws s3 ls s3://my-bucket-name/path/ --recursive
aws s3 ls s3://my-bucket-name --recursive --human-readable --summarize

# Remove bucket (must be empty)
aws s3 rb s3://my-bucket-name

# Remove bucket and all contents
aws s3 rb s3://my-bucket-name --force
```

### Object Operations

```bash
# Upload file
aws s3 cp file.txt s3://my-bucket-name/

# Upload with metadata
aws s3 cp file.txt s3://my-bucket-name/ --metadata key1=value1,key2=value2

# Upload directory
aws s3 cp /local/path s3://my-bucket-name/path/ --recursive

# Upload with specific storage class
aws s3 cp file.txt s3://my-bucket-name/ --storage-class GLACIER

# Download file
aws s3 cp s3://my-bucket-name/file.txt ./

# Download directory
aws s3 cp s3://my-bucket-name/path/ /local/path/ --recursive

# Sync local to S3
aws s3 sync /local/path s3://my-bucket-name/path/

# Sync S3 to local
aws s3 sync s3://my-bucket-name/path/ /local/path/

# Sync with delete
aws s3 sync /local/path s3://my-bucket-name/path/ --delete

# Move file
aws s3 mv file.txt s3://my-bucket-name/
aws s3 mv s3://my-bucket-name/file.txt s3://my-bucket-name/newpath/

# Remove file
aws s3 rm s3://my-bucket-name/file.txt

# Remove directory
aws s3 rm s3://my-bucket-name/path/ --recursive

# Get object metadata
aws s3api head-object --bucket my-bucket-name --key file.txt

# Copy between buckets
aws s3 cp s3://source-bucket/file.txt s3://dest-bucket/
```

### Bucket Configuration

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled

# Suspend versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Suspended

# Get versioning status
aws s3api get-bucket-versioning --bucket my-bucket-name

# List object versions
aws s3api list-object-versions --bucket my-bucket-name

# Enable server-side encryption
aws s3api put-bucket-encryption \
  --bucket my-bucket-name \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Enable server-side encryption with KMS
aws s3api put-bucket-encryption \
  --bucket my-bucket-name \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
      }
    }]
  }'

# Block public access
aws s3api put-public-access-block \
  --bucket my-bucket-name \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable static website hosting
aws s3api put-bucket-website \
  --bucket my-bucket-name \
  --website-configuration '{
    "IndexDocument": {"Suffix": "index.html"},
    "ErrorDocument": {"Key": "error.html"}
  }'

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket-name \
  --lifecycle-configuration file://lifecycle.json
```

```json
// lifecycle.json
{
  "Rules": [
    {
      "Id": "Move to Glacier after 30 days",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

### Bucket Policy

```bash
# Set bucket policy
aws s3api put-bucket-policy --bucket my-bucket-name --policy file://policy.json

# Get bucket policy
aws s3api get-bucket-policy --bucket my-bucket-name

# Delete bucket policy
aws s3api delete-bucket-policy --bucket my-bucket-name
```

```json
// policy.json - Public read access
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket-name/*"
    }
  ]
}
```

### CORS Configuration

```bash
# Set CORS configuration
aws s3api put-bucket-cors --bucket my-bucket-name --cors-configuration file://cors.json

# Get CORS configuration
aws s3api get-bucket-cors --bucket my-bucket-name

# Delete CORS configuration
aws s3api delete-bucket-cors --bucket my-bucket-name
```

```json
// cors.json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedHeaders": ["*"],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

### Presigned URLs

```bash
# Generate presigned URL for upload (valid for 1 hour)
aws s3 presign s3://my-bucket-name/file.txt --expires-in 3600

# Generate presigned URL for download
aws s3 presign s3://my-bucket-name/file.txt

# Upload using presigned URL
curl -X PUT -T file.txt "PRESIGNED_URL"
```

---

## RDS (Relational Database Service)

### DB Instance Management

```bash
# Create DB instance (MySQL)
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password MyPassword123! \
  --allocated-storage 20 \
  --storage-type gp3 \
  --vpc-security-group-ids sg-0123456789abcdef0 \
  --db-subnet-group-name mydbsubnetgroup \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "mon:04:00-mon:05:00" \
  --storage-encrypted \
  --publicly-accessible

# Create DB instance (PostgreSQL)
aws rds create-db-instance \
  --db-instance-identifier mypostgres \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 15.3 \
  --master-username postgres \
  --master-user-password MyPassword123! \
  --allocated-storage 20

# List DB instances
aws rds describe-db-instances

# Get specific DB instance
aws rds describe-db-instances --db-instance-identifier mydb

# Modify DB instance
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.small \
  --apply-immediately

# Start DB instance
aws rds start-db-instance --db-instance-identifier mydb

# Stop DB instance
aws rds stop-db-instance --db-instance-identifier mydb

# Reboot DB instance
aws rds reboot-db-instance --db-instance-identifier mydb

# Delete DB instance (with final snapshot)
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --final-db-snapshot-identifier mydb-final-snapshot

# Delete DB instance (without snapshot)
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
```

### DB Snapshots

```bash
# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot-20240101

# Copy snapshot
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier mydb-snapshot-20240101 \
  --target-db-snapshot-identifier mydb-snapshot-copy

# Delete snapshot
aws rds delete-db-snapshot --db-snapshot-identifier mydb-snapshot-20240101
```

### DB Subnet Groups

```bash
# Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name mydbsubnetgroup \
  --db-subnet-group-description "My DB Subnet Group" \
  --subnet-ids subnet-0123456789abcdef0 subnet-9876543210abcdef0

# List DB subnet groups
aws rds describe-db-subnet-groups

# Delete DB subnet group
aws rds delete-db-subnet-group --db-subnet-group-name mydbsubnetgroup
```

### Parameter Groups

```bash
# Create parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name myparametergroup \
  --db-parameter-group-family mysql8.0 \
  --description "My parameter group"

# List parameter groups
aws rds describe-db-parameter-groups

# Modify parameter
aws rds modify-db-parameter-group \
  --db-parameter-group-name myparametergroup \
  --parameters "ParameterName=max_connections,ParameterValue=200,ApplyMethod=immediate"

# List parameters
aws rds describe-db-parameters --db-parameter-group-name myparametergroup

# Delete parameter group
aws rds delete-db-parameter-group --db-parameter-group-name myparametergroup
```

### Aurora Clusters

```bash
# Create Aurora cluster
aws rds create-db-cluster \
  --db-cluster-identifier myaurora \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.04.0 \
  --master-username admin \
  --master-user-password MyPassword123! \
  --vpc-security-group-ids sg-0123456789abcdef0 \
  --db-subnet-group-name mydbsubnetgroup

# Create Aurora instance in cluster
aws rds create-db-instance \
  --db-instance-identifier myaurora-instance-1 \
  --db-instance-class db.t3.medium \
  --engine aurora-mysql \
  --db-cluster-identifier myaurora

# List clusters
aws rds describe-db-clusters

# Delete cluster
aws rds delete-db-cluster \
  --db-cluster-identifier myaurora \
  --skip-final-snapshot
```

---

## Lambda

### Function Management

```bash
# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime nodejs18.x \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 256 \
  --environment "Variables={ENV=production,DB_HOST=mydb.example.com}"

# List functions
aws lambda list-functions

# Get function configuration
aws lambda get-function --function-name my-function

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update function configuration
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 60 \
  --memory-size 512 \
  --environment "Variables={ENV=production,DB_HOST=newdb.example.com}"

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json

# Invoke function asynchronously
aws lambda invoke \
  --function-name my-function \
  --invocation-type Event \
  --payload '{"key": "value"}' \
  response.json

# Delete function
aws lambda delete-function --function-name my-function

# Get function logs
aws logs tail /aws/lambda/my-function --follow
```

### Function Versions & Aliases

```bash
# Publish version
aws lambda publish-version --function-name my-function

# List versions
aws lambda list-versions-by-function --function-name my-function

# Create alias
aws lambda create-alias \
  --function-name my-function \
  --name production \
  --function-version 1

# Update alias
aws lambda update-alias \
  --function-name my-function \
  --name production \
  --function-version 2

# Delete alias
aws lambda delete-alias --function-name my-function --name production
```

### Event Source Mappings

```bash
# Create event source mapping (SQS)
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
  --batch-size 10

# Create event source mapping (DynamoDB Stream)
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:dynamodb:us-east-1:123456789012:table/my-table/stream/2024-01-01T00:00:00.000 \
  --starting-position LATEST

# List event source mappings
aws lambda list-event-source-mappings --function-name my-function

# Delete event source mapping
aws lambda delete-event-source-mapping --uuid 12345678-1234-1234-1234-123456789012
```

### Layers

```bash
# Publish layer version
aws lambda publish-layer-version \
  --layer-name my-layer \
  --description "My Lambda Layer" \
  --zip-file fileb://layer.zip \
  --compatible-runtimes nodejs18.x python3.11

# List layers
aws lambda list-layers

# List layer versions
aws lambda list-layer-versions --layer-name my-layer

# Add layer to function
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:my-layer:1

# Delete layer version
aws lambda delete-layer-version --layer-name my-layer --version-number 1
```

---

## ECS & Fargate

### Cluster Management

```bash
# Create ECS cluster
aws ecs create-cluster --cluster-name my-cluster

# List clusters
aws ecs list-clusters

# Describe cluster
aws ecs describe-clusters --clusters my-cluster

# Delete cluster
aws ecs delete-cluster --cluster my-cluster
```

### Task Definitions

```bash
# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# List task definitions
aws ecs list-task-definitions

# Describe task definition
aws ecs describe-task-definition --task-definition my-task:1

# Deregister task definition
aws ecs deregister-task-definition --task-definition my-task:1
```

```json
// task-definition.json
{
  "family": "my-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "nginx:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-task",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### Service Management

```bash
# Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-service \
  --task-definition my-task:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-0123456789abcdef0],securityGroups=[sg-0123456789abcdef0],assignPublicIp=ENABLED}"

# List services
aws ecs list-services --cluster my-cluster

# Describe service
aws ecs describe-services --cluster my-cluster --services my-service

# Update service
aws ecs update-service \
  --cluster my-cluster \
  --service my-service \
  --desired-count 3 \
  --task-definition my-task:2

# Delete service
aws ecs delete-service --cluster my-cluster --service my-service --force
```

### Task Management

```bash
# Run task
aws ecs run-task \
  --cluster my-cluster \
  --task-definition my-task:1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-0123456789abcdef0],securityGroups=[sg-0123456789abcdef0],assignPublicIp=ENABLED}"

# List tasks
aws ecs list-tasks --cluster my-cluster

# Describe tasks
aws ecs describe-tasks --cluster my-cluster --tasks TASK_ARN

# Stop task
aws ecs stop-task --cluster my-cluster --task TASK_ARN
```

---

## EKS (Elastic Kubernetes Service)

### Cluster Management

```bash
# Create EKS cluster
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::123456789012:role/eks-cluster-role \
  --resources-vpc-config subnetIds=subnet-0123456789abcdef0,subnet-9876543210abcdef0,securityGroupIds=sg-0123456789abcdef0

# List clusters
aws eks list-clusters

# Describe cluster
aws eks describe-cluster --name my-cluster

# Update cluster version
aws eks update-cluster-version --name my-cluster --kubernetes-version 1.28

# Delete cluster
aws eks delete-cluster --name my-cluster

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1

# Update kubeconfig with specific profile
aws eks update-kubeconfig --name my-cluster --region us-east-1 --profile production
```

### Node Groups

```bash
# Create node group
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name my-nodegroup \
  --scaling-config minSize=1,maxSize=3,desiredSize=2 \
  --disk-size 20 \
  --subnets subnet-0123456789abcdef0 subnet-9876543210abcdef0 \
  --instance-types t3.medium \
  --node-role arn:aws:iam::123456789012:role/eks-node-role

# List node groups
aws eks list-nodegroups --cluster-name my-cluster

# Describe node group
aws eks describe-nodegroup --cluster-name my-cluster --nodegroup-name my-nodegroup

# Update node group
aws eks update-nodegroup-config \
  --cluster-name my-cluster \
  --nodegroup-name my-nodegroup \
  --scaling-config minSize=2,maxSize=5,desiredSize=3

# Delete node group
aws eks delete-nodegroup --cluster-name my-cluster --nodegroup-name my-nodegroup
```

### Add-ons

```bash
# List available add-ons
aws eks describe-addon-versions

# Create add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni \
  --addon-version v1.15.0-eksbuild.2

# List add-ons
aws eks list-addons --cluster-name my-cluster

# Describe add-on
aws eks describe-addon --cluster-name my-cluster --addon-name vpc-cni

# Update add-on
aws eks update-addon --cluster-name my-cluster --addon-name vpc-cni --addon-version v1.16.0-eksbuild.1

# Delete add-on
aws eks delete-addon --cluster-name my-cluster --addon-name vpc-cni
```

---

## CloudFormation

### Stack Management

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=my-key \
  --capabilities CAPABILITY_IAM

# List stacks
aws cloudformation list-stacks

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Get stack events
aws cloudformation describe-stack-events --stack-name my-stack

# Get stack resources
aws cloudformation describe-stack-resources --stack-name my-stack

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=new-key

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Wait for stack creation
aws cloudformation wait stack-create-complete --stack-name my-stack

# Wait for stack deletion
aws cloudformation wait stack-delete-complete --stack-name my-stack
```

### Change Sets

```bash
# Create change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# Describe change set
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Execute change set
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Delete change set
aws cloudformation delete-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

### Template Validation

```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Estimate template cost
aws cloudformation estimate-template-cost --template-body file://template.yaml
```

---

## CloudWatch

### Logs

```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# List log groups
aws logs describe-log-groups

# Delete log group
aws logs delete-log-group --log-group-name /aws/lambda/my-function

# List log streams
aws logs describe-log-streams --log-group-name /aws/lambda/my-function

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name 2024/01/01/[$LATEST]abc123

# Filter log events
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR" \
  --start-time 1704067200000

# Tail logs
aws logs tail /aws/lambda/my-function --follow

# Put log events
aws logs put-log-events \
  --log-group-name my-log-group \
  --log-stream-name my-log-stream \
  --log-events timestamp=1704067200000,message="Log message"
```

### Metrics

```bash
# List metrics
aws cloudwatch list-metrics --namespace AWS/EC2

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average,Maximum

# Put metric data
aws cloudwatch put-metric-data \
  --namespace MyApp \
  --metric-name RequestCount \
  --value 100 \
  --timestamp 2024-01-01T12:00:00Z
```

### Alarms

```bash
# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:my-topic

# List alarms
aws cloudwatch describe-alarms

# Describe specific alarm
aws cloudwatch describe-alarms --alarm-names high-cpu

# Delete alarm
aws cloudwatch delete-alarms --alarm-names high-cpu

# Disable alarm actions
aws cloudwatch disable-alarm-actions --alarm-names high-cpu

# Enable alarm actions
aws cloudwatch enable-alarm-actions --alarm-names high-cpu
```

### Dashboards

```bash
# Create dashboard
aws cloudwatch put-dashboard \
  --dashboard-name my-dashboard \
  --dashboard-body file://dashboard.json

# List dashboards
aws cloudwatch list-dashboards

# Get dashboard
aws cloudwatch get-dashboard --dashboard-name my-dashboard

# Delete dashboard
aws cloudwatch delete-dashboards --dashboard-names my-dashboard
```

---

## Route 53

### Hosted Zones

```bash
# Create hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s)

# List hosted zones
aws route53 list-hosted-zones

# Get hosted zone
aws route53 get-hosted-zone --id Z1234567890ABC

# Delete hosted zone
aws route53 delete-hosted-zone --id Z1234567890ABC
```

### Record Sets

```bash
# List record sets
aws route53 list-resource-record-sets --hosted-zone-id Z1234567890ABC

# Create A record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch file://change-batch.json
```

```json
// change-batch.json
{
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "192.0.2.1"
          }
        ]
      }
    }
  ]
}
```

### Health Checks

```bash
# Create health check
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config IPAddress=192.0.2.1,Port=80,Type=HTTP,ResourcePath=/health

# List health checks
aws route53 list-health-checks

# Delete health check
aws route53 delete-health-check --health-check-id abc12345-1234-1234-1234-123456789012
```

---

## Load Balancers

### Application Load Balancer (ALB)

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-0123456789abcdef0 subnet-9876543210abcdef0 \
  --security-groups sg-0123456789abcdef0 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4

# List load balancers
aws elbv2 describe-load-balancers

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-0123456789abcdef0 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 2

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/1234567890123456 \
  --targets Id=i-1234567890abcdef0 Id=i-0987654321fedcba0

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890123456 \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/1234567890123456

# Delete load balancer
aws elbv2 delete-load-balancer \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890123456
```

### Network Load Balancer (NLB)

```bash
# Create NLB
aws elbv2 create-load-balancer \
  --name my-nlb \
  --subnets subnet-0123456789abcdef0 subnet-9876543210abcdef0 \
  --type network \
  --scheme internet-facing
```

---

## Auto Scaling

### Launch Templates

```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --version-description "Version 1" \
  --launch-template-data file://template-data.json

# List launch templates
aws ec2 describe-launch-templates

# Create new version
aws ec2 create-launch-template-version \
  --launch-template-name my-template \
  --version-description "Version 2" \
  --launch-template-data file://template-data.json

# Delete launch template
aws ec2 delete-launch-template --launch-template-name my-template
```

```json
// template-data.json
{
  "ImageId": "ami-0c55b159cbfafe1f0",
  "InstanceType": "t2.micro",
  "KeyName": "my-key-pair",
  "SecurityGroupIds": ["sg-0123456789abcdef0"],
  "UserData": "IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8gV29ybGQi"
}
```

### Auto Scaling Groups

```bash
# Create auto scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version='$Latest' \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-0123456789abcdef0,subnet-9876543210abcdef0" \
  --target-group-arns arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/1234567890123456 \
  --health-check-type ELB \
  --health-check-grace-period 300

# List auto scaling groups
aws autoscaling describe-auto-scaling-groups

# Update auto scaling group
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3

# Set desired capacity
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 4

# Terminate instance in auto scaling group
aws autoscaling terminate-instance-in-auto-scaling-group \
  --instance-id i-1234567890abcdef0 \
  --should-decrement-desired-capacity

# Delete auto scaling group
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --force-delete
```

### Scaling Policies

```bash
# Create target tracking scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://target-tracking-config.json

# Create step scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name scale-up \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments MetricIntervalLowerBound=0,ScalingAdjustment=1

# List scaling policies
aws autoscaling describe-policies --auto-scaling-group-name my-asg

# Delete scaling policy
aws autoscaling delete-policy --policy-name cpu-target-tracking --auto-scaling-group-name my-asg
```

```json
// target-tracking-config.json
{
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ASGAverageCPUUtilization"
  },
  "TargetValue": 70.0
}
```

---

## SNS & SQS

### SNS (Simple Notification Service)

```bash
# Create topic
aws sns create-topic --name my-topic

# List topics
aws sns list-topics

# Get topic attributes
aws sns get-topic-attributes --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic

# Subscribe to topic (email)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --protocol email \
  --notification-endpoint user@example.com

# Subscribe to topic (SQS)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:my-queue

# List subscriptions
aws sns list-subscriptions

# Publish message
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --message "Hello World" \
  --subject "Test Message"

# Publish with attributes
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --message "Hello World" \
  --message-attributes '{"priority":{"DataType":"String","StringValue":"high"}}'

# Unsubscribe
aws sns unsubscribe --subscription-arn arn:aws:sns:us-east-1:123456789012:my-topic:12345678-1234-1234-1234-123456789012

# Delete topic
aws sns delete-topic --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic
```

### SQS (Simple Queue Service)

```bash
# Create queue
aws sqs create-queue --queue-name my-queue

# Create FIFO queue
aws sqs create-queue \
  --queue-name my-queue.fifo \
  --attributes FifoQueue=true,ContentBasedDeduplication=true

# List queues
aws sqs list-queues

# Get queue URL
aws sqs get-queue-url --queue-name my-queue

# Get queue attributes
aws sqs get-queue-attributes \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --attribute-names All

# Send message
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --message-body "Hello World"

# Send message with attributes
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --message-body "Hello World" \
  --message-attributes '{"Priority":{"DataType":"String","StringValue":"high"}}'

# Receive messages
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --max-number-of-messages 10 \
  --wait-time-seconds 20

# Delete message
aws sqs delete-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --receipt-handle "MESSAGE_RECEIPT_HANDLE"

# Purge queue
aws sqs purge-queue --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue

# Delete queue
aws sqs delete-queue --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue
```

---

## DynamoDB

### Table Management

```bash
# Create table
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions \
    AttributeName=UserId,AttributeType=S \
    AttributeName=Email,AttributeType=S \
  --key-schema \
    AttributeName=UserId,KeyType=HASH \
  --global-secondary-indexes \
    "[{\"IndexName\":\"EmailIndex\",\"KeySchema\":[{\"AttributeName\":\"Email\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"},\"ProvisionedThroughput\":{\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}}]" \
  --billing-mode PAY_PER_REQUEST

# List tables
aws dynamodb list-tables

# Describe table
aws dynamodb describe-table --table-name Users

# Update table
aws dynamodb update-table \
  --table-name Users \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10

# Delete table
aws dynamodb delete-table --table-name Users
```

### Item Operations

```bash
# Put item
aws dynamodb put-item \
  --table-name Users \
  --item '{
    "UserId": {"S": "user123"},
    "Email": {"S": "user@example.com"},
    "Name": {"S": "John Doe"},
    "Age": {"N": "30"}
  }'

# Get item
aws dynamodb get-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Update item
aws dynamodb update-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}' \
  --update-expression "SET Age = :age" \
  --expression-attribute-values '{":age": {"N": "31"}}'

# Delete item
aws dynamodb delete-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Query items
aws dynamodb query \
  --table-name Users \
  --key-condition-expression "UserId = :userId" \
  --expression-attribute-values '{":userId": {"S": "user123"}}'

# Scan table
aws dynamodb scan --table-name Users

# Batch write items
aws dynamodb batch-write-item --request-items file://batch-write.json
```

```json
// batch-write.json
{
  "Users": [
    {
      "PutRequest": {
        "Item": {
          "UserId": {"S": "user123"},
          "Email": {"S": "user@example.com"}
        }
      }
    },
    {
      "PutRequest": {
        "Item": {
          "UserId": {"S": "user456"},
          "Email": {"S": "user2@example.com"}
        }
      }
    }
  ]
}
```

---

## ElastiCache

### Redis

```bash
# Create Redis cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id my-redis \
  --engine redis \
  --cache-node-type cache.t3.micro \
  --num-cache-nodes 1 \
  --cache-subnet-group-name my-subnet-group \
  --security-group-ids sg-0123456789abcdef0

# Create Redis replication group
aws elasticache create-replication-group \
  --replication-group-id my-redis-cluster \
  --replication-group-description "My Redis Cluster" \
  --engine redis \
  --cache-node-type cache.t3.micro \
  --num-cache-clusters 3 \
  --automatic-failover-enabled \
  --cache-subnet-group-name my-subnet-group

# List cache clusters
aws elasticache describe-cache-clusters

# Delete cache cluster
aws elasticache delete-cache-cluster --cache-cluster-id my-redis
```

### Memcached

```bash
# Create Memcached cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id my-memcached \
  --engine memcached \
  --cache-node-type cache.t3.micro \
  --num-cache-nodes 2 \
  --cache-subnet-group-name my-subnet-group
```

---

## CloudFront

### Distribution Management

```bash
# Create distribution
aws cloudfront create-distribution --distribution-config file://distribution-config.json

# List distributions
aws cloudfront list-distributions

# Get distribution
aws cloudfront get-distribution --id E1234567890ABC

# Update distribution
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --if-match ETAG_VALUE \
  --distribution-config file://distribution-config.json

# Create invalidation
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"

# Delete distribution
aws cloudfront delete-distribution --id E1234567890ABC --if-match ETAG_VALUE
```

---

## API Gateway

### REST API

```bash
# Create REST API
aws apigateway create-rest-api --name my-api --description "My API"

# Get REST APIs
aws apigateway get-rest-apis

# Get resources
aws apigateway get-resources --rest-api-id abc123

# Create resource
aws apigateway create-resource \
  --rest-api-id abc123 \
  --parent-id xyz789 \
  --path-part users

# Create method
aws apigateway put-method \
  --rest-api-id abc123 \
  --resource-id xyz789 \
  --http-method GET \
  --authorization-type NONE

# Create integration
aws apigateway put-integration \
  --rest-api-id abc123 \
  --resource-id xyz789 \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:my-function/invocations

# Deploy API
aws apigateway create-deployment \
  --rest-api-id abc123 \
  --stage-name prod

# Delete REST API
aws apigateway delete-rest-api --rest-api-id abc123
```

---

## Systems Manager (SSM)

### Parameter Store

```bash
# Put parameter
aws ssm put-parameter \
  --name /myapp/database/host \
  --value "db.example.com" \
  --type String

# Put secure parameter
aws ssm put-parameter \
  --name /myapp/database/password \
  --value "MySecretPassword" \
  --type SecureString

# Get parameter
aws ssm get-parameter --name /myapp/database/host

# Get parameter with decryption
aws ssm get-parameter --name /myapp/database/password --with-decryption

# Get parameters by path
aws ssm get-parameters-by-path --path /myapp --recursive

# Delete parameter
aws ssm delete-parameter --name /myapp/database/host
```

### Run Command

```bash
# Send command to instances
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=tag:Environment,Values=Production" \
  --parameters 'commands=["sudo yum update -y"]'

# Get command invocation
aws ssm get-command-invocation \
  --command-id abc-123-def-456 \
  --instance-id i-1234567890abcdef0
```

### Session Manager

```bash
# Start session
aws ssm start-session --target i-1234567890abcdef0

# List sessions
aws ssm describe-sessions --state Active

# Terminate session
aws ssm terminate-session --session-id session-0123456789abcdef0
```

---

## Secrets Manager

```bash
# Create secret
aws secretsmanager create-secret \
  --name MyAppSecret \
  --description "My application secret" \
  --secret-string '{"username":"admin","password":"MyPassword123!"}'

# List secrets
aws secretsmanager list-secrets

# Get secret value
aws secretsmanager get-secret-value --secret-id MyAppSecret

# Update secret
aws secretsmanager update-secret \
  --secret-id MyAppSecret \
  --secret-string '{"username":"admin","password":"NewPassword456!"}'

# Rotate secret
aws secretsmanager rotate-secret \
  --secret-id MyAppSecret \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:MyRotationFunction

# Delete secret
aws secretsmanager delete-secret --secret-id MyAppSecret

# Restore secret
aws secretsmanager restore-secret --secret-id MyAppSecret
```

---

## ECR (Elastic Container Registry)

```bash
# Create repository
aws ecr create-repository --repository-name my-app

# List repositories
aws ecr describe-repositories

# Get login password
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Tag image
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# Push image
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# Pull image
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# List images
aws ecr list-images --repository-name my-app

# Delete image
aws ecr batch-delete-image \
  --repository-name my-app \
  --image-ids imageTag=latest

# Delete repository
aws ecr delete-repository --repository-name my-app --force

# Set lifecycle policy
aws ecr put-lifecycle-policy \
  --repository-name my-app \
  --lifecycle-policy-text file://lifecycle-policy.json
```

```json
// lifecycle-policy.json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep only 10 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

---

## Cost Management

### Cost Explorer

```bash
# Get cost and usage
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost UnblendedCost

# Get cost forecast
aws ce get-cost-forecast \
  --time-period Start=2024-02-01,End=2024-02-29 \
  --metric BLENDED_COST \
  --granularity MONTHLY
```

### Budgets

```bash
# Create budget
aws budgets create-budget \
  --account-id 123456789012 \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

```json
// budget.json
{
  "BudgetName": "Monthly Budget",
  "BudgetLimit": {
    "Amount": "1000",
    "Unit": "USD"
  },
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
```

### Cost Tags

```bash
# Activate cost allocation tags
aws ce create-cost-category-definition \
  --name Environment \
  --rules file://rules.json
```

---

## Security Best Practices

### Enable MFA for Root Account

```bash
# This must be done through AWS Console
# 1. Sign in as root user
# 2. Go to IAM > Dashboard > Security Status
# 3. Activate MFA on your root account
```

### Enable CloudTrail

```bash
# Create trail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-cloudtrail-bucket

# Start logging
aws cloudtrail start-logging --name my-trail

# Get trail status
aws cloudtrail get-trail-status --name my-trail
```

### Enable AWS Config

```bash
# Create configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/config-role

# Start configuration recorder
aws configservice start-configuration-recorder --configuration-recorder-name default
```

### Enable GuardDuty

```bash
# Create detector
aws guardduty create-detector --enable

# List detectors
aws guardduty list-detectors

# Get findings
aws guardduty list-findings --detector-id abc123
```

### Security Hub

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Get findings
aws securityhub get-findings
```

### IAM Access Analyzer

```bash
# Create analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name my-analyzer \
  --type ACCOUNT

# List findings
aws accessanalyzer list-findings --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-analyzer
```

---

## Multi-Account Strategy

### AWS Organizations

```bash
# Create organization
aws organizations create-organization

# List accounts
aws organizations list-accounts

# Create account
aws organizations create-account \
  --email new-account@example.com \
  --account-name "Development Account"

# Create organizational unit
aws organizations create-organizational-unit \
  --parent-id r-abc123 \
  --name Development

# Move account to OU
aws organizations move-account \
  --account-id 123456789012 \
  --source-parent-id r-abc123 \
  --destination-parent-id ou-abc123

# Attach policy to account
aws organizations attach-policy \
  --policy-id p-abc123 \
  --target-id 123456789012
```

### Service Control Policies (SCPs)

```bash
# Create policy
aws organizations create-policy \
  --content file://scp.json \
  --description "Deny all S3 delete operations" \
  --name DenyS3Delete \
  --type SERVICE_CONTROL_POLICY

# List policies
aws organizations list-policies --filter SERVICE_CONTROL_POLICY

# Attach policy
aws organizations attach-policy \
  --policy-id p-abc123 \
  --target-id ou-abc123
```

```json
// scp.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "s3:DeleteBucket",
        "s3:DeleteObject"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Conclusion

This comprehensive AWS guide covers the essential services and CLI commands for managing AWS infrastructure. Key takeaways:

1. **IAM**: Always follow the principle of least privilege
2. **EC2**: Use Auto Scaling and Load Balancers for high availability
3. **VPC**: Design proper network segmentation
4. **S3**: Enable versioning and encryption for important data
5. **RDS**: Regular backups and use read replicas for scaling
6. **Lambda**: Serverless for event-driven architectures
7. **ECS/EKS**: Container orchestration options
8. **CloudFormation**: Infrastructure as Code
9. **CloudWatch**: Monitor and set alarms
10. **Security**: Enable CloudTrail, GuardDuty, and use MFA

### Additional Resources

- [AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [AWS Cost Optimization](https://aws.amazon.com/pricing/cost-optimization/)

---

**Remember**: Always test commands in a non-production environment first and follow AWS best practices for security and cost optimization.
