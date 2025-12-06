# Amazon EKS Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation and Setup](#installation-and-setup)
- [Creating an EKS Cluster](#creating-an-eks-cluster)
- [Node Groups](#node-groups)
- [Networking](#networking)
- [Storage](#storage)
- [IAM and RBAC](#iam-and-rbac)
- [Load Balancing](#load-balancing)
- [Auto Scaling](#auto-scaling)
- [Monitoring and Logging](#monitoring-and-logging)
- [Security](#security)
- [Add-ons and Integrations](#add-ons-and-integrations)
- [Cost Optimization](#cost-optimization)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Advanced Topics](#advanced-topics)
- [Practical Examples](#practical-examples)

---

## Introduction

Amazon Elastic Kubernetes Service (EKS) is a fully managed Kubernetes service that makes it easy to run Kubernetes on AWS without needing to install and operate your own Kubernetes control plane.

### Key Features

- **Managed Control Plane**: AWS manages the Kubernetes control plane
- **High Availability**: Control plane runs across multiple AZs
- **Security**: Integration with AWS IAM, VPC, and security services
- **Scalability**: Auto-scaling of nodes and pods
- **Updates**: Automated version updates and patching
- **Integration**: Native AWS service integration
- **Compliance**: SOC, PCI, ISO, and other certifications

### EKS Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Cloud                             │
│  ┌───────────────────────────────────────────────────┐  │
│  │         EKS Control Plane (Managed by AWS)        │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  │
│  │  │   API    │  │ Scheduler│  │Controller│        │  │
│  │  │  Server  │  │          │  │ Manager  │        │  │
│  │  └──────────┘  └──────────┘  └──────────┘        │  │
│  └───────────────────────────────────────────────────┘  │
│                         │                                │
│  ┌──────────────────────┴───────────────────────────┐  │
│  │              Worker Nodes (Your VPC)             │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │  │
│  │  │  Node 1  │  │  Node 2  │  │  Node 3  │       │  │
│  │  │  (EC2)   │  │  (EC2)   │  │ (Fargate)│       │  │
│  │  └──────────┘  └──────────┘  └──────────┘       │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Prerequisites

### Required Tools

```bash
# Install AWS CLI
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# macOS
brew install awscli

# Verify
aws --version

# Configure AWS CLI
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region: us-east-1
# Default output format: json

# Install kubectl
# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# macOS
brew install kubectl

# Verify
kubectl version --client

# Install eksctl
# Linux
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# macOS
brew install eksctl

# Verify
eksctl version

# Install Helm
# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# macOS
brew install helm

# Verify
helm version
```

### IAM Permissions Required

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:*",
        "ec2:*",
        "elasticloadbalancing:*",
        "autoscaling:*",
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:PassRole",
        "cloudformation:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Installation and Setup

### Configure AWS Credentials

```bash
# Set up AWS credentials
export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
export AWS_DEFAULT_REGION=us-east-1

# Or use AWS CLI configure
aws configure

# Verify credentials
aws sts get-caller-identity

# Set default region
export AWS_REGION=us-east-1
```

### Install AWS IAM Authenticator

```bash
# Linux
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv ./aws-iam-authenticator /usr/local/bin

# macOS
brew install aws-iam-authenticator

# Verify
aws-iam-authenticator version
```

---

## Creating an EKS Cluster

### Using eksctl (Recommended)

```bash
# Create basic cluster
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Create cluster with YAML config
cat > cluster.yaml <<EOF
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1
  version: "1.28"

vpc:
  cidr: 10.0.0.0/16
  nat:
    gateway: Single

managedNodeGroups:
  - name: standard-workers
    instanceType: t3.medium
    minSize: 2
    maxSize: 4
    desiredCapacity: 3
    volumeSize: 20
    ssh:
      allow: true
      publicKeyPath: ~/.ssh/id_rsa.pub
    labels:
      role: worker
      environment: production
    tags:
      nodegroup-role: worker
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        externalDNS: true
        certManager: true
        albIngress: true
        ebs: true
        efs: true
        cloudWatch: true

  - name: spot-workers
    instanceTypes: ["t3.medium", "t3a.medium"]
    minSize: 0
    maxSize: 10
    desiredCapacity: 2
    spot: true
    labels:
      lifecycle: spot

iam:
  withOIDC: true
  serviceAccounts:
    - metadata:
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true
    - metadata:
        name: cluster-autoscaler
        namespace: kube-system
      wellKnownPolicies:
        autoScaler: true

addons:
  - name: vpc-cni
    version: latest
  - name: coredns
    version: latest
  - name: kube-proxy
    version: latest

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]
EOF

# Create cluster from config
eksctl create cluster -f cluster.yaml

# List clusters
eksctl get cluster

# Get cluster info
aws eks describe-cluster --name my-cluster --query cluster

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1

# Verify connection
kubectl get nodes
kubectl cluster-info
```

### Using AWS CLI and CloudFormation

```bash
# Create IAM role for cluster
cat > cluster-role-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name myEKSClusterRole \
  --assume-role-policy-document file://cluster-role-trust-policy.json

aws iam attach-role-policy \
  --role-name myEKSClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy

# Create VPC (using CloudFormation)
aws cloudformation create-stack \
  --stack-name my-eks-vpc \
  --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

# Wait for stack creation
aws cloudformation wait stack-create-complete --stack-name my-eks-vpc

# Get VPC details
SECURITY_GROUP=$(aws cloudformation describe-stacks \
  --stack-name my-eks-vpc \
  --query "Stacks[0].Outputs[?OutputKey=='SecurityGroups'].OutputValue" \
  --output text)

SUBNET_IDS=$(aws cloudformation describe-stacks \
  --stack-name my-eks-vpc \
  --query "Stacks[0].Outputs[?OutputKey=='SubnetIds'].OutputValue" \
  --output text)

# Create EKS cluster
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/myEKSClusterRole \
  --resources-vpc-config subnetIds=$SUBNET_IDS,securityGroupIds=$SECURITY_GROUP

# Wait for cluster to be active
aws eks wait cluster-active --name my-cluster

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster
```

### Delete Cluster

```bash
# Delete cluster with eksctl
eksctl delete cluster --name my-cluster

# Delete cluster with AWS CLI
aws eks delete-cluster --name my-cluster

# Delete associated resources
aws cloudformation delete-stack --stack-name my-eks-vpc
```

---

## Node Groups

### Managed Node Groups

```bash
# Create managed node group
eksctl create nodegroup \
  --cluster my-cluster \
  --name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Create node group with AWS CLI
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name standard-workers \
  --scaling-config minSize=1,maxSize=4,desiredSize=3 \
  --subnets subnet-xxx subnet-yyy \
  --instance-types t3.medium \
  --node-role arn:aws:iam::ACCOUNT_ID:role/NodeInstanceRole

# List node groups
eksctl get nodegroup --cluster my-cluster
aws eks list-nodegroups --cluster-name my-cluster

# Describe node group
aws eks describe-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name standard-workers

# Update node group
eksctl scale nodegroup \
  --cluster my-cluster \
  --name standard-workers \
  --nodes 5 \
  --nodes-min 2 \
  --nodes-max 6

# Delete node group
eksctl delete nodegroup \
  --cluster my-cluster \
  --name standard-workers
```

### Spot Instances Node Group

```yaml
# spot-nodegroup.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1

managedNodeGroups:
  - name: spot-workers
    instanceTypes:
      - t3.medium
      - t3a.medium
      - t3.large
    minSize: 0
    maxSize: 10
    desiredCapacity: 3
    spot: true
    labels:
      lifecycle: spot
      instance-type: mixed
    tags:
      k8s.io/cluster-autoscaler/enabled: "true"
      k8s.io/cluster-autoscaler/my-cluster: "owned"
    iam:
      withAddonPolicies:
        autoScaler: true
```

```bash
# Create spot node group
eksctl create nodegroup -f spot-nodegroup.yaml
```

### Fargate Profiles

```bash
# Create Fargate profile with eksctl
eksctl create fargateprofile \
  --cluster my-cluster \
  --name fp-default \
  --namespace default

# Create Fargate profile with YAML
cat > fargate-profile.yaml <<EOF
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1

fargateProfiles:
  - name: fp-app
    selectors:
      - namespace: production
        labels:
          app: myapp
  - name: fp-kube-system
    selectors:
      - namespace: kube-system
EOF

eksctl create fargateprofile -f fargate-profile.yaml

# List Fargate profiles
eksctl get fargateprofile --cluster my-cluster

# Delete Fargate profile
eksctl delete fargateprofile --cluster my-cluster --name fp-default
```

### Custom AMI for Node Groups

```bash
# Build custom AMI using Packer
cat > eks-node-ami.json <<EOF
{
  "variables": {
    "aws_region": "us-east-1",
    "eks_version": "1.28"
  },
  "builders": [{
    "type": "amazon-ebs",
    "region": "{{user `aws_region`}}",
    "source_ami_filter": {
      "filters": {
        "name": "amazon-eks-node-{{user `eks_version`}}-*"
      },
      "owners": ["amazon"],
      "most_recent": true
    },
    "instance_type": "t3.medium",
    "ssh_username": "ec2-user",
    "ami_name": "custom-eks-node-{{timestamp}}"
  }],
  "provisioners": [{
    "type": "shell",
    "inline": [
      "sudo yum update -y",
      "sudo yum install -y docker htop"
    ]
  }]
}
EOF

# Use custom AMI in node group
eksctl create nodegroup \
  --cluster my-cluster \
  --name custom-nodes \
  --node-ami ami-xxxxx \
  --node-type t3.medium \
  --nodes 3
```

---

## Networking

### VPC Configuration

```yaml
# vpc-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1

vpc:
  cidr: 10.0.0.0/16
  nat:
    gateway: HighlyAvailable  # Single, HighlyAvailable, or Disable
  clusterEndpoints:
    privateAccess: true
    publicAccess: true
  publicAccessCIDRs: ["0.0.0.0/0"]

availabilityZones:
  - us-east-1a
  - us-east-1b
  - us-east-1c

subnets:
  private:
    us-east-1a:
      id: subnet-xxx
      cidr: 10.0.1.0/24
    us-east-1b:
      id: subnet-yyy
      cidr: 10.0.2.0/24
    us-east-1c:
      id: subnet-zzz
      cidr: 10.0.3.0/24
  public:
    us-east-1a:
      id: subnet-aaa
      cidr: 10.0.101.0/24
    us-east-1b:
      id: subnet-bbb
      cidr: 10.0.102.0/24
    us-east-1c:
      id: subnet-ccc
      cidr: 10.0.103.0/24
```

### VPC CNI Plugin

```bash
# Install/Update VPC CNI plugin
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.13/config/master/aws-k8s-cni.yaml

# Enable prefix delegation (more IPs per node)
kubectl set env daemonset aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true

# Configure custom networking
kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true

# Set ENI config label
kubectl set env daemonset aws-node -n kube-system ENI_CONFIG_LABEL_DEF=topology.kubernetes.io/zone

# Create ENI config
cat > eniconfig.yaml <<EOF
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: us-east-1a
spec:
  subnet: subnet-xxx
  securityGroups:
    - sg-xxx
EOF

kubectl apply -f eniconfig.yaml
```

### Network Policies

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-same-namespace
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector: {}

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-traffic
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

```bash
# Apply network policies
kubectl apply -f network-policy.yaml
```

### Security Groups for Pods

```bash
# Enable security groups for pods
kubectl set env daemonset aws-node \
  -n kube-system \
  ENABLE_POD_ENI=true

# Create SecurityGroupPolicy
cat > security-group-policy.yaml <<EOF
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
  name: my-security-group-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: myapp
  securityGroups:
    groupIds:
      - sg-xxxxx
EOF

kubectl apply -f security-group-policy.yaml
```

---

## Storage

### EBS CSI Driver

```bash
# Add EBS CSI driver IAM policy
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace kube-system \
  --name ebs-csi-controller-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --override-existing-serviceaccounts

# Install EBS CSI driver
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.24"

# Or using eksctl addon
eksctl create addon \
  --cluster my-cluster \
  --name aws-ebs-csi-driver \
  --version latest \
  --service-account-role-arn arn:aws:iam::ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole \
  --force
```

### StorageClass for EBS

```yaml
# ebs-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
  kmsKeyId: arn:aws:kms:us-east-1:ACCOUNT_ID:key/KEY_ID
allowVolumeExpansion: true

---
# PVC using EBS
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 10Gi

---
# Pod using PVC
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: /data
          name: persistent-storage
  volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: ebs-claim
```

### EFS CSI Driver

```bash
# Create EFS file system
aws efs create-file-system \
  --creation-token my-efs \
  --performance-mode generalPurpose \
  --throughput-mode bursting \
  --encrypted \
  --tags Key=Name,Value=my-efs

# Get EFS ID
EFS_ID=$(aws efs describe-file-systems \
  --query "FileSystems[?Name=='my-efs'].FileSystemId" \
  --output text)

# Create mount targets in each subnet
SUBNETS=$(aws eks describe-cluster \
  --name my-cluster \
  --query "cluster.resourcesVpcConfig.subnetIds" \
  --output text)

for subnet in $SUBNETS; do
  aws efs create-mount-target \
    --file-system-id $EFS_ID \
    --subnet-id $subnet \
    --security-groups sg-xxx
done

# Install EFS CSI driver
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.5"
```

### StorageClass for EFS

```yaml
# efs-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-xxxxx
  directoryPerms: "700"

---
# PVC using EFS
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi

---
# Deployment using EFS
apiVersion: apps/v1
kind: Deployment
metadata:
  name: efs-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: efs-app
  template:
    metadata:
      labels:
        app: efs-app
    spec:
      containers:
        - name: app
          image: nginx
          volumeMounts:
            - name: efs-storage
              mountPath: /data
      volumes:
        - name: efs-storage
          persistentVolumeClaim:
            claimName: efs-claim
```

---

## IAM and RBAC

### IAM Roles for Service Accounts (IRSA)

```bash
# Enable OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve

# Create IAM role and service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace default \
  --name my-service-account \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve \
  --override-existing-serviceaccounts

# Manually create IAM role
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/oidc.eks.REGION.amazonaws.com/id/OIDC_ID"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.REGION.amazonaws.com/id/OIDC_ID:sub": "system:serviceaccount:NAMESPACE:SERVICE_ACCOUNT_NAME"
        }
      }
    }
  ]
}
EOF

aws iam create-role \
  --role-name my-pod-role \
  --assume-role-policy-document file://trust-policy.json

aws iam attach-role-policy \
  --role-name my-pod-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create service account with annotation
cat > service-account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT_ID:role/my-pod-role
EOF

kubectl apply -f service-account.yaml
```

### RBAC Configuration

```yaml
# rbac.yaml
---
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
  - kind: ServiceAccount
    name: my-service-account
    namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
  - apiGroups: [""]
    resources: ["nodes", "namespaces"]
    verbs: ["get", "list"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-read-all
subjects:
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

### AWS IAM to Kubernetes RBAC Mapping

```bash
# Edit aws-auth ConfigMap
kubectl edit configmap aws-auth -n kube-system
```

```yaml
# Add mapRoles and mapUsers
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::ACCOUNT_ID:role/developers
      username: developer:{{SessionName}}
      groups:
        - developers
    - rolearn: arn:aws:iam::ACCOUNT_ID:role/admins
      username: admin:{{SessionName}}
      groups:
        - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::ACCOUNT_ID:user/john
      username: john
      groups:
        - developers
```

---

## Load Balancing

### AWS Load Balancer Controller

```bash
# Create IAM policy
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.6.0/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json

# Create service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --override-existing-serviceaccounts

# Install controller with Helm
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

# Verify installation
kubectl get deployment -n kube-system aws-load-balancer-controller
```

### Application Load Balancer (ALB) Ingress

```yaml
# alb-ingress.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/xxx
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '30'
    alb.ingress.kubernetes.io/success-codes: '200'
    alb.ingress.kubernetes.io/tags: Environment=production,Team=platform
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
```

### Network Load Balancer (NLB)

```yaml
# nlb-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-nlb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:region:account:certificate/xxx
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8080
```

---

## Auto Scaling

### Cluster Autoscaler

```bash
# Create IAM policy
cat > cluster-autoscaler-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeTags",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name AmazonEKSClusterAutoscalerPolicy \
  --policy-document file://cluster-autoscaler-policy.json

# Create service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace kube-system \
  --name cluster-autoscaler \
  --attach-policy-arn arn:aws:iam::ACCOUNT_ID:policy/AmazonEKSClusterAutoscalerPolicy \
  --approve \
  --override-existing-serviceaccounts

# Deploy cluster autoscaler
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

# Set cluster name
kubectl -n kube-system edit deployment cluster-autoscaler

# Add these flags:
# --balance-similar-node-groups
# --skip-nodes-with-system-pods=false
# --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster

# Set image version
kubectl -n kube-system set image deployment.apps/cluster-autoscaler \
  cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v1.28.0
```

### Horizontal Pod Autoscaler (HPA)

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

### Vertical Pod Autoscaler (VPA)

```bash
# Install VPA
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Recreate, Auto
  resourcePolicy:
    containerPolicies:
      - containerName: app
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2
          memory: 2Gi
```

### Karpenter (Advanced Auto Scaling)

```bash
# Install Karpenter
export CLUSTER_NAME=my-cluster
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version v0.32.0 \
  --namespace karpenter \
  --create-namespace \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"="arn:aws:iam::${AWS_ACCOUNT_ID}:role/KarpenterControllerRole-${CLUSTER_NAME}" \
  --set settings.aws.clusterName=${CLUSTER_NAME} \
  --set settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile-${CLUSTER_NAME} \
  --set settings.aws.interruptionQueueName=${CLUSTER_NAME} \
  --wait
```

```yaml
# karpenter-provisioner.yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot", "on-demand"]
    - key: kubernetes.io/arch
      operator: In
      values: ["amd64"]
    - key: node.kubernetes.io/instance-type
      operator: In
      values: ["t3.medium", "t3.large", "t3.xlarge"]
  limits:
    resources:
      cpu: 1000
      memory: 1000Gi
  providerRef:
    name: default
  ttlSecondsAfterEmpty: 30
  ttlSecondsUntilExpired: 2592000

---
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: default
spec:
  subnetSelector:
    karpenter.sh/discovery: my-cluster
  securityGroupSelector:
    karpenter.sh/discovery: my-cluster
  tags:
    karpenter.sh/discovery: my-cluster
```

---

## Monitoring and Logging

### CloudWatch Container Insights

```bash
# Install CloudWatch agent
ClusterName=my-cluster
RegionName=us-east-1
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
FluentBitHttpServer='On'

curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | sed "s/{{cluster_name}}/${ClusterName}/;s/{{region_name}}/${RegionName}/;s/{{http_server_toggle}}/${FluentBitHttpServer}/;s/{{http_server_port}}/${FluentBitHttpPort}/;s/{{read_from_head}}/${FluentBitReadFromHead}/" | kubectl apply -f -

# Verify installation
kubectl get pods -n amazon-cloudwatch
```

### Prometheus and Grafana

```bash
# Add Prometheus community Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set grafana.adminPassword=admin123

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Create ServiceMonitor for custom metrics
cat > servicemonitor.yaml <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
    - port: metrics
      interval: 30s
EOF

kubectl apply -f servicemonitor.yaml
```

### AWS X-Ray

```bash
# Install X-Ray daemon
kubectl apply -f https://github.com/aws/aws-xray-daemon/tree/master/kubernetes/xray-daemon-config.yaml
```

```yaml
# xray-sidecar.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: myapp:latest
          env:
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: xray-service.default:2000
        - name: xray-daemon
          image: amazon/aws-xray-daemon:latest
          ports:
            - containerPort: 2000
              protocol: UDP
```

### Logging with FluentBit

```yaml
# fluentbit-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush        5
        Daemon       Off
        Log_Level    info

    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5

    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token

    [OUTPUT]
        Name cloudwatch_logs
        Match   *
        region us-east-1
        log_group_name /aws/eks/my-cluster
        auto_create_group true
```

---

## Security

### Pod Security Standards

```yaml
# pod-security.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Network Policies (Calico)

```bash
# Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

### Secrets Management with AWS Secrets Manager

```bash
# Install Secrets Store CSI Driver
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --namespace kube-system

# Install AWS provider
kubectl apply -f https://raw.githubusercontent.com/aws/secrets-store-csi-driver-provider-aws/main/deployment/aws-provider-installer.yaml
```

```yaml
# secretproviderclass.yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: aws-secrets
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "MySecret"
        objectType: "secretsmanager"
        objectAlias: "database-creds"

---
# Pod using secrets
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  serviceAccountName: my-service-account
  volumes:
    - name: secrets-store
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "aws-secrets"
  containers:
    - name: app
      image: myapp:latest
      volumeMounts:
        - name: secrets-store
          mountPath: "/mnt/secrets"
          readOnly: true
```

### Image Scanning

```bash
# Enable ECR image scanning
aws ecr put-image-scanning-configuration \
  --repository-name my-repo \
  --image-scanning-configuration scanOnPush=true

# Scan existing image
aws ecr start-image-scan \
  --repository-name my-repo \
  --image-id imageTag=latest

# Get scan findings
aws ecr describe-image-scan-findings \
  --repository-name my-repo \
  --image-id imageTag=latest
```

---

## Add-ons and Integrations

### EKS Add-ons

```bash
# List available add-ons
aws eks describe-addon-versions

# Install add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni \
  --addon-version v1.13.0-eksbuild.1

# List installed add-ons
aws eks list-addons --cluster-name my-cluster

# Update add-on
aws eks update-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni \
  --addon-version v1.14.0-eksbuild.1

# Delete add-on
aws eks delete-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni
```

### Cert-Manager

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create ClusterIssuer for Let's Encrypt
cat > letsencrypt-issuer.yaml <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: alb
EOF

kubectl apply -f letsencrypt-issuer.yaml
```

### ExternalDNS

```bash
# Create IAM policy
cat > external-dns-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
EOF

# Create service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --name external-dns \
  --namespace kube-system \
  --attach-policy-arn arn:aws:iam::ACCOUNT_ID:policy/ExternalDNSPolicy \
  --approve

# Install ExternalDNS
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/external-dns/master/docs/tutorials/aws.yaml
```

---

## Cost Optimization

### Cost Analysis

```bash
# Install kubecost
kubectl create namespace kubecost
helm install kubecost kubecost/cost-analyzer \
  --namespace kubecost \
  --set kubecostToken="YOUR_TOKEN"

# Access kubecost UI
kubectl port-forward -n kubecost svc/kubecost-cost-analyzer 9090:9090
```

### Right-Sizing Recommendations

```bash
# Get resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Install Vertical Pod Autoscaler
./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh

# Get VPA recommendations
kubectl describe vpa myapp-vpa
```

### Spot Instances

```yaml
# See Node Groups section for spot configuration

# Pod scheduling preferences for spot
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              preference:
                matchExpressions:
                  - key: lifecycle
                    operator: In
                    values:
                      - spot
      tolerations:
        - key: spotInstance
          operator: Equal
          value: "true"
          effect: NoSchedule
```

---

## Troubleshooting

### Common Issues

```bash
# Check cluster status
aws eks describe-cluster --name my-cluster --query cluster.status

# Check node status
kubectl get nodes
kubectl describe node NODE_NAME

# Check pod issues
kubectl get pods --all-namespaces
kubectl describe pod POD_NAME -n NAMESPACE
kubectl logs POD_NAME -n NAMESPACE
kubectl logs POD_NAME -n NAMESPACE --previous

# Check events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Debug pod
kubectl run debug --image=nicolaka/netshoot -it --rm -- bash

# Check DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# Check connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- bash
# Inside pod:
curl kubernetes.default.svc.cluster.local

# View API server logs (CloudWatch)
aws logs tail /aws/eks/my-cluster/cluster --follow

# Check IAM authentication
kubectl auth can-i get pods
kubectl auth can-i '*' '*' --all-namespaces

# Verify aws-auth ConfigMap
kubectl get configmap aws-auth -n kube-system -o yaml
```

### Node Issues

```bash
# Cordon node (prevent new pods)
kubectl cordon NODE_NAME

# Drain node (evict pods)
kubectl drain NODE_NAME --ignore-daemonsets --delete-emptydir-data

# Uncordon node
kubectl uncordon NODE_NAME

# SSH to node
# Get instance ID
INSTANCE_ID=$(kubectl get node NODE_NAME -o jsonpath='{.spec.providerID}' | cut -d'/' -f5)

# SSM Session
aws ssm start-session --target $INSTANCE_ID

# Check kubelet logs
sudo journalctl -u kubelet -f

# Check docker/containerd
sudo systemctl status docker
sudo systemctl status containerd
```

---

## Best Practices

### Production Checklist

```yaml
# 1. Multi-AZ deployment
managedNodeGroups:
  - name: workers
    instanceType: t3.medium
    minSize: 3
    maxSize: 9
    desiredCapacity: 6
    availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]

# 2. Enable control plane logging
cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]

# 3. Private cluster endpoint
vpc:
  clusterEndpoints:
    privateAccess: true
    publicAccess: false

# 4. Use IRSA instead of instance profiles
iam:
  withOIDC: true

# 5. Enable network policies

# 6. Use pod security policies

# 7. Encrypt secrets
# Use AWS Secrets Manager or KMS

# 8. Resource quotas per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    limits.cpu: "200"
    limits.memory: 400Gi
    pods: "100"

# 9. Limit ranges
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
  namespace: production
spec:
  limits:
    - max:
        cpu: "2"
        memory: 4Gi
      min:
        cpu: 100m
        memory: 128Mi
      type: Container

# 10. Pod Disruption Budgets
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

---

## Advanced Topics

### GitOps with ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Service Mesh with AWS App Mesh

```bash
# Install App Mesh controller
helm repo add eks https://aws.github.io/eks-charts
helm install appmesh-controller eks/appmesh-controller \
  --namespace appmesh-system \
  --create-namespace
```

### Multi-Cluster Management

```bash
# Register additional clusters
aws eks update-kubeconfig --name cluster-1 --alias cluster-1
aws eks update-kubeconfig --name cluster-2 --alias cluster-2

# Switch context
kubectl config use-context cluster-1
kubectl config use-context cluster-2

# View all contexts
kubectl config get-contexts
```

---

## Practical Examples

### Complete Application Deployment

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-prod

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      serviceAccountName: myapp-sa
      containers:
        - name: app
          image: ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          env:
            - name: DATABASE_HOST
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: host

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: myapp-prod
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp-prod
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:ACCOUNT_ID:certificate/xxx
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: myapp-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

```bash
# Deploy application
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f hpa.yaml

# Verify deployment
kubectl get all -n myapp-prod
kubectl get ingress -n myapp-prod
```

---

## Conclusion

This comprehensive Amazon EKS guide covers:

1. **Setup**: Complete installation and cluster creation
2. **Node Management**: Managed nodes, Spot, and Fargate
3. **Networking**: VPC, CNI, load balancers, and network policies
4. **Storage**: EBS and EFS persistent storage
5. **Security**: IAM, RBAC, secrets, and network security
6. **Scaling**: Cluster, pod, and vertical auto-scaling
7. **Monitoring**: CloudWatch, Prometheus, and logging
8. **Cost Optimization**: Right-sizing and spot instances
9. **Best Practices**: Production-ready configurations

### Additional Resources

- [EKS Documentation](https://docs.aws.amazon.com/eks/)
- [EKS Best Practices Guide](https://aws.github.io/aws-eks-best-practices/)
- [eksctl Documentation](https://eksctl.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [AWS Workshops](https://eksworkshop.com/)

---

**Remember**: Always test in non-production environments, follow security best practices, monitor costs, and keep your cluster updated!
