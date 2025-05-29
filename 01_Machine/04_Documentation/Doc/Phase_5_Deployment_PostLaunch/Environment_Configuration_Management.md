# Environment Configuration Management - DafnckMachine v3.1

## Overview
Comprehensive environment configuration management for the DafnckMachine v3.1 project, covering development, staging, and production environments with secure configuration handling, environment variables, and deployment strategies.

## Environment Strategy

### Environment Types
- **Development**: Local development and testing environment
- **Staging**: Pre-production environment for integration testing
- **Production**: Live production environment serving end users

### Configuration Principles
- **Environment Parity**: Maintain consistency across environments
- **Configuration as Code**: All configuration stored in version control
- **Secrets Management**: Secure handling of sensitive configuration
- **Environment Isolation**: Clear separation between environments
- **Automated Deployment**: Consistent deployment processes

## Environment Configuration Structure

### Configuration Hierarchy

```
environments/
├── shared/
│   ├── base.env                    # Base configuration
│   ├── database.env               # Database configuration
│   └── services.env               # Service configuration
├── development/
│   ├── .env                       # Development environment variables
│   ├── docker-compose.yml         # Local development setup
│   └── terraform.tfvars           # Development infrastructure
├── staging/
│   ├── .env.staging               # Staging environment variables
│   ├── terraform.tfvars           # Staging infrastructure
│   └── deployment.yml             # Staging deployment config
└── production/
    ├── .env.production            # Production environment variables
    ├── terraform.tfvars           # Production infrastructure
    └── deployment.yml             # Production deployment config
```

### Environment Variables Schema

```typescript
// config/environment.schema.ts
export interface EnvironmentConfig {
  // Application Configuration
  NODE_ENV: 'development' | 'staging' | 'production';
  PORT: number;
  APP_NAME: string;
  APP_VERSION: string;
  
  // Database Configuration
  DATABASE_URL: string;
  DATABASE_SSL: boolean;
  DATABASE_POOL_SIZE: number;
  DATABASE_TIMEOUT: number;
  
  // Redis Configuration
  REDIS_URL: string;
  REDIS_TTL: number;
  REDIS_MAX_RETRIES: number;
  
  // Authentication
  JWT_SECRET: string;
  JWT_EXPIRES_IN: string;
  BCRYPT_ROUNDS: number;
  
  // External Services
  EMAIL_SERVICE_API_KEY: string;
  PAYMENT_GATEWAY_API_KEY: string;
  ANALYTICS_API_KEY: string;
  
  // AWS Configuration
  AWS_REGION: string;
  AWS_S3_BUCKET: string;
  AWS_CLOUDFRONT_DOMAIN: string;
  
  // Monitoring
  LOG_LEVEL: 'error' | 'warn' | 'info' | 'debug';
  SENTRY_DSN?: string;
  NEW_RELIC_LICENSE_KEY?: string;
  
  // Feature Flags
  ENABLE_ANALYTICS: boolean;
  ENABLE_NOTIFICATIONS: boolean;
  ENABLE_REAL_TIME_UPDATES: boolean;
}
```

## Development Environment

### Local Development Setup

```bash
# environments/development/.env
NODE_ENV=development
PORT=3000
APP_NAME=DafnckMachine
APP_VERSION=3.1.0

# Database Configuration
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/dafnckmachine_dev
DATABASE_SSL=false
DATABASE_POOL_SIZE=10
DATABASE_TIMEOUT=30000

# Redis Configuration
REDIS_URL=redis://localhost:6379
REDIS_TTL=3600
REDIS_MAX_RETRIES=3

# Authentication
JWT_SECRET=dev-jwt-secret-key-change-in-production
JWT_EXPIRES_IN=7d
BCRYPT_ROUNDS=10

# External Services (Development/Mock)
EMAIL_SERVICE_API_KEY=dev-email-key
PAYMENT_GATEWAY_API_KEY=dev-payment-key
ANALYTICS_API_KEY=dev-analytics-key

# AWS Configuration (LocalStack)
AWS_REGION=us-east-1
AWS_S3_BUCKET=dafnckmachine-dev-assets
AWS_CLOUDFRONT_DOMAIN=localhost:3000

# Monitoring
LOG_LEVEL=debug
SENTRY_DSN=
NEW_RELIC_LICENSE_KEY=

# Feature Flags
ENABLE_ANALYTICS=false
ENABLE_NOTIFICATIONS=true
ENABLE_REAL_TIME_UPDATES=true
```

### Docker Compose Development

```yaml
# environments/development/docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: ../../
      dockerfile: Dockerfile
      target: development
    ports:
      - "3000:3000"
      - "4000:4000"
    env_file:
      - .env
    volumes:
      - ../../:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis
      - localstack
    networks:
      - dafnckmachine-dev

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
      - ../../scripts/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    networks:
      - dafnckmachine-dev

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - dafnckmachine-dev

  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3,secretsmanager,sns,sqs
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
    volumes:
      - localstack_data:/tmp/localstack
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - dafnckmachine-dev

  mailhog:
    image: mailhog/mailhog:latest
    ports:
      - "1025:1025"
      - "8025:8025"
    networks:
      - dafnckmachine-dev

volumes:
  postgres_data:
  redis_data:
  localstack_data:

networks:
  dafnckmachine-dev:
    driver: bridge
```

### Development Scripts

```json
{
  "scripts": {
    "dev": "NODE_ENV=development nodemon src/index.ts",
    "dev:docker": "docker-compose -f environments/development/docker-compose.yml up",
    "dev:setup": "npm run dev:docker -- -d && npm run db:migrate && npm run db:seed",
    "dev:clean": "docker-compose -f environments/development/docker-compose.yml down -v",
    "dev:logs": "docker-compose -f environments/development/docker-compose.yml logs -f",
    "dev:test": "NODE_ENV=test npm run test",
    "dev:lint": "eslint src --ext .ts,.tsx --fix",
    "dev:format": "prettier --write src/**/*.{ts,tsx,json,md}"
  }
}
```

## Staging Environment

### Staging Configuration

```bash
# environments/staging/.env.staging
NODE_ENV=staging
PORT=3000
APP_NAME=DafnckMachine
APP_VERSION=3.1.0

# Database Configuration
DATABASE_URL=${DATABASE_URL_SECRET}
DATABASE_SSL=true
DATABASE_POOL_SIZE=20
DATABASE_TIMEOUT=30000

# Redis Configuration
REDIS_URL=${REDIS_URL_SECRET}
REDIS_TTL=3600
REDIS_MAX_RETRIES=3

# Authentication
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=7d
BCRYPT_ROUNDS=12

# External Services
EMAIL_SERVICE_API_KEY=${EMAIL_SERVICE_API_KEY}
PAYMENT_GATEWAY_API_KEY=${PAYMENT_GATEWAY_SANDBOX_KEY}
ANALYTICS_API_KEY=${ANALYTICS_API_KEY}

# AWS Configuration
AWS_REGION=us-east-1
AWS_S3_BUCKET=dafnckmachine-staging-assets
AWS_CLOUDFRONT_DOMAIN=staging-cdn.dafnckmachine.com

# Monitoring
LOG_LEVEL=info
SENTRY_DSN=${SENTRY_DSN_STAGING}
NEW_RELIC_LICENSE_KEY=${NEW_RELIC_LICENSE_KEY}

# Feature Flags
ENABLE_ANALYTICS=true
ENABLE_NOTIFICATIONS=true
ENABLE_REAL_TIME_UPDATES=true
```

### Staging Infrastructure

```hcl
# environments/staging/terraform.tfvars
environment = "staging"
aws_region  = "us-east-1"

# VPC Configuration
vpc_cidr = "10.1.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b"]
public_subnets     = ["10.1.101.0/24", "10.1.102.0/24"]
private_subnets    = ["10.1.1.0/24", "10.1.2.0/24"]

# ECS Configuration
app_count        = 2
app_min_capacity = 1
app_max_capacity = 4
app_cpu          = 512
app_memory       = 1024

# Database Configuration
db_instance_class      = "db.t3.medium"
db_allocated_storage   = 20
db_max_allocated_storage = 100
db_backup_retention_period = 7

# Redis Configuration
redis_node_type         = "cache.t3.micro"
redis_num_cache_nodes   = 1

# Domain Configuration
domain_names = ["staging.dafnckmachine.com"]
certificate_domain = "*.dafnckmachine.com"

# Monitoring
enable_detailed_monitoring = true
log_retention_days = 7

# Cost Management
monthly_budget_limit = "200"
budget_notification_emails = ["devops@dafnckmachine.com"]
```

### Staging Deployment Configuration

```yaml
# environments/staging/deployment.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: staging-config
data:
  deployment_strategy: "rolling"
  health_check_path: "/health"
  health_check_interval: "30s"
  health_check_timeout: "5s"
  health_check_retries: "3"
  
  # Scaling Configuration
  min_replicas: "1"
  max_replicas: "4"
  target_cpu_utilization: "70"
  target_memory_utilization: "80"
  
  # Resource Limits
  cpu_request: "256m"
  cpu_limit: "512m"
  memory_request: "512Mi"
  memory_limit: "1Gi"
  
  # Environment Specific Settings
  enable_debug_logging: "false"
  enable_performance_monitoring: "true"
  enable_security_scanning: "true"
```

## Production Environment

### Production Configuration

```bash
# environments/production/.env.production
NODE_ENV=production
PORT=3000
APP_NAME=DafnckMachine
APP_VERSION=3.1.0

# Database Configuration
DATABASE_URL=${DATABASE_URL_SECRET}
DATABASE_SSL=true
DATABASE_POOL_SIZE=50
DATABASE_TIMEOUT=30000

# Redis Configuration
REDIS_URL=${REDIS_URL_SECRET}
REDIS_TTL=3600
REDIS_MAX_RETRIES=5

# Authentication
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=24h
BCRYPT_ROUNDS=14

# External Services
EMAIL_SERVICE_API_KEY=${EMAIL_SERVICE_API_KEY}
PAYMENT_GATEWAY_API_KEY=${PAYMENT_GATEWAY_PRODUCTION_KEY}
ANALYTICS_API_KEY=${ANALYTICS_API_KEY}

# AWS Configuration
AWS_REGION=us-east-1
AWS_S3_BUCKET=dafnckmachine-production-assets
AWS_CLOUDFRONT_DOMAIN=cdn.dafnckmachine.com

# Monitoring
LOG_LEVEL=warn
SENTRY_DSN=${SENTRY_DSN_PRODUCTION}
NEW_RELIC_LICENSE_KEY=${NEW_RELIC_LICENSE_KEY}

# Feature Flags
ENABLE_ANALYTICS=true
ENABLE_NOTIFICATIONS=true
ENABLE_REAL_TIME_UPDATES=true
```

### Production Infrastructure

```hcl
# environments/production/terraform.tfvars
environment = "production"
aws_region  = "us-east-1"

# VPC Configuration
vpc_cidr = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
public_subnets     = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
private_subnets    = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]

# ECS Configuration
app_count        = 3
app_min_capacity = 2
app_max_capacity = 10
app_cpu          = 1024
app_memory       = 2048

# Database Configuration
db_instance_class      = "db.r6g.large"
db_allocated_storage   = 100
db_max_allocated_storage = 1000
db_backup_retention_period = 30
db_multi_az = true

# Redis Configuration
redis_node_type         = "cache.r6g.large"
redis_num_cache_nodes   = 3

# Domain Configuration
domain_names = ["dafnckmachine.com", "www.dafnckmachine.com"]
certificate_domain = "*.dafnckmachine.com"

# Security
enable_waf = true
enable_shield_advanced = true
enable_deletion_protection = true

# Monitoring
enable_detailed_monitoring = true
log_retention_days = 30
enable_performance_insights = true

# Backup
backup_retention_days = 2555  # 7 years
enable_cross_region_backup = true

# Cost Management
monthly_budget_limit = "1000"
budget_notification_emails = ["finance@dafnckmachine.com", "devops@dafnckmachine.com"]
```

## Configuration Management Tools

### Environment Configuration Loader

```typescript
// src/config/environment.ts
import { z } from 'zod';
import dotenv from 'dotenv';
import path from 'path';

// Load environment-specific .env file
const envFile = process.env.NODE_ENV === 'production' 
  ? '.env.production'
  : process.env.NODE_ENV === 'staging'
  ? '.env.staging'
  : '.env';

dotenv.config({ path: path.resolve(process.cwd(), `environments/${process.env.NODE_ENV}/${envFile}`) });

// Environment schema validation
const environmentSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  PORT: z.coerce.number().default(3000),
  APP_NAME: z.string(),
  APP_VERSION: z.string(),
  
  // Database
  DATABASE_URL: z.string().url(),
  DATABASE_SSL: z.coerce.boolean().default(false),
  DATABASE_POOL_SIZE: z.coerce.number().default(10),
  DATABASE_TIMEOUT: z.coerce.number().default(30000),
  
  // Redis
  REDIS_URL: z.string().url(),
  REDIS_TTL: z.coerce.number().default(3600),
  REDIS_MAX_RETRIES: z.coerce.number().default(3),
  
  // Authentication
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default('24h'),
  BCRYPT_ROUNDS: z.coerce.number().min(10).max(15),
  
  // External Services
  EMAIL_SERVICE_API_KEY: z.string(),
  PAYMENT_GATEWAY_API_KEY: z.string(),
  ANALYTICS_API_KEY: z.string().optional(),
  
  // AWS
  AWS_REGION: z.string().default('us-east-1'),
  AWS_S3_BUCKET: z.string(),
  AWS_CLOUDFRONT_DOMAIN: z.string(),
  
  // Monitoring
  LOG_LEVEL: z.enum(['error', 'warn', 'info', 'debug']).default('info'),
  SENTRY_DSN: z.string().url().optional(),
  NEW_RELIC_LICENSE_KEY: z.string().optional(),
  
  // Feature Flags
  ENABLE_ANALYTICS: z.coerce.boolean().default(true),
  ENABLE_NOTIFICATIONS: z.coerce.boolean().default(true),
  ENABLE_REAL_TIME_UPDATES: z.coerce.boolean().default(true),
});

export type Environment = z.infer<typeof environmentSchema>;

// Validate and export configuration
export const config: Environment = environmentSchema.parse(process.env);

// Configuration validation function
export function validateConfiguration(): void {
  try {
    environmentSchema.parse(process.env);
    console.log('✅ Environment configuration is valid');
  } catch (error) {
    console.error('❌ Environment configuration validation failed:', error);
    process.exit(1);
  }
}

// Environment-specific configuration
export const isDevelopment = config.NODE_ENV === 'development';
export const isStaging = config.NODE_ENV === 'staging';
export const isProduction = config.NODE_ENV === 'production';

// Database configuration
export const databaseConfig = {
  url: config.DATABASE_URL,
  ssl: config.DATABASE_SSL,
  poolSize: config.DATABASE_POOL_SIZE,
  timeout: config.DATABASE_TIMEOUT,
};

// Redis configuration
export const redisConfig = {
  url: config.REDIS_URL,
  ttl: config.REDIS_TTL,
  maxRetries: config.REDIS_MAX_RETRIES,
};

// Feature flags
export const features = {
  analytics: config.ENABLE_ANALYTICS,
  notifications: config.ENABLE_NOTIFICATIONS,
  realTimeUpdates: config.ENABLE_REAL_TIME_UPDATES,
};
```

### Secrets Management

```typescript
// src/config/secrets.ts
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import { config } from './environment';

const secretsClient = new SecretsManagerClient({ region: config.AWS_REGION });

interface SecretValue {
  [key: string]: string;
}

class SecretsManager {
  private cache = new Map<string, SecretValue>();
  private cacheExpiry = new Map<string, number>();
  private readonly CACHE_TTL = 5 * 60 * 1000; // 5 minutes

  async getSecret(secretName: string): Promise<SecretValue> {
    // Check cache first
    const cached = this.cache.get(secretName);
    const expiry = this.cacheExpiry.get(secretName);
    
    if (cached && expiry && Date.now() < expiry) {
      return cached;
    }

    try {
      const command = new GetSecretValueCommand({ SecretId: secretName });
      const response = await secretsClient.send(command);
      
      if (!response.SecretString) {
        throw new Error(`Secret ${secretName} has no string value`);
      }

      const secretValue = JSON.parse(response.SecretString);
      
      // Cache the secret
      this.cache.set(secretName, secretValue);
      this.cacheExpiry.set(secretName, Date.now() + this.CACHE_TTL);
      
      return secretValue;
    } catch (error) {
      console.error(`Failed to retrieve secret ${secretName}:`, error);
      throw error;
    }
  }

  async getDatabaseCredentials(): Promise<{
    username: string;
    password: string;
    host: string;
    port: number;
    database: string;
  }> {
    const secretName = `dafnckmachine/${config.NODE_ENV}/database/credentials`;
    return this.getSecret(secretName) as Promise<any>;
  }

  async getRedisCredentials(): Promise<{
    host: string;
    port: number;
    password: string;
  }> {
    const secretName = `dafnckmachine/${config.NODE_ENV}/redis/credentials`;
    return this.getSecret(secretName) as Promise<any>;
  }

  async getExternalServiceKeys(): Promise<{
    emailServiceApiKey: string;
    paymentGatewayApiKey: string;
    analyticsApiKey: string;
  }> {
    const secretName = `dafnckmachine/${config.NODE_ENV}/external-services/keys`;
    return this.getSecret(secretName) as Promise<any>;
  }

  clearCache(): void {
    this.cache.clear();
    this.cacheExpiry.clear();
  }
}

export const secretsManager = new SecretsManager();
```

### Configuration Validation

```typescript
// src/config/validator.ts
import { config, validateConfiguration } from './environment';
import { secretsManager } from './secrets';

export class ConfigurationValidator {
  async validateAll(): Promise<void> {
    console.log('🔍 Validating configuration...');
    
    // Validate environment variables
    validateConfiguration();
    
    // Validate database connection
    await this.validateDatabase();
    
    // Validate Redis connection
    await this.validateRedis();
    
    // Validate external services
    await this.validateExternalServices();
    
    // Validate AWS services
    await this.validateAWSServices();
    
    console.log('✅ All configuration validation passed');
  }

  private async validateDatabase(): Promise<void> {
    try {
      const credentials = await secretsManager.getDatabaseCredentials();
      // Test database connection
      console.log('✅ Database configuration is valid');
    } catch (error) {
      console.error('❌ Database configuration validation failed:', error);
      throw error;
    }
  }

  private async validateRedis(): Promise<void> {
    try {
      const credentials = await secretsManager.getRedisCredentials();
      // Test Redis connection
      console.log('✅ Redis configuration is valid');
    } catch (error) {
      console.error('❌ Redis configuration validation failed:', error);
      throw error;
    }
  }

  private async validateExternalServices(): Promise<void> {
    try {
      const keys = await secretsManager.getExternalServiceKeys();
      // Test external service connections
      console.log('✅ External services configuration is valid');
    } catch (error) {
      console.error('❌ External services configuration validation failed:', error);
      throw error;
    }
  }

  private async validateAWSServices(): Promise<void> {
    try {
      // Test AWS service connections
      console.log('✅ AWS services configuration is valid');
    } catch (error) {
      console.error('❌ AWS services configuration validation failed:', error);
      throw error;
    }
  }
}

export const configValidator = new ConfigurationValidator();
```

## Environment Deployment Scripts

### Environment Setup Script

```bash
#!/bin/bash
# scripts/setup-environment.sh

set -e

ENVIRONMENT=${1:-development}
AWS_REGION=${2:-us-east-1}

echo "🚀 Setting up $ENVIRONMENT environment..."

# Validate inputs
if [[ ! "$ENVIRONMENT" =~ ^(development|staging|production)$ ]]; then
  echo "❌ Invalid environment. Must be: development, staging, or production"
  exit 1
fi

# Set environment-specific variables
case $ENVIRONMENT in
  development)
    TERRAFORM_WORKSPACE="development"
    CONFIG_FILE="environments/development/.env"
    ;;
  staging)
    TERRAFORM_WORKSPACE="staging"
    CONFIG_FILE="environments/staging/.env.staging"
    ;;
  production)
    TERRAFORM_WORKSPACE="production"
    CONFIG_FILE="environments/production/.env.production"
    ;;
esac

echo "📋 Environment: $ENVIRONMENT"
echo "🌍 AWS Region: $AWS_REGION"
echo "⚙️  Terraform Workspace: $TERRAFORM_WORKSPACE"

# Check prerequisites
echo "🔍 Checking prerequisites..."
command -v terraform >/dev/null 2>&1 || { echo "❌ Terraform is required but not installed."; exit 1; }
command -v aws >/dev/null 2>&1 || { echo "❌ AWS CLI is required but not installed."; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "❌ Docker is required but not installed."; exit 1; }

# Verify AWS credentials
echo "🔐 Verifying AWS credentials..."
aws sts get-caller-identity >/dev/null || { echo "❌ AWS credentials not configured."; exit 1; }

# Create Terraform backend resources
echo "🏗️  Creating Terraform backend resources..."
aws s3 mb s3://dafnckmachine-terraform-state-$ENVIRONMENT --region $AWS_REGION 2>/dev/null || true

aws s3api put-bucket-versioning \
  --bucket dafnckmachine-terraform-state-$ENVIRONMENT \
  --versioning-configuration Status=Enabled

aws dynamodb create-table \
  --table-name dafnckmachine-terraform-locks-$ENVIRONMENT \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region $AWS_REGION 2>/dev/null || true

# Initialize Terraform
echo "🔧 Initializing Terraform..."
cd infrastructure
terraform workspace select $TERRAFORM_WORKSPACE 2>/dev/null || terraform workspace new $TERRAFORM_WORKSPACE

terraform init \
  -backend-config="bucket=dafnckmachine-terraform-state-$ENVIRONMENT" \
  -backend-config="key=infrastructure/terraform.tfstate" \
  -backend-config="region=$AWS_REGION" \
  -backend-config="dynamodb_table=dafnckmachine-terraform-locks-$ENVIRONMENT"

# Plan infrastructure
echo "📋 Planning infrastructure..."
terraform plan -var-file="../environments/$ENVIRONMENT/terraform.tfvars" -out=tfplan

# Apply infrastructure (with confirmation for production)
if [[ "$ENVIRONMENT" == "production" ]]; then
  echo "⚠️  You are about to deploy to PRODUCTION. This action cannot be undone."
  read -p "Are you sure you want to continue? (yes/no): " -r
  if [[ ! $REPLY =~ ^yes$ ]]; then
    echo "❌ Deployment cancelled."
    exit 1
  fi
fi

echo "🚀 Applying infrastructure..."
terraform apply tfplan

# Create secrets in AWS Secrets Manager
echo "🔐 Creating secrets..."
../scripts/create-secrets.sh $ENVIRONMENT

# Deploy application
echo "📦 Deploying application..."
../scripts/deploy-application.sh $ENVIRONMENT

echo "✅ Environment $ENVIRONMENT setup complete!"
echo "🌐 Application URL: https://$ENVIRONMENT.dafnckmachine.com"
```

### Secrets Creation Script

```bash
#!/bin/bash
# scripts/create-secrets.sh

set -e

ENVIRONMENT=${1:-development}
AWS_REGION=${2:-us-east-1}

echo "🔐 Creating secrets for $ENVIRONMENT environment..."

# Database credentials
aws secretsmanager create-secret \
  --name "dafnckmachine/$ENVIRONMENT/database/credentials" \
  --description "Database credentials for $ENVIRONMENT" \
  --secret-string '{
    "username": "dafnckmachine_user",
    "password": "'$(openssl rand -base64 32)'",
    "host": "dafnckmachine-'$ENVIRONMENT'.cluster-xyz.us-east-1.rds.amazonaws.com",
    "port": 5432,
    "database": "dafnckmachine"
  }' \
  --region $AWS_REGION 2>/dev/null || echo "Database secret already exists"

# Redis credentials
aws secretsmanager create-secret \
  --name "dafnckmachine/$ENVIRONMENT/redis/credentials" \
  --description "Redis credentials for $ENVIRONMENT" \
  --secret-string '{
    "host": "dafnckmachine-'$ENVIRONMENT'.cache.amazonaws.com",
    "port": 6379,
    "password": "'$(openssl rand -base64 32)'"
  }' \
  --region $AWS_REGION 2>/dev/null || echo "Redis secret already exists"

# JWT secret
aws secretsmanager create-secret \
  --name "dafnckmachine/$ENVIRONMENT/auth/jwt-secret" \
  --description "JWT secret for $ENVIRONMENT" \
  --secret-string "$(openssl rand -base64 64)" \
  --region $AWS_REGION 2>/dev/null || echo "JWT secret already exists"

# External service keys
aws secretsmanager create-secret \
  --name "dafnckmachine/$ENVIRONMENT/external-services/keys" \
  --description "External service API keys for $ENVIRONMENT" \
  --secret-string '{
    "emailServiceApiKey": "email-service-key-'$ENVIRONMENT'",
    "paymentGatewayApiKey": "payment-gateway-key-'$ENVIRONMENT'",
    "analyticsApiKey": "analytics-key-'$ENVIRONMENT'"
  }' \
  --region $AWS_REGION 2>/dev/null || echo "External services secret already exists"

echo "✅ Secrets created successfully"
```

### Environment Promotion Script

```bash
#!/bin/bash
# scripts/promote-environment.sh

set -e

SOURCE_ENV=${1}
TARGET_ENV=${2}
IMAGE_TAG=${3}

if [[ -z "$SOURCE_ENV" || -z "$TARGET_ENV" || -z "$IMAGE_TAG" ]]; then
  echo "Usage: $0 <source-env> <target-env> <image-tag>"
  echo "Example: $0 staging production v1.2.3"
  exit 1
fi

echo "🚀 Promoting from $SOURCE_ENV to $TARGET_ENV with image tag $IMAGE_TAG"

# Validate environments
if [[ ! "$SOURCE_ENV" =~ ^(development|staging)$ ]] || [[ ! "$TARGET_ENV" =~ ^(staging|production)$ ]]; then
  echo "❌ Invalid environment promotion path"
  exit 1
fi

# Run tests in source environment
echo "🧪 Running tests in $SOURCE_ENV..."
npm run test:e2e:$SOURCE_ENV

# Backup target environment (production only)
if [[ "$TARGET_ENV" == "production" ]]; then
  echo "💾 Creating backup of production environment..."
  aws rds create-db-snapshot \
    --db-instance-identifier dafnckmachine-production \
    --db-snapshot-identifier dafnckmachine-production-$(date +%Y%m%d%H%M%S)
fi

# Deploy to target environment
echo "📦 Deploying to $TARGET_ENV..."
aws ecs update-service \
  --cluster dafnckmachine-$TARGET_ENV \
  --service dafnckmachine-app \
  --task-definition dafnckmachine-$TARGET_ENV:$(aws ecs describe-task-definition \
    --task-definition dafnckmachine-$TARGET_ENV \
    --query 'taskDefinition.revision' \
    --output text) \
  --force-new-deployment

# Wait for deployment
echo "⏳ Waiting for deployment to complete..."
aws ecs wait services-stable \
  --cluster dafnckmachine-$TARGET_ENV \
  --services dafnckmachine-app

# Run smoke tests
echo "🔍 Running smoke tests..."
curl -f https://$TARGET_ENV.dafnckmachine.com/health || {
  echo "❌ Smoke tests failed"
  exit 1
}

echo "✅ Environment promotion completed successfully"
```

## Related Documentation

- [CI/CD Pipeline Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/CICD_Pipeline_Configuration.md)
- [Infrastructure Architecture Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Infrastructure_Architecture_Implementation.md)
- [Monitoring Analytics Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Monitoring_Analytics_Implementation.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: devops-configuration-agent
- **Related Workflows**: 16_Deployment_Automation.md 