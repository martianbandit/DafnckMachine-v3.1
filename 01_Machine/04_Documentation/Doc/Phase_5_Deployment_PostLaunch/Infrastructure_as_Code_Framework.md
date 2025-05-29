# Infrastructure as Code Framework

## Overview
This document provides comprehensive guidance for implementing Infrastructure as Code (IaC) for DafnckMachine v3.1, covering Terraform configurations, cloud resource management, environment provisioning, and infrastructure automation best practices.

## Terraform Configuration

### Provider Setup
```hcl
# providers.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.10"
    }
  }
  
  backend "s3" {
    bucket         = "dafnckmachine-terraform-state"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = "DafnckMachine"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

### Variables Configuration
```hcl
# variables.tf
variable "environment" {
  description = "Environment name (dev, staging, production)"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-west-2"
}

variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
  default     = "dafnckmachine"
}

variable "node_instance_types" {
  description = "EC2 instance types for EKS nodes"
  type        = list(string)
  default     = ["t3.medium", "t3.large"]
}

variable "min_nodes" {
  description = "Minimum number of nodes in the cluster"
  type        = number
  default     = 2
}

variable "max_nodes" {
  description = "Maximum number of nodes in the cluster"
  type        = number
  default     = 10
}

variable "desired_nodes" {
  description = "Desired number of nodes in the cluster"
  type        = number
  default     = 3
}
```

## Network Infrastructure

### VPC Configuration
```hcl
# vpc.tf
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "${var.cluster_name}-${var.environment}"
  cidr = "10.0.0.0/16"
  
  azs             = data.aws_availability_zones.available.names
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  enable_dns_hostnames = true
  enable_dns_support = true
  
  public_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}-${var.environment}" = "shared"
    "kubernetes.io/role/elb" = "1"
  }
  
  private_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}-${var.environment}" = "shared"
    "kubernetes.io/role/internal-elb" = "1"
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}
```

### Security Groups
```hcl
# security-groups.tf
resource "aws_security_group" "eks_cluster" {
  name_prefix = "${var.cluster_name}-${var.environment}-cluster-"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port = 443
    to_port   = 443
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.cluster_name}-${var.environment}-cluster-sg"
  }
}

resource "aws_security_group" "eks_nodes" {
  name_prefix = "${var.cluster_name}-${var.environment}-nodes-"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port = 0
    to_port   = 65535
    protocol  = "tcp"
    self      = true
  }
  
  ingress {
    from_port       = 1025
    to_port         = 65535
    protocol        = "tcp"
    security_groups = [aws_security_group.eks_cluster.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.cluster_name}-${var.environment}-nodes-sg"
  }
}
```

## EKS Cluster Configuration

### Cluster Setup
```hcl
# eks.tf
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "${var.cluster_name}-${var.environment}"
  cluster_version = "1.27"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = true
  cluster_endpoint_public_access_cidrs = ["0.0.0.0/0"]
  
  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent = true
    }
    aws-ebs-csi-driver = {
      most_recent = true
    }
  }
  
  eks_managed_node_groups = {
    main = {
      name = "main-${var.environment}"
      
      instance_types = var.node_instance_types
      
      min_size     = var.min_nodes
      max_size     = var.max_nodes
      desired_size = var.desired_nodes
      
      disk_size = 50
      disk_type = "gp3"
      
      labels = {
        Environment = var.environment
        NodeGroup   = "main"
      }
      
      update_config = {
        max_unavailable_percentage = 25
      }
    }
  }
  
  node_security_group_additional_rules = {
    ingress_self_all = {
      description = "Node to node all ports/protocols"
      protocol    = "-1"
      from_port   = 0
      to_port     = 0
      type        = "ingress"
      self        = true
    }
  }
}
```

### IRSA Configuration
```hcl
# irsa.tf
module "load_balancer_controller_irsa_role" {
  source = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  
  role_name = "${var.cluster_name}-${var.environment}-load-balancer-controller"
  
  attach_load_balancer_controller_policy = true
  
  oidc_providers = {
    ex = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:aws-load-balancer-controller"]
    }
  }
}

module "ebs_csi_irsa_role" {
  source = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  
  role_name = "${var.cluster_name}-${var.environment}-ebs-csi"
  
  attach_ebs_csi_policy = true
  
  oidc_providers = {
    ex = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:ebs-csi-controller-sa"]
    }
  }
}
```

## Database Infrastructure

### RDS Configuration
```hcl
# rds.tf
module "db" {
  source = "terraform-aws-modules/rds/aws"
  
  identifier = "${var.cluster_name}-${var.environment}"
  
  engine            = "postgres"
  engine_version    = "15.3"
  instance_class    = var.environment == "production" ? "db.r6g.large" : "db.t3.micro"
  allocated_storage = var.environment == "production" ? 100 : 20
  storage_encrypted = true
  
  db_name  = "dafnckmachine"
  username = "postgres"
  password = random_password.db_password.result
  port     = "5432"
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  maintenance_window = "Mon:00:00-Mon:03:00"
  backup_window      = "03:00-06:00"
  
  backup_retention_period = var.environment == "production" ? 30 : 7
  
  db_subnet_group_name = module.vpc.database_subnet_group
  
  family = "postgres15"
  
  major_engine_version = "15"
  
  deletion_protection = var.environment == "production"
  
  parameters = [
    {
      name  = "log_connections"
      value = "1"
    }
  ]
}

resource "random_password" "db_password" {
  length  = 16
  special = true
}

resource "aws_security_group" "rds" {
  name_prefix = "${var.cluster_name}-${var.environment}-rds-"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.eks_nodes.id]
  }
  
  tags = {
    Name = "${var.cluster_name}-${var.environment}-rds-sg"
  }
}
```

### Redis Configuration
```hcl
# redis.tf
module "redis" {
  source = "terraform-aws-modules/elasticache/aws"
  
  cluster_id           = "${var.cluster_name}-${var.environment}"
  description          = "Redis cluster for ${var.cluster_name} ${var.environment}"
  
  node_type            = var.environment == "production" ? "cache.r6g.large" : "cache.t3.micro"
  port                 = 6379
  parameter_group_name = "default.redis7"
  
  num_cache_nodes = 1
  
  subnet_group_name = module.vpc.elasticache_subnet_group_name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  auth_token = random_password.redis_password.result
}

resource "random_password" "redis_password" {
  length  = 32
  special = false
}

resource "aws_security_group" "redis" {
  name_prefix = "${var.cluster_name}-${var.environment}-redis-"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.eks_nodes.id]
  }
  
  tags = {
    Name = "${var.cluster_name}-${var.environment}-redis-sg"
  }
}
```

## Monitoring Infrastructure

### CloudWatch Configuration
```hcl
# monitoring.tf
resource "aws_cloudwatch_log_group" "eks_cluster" {
  name              = "/aws/eks/${var.cluster_name}-${var.environment}/cluster"
  retention_in_days = var.environment == "production" ? 30 : 7
}

resource "aws_cloudwatch_log_group" "application" {
  name              = "/aws/eks/${var.cluster_name}-${var.environment}/application"
  retention_in_days = var.environment == "production" ? 30 : 7
}

# SNS Topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "${var.cluster_name}-${var.environment}-alerts"
}

resource "aws_sns_topic_subscription" "email_alerts" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "${var.cluster_name}-${var.environment}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EKS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "This metric monitors eks cpu utilization"
  alarm_actions       = [aws_sns_topic.alerts.arn]
  
  dimensions = {
    ClusterName = module.eks.cluster_name
  }
}
```

## Helm Charts Deployment

### Application Deployment
```hcl
# helm.tf
provider "helm" {
  kubernetes {
    host                   = module.eks.cluster_endpoint
    cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
    
    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      command     = "aws"
      args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
    }
  }
}

resource "helm_release" "aws_load_balancer_controller" {
  name       = "aws-load-balancer-controller"
  repository = "https://aws.github.io/eks-charts"
  chart      = "aws-load-balancer-controller"
  namespace  = "kube-system"
  version    = "1.5.4"
  
  set {
    name  = "clusterName"
    value = module.eks.cluster_name
  }
  
  set {
    name  = "serviceAccount.create"
    value = "true"
  }
  
  set {
    name  = "serviceAccount.name"
    value = "aws-load-balancer-controller"
  }
  
  set {
    name  = "serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = module.load_balancer_controller_irsa_role.iam_role_arn
  }
  
  depends_on = [module.eks]
}

resource "helm_release" "metrics_server" {
  name       = "metrics-server"
  repository = "https://kubernetes-sigs.github.io/metrics-server/"
  chart      = "metrics-server"
  namespace  = "kube-system"
  version    = "3.10.0"
  
  depends_on = [module.eks]
}
```

## Environment-Specific Configurations

### Development Environment
```hcl
# environments/dev.tfvars
environment = "dev"
aws_region  = "us-west-2"

# Smaller instances for cost optimization
node_instance_types = ["t3.small"]
min_nodes          = 1
max_nodes          = 3
desired_nodes      = 2

# Reduced backup retention
db_backup_retention = 7
log_retention_days  = 7

# Development-specific settings
enable_deletion_protection = false
enable_detailed_monitoring = false
```

### Production Environment
```hcl
# environments/prod.tfvars
environment = "production"
aws_region  = "us-west-2"

# Production-grade instances
node_instance_types = ["t3.large", "t3.xlarge"]
min_nodes          = 3
max_nodes          = 20
desired_nodes      = 5

# Extended backup retention
db_backup_retention = 30
log_retention_days  = 30

# Production-specific settings
enable_deletion_protection = true
enable_detailed_monitoring = true
enable_multi_az           = true
```

## State Management

### Remote State Configuration
```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "dafnckmachine-terraform-state"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    
    # Workspace-specific state files
    workspace_key_prefix = "env:"
  }
}

# State bucket setup (run once)
resource "aws_s3_bucket" "terraform_state" {
  bucket = "dafnckmachine-terraform-state"
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_dynamodb_table" "terraform_state_lock" {
  name           = "terraform-state-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

## Outputs Configuration

### Infrastructure Outputs
```hcl
# outputs.tf
output "cluster_endpoint" {
  description = "Endpoint for EKS control plane"
  value       = module.eks.cluster_endpoint
}

output "cluster_security_group_id" {
  description = "Security group ids attached to the cluster control plane"
  value       = module.eks.cluster_security_group_id
}

output "cluster_iam_role_name" {
  description = "IAM role name associated with EKS cluster"
  value       = module.eks.cluster_iam_role_name
}

output "cluster_certificate_authority_data" {
  description = "Base64 encoded certificate data required to communicate with the cluster"
  value       = module.eks.cluster_certificate_authority_data
}

output "cluster_name" {
  description = "The name/id of the EKS cluster"
  value       = module.eks.cluster_name
}

output "vpc_id" {
  description = "ID of the VPC where the cluster is deployed"
  value       = module.vpc.vpc_id
}

output "private_subnets" {
  description = "List of IDs of private subnets"
  value       = module.vpc.private_subnets
}

output "public_subnets" {
  description = "List of IDs of public subnets"
  value       = module.vpc.public_subnets
}

output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = module.db.db_instance_endpoint
  sensitive   = true
}

output "redis_endpoint" {
  description = "Redis cluster endpoint"
  value       = module.redis.cluster_address
  sensitive   = true
}
```

## Deployment Scripts

### Infrastructure Deployment
```bash
#!/bin/bash
# scripts/deploy-infrastructure.sh

set -e

ENVIRONMENT=${1:-dev}
REGION=${2:-us-west-2}

echo "Deploying infrastructure for environment: $ENVIRONMENT"

# Initialize Terraform
terraform init

# Select workspace
terraform workspace select $ENVIRONMENT || terraform workspace new $ENVIRONMENT

# Plan deployment
terraform plan -var-file="environments/${ENVIRONMENT}.tfvars" -out=tfplan

# Apply changes
terraform apply tfplan

# Update kubeconfig
aws eks update-kubeconfig --region $REGION --name dafnckmachine-$ENVIRONMENT

echo "Infrastructure deployment completed for $ENVIRONMENT"
```

### Destruction Script
```bash
#!/bin/bash
# scripts/destroy-infrastructure.sh

set -e

ENVIRONMENT=${1:-dev}

echo "WARNING: This will destroy all infrastructure for environment: $ENVIRONMENT"
read -p "Are you sure? (yes/no): " confirm

if [ "$confirm" = "yes" ]; then
    terraform workspace select $ENVIRONMENT
    terraform destroy -var-file="environments/${ENVIRONMENT}.tfvars"
    echo "Infrastructure destroyed for $ENVIRONMENT"
else
    echo "Destruction cancelled"
fi
```

## Best Practices

### Security Guidelines
- Use IAM roles and service accounts for authentication
- Enable encryption at rest and in transit
- Implement least privilege access controls
- Regular security scanning and updates

### Cost Optimization
- Use appropriate instance types for workloads
- Implement auto-scaling policies
- Regular resource cleanup and optimization
- Monitor and alert on cost thresholds

### Reliability Patterns
- Multi-AZ deployments for production
- Automated backup and recovery procedures
- Health checks and monitoring
- Disaster recovery planning

## Troubleshooting

### Common Issues
1. **State Lock Issues**: Check DynamoDB table and release locks if needed
2. **Permission Errors**: Verify IAM roles and policies
3. **Resource Conflicts**: Check for existing resources with same names
4. **Network Issues**: Verify VPC and subnet configurations

### Debug Commands
```bash
# Check Terraform state
terraform state list
terraform state show <resource>

# Validate configuration
terraform validate
terraform fmt -check

# Import existing resources
terraform import <resource_type>.<name> <resource_id>

# Refresh state
terraform refresh
```

---

**Last Updated**: 2025-01-27  
**Version**: 1.0  
**Related Documents**: 
- [CI/CD Pipeline Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/CI_CD_Pipeline_Implementation.md)
- [Deployment Strategy Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Deployment_Strategy_Configuration.md)
- [Security Integration Framework](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Security_Integration_Framework.md) 