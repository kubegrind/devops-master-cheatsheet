# Terraform Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Installation](#installation)
- [Core Terraform Commands](#core-terraform-commands)
- [Terraform Configuration](#terraform-configuration)
- [AWS (Amazon Web Services)](#aws-amazon-web-services)
- [Azure (Microsoft Azure)](#azure-microsoft-azure)
- [GCP (Google Cloud Platform)](#gcp-google-cloud-platform)
- [Multi-Cloud Examples](#multi-cloud-examples)
- [Advanced Topics](#advanced-topics)
- [Best Practices](#best-practices)

---

## Introduction

Terraform is an Infrastructure as Code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently across multiple cloud providers.

### Key Concepts
- **Provider**: Plugin that interacts with cloud platforms (AWS, Azure, GCP, etc.)
- **Resource**: Infrastructure component (VM, network, storage, etc.)
- **Module**: Reusable Terraform configuration
- **State**: Mapping of resources to configuration
- **Plan**: Preview of changes before applying

---

## Installation

### Linux
```bash
# Download Terraform
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip

# Unzip
unzip terraform_1.7.0_linux_amd64.zip

# Move to PATH
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

### macOS
```bash
# Using Homebrew
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify
terraform version
```

### Windows
```powershell
# Using Chocolatey
choco install terraform

# Or download from https://www.terraform.io/downloads
# Add to PATH manually

# Verify
terraform version
```

---

## Core Terraform Commands

### Initialization & Setup
```bash
# Initialize working directory (downloads providers)
terraform init

# Initialize with backend configuration
terraform init -backend-config="bucket=my-bucket"

# Upgrade provider plugins
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure

# Copy modules from source
terraform init -from-module=MODULE-SOURCE
```

### Planning & Validation
```bash
# Validate configuration syntax
terraform validate

# Format configuration files
terraform fmt

# Format and list changed files
terraform fmt -list=true

# Recursively format all files
terraform fmt -recursive

# Check formatting (CI/CD)
terraform fmt -check

# Create execution plan
terraform plan

# Save plan to file
terraform plan -out=tfplan

# Plan with variable file
terraform plan -var-file="prod.tfvars"

# Plan with inline variable
terraform plan -var="instance_type=t2.micro"

# Detailed plan output
terraform plan -detailed-exitcode

# Refresh state before planning
terraform plan -refresh=true
```

### Applying Changes
```bash
# Apply changes
terraform apply

# Apply without confirmation prompt
terraform apply -auto-approve

# Apply saved plan
terraform apply tfplan

# Apply with variable file
terraform apply -var-file="prod.tfvars"

# Apply specific resource
terraform apply -target=aws_instance.web

# Apply with parallelism control
terraform apply -parallelism=10
```

### Destroying Resources
```bash
# Destroy all resources
terraform destroy

# Destroy without confirmation
terraform destroy -auto-approve

# Destroy specific resource
terraform destroy -target=aws_instance.web

# Destroy with variable file
terraform destroy -var-file="prod.tfvars"
```

### State Management
```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Pull remote state
terraform state pull

# Push local state to remote
terraform state push

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state
terraform state rm aws_instance.web

# Replace provider in state
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws

# Refresh state
terraform refresh
```

### Workspace Management
```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev

# Select workspace
terraform workspace select prod

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### Output & Console
```bash
# Show outputs
terraform output

# Show specific output
terraform output instance_ip

# Output in JSON format
terraform output -json

# Interactive console
terraform console

# Graph visualization (requires Graphviz)
terraform graph | dot -Tsvg > graph.svg
```

### Import & Taint
```bash
# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Mark resource for recreation
terraform taint aws_instance.web

# Remove taint
terraform untaint aws_instance.web
```

### Provider Management
```bash
# Show provider requirements
terraform providers

# Lock provider versions
terraform providers lock

# Show provider schemas
terraform providers schema -json
```

### Advanced Commands
```bash
# Show Terraform version
terraform version

# Get help for command
terraform plan -help

# Enable detailed logging
export TF_LOG=DEBUG
terraform apply

# Set log file
export TF_LOG_PATH=terraform.log

# Force unlock state
terraform force-unlock LOCK_ID

# Test configuration
terraform test
```

---

## Terraform Configuration

### Basic Structure
```hcl
# main.tf
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "WebServer"
    Environment = var.environment
  }
}
```

### Variables
```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "allowed_ports" {
  description = "List of allowed ports"
  type        = list(number)
  default     = [80, 443]
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

### Outputs
```hcl
# outputs.tf
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address"
  value       = aws_instance.web.public_ip
}

output "instance_private_ip" {
  description = "Private IP address"
  value       = aws_instance.web.private_ip
  sensitive   = true
}
```

### Variable Files
```hcl
# terraform.tfvars
aws_region    = "us-west-2"
instance_type = "t3.medium"

tags = {
  Environment = "Production"
  Team        = "DevOps"
}

# prod.tfvars
environment = "production"
instance_type = "t3.large"

# dev.tfvars
environment = "development"
instance_type = "t2.micro"
```

---

## AWS (Amazon Web Services)

### Provider Configuration
```hcl
# AWS Provider Setup
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region     = "us-east-1"
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key

  # Alternative: Use profile
  profile = "default"

  # Alternative: Use assume role
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}

# Multiple AWS providers
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```

### EC2 Instances
```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  key_name              = aws_key_pair.deployer.key_name
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id             = aws_subnet.public.id

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              EOF

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
    encrypted   = true
  }

  tags = {
    Name = "WebServer"
  }
}

# Key Pair
resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

# Elastic IP
resource "aws_eip" "web" {
  instance = aws_instance.web.id
  domain   = "vpc"
}
```

### VPC & Networking
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Subnets
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1b"

  tags = {
    Name = "private-subnet"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-rt"
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "main-nat"
  }
}
```

### Security Groups
```hcl
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
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
    Name = "web-sg"
  }
}
```

### S3 Buckets
```hcl
resource "aws_s3_bucket" "app" {
  bucket = "my-app-bucket-${random_id.bucket_suffix.hex}"

  tags = {
    Name = "App Bucket"
  }
}

resource "aws_s3_bucket_versioning" "app" {
  bucket = aws_s3_bucket.app.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_public_access_block" "app" {
  bucket = aws_s3_bucket.app.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "app" {
  bucket = aws_s3_bucket.app.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}
```

### RDS Database
```hcl
resource "aws_db_instance" "postgres" {
  identifier             = "myapp-db"
  engine                 = "postgres"
  engine_version         = "15.3"
  instance_class         = "db.t3.micro"
  allocated_storage      = 20
  storage_type           = "gp3"
  storage_encrypted      = true

  db_name  = "myappdb"
  username = var.db_username
  password = var.db_password

  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  skip_final_snapshot = false
  final_snapshot_identifier = "myapp-db-final-snapshot"

  tags = {
    Name = "MyApp Database"
  }
}

resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = [aws_subnet.private.id, aws_subnet.private2.id]

  tags = {
    Name = "Main DB Subnet Group"
  }
}
```

### ECS & Fargate
```hcl
resource "aws_ecs_cluster" "main" {
  name = "app-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

resource "aws_ecs_task_definition" "app" {
  family                   = "app-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
  execution_role_arn       = aws_iam_role.ecs_execution.arn

  container_definitions = jsonencode([
    {
      name  = "app"
      image = "nginx:latest"
      portMappings = [
        {
          containerPort = 80
          protocol      = "tcp"
        }
      ]
    }
  ])
}

resource "aws_ecs_service" "app" {
  name            = "app-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = [aws_subnet.private.id]
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 80
  }
}
```

### Lambda Functions
```hcl
resource "aws_lambda_function" "api" {
  filename      = "lambda_function.zip"
  function_name = "api_handler"
  role          = aws_iam_role.lambda.arn
  handler       = "index.handler"
  runtime       = "nodejs18.x"

  source_code_hash = filebase64sha256("lambda_function.zip")

  environment {
    variables = {
      ENV = "production"
    }
  }

  timeout     = 30
  memory_size = 256
}

resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api.function_name
  principal     = "apigateway.amazonaws.com"
}
```

### IAM Roles & Policies
```hcl
resource "aws_iam_role" "ec2_role" {
  name = "ec2-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ec2_s3" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}

resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-profile"
  role = aws_iam_role.ec2_role.name
}
```

### Load Balancers
```hcl
resource "aws_lb" "app" {
  name               = "app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb.id]
  subnets            = [aws_subnet.public.id, aws_subnet.public2.id]

  enable_deletion_protection = false
}

resource "aws_lb_target_group" "app" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
  }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

### Auto Scaling
```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "web-"
  image_id      = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              EOF
  )
}

resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = [aws_subnet.public.id, aws_subnet.public2.id]
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"

  min_size         = 2
  max_size         = 6
  desired_capacity = 3

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "web-server"
    propagate_at_launch = true
  }
}

resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  scaling_adjustment     = 1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.web.name
}
```

---

## Azure (Microsoft Azure)

### Provider Configuration
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}

  subscription_id = var.azure_subscription_id
  client_id       = var.azure_client_id
  client_secret   = var.azure_client_secret
  tenant_id       = var.azure_tenant_id

  # Alternative: Use Azure CLI authentication
  # az login
}
```

### Resource Groups
```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-production"
  location = "East US"

  tags = {
    Environment = "Production"
    Team        = "DevOps"
  }
}
```

### Virtual Machines
```hcl
resource "azurerm_virtual_network" "main" {
  name                = "vnet-main"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "subnet-internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "main" {
  name                = "nic-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

resource "azurerm_public_ip" "main" {
  name                = "pip-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_linux_virtual_machine" "web" {
  name                = "vm-web"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B2s"
  admin_username      = "adminuser"

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 30
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  custom_data = base64encode(<<-EOF
                #!/bin/bash
                apt-get update
                apt-get install -y nginx
                systemctl start nginx
                EOF
  )
}
```

### Network Security Groups
```hcl
resource "azurerm_network_security_group" "web" {
  name                = "nsg-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "HTTP"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "HTTPS"
    priority                   = 1003
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "web" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.web.id
}
```

### Storage Accounts
```hcl
resource "azurerm_storage_account" "app" {
  name                     = "stappdata${random_string.suffix.result}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "GRS"

  blob_properties {
    versioning_enabled = true

    delete_retention_policy {
      days = 7
    }
  }

  network_rules {
    default_action = "Deny"
    ip_rules       = ["100.0.0.1"]
    bypass         = ["AzureServices"]
  }
}

resource "azurerm_storage_container" "data" {
  name                  = "data"
  storage_account_name  = azurerm_storage_account.app.name
  container_access_type = "private"
}

resource "random_string" "suffix" {
  length  = 8
  special = false
  upper   = false
}
```

### SQL Database
```hcl
resource "azurerm_mssql_server" "main" {
  name                         = "sqlserver-${random_string.suffix.result}"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = var.sql_admin_username
  administrator_login_password = var.sql_admin_password

  minimum_tls_version = "1.2"
}

resource "azurerm_mssql_database" "app" {
  name           = "appdb"
  server_id      = azurerm_mssql_server.main.id
  collation      = "SQL_Latin1_General_CP1_CI_AS"
  license_type   = "LicenseIncluded"
  max_size_gb    = 2
  sku_name       = "S0"
  zone_redundant = false
}

resource "azurerm_mssql_firewall_rule" "allow_azure" {
  name             = "AllowAzureServices"
  server_id        = azurerm_mssql_server.main.id
  start_ip_address = "0.0.0.0"
  end_ip_address   = "0.0.0.0"
}
```

### AKS (Azure Kubernetes Service)
```hcl
resource "azurerm_kubernetes_cluster" "main" {
  name                = "aks-cluster"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "myaks"

  default_node_pool {
    name            = "default"
    node_count      = 3
    vm_size         = "Standard_D2_v2"
    os_disk_size_gb = 30
    vnet_subnet_id  = azurerm_subnet.internal.id
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    load_balancer_sku = "standard"
  }
}

output "kube_config" {
  value     = azurerm_kubernetes_cluster.main.kube_config_raw
  sensitive = true
}
```

### App Service
```hcl
resource "azurerm_service_plan" "main" {
  name                = "asp-webapp"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = "P1v2"
}

resource "azurerm_linux_web_app" "main" {
  name                = "webapp-${random_string.suffix.result}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_service_plan.main.location
  service_plan_id     = azurerm_service_plan.main.id

  site_config {
    application_stack {
      node_version = "18-lts"
    }

    always_on = true
  }

  app_settings = {
    "WEBSITE_NODE_DEFAULT_VERSION" = "~18"
    "ENV"                          = "production"
  }
}
```

### Load Balancer
```hcl
resource "azurerm_lb" "main" {
  name                = "lb-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "Standard"

  frontend_ip_configuration {
    name                 = "PublicIPAddress"
    public_ip_address_id = azurerm_public_ip.lb.id
  }
}

resource "azurerm_public_ip" "lb" {
  name                = "pip-lb"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_lb_backend_address_pool" "main" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "BackEndAddressPool"
}

resource "azurerm_lb_probe" "http" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "http-probe"
  port            = 80
  protocol        = "Http"
  request_path    = "/health"
}

resource "azurerm_lb_rule" "http" {
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "HTTP"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "PublicIPAddress"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.main.id]
  probe_id                       = azurerm_lb_probe.http.id
}
```

---

## GCP (Google Cloud Platform)

### Provider Configuration
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project     = var.gcp_project_id
  region      = "us-central1"
  zone        = "us-central1-a"
  credentials = file("service-account-key.json")

  # Alternative: Use Application Default Credentials
  # gcloud auth application-default login
}

provider "google-beta" {
  project     = var.gcp_project_id
  region      = "us-central1"
  credentials = file("service-account-key.json")
}
```

### Compute Instances
```hcl
resource "google_compute_instance" "web" {
  name         = "web-server"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      size  = 20
      type  = "pd-standard"
    }
  }

  network_interface {
    network    = google_compute_network.vpc.id
    subnetwork = google_compute_subnetwork.subnet.id

    access_config {
      // Ephemeral public IP
    }
  }

  metadata = {
    ssh-keys = "admin:${file("~/.ssh/id_rsa.pub")}"
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
  EOF

  tags = ["web", "http-server", "https-server"]

  labels = {
    environment = "production"
  }
}

resource "google_compute_address" "static" {
  name = "ipv4-address"
}
```

### VPC & Networking
```hcl
resource "google_compute_network" "vpc" {
  name                    = "main-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "main-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-central1"
  network       = google_compute_network.vpc.id

  secondary_ip_range {
    range_name    = "services-range"
    ip_cidr_range = "192.168.1.0/24"
  }

  secondary_ip_range {
    range_name    = "pod-ranges"
    ip_cidr_range = "192.168.64.0/22"
  }
}

resource "google_compute_router" "router" {
  name    = "main-router"
  region  = google_compute_subnetwork.subnet.region
  network = google_compute_network.vpc.id
}

resource "google_compute_router_nat" "nat" {
  name                               = "main-nat"
  router                             = google_compute_router.router.name
  region                             = google_compute_router.router.region
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
}
```

### Firewall Rules
```hcl
resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["http-server"]
}

resource "google_compute_firewall" "allow_https" {
  name    = "allow-https"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["https-server"]
}

resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["ssh"]
}
```

### Cloud Storage
```hcl
resource "google_storage_bucket" "app" {
  name          = "app-bucket-${random_id.bucket_suffix.hex}"
  location      = "US"
  force_destroy = false

  uniform_bucket_level_access = true

  versioning {
    enabled = true
  }

  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "Delete"
    }
  }

  encryption {
    default_kms_key_name = google_kms_crypto_key.bucket_key.id
  }

  labels = {
    environment = "production"
  }
}

resource "google_storage_bucket_object" "config" {
  name   = "config/app.json"
  bucket = google_storage_bucket.app.name
  source = "config/app.json"
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}
```

### Cloud SQL
```hcl
resource "google_sql_database_instance" "postgres" {
  name             = "postgres-instance"
  database_version = "POSTGRES_15"
  region           = "us-central1"

  settings {
    tier              = "db-f1-micro"
    availability_type = "ZONAL"
    disk_size         = 20
    disk_type         = "PD_SSD"

    backup_configuration {
      enabled                        = true
      start_time                     = "03:00"
      point_in_time_recovery_enabled = true
      backup_retention_settings {
        retained_backups = 7
      }
    }

    ip_configuration {
      ipv4_enabled    = true
      private_network = google_compute_network.vpc.id

      authorized_networks {
        name  = "office"
        value = "203.0.113.0/24"
      }
    }

    database_flags {
      name  = "max_connections"
      value = "100"
    }
  }

  deletion_protection = true
}

resource "google_sql_database" "app" {
  name     = "appdb"
  instance = google_sql_database_instance.postgres.name
}

resource "google_sql_user" "app" {
  name     = var.db_username
  instance = google_sql_database_instance.postgres.name
  password = var.db_password
}
```

### GKE (Google Kubernetes Engine)
```hcl
resource "google_container_cluster" "primary" {
  name     = "gke-cluster"
  location = "us-central1"

  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.vpc.name
  subnetwork = google_compute_subnetwork.subnet.name

  ip_allocation_policy {
    cluster_secondary_range_name  = "pod-ranges"
    services_secondary_range_name = "services-range"
  }

  master_auth {
    client_certificate_config {
      issue_client_certificate = false
    }
  }

  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
  }
}

resource "google_container_node_pool" "primary_nodes" {
  name       = "primary-node-pool"
  location   = "us-central1"
  cluster    = google_container_cluster.primary.name
  node_count = 3

  node_config {
    preemptible  = false
    machine_type = "e2-medium"
    disk_size_gb = 50
    disk_type    = "pd-standard"

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      env = "production"
    }

    tags = ["gke-node"]
  }

  autoscaling {
    min_node_count = 1
    max_node_count = 5
  }
}
```

### Cloud Functions
```hcl
resource "google_storage_bucket" "function_bucket" {
  name     = "function-source-${random_id.bucket_suffix.hex}"
  location = "US"
}

resource "google_storage_bucket_object" "function_source" {
  name   = "function-source.zip"
  bucket = google_storage_bucket.function_bucket.name
  source = "function-source.zip"
}

resource "google_cloudfunctions_function" "api" {
  name        = "api-function"
  description = "API Handler"
  runtime     = "nodejs18"

  available_memory_mb   = 256
  source_archive_bucket = google_storage_bucket.function_bucket.name
  source_archive_object = google_storage_bucket_object.function_source.name
  trigger_http          = true
  entry_point           = "handler"

  environment_variables = {
    ENV = "production"
  }
}

resource "google_cloudfunctions_function_iam_member" "invoker" {
  project        = google_cloudfunctions_function.api.project
  region         = google_cloudfunctions_function.api.region
  cloud_function = google_cloudfunctions_function.api.name

  role   = "roles/cloudfunctions.invoker"
  member = "allUsers"
}
```

### Load Balancer
```hcl
resource "google_compute_global_address" "default" {
  name = "lb-ipv4-address"
}

resource "google_compute_health_check" "http" {
  name = "http-health-check"

  http_health_check {
    port         = 80
    request_path = "/health"
  }
}

resource "google_compute_backend_service" "default" {
  name          = "backend-service"
  protocol      = "HTTP"
  timeout_sec   = 10
  health_checks = [google_compute_health_check.http.id]

  backend {
    group = google_compute_instance_group_manager.web.instance_group
  }
}

resource "google_compute_url_map" "default" {
  name            = "url-map"
  default_service = google_compute_backend_service.default.id
}

resource "google_compute_target_http_proxy" "default" {
  name    = "http-proxy"
  url_map = google_compute_url_map.default.id
}

resource "google_compute_global_forwarding_rule" "default" {
  name       = "forwarding-rule"
  target     = google_compute_target_http_proxy.default.id
  port_range = "80"
  ip_address = google_compute_global_address.default.address
}
```

### Instance Groups & Auto Scaling
```hcl
resource "google_compute_instance_template" "web" {
  name_prefix  = "web-template-"
  machine_type = "e2-medium"

  disk {
    source_image = "debian-cloud/debian-11"
    auto_delete  = true
    boot         = true
  }

  network_interface {
    network    = google_compute_network.vpc.id
    subnetwork = google_compute_subnetwork.subnet.id
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
  EOF

  tags = ["web", "http-server"]

  lifecycle {
    create_before_destroy = true
  }
}

resource "google_compute_instance_group_manager" "web" {
  name = "web-igm"
  zone = "us-central1-a"

  base_instance_name = "web"

  version {
    instance_template = google_compute_instance_template.web.id
  }

  target_size = 3

  named_port {
    name = "http"
    port = 80
  }

  auto_healing_policies {
    health_check      = google_compute_health_check.http.id
    initial_delay_sec = 300
  }
}

resource "google_compute_autoscaler" "web" {
  name   = "web-autoscaler"
  zone   = "us-central1-a"
  target = google_compute_instance_group_manager.web.id

  autoscaling_policy {
    max_replicas    = 10
    min_replicas    = 2
    cooldown_period = 60

    cpu_utilization {
      target = 0.7
    }
  }
}
```

---

## Multi-Cloud Examples

### Multi-Cloud Deployment
```hcl
# Deploy to AWS, Azure, and GCP simultaneously

# AWS EC2
resource "aws_instance" "app" {
  provider      = aws
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name  = "App-AWS"
    Cloud = "AWS"
  }
}

# Azure VM
resource "azurerm_linux_virtual_machine" "app" {
  provider            = azurerm
  name                = "vm-app-azure"
  resource_group_name = azurerm_resource_group.main.name
  location            = "East US"
  size                = "Standard_B2s"
  admin_username      = "adminuser"

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  tags = {
    Name  = "App-Azure"
    Cloud = "Azure"
  }
}

# GCP Compute Instance
resource "google_compute_instance" "app" {
  provider     = google
  name         = "app-gcp"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }

  labels = {
    name  = "app-gcp"
    cloud = "gcp"
  }
}
```

### Multi-Region Deployment
```hcl
# Deploy to multiple AWS regions
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

provider "aws" {
  alias  = "eu_west"
  region = "eu-west-1"
}

resource "aws_instance" "us_east" {
  provider      = aws.us_east
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name   = "App-US-East"
    Region = "us-east-1"
  }
}

resource "aws_instance" "us_west" {
  provider      = aws.us_west
  ami           = "ami-0d1cd67c26f5fca19"
  instance_type = "t2.micro"

  tags = {
    Name   = "App-US-West"
    Region = "us-west-2"
  }
}

resource "aws_instance" "eu_west" {
  provider      = aws.eu_west
  ami           = "ami-0dad359ff462124ca"
  instance_type = "t2.micro"

  tags = {
    Name   = "App-EU-West"
    Region = "eu-west-1"
  }
}
```

---

## Advanced Topics

### Modules
```hcl
# modules/vpc/main.tf
variable "vpc_cidr" {
  type = string
}

variable "environment" {
  type = string
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name        = "vpc-${var.environment}"
    Environment = var.environment
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

# Root main.tf - Using the module
module "vpc_dev" {
  source = "./modules/vpc"

  vpc_cidr    = "10.0.0.0/16"
  environment = "dev"
}

module "vpc_prod" {
  source = "./modules/vpc"

  vpc_cidr    = "10.1.0.0/16"
  environment = "prod"
}

output "dev_vpc_id" {
  value = module.vpc_dev.vpc_id
}
```

### Remote State
```hcl
# S3 Backend (AWS)
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Azure Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# GCS Backend (GCP)
terraform {
  backend "gcs" {
    bucket = "tf-state-bucket"
    prefix = "prod"
  }
}

# Terraform Cloud
terraform {
  backend "remote" {
    organization = "my-org"

    workspaces {
      name = "production"
    }
  }
}
```

### Data Sources
```hcl
# AWS
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_caller_identity" "current" {}

# Azure
data "azurerm_client_config" "current" {}

data "azurerm_subscription" "current" {}

# GCP
data "google_compute_zones" "available" {
  region = "us-central1"
}

data "google_project" "current" {}
```

### Dynamic Blocks
```hcl
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP"
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS"
    }
  ]
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
}
```

### Count & For Each
```hcl
# Using count
resource "aws_instance" "web" {
  count = 3

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "web-${count.index}"
  }
}

# Using for_each with map
variable "instances" {
  type = map(object({
    instance_type = string
    ami           = string
  }))
  default = {
    web = {
      instance_type = "t2.micro"
      ami           = "ami-0c55b159cbfafe1f0"
    }
    app = {
      instance_type = "t2.small"
      ami           = "ami-0c55b159cbfafe1f0"
    }
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances

  ami           = each.value.ami
  instance_type = each.value.instance_type

  tags = {
    Name = each.key
  }
}

# Using for_each with set
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "charlie"])

  name = each.key
}
```

### Conditional Expressions
```hcl
variable "environment" {
  type = string
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.environment == "prod" ? "t3.large" : "t2.micro"

  tags = {
    Name = var.environment == "prod" ? "Production Server" : "Development Server"
  }
}

# Conditional resource creation
resource "aws_instance" "optional" {
  count = var.create_instance ? 1 : 0

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

### Local Values
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
  }

  name_prefix = "${var.project_name}-${var.environment}"

  vpc_cidr = var.environment == "prod" ? "10.0.0.0/16" : "10.1.0.0/16"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-web"
    }
  )
}
```

---

## Best Practices

### Directory Structure
```
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── terraform.tfvars
├── dev.tfvars
├── prod.tfvars
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ec2/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── rds/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/
    ├── dev/
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prod/
        ├── main.tf
        └── terraform.tfvars
```

### Version Constraints
```hcl
# versions.tf
terraform {
  required_version = ">= 1.0, < 2.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}
```

### Naming Conventions
```hcl
# Resource naming patterns
# Format: <resource_type>-<environment>-<region>-<purpose>

resource "aws_instance" "web" {
  tags = {
    Name = "ec2-prod-us-east-1-web"
  }
}

resource "azurerm_virtual_network" "main" {
  name = "vnet-prod-eastus-main"
}

resource "google_compute_instance" "app" {
  name = "vm-prod-us-central1-app"
}
```

### Security Best Practices
```hcl
# Never hardcode credentials
# Use environment variables or secret management

# Bad
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}

# Good
provider "aws" {
  # Uses AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY env vars
}

# Use sensitive flag for outputs
output "db_password" {
  value     = aws_db_instance.main.password
  sensitive = true
}

# Use AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### State File Security
```bash
# Encrypt state files
# Enable versioning
# Restrict access with IAM/RBAC
# Never commit state files to version control

# .gitignore
*.tfstate
*.tfstate.*
.terraform/
*.tfvars
!example.tfvars
```

### CI/CD Integration
```yaml
# GitHub Actions example
name: Terraform

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.7.0

    - name: Terraform Init
      run: terraform init

    - name: Terraform Format
      run: terraform fmt -check

    - name: Terraform Validate
      run: terraform validate

    - name: Terraform Plan
      run: terraform plan -out=tfplan

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve tfplan
```

### Common Commands Workflow
```bash
# Complete workflow
terraform fmt              # Format code
terraform validate         # Validate syntax
terraform init            # Initialize
terraform plan            # Preview changes
terraform apply           # Apply changes
terraform output          # Show outputs
terraform state list      # List resources
terraform destroy         # Cleanup (when needed)

# CI/CD workflow
terraform fmt -check -recursive
terraform init -backend-config=backend.hcl
terraform validate
terraform plan -out=tfplan -var-file=prod.tfvars
terraform apply tfplan
```

---

## Useful Tips

### Terraform Environment Variables
```bash
# Logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

# Input variables
export TF_VAR_region=us-east-1
export TF_VAR_instance_type=t2.micro

# CLI configuration
export TF_CLI_ARGS_plan="-parallelism=30"
export TF_CLI_ARGS_apply="-auto-approve"

# Workspace
export TF_WORKSPACE=production
```

### Quick Reference Commands
```bash
# State manipulation
terraform state list
terraform state show <resource>
terraform state mv <source> <destination>
terraform state rm <resource>
terraform state pull > backup.tfstate

# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0
terraform import azurerm_resource_group.main /subscriptions/.../resourceGroups/mygroup
terraform import google_compute_instance.default projects/myproject/zones/us-central1-a/instances/myinstance

# Debugging
terraform console
terraform graph
terraform show
terraform providers

# Cleanup
rm -rf .terraform
rm terraform.tfstate*
rm tfplan
```

---

## Conclusion

This guide covers comprehensive Terraform commands and configurations for AWS, Azure, and GCP. Use this as a reference for:
- Infrastructure provisioning
- Multi-cloud deployments
- State management
- Best practices implementation
- CI/CD integration

For more information, visit:
- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Provider](https://registry.terraform.io/providers/hashicorp/aws)
- [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm)
- [Google Provider](https://registry.terraform.io/providers/hashicorp/google)
