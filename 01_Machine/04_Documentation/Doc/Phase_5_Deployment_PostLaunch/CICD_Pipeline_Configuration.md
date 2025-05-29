# CI/CD Pipeline Configuration - DafnckMachine v3.1

## Overview
Comprehensive CI/CD pipeline configuration for the DafnckMachine v3.1 project, covering automated testing, building, deployment, and monitoring across multiple environments.

## Pipeline Architecture

### CI/CD Strategy
- **Continuous Integration**: Automated testing and building on every commit
- **Continuous Deployment**: Automated deployment to staging and production
- **GitOps Approach**: Infrastructure and deployment configuration as code
- **Multi-Environment**: Development, staging, and production environments
- **Security First**: Integrated security scanning and compliance checks

### Pipeline Stages
1. **Code Quality**: Linting, formatting, and static analysis
2. **Testing**: Unit, integration, and E2E tests
3. **Security**: Vulnerability scanning and dependency checks
4. **Build**: Application building and artifact creation
5. **Deploy**: Environment-specific deployments
6. **Monitor**: Post-deployment health checks and monitoring

## GitHub Actions Configuration

### Main Workflow File

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  release:
    types: [published]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Code Quality and Linting
  code-quality:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run Prettier check
        run: npm run format:check

      - name: TypeScript type check
        run: npm run type-check

      - name: Run security audit
        run: npm audit --audit-level=high

  # Unit and Integration Tests
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    needs: code-quality
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run database migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Run unit tests
        run: npm run test:unit
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

  # E2E Tests
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Build application
        run: npm run build

      - name: Start application
        run: |
          npm run start &
          npx wait-on http://localhost:3000

      - name: Run E2E tests
        run: npm run test:e2e
        env:
          BASE_URL: http://localhost:3000

      - name: Upload E2E test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

  # Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: code-quality
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript

  # Build and Push Docker Images
  build:
    name: Build and Push
    runs-on: ubuntu-latest
    needs: [test, e2e-tests, security]
    if: github.event_name != 'pull_request'
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster dafnckmachine-staging \
            --service dafnckmachine-app \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster dafnckmachine-staging \
            --services dafnckmachine-app

      - name: Run smoke tests
        run: |
          curl -f https://staging.dafnckmachine.com/health || exit 1

  # Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_PROD }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}
          aws-region: us-east-1

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster dafnckmachine-production \
            --service dafnckmachine-app \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster dafnckmachine-production \
            --services dafnckmachine-app

      - name: Run smoke tests
        run: |
          curl -f https://dafnckmachine.com/health || exit 1

      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Environment-Specific Workflows

```yaml
# .github/workflows/staging-deploy.yml
name: Staging Deployment

on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: 'Docker image tag to deploy'
        required: true
        default: 'develop'

jobs:
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Update ECS task definition
        run: |
          TASK_DEFINITION=$(aws ecs describe-task-definition \
            --task-definition dafnckmachine-staging \
            --query taskDefinition)
          
          NEW_TASK_DEFINITION=$(echo $TASK_DEFINITION | jq \
            --arg IMAGE "ghcr.io/${{ github.repository }}:${{ github.event.inputs.image_tag }}" \
            '.containerDefinitions[0].image = $IMAGE')
          
          echo $NEW_TASK_DEFINITION | jq 'del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .placementConstraints, .compatibilities, .registeredAt, .registeredBy)' > task-definition.json

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: task-definition.json
          service: dafnckmachine-app
          cluster: dafnckmachine-staging
          wait-for-service-stability: true
```

## Docker Configuration

### Multi-Stage Dockerfile

```dockerfile
# Dockerfile
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Development stage
FROM base AS development
RUN npm ci
COPY . .
EXPOSE 3000 4000
CMD ["npm", "run", "dev"]

# Build stage
FROM base AS build
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy built application
COPY --from=build --chown=nextjs:nodejs /app/.next ./.next
COPY --from=build --chown=nextjs:nodejs /app/public ./public
COPY --from=build --chown=nextjs:nodejs /app/package*.json ./
COPY --from=build --chown=nextjs:nodejs /app/node_modules ./node_modules

USER nextjs

EXPOSE 3000
ENV NODE_ENV=production
ENV PORT=3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["npm", "start"]
```

### Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    ports:
      - "3000:3000"
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/dafnckmachine_dev
      - REDIS_URL=redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: dafnckmachine_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Infrastructure as Code

### Terraform Configuration

```hcl
# infrastructure/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "dafnckmachine-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC Configuration
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "${var.project_name}-${var.environment}"
  cidr = var.vpc_cidr
  
  azs             = var.availability_zones
  private_subnets = var.private_subnets
  public_subnets  = var.public_subnets
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = var.common_tags
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.project_name}-${var.environment}"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
  
  tags = var.common_tags
}

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "${var.project_name}-${var.environment}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets           = module.vpc.public_subnets
  
  enable_deletion_protection = var.environment == "production"
  
  tags = var.common_tags
}

# ECS Service
resource "aws_ecs_service" "app" {
  name            = "${var.project_name}-app"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.app_count
  
  launch_type = "FARGATE"
  
  network_configuration {
    security_groups = [aws_security_group.ecs_tasks.id]
    subnets         = module.vpc.private_subnets
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 3000
  }
  
  depends_on = [aws_lb_listener.app]
  
  tags = var.common_tags
}
```

### Environment Variables

```hcl
# infrastructure/variables.tf
variable "project_name" {
  description = "Name of the project"
  type        = string
  default     = "dafnckmachine"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "private_subnets" {
  description = "Private subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

variable "public_subnets" {
  description = "Public subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}

variable "app_count" {
  description = "Number of app instances"
  type        = number
  default     = 2
}

variable "common_tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default = {
    Project     = "DafnckMachine"
    ManagedBy   = "Terraform"
  }
}
```

## Deployment Strategies

### Blue-Green Deployment

```yaml
# .github/workflows/blue-green-deploy.yml
name: Blue-Green Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'production'
        type: choice
        options:
        - staging
        - production

jobs:
  deploy:
    name: Blue-Green Deploy
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Get current deployment
        id: current
        run: |
          CURRENT_COLOR=$(aws elbv2 describe-target-groups \
            --names "dafnckmachine-${{ github.event.inputs.environment }}-blue" \
            --query 'TargetGroups[0].HealthyThresholdCount' \
            --output text 2>/dev/null || echo "0")
          
          if [ "$CURRENT_COLOR" != "0" ]; then
            echo "current=blue" >> $GITHUB_OUTPUT
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "current=green" >> $GITHUB_OUTPUT
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

      - name: Deploy to target environment
        run: |
          aws ecs update-service \
            --cluster "dafnckmachine-${{ github.event.inputs.environment }}" \
            --service "dafnckmachine-app-${{ steps.current.outputs.target }}" \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster "dafnckmachine-${{ github.event.inputs.environment }}" \
            --services "dafnckmachine-app-${{ steps.current.outputs.target }}"

      - name: Run health checks
        run: |
          TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups \
            --names "dafnckmachine-${{ github.event.inputs.environment }}-${{ steps.current.outputs.target }}" \
            --query 'TargetGroups[0].TargetGroupArn' \
            --output text)
          
          # Wait for healthy targets
          aws elbv2 wait target-in-service \
            --target-group-arn $TARGET_GROUP_ARN

      - name: Switch traffic
        run: |
          LISTENER_ARN=$(aws elbv2 describe-listeners \
            --load-balancer-arn $(aws elbv2 describe-load-balancers \
              --names "dafnckmachine-${{ github.event.inputs.environment }}-alb" \
              --query 'LoadBalancers[0].LoadBalancerArn' \
              --output text) \
            --query 'Listeners[0].ListenerArn' \
            --output text)
          
          TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups \
            --names "dafnckmachine-${{ github.event.inputs.environment }}-${{ steps.current.outputs.target }}" \
            --query 'TargetGroups[0].TargetGroupArn' \
            --output text)
          
          aws elbv2 modify-listener \
            --listener-arn $LISTENER_ARN \
            --default-actions Type=forward,TargetGroupArn=$TARGET_GROUP_ARN

      - name: Verify deployment
        run: |
          sleep 30
          curl -f "https://${{ github.event.inputs.environment }}.dafnckmachine.com/health" || exit 1
```

### Canary Deployment

```yaml
# .github/workflows/canary-deploy.yml
name: Canary Deployment

on:
  workflow_dispatch:
    inputs:
      traffic_percentage:
        description: 'Percentage of traffic to route to canary'
        required: true
        default: '10'
        type: choice
        options:
        - '10'
        - '25'
        - '50'
        - '100'

jobs:
  canary-deploy:
    name: Canary Deploy
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy canary version
        run: |
          aws ecs update-service \
            --cluster dafnckmachine-production \
            --service dafnckmachine-app-canary \
            --force-new-deployment

      - name: Wait for canary deployment
        run: |
          aws ecs wait services-stable \
            --cluster dafnckmachine-production \
            --services dafnckmachine-app-canary

      - name: Update traffic routing
        run: |
          LISTENER_ARN=$(aws elbv2 describe-listeners \
            --load-balancer-arn $(aws elbv2 describe-load-balancers \
              --names dafnckmachine-production-alb \
              --query 'LoadBalancers[0].LoadBalancerArn' \
              --output text) \
            --query 'Listeners[0].ListenerArn' \
            --output text)
          
          STABLE_TG_ARN=$(aws elbv2 describe-target-groups \
            --names dafnckmachine-production-stable \
            --query 'TargetGroups[0].TargetGroupArn' \
            --output text)
          
          CANARY_TG_ARN=$(aws elbv2 describe-target-groups \
            --names dafnckmachine-production-canary \
            --query 'TargetGroups[0].TargetGroupArn' \
            --output text)
          
          STABLE_WEIGHT=$((100 - ${{ github.event.inputs.traffic_percentage }}))
          CANARY_WEIGHT=${{ github.event.inputs.traffic_percentage }}
          
          aws elbv2 modify-listener \
            --listener-arn $LISTENER_ARN \
            --default-actions Type=forward,ForwardConfig='{
              "TargetGroups": [
                {
                  "TargetGroupArn": "'$STABLE_TG_ARN'",
                  "Weight": '$STABLE_WEIGHT'
                },
                {
                  "TargetGroupArn": "'$CANARY_TG_ARN'",
                  "Weight": '$CANARY_WEIGHT'
                }
              ]
            }'

      - name: Monitor canary metrics
        run: |
          echo "Monitoring canary deployment for 10 minutes..."
          sleep 600
          
          # Check error rates and response times
          # This would integrate with your monitoring system
```

## Environment Management

### Environment Configuration

```bash
# scripts/setup-environment.sh
#!/bin/bash

set -e

ENVIRONMENT=${1:-development}
AWS_REGION=${2:-us-east-1}

echo "Setting up $ENVIRONMENT environment in $AWS_REGION..."

# Create S3 bucket for Terraform state
aws s3 mb s3://dafnckmachine-terraform-state-$ENVIRONMENT --region $AWS_REGION

# Enable versioning on state bucket
aws s3api put-bucket-versioning \
  --bucket dafnckmachine-terraform-state-$ENVIRONMENT \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name dafnckmachine-terraform-locks-$ENVIRONMENT \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region $AWS_REGION

# Initialize Terraform
cd infrastructure
terraform init \
  -backend-config="bucket=dafnckmachine-terraform-state-$ENVIRONMENT" \
  -backend-config="key=infrastructure/terraform.tfstate" \
  -backend-config="region=$AWS_REGION" \
  -backend-config="dynamodb_table=dafnckmachine-terraform-locks-$ENVIRONMENT"

# Plan and apply infrastructure
terraform plan -var="environment=$ENVIRONMENT" -out=tfplan
terraform apply tfplan

echo "$ENVIRONMENT environment setup complete!"
```

### Secrets Management

```yaml
# .github/workflows/secrets-rotation.yml
name: Secrets Rotation

on:
  schedule:
    - cron: '0 2 1 * *' # Monthly on the 1st at 2 AM
  workflow_dispatch:

jobs:
  rotate-secrets:
    name: Rotate Secrets
    runs-on: ubuntu-latest
    
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Rotate database password
        run: |
          NEW_PASSWORD=$(openssl rand -base64 32)
          
          # Update password in RDS
          aws rds modify-db-instance \
            --db-instance-identifier dafnckmachine-production \
            --master-user-password $NEW_PASSWORD \
            --apply-immediately
          
          # Update secret in Secrets Manager
          aws secretsmanager update-secret \
            --secret-id dafnckmachine/database/password \
            --secret-string $NEW_PASSWORD

      - name: Update application configuration
        run: |
          # Trigger ECS service update to pick up new secrets
          aws ecs update-service \
            --cluster dafnckmachine-production \
            --service dafnckmachine-app \
            --force-new-deployment
```

## Monitoring and Alerting

### Health Check Configuration

```typescript
// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator, MemoryHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024),
      () => this.memory.checkRSS('memory_rss', 150 * 1024 * 1024),
    ]);
  }

  @Get('ready')
  @HealthCheck()
  readiness() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }

  @Get('live')
  @HealthCheck()
  liveness() {
    return this.health.check([
      () => this.memory.checkHeap('memory_heap', 200 * 1024 * 1024),
    ]);
  }
}
```

### Deployment Notifications

```yaml
# .github/workflows/notifications.yml
name: Deployment Notifications

on:
  workflow_run:
    workflows: ["CI/CD Pipeline"]
    types:
      - completed

jobs:
  notify:
    name: Send Notifications
    runs-on: ubuntu-latest
    
    steps:
      - name: Notify Slack on success
        if: ${{ github.event.workflow_run.conclusion == 'success' }}
        uses: 8398a7/action-slack@v3
        with:
          status: success
          channel: '#deployments'
          message: |
            ✅ Deployment successful!
            Branch: ${{ github.event.workflow_run.head_branch }}
            Commit: ${{ github.event.workflow_run.head_sha }}
            Environment: ${{ github.event.workflow_run.name }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

      - name: Notify Slack on failure
        if: ${{ github.event.workflow_run.conclusion == 'failure' }}
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          channel: '#deployments'
          message: |
            ❌ Deployment failed!
            Branch: ${{ github.event.workflow_run.head_branch }}
            Commit: ${{ github.event.workflow_run.head_sha }}
            Environment: ${{ github.event.workflow_run.name }}
            Please check the logs: ${{ github.event.workflow_run.html_url }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

      - name: Create GitHub issue on failure
        if: ${{ github.event.workflow_run.conclusion == 'failure' }}
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Deployment failed: ${context.payload.workflow_run.head_branch}`,
              body: `
                Deployment failed for branch: ${context.payload.workflow_run.head_branch}
                Commit: ${context.payload.workflow_run.head_sha}
                Workflow: ${context.payload.workflow_run.html_url}
                
                Please investigate and fix the deployment issues.
              `,
              labels: ['deployment', 'bug', 'high-priority']
            });
```

## Related Documentation

- [Infrastructure Architecture Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Infrastructure_Architecture_Implementation.md)
- [Environment Configuration Management](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Environment_Configuration_Management.md)
- [Monitoring Analytics Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Monitoring_Analytics_Implementation.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: devops-automation-agent
- **Related Workflows**: 16_Deployment_Automation.md 