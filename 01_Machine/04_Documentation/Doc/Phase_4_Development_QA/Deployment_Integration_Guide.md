# Deployment Integration Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Performance_Optimization_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Guide.md)

## Overview

This guide provides comprehensive deployment integration strategies for DafnckMachine v3.1, covering CI/CD pipeline setup, build optimization, environment configuration, deployment strategies, and production monitoring to ensure reliable and efficient application deployment.

## Table of Contents

1. [Deployment Strategy](#deployment-strategy)
2. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
3. [Build Optimization](#build-optimization)
4. [Environment Configuration](#environment-configuration)
5. [Deployment Strategies](#deployment-strategies)
6. [Production Monitoring](#production-monitoring)

## Deployment Strategy

### Deployment Architecture

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm run test:ci
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.image.outputs.image }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.REACT_APP_API_URL }}
          REACT_APP_GA_TRACKING_ID: ${{ secrets.REACT_APP_GA_TRACKING_ID }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-files
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying to staging environment"
          # Add staging deployment commands

  deploy-production:
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to production environment"
          # Add production deployment commands
```

## CI/CD Pipeline Setup

### GitHub Actions Configuration

```typescript
// scripts/deploy.ts
import { execSync } from 'child_process';
import { readFileSync, writeFileSync } from 'fs';

interface DeploymentConfig {
  environment: 'staging' | 'production';
  apiUrl: string;
  cdnUrl: string;
  buildCommand: string;
  deployCommand: string;
}

class DeploymentManager {
  private config: DeploymentConfig;
  
  constructor(environment: 'staging' | 'production') {
    this.config = this.loadConfig(environment);
  }
  
  private loadConfig(environment: string): DeploymentConfig {
    const configs = {
      staging: {
        environment: 'staging' as const,
        apiUrl: process.env.STAGING_API_URL!,
        cdnUrl: process.env.STAGING_CDN_URL!,
        buildCommand: 'npm run build:staging',
        deployCommand: 'npm run deploy:staging'
      },
      production: {
        environment: 'production' as const,
        apiUrl: process.env.PRODUCTION_API_URL!,
        cdnUrl: process.env.PRODUCTION_CDN_URL!,
        buildCommand: 'npm run build:production',
        deployCommand: 'npm run deploy:production'
      }
    };
    
    return configs[environment as keyof typeof configs];
  }
  
  async deploy(): Promise<void> {
    try {
      console.log(`Starting deployment to ${this.config.environment}`);
      
      // Pre-deployment checks
      await this.runPreDeploymentChecks();
      
      // Build application
      await this.buildApplication();
      
      // Run deployment
      await this.runDeployment();
      
      // Post-deployment verification
      await this.runPostDeploymentChecks();
      
      console.log(`Deployment to ${this.config.environment} completed successfully`);
    } catch (error) {
      console.error(`Deployment failed:`, error);
      throw error;
    }
  }
  
  private async runPreDeploymentChecks(): Promise<void> {
    console.log('Running pre-deployment checks...');
    
    // Check environment variables
    const requiredEnvVars = ['API_URL', 'CDN_URL'];
    for (const envVar of requiredEnvVars) {
      if (!process.env[envVar]) {
        throw new Error(`Missing required environment variable: ${envVar}`);
      }
    }
    
    // Run tests
    execSync('npm run test:ci', { stdio: 'inherit' });
    
    // Check build dependencies
    execSync('npm audit --audit-level moderate', { stdio: 'inherit' });
  }
  
  private async buildApplication(): Promise<void> {
    console.log('Building application...');
    
    // Generate build info
    const buildInfo = {
      version: process.env.npm_package_version,
      commit: process.env.GITHUB_SHA,
      branch: process.env.GITHUB_REF_NAME,
      buildTime: new Date().toISOString(),
      environment: this.config.environment
    };
    
    writeFileSync('public/build-info.json', JSON.stringify(buildInfo, null, 2));
    
    // Run build command
    execSync(this.config.buildCommand, { stdio: 'inherit' });
  }
  
  private async runDeployment(): Promise<void> {
    console.log('Running deployment...');
    execSync(this.config.deployCommand, { stdio: 'inherit' });
  }
  
  private async runPostDeploymentChecks(): Promise<void> {
    console.log('Running post-deployment checks...');
    
    // Health check
    const healthCheckUrl = `${this.config.apiUrl}/health`;
    const response = await fetch(healthCheckUrl);
    
    if (!response.ok) {
      throw new Error(`Health check failed: ${response.status}`);
    }
    
    console.log('Health check passed');
  }
}

// Usage
const environment = process.argv[2] as 'staging' | 'production';
if (!environment || !['staging', 'production'].includes(environment)) {
  console.error('Usage: npm run deploy <staging|production>');
  process.exit(1);
}

const deployment = new DeploymentManager(environment);
deployment.deploy().catch((error) => {
  console.error('Deployment failed:', error);
  process.exit(1);
});
```

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built application
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
    
    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
    
    server {
        listen 80;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;
        
        # Handle client-side routing
        location / {
            try_files $uri $uri/ /index.html;
        }
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        # API proxy
        location /api/ {
            proxy_pass $API_URL;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

## Build Optimization

### Vite Build Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig(({ mode }) => ({
  plugins: [
    react(),
    // Bundle analyzer
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true
    })
  ],
  
  build: {
    // Output directory
    outDir: 'dist',
    
    // Generate source maps for production debugging
    sourcemap: mode === 'production' ? 'hidden' : true,
    
    // Optimize bundle size
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: mode === 'production',
        drop_debugger: mode === 'production'
      }
    },
    
    // Rollup options
    rollupOptions: {
      output: {
        // Manual chunk splitting
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          ui: ['@headlessui/react', '@heroicons/react'],
          utils: ['date-fns', 'lodash-es']
        },
        
        // Asset file naming
        assetFileNames: (assetInfo) => {
          const info = assetInfo.name!.split('.');
          const ext = info[info.length - 1];
          
          if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(ext)) {
            return `assets/images/[name]-[hash][extname]`;
          }
          
          if (/woff2?|eot|ttf|otf/i.test(ext)) {
            return `assets/fonts/[name]-[hash][extname]`;
          }
          
          return `assets/[name]-[hash][extname]`;
        },
        
        // Chunk file naming
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js'
      }
    },
    
    // Chunk size warning limit
    chunkSizeWarningLimit: 1000
  },
  
  // Development server
  server: {
    port: 3000,
    host: true,
    proxy: {
      '/api': {
        target: process.env.VITE_API_URL || 'http://localhost:8000',
        changeOrigin: true,
        secure: false
      }
    }
  },
  
  // Path resolution
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@hooks': resolve(__dirname, 'src/hooks'),
      '@types': resolve(__dirname, 'src/types')
    }
  },
  
  // Environment variables
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString())
  }
}));
```

### Build Scripts

```json
{
  "scripts": {
    "build": "vite build",
    "build:staging": "cross-env NODE_ENV=staging vite build --mode staging",
    "build:production": "cross-env NODE_ENV=production vite build --mode production",
    "build:analyze": "npm run build && npx vite-bundle-analyzer dist/stats.html",
    "preview": "vite preview",
    "deploy:staging": "npm run build:staging && aws s3 sync dist/ s3://staging-bucket --delete",
    "deploy:production": "npm run build:production && aws s3 sync dist/ s3://production-bucket --delete"
  }
}
```

## Environment Configuration

### Environment Variables Management

```typescript
// src/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  cdnUrl: string;
  environment: 'development' | 'staging' | 'production';
  features: {
    analytics: boolean;
    monitoring: boolean;
    debugging: boolean;
  };
  analytics: {
    googleAnalyticsId?: string;
    mixpanelToken?: string;
  };
  monitoring: {
    sentryDsn?: string;
    datadogApiKey?: string;
  };
}

const createEnvironmentConfig = (): EnvironmentConfig => {
  const environment = import.meta.env.MODE as EnvironmentConfig['environment'];
  
  const baseConfig: EnvironmentConfig = {
    apiUrl: import.meta.env.VITE_API_URL || 'http://localhost:8000',
    cdnUrl: import.meta.env.VITE_CDN_URL || '',
    environment,
    features: {
      analytics: environment !== 'development',
      monitoring: environment === 'production',
      debugging: environment === 'development'
    },
    analytics: {
      googleAnalyticsId: import.meta.env.VITE_GA_TRACKING_ID,
      mixpanelToken: import.meta.env.VITE_MIXPANEL_TOKEN
    },
    monitoring: {
      sentryDsn: import.meta.env.VITE_SENTRY_DSN,
      datadogApiKey: import.meta.env.VITE_DATADOG_API_KEY
    }
  };
  
  // Environment-specific overrides
  switch (environment) {
    case 'staging':
      return {
        ...baseConfig,
        features: {
          ...baseConfig.features,
          debugging: true
        }
      };
    
    case 'production':
      return {
        ...baseConfig,
        features: {
          ...baseConfig.features,
          debugging: false
        }
      };
    
    default:
      return baseConfig;
  }
};

export const config = createEnvironmentConfig();

// Validate required environment variables
const validateEnvironment = () => {
  const required = ['VITE_API_URL'];
  const missing = required.filter(key => !import.meta.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
};

validateEnvironment();
```

### Environment Files

```bash
# .env.development
VITE_API_URL=http://localhost:8000
VITE_CDN_URL=http://localhost:3000
VITE_GA_TRACKING_ID=
VITE_SENTRY_DSN=
```

```bash
# .env.staging
VITE_API_URL=https://api-staging.dafnckmachine.com
VITE_CDN_URL=https://cdn-staging.dafnckmachine.com
VITE_GA_TRACKING_ID=GA-STAGING-ID
VITE_SENTRY_DSN=https://staging-sentry-dsn
```

```bash
# .env.production
VITE_API_URL=https://api.dafnckmachine.com
VITE_CDN_URL=https://cdn.dafnckmachine.com
VITE_GA_TRACKING_ID=GA-PRODUCTION-ID
VITE_SENTRY_DSN=https://production-sentry-dsn
```

## Deployment Strategies

### Blue-Green Deployment

```typescript
// scripts/blue-green-deploy.ts
import { execSync } from 'child_process';

interface DeploymentSlot {
  name: 'blue' | 'green';
  url: string;
  active: boolean;
}

class BlueGreenDeployment {
  private slots: DeploymentSlot[] = [
    { name: 'blue', url: 'https://blue.dafnckmachine.com', active: false },
    { name: 'green', url: 'https://green.dafnckmachine.com', active: false }
  ];
  
  async deploy(): Promise<void> {
    const activeSlot = this.getActiveSlot();
    const inactiveSlot = this.getInactiveSlot();
    
    console.log(`Deploying to ${inactiveSlot.name} slot`);
    
    try {
      // Deploy to inactive slot
      await this.deployToSlot(inactiveSlot);
      
      // Run health checks
      await this.healthCheck(inactiveSlot);
      
      // Switch traffic
      await this.switchTraffic(inactiveSlot, activeSlot);
      
      console.log(`Successfully deployed to ${inactiveSlot.name} slot`);
    } catch (error) {
      console.error(`Deployment failed:`, error);
      throw error;
    }
  }
  
  private getActiveSlot(): DeploymentSlot {
    return this.slots.find(slot => slot.active) || this.slots[0];
  }
  
  private getInactiveSlot(): DeploymentSlot {
    return this.slots.find(slot => !slot.active) || this.slots[1];
  }
  
  private async deployToSlot(slot: DeploymentSlot): Promise<void> {
    const deployCommand = `aws s3 sync dist/ s3://${slot.name}-bucket --delete`;
    execSync(deployCommand, { stdio: 'inherit' });
  }
  
  private async healthCheck(slot: DeploymentSlot): Promise<void> {
    const maxRetries = 10;
    const retryDelay = 5000;
    
    for (let i = 0; i < maxRetries; i++) {
      try {
        const response = await fetch(`${slot.url}/health`);
        if (response.ok) {
          console.log(`Health check passed for ${slot.name} slot`);
          return;
        }
      } catch (error) {
        console.log(`Health check attempt ${i + 1} failed for ${slot.name} slot`);
      }
      
      if (i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, retryDelay));
      }
    }
    
    throw new Error(`Health check failed for ${slot.name} slot after ${maxRetries} attempts`);
  }
  
  private async switchTraffic(newActive: DeploymentSlot, oldActive: DeploymentSlot): Promise<void> {
    // Update load balancer configuration
    const switchCommand = `aws elbv2 modify-target-group --target-group-arn ${newActive.name}-tg`;
    execSync(switchCommand, { stdio: 'inherit' });
    
    // Update slot status
    newActive.active = true;
    oldActive.active = false;
    
    console.log(`Traffic switched from ${oldActive.name} to ${newActive.name}`);
  }
}
```

### Canary Deployment

```typescript
// scripts/canary-deploy.ts
class CanaryDeployment {
  private trafficPercentages = [5, 25, 50, 100];
  private currentStep = 0;
  
  async deploy(): Promise<void> {
    console.log('Starting canary deployment');
    
    try {
      // Deploy canary version
      await this.deployCanary();
      
      // Gradually increase traffic
      for (const percentage of this.trafficPercentages) {
        await this.updateTraffic(percentage);
        await this.monitorMetrics(percentage);
        this.currentStep++;
      }
      
      console.log('Canary deployment completed successfully');
    } catch (error) {
      console.error('Canary deployment failed, rolling back');
      await this.rollback();
      throw error;
    }
  }
  
  private async deployCanary(): Promise<void> {
    execSync('aws s3 sync dist/ s3://canary-bucket --delete', { stdio: 'inherit' });
  }
  
  private async updateTraffic(percentage: number): Promise<void> {
    console.log(`Updating traffic to ${percentage}% canary`);
    
    const updateCommand = `aws elbv2 modify-rule --rule-arn canary-rule --actions Type=forward,TargetGroupArn=canary-tg,Weight=${percentage}`;
    execSync(updateCommand, { stdio: 'inherit' });
    
    // Wait for traffic to stabilize
    await new Promise(resolve => setTimeout(resolve, 30000));
  }
  
  private async monitorMetrics(percentage: number): Promise<void> {
    console.log(`Monitoring metrics for ${percentage}% traffic`);
    
    const metrics = await this.getMetrics();
    
    if (metrics.errorRate > 0.01 || metrics.responseTime > 2000) {
      throw new Error(`Metrics threshold exceeded: error rate ${metrics.errorRate}, response time ${metrics.responseTime}ms`);
    }
    
    console.log(`Metrics look good for ${percentage}% traffic`);
  }
  
  private async getMetrics(): Promise<{ errorRate: number; responseTime: number }> {
    // Simulate metrics collection
    return {
      errorRate: Math.random() * 0.005,
      responseTime: 500 + Math.random() * 200
    };
  }
  
  private async rollback(): Promise<void> {
    console.log('Rolling back canary deployment');
    
    const rollbackCommand = 'aws elbv2 modify-rule --rule-arn canary-rule --actions Type=forward,TargetGroupArn=production-tg,Weight=100';
    execSync(rollbackCommand, { stdio: 'inherit' });
  }
}
```

## Production Monitoring

### Deployment Monitoring

```typescript
// src/utils/deploymentMonitoring.ts
interface DeploymentMetrics {
  version: string;
  deploymentTime: number;
  healthStatus: 'healthy' | 'unhealthy';
  errorRate: number;
  responseTime: number;
  throughput: number;
}

class DeploymentMonitor {
  private metrics: DeploymentMetrics[] = [];
  
  async startMonitoring(): Promise<void> {
    setInterval(async () => {
      const metrics = await this.collectMetrics();
      this.metrics.push(metrics);
      
      if (this.shouldAlert(metrics)) {
        await this.sendAlert(metrics);
      }
    }, 60000); // Monitor every minute
  }
  
  private async collectMetrics(): Promise<DeploymentMetrics> {
    const buildInfo = await this.getBuildInfo();
    const healthStatus = await this.checkHealth();
    const performanceMetrics = await this.getPerformanceMetrics();
    
    return {
      version: buildInfo.version,
      deploymentTime: Date.now(),
      healthStatus,
      errorRate: performanceMetrics.errorRate,
      responseTime: performanceMetrics.responseTime,
      throughput: performanceMetrics.throughput
    };
  }
  
  private async getBuildInfo(): Promise<{ version: string }> {
    try {
      const response = await fetch('/build-info.json');
      return await response.json();
    } catch {
      return { version: 'unknown' };
    }
  }
  
  private async checkHealth(): Promise<'healthy' | 'unhealthy'> {
    try {
      const response = await fetch('/health');
      return response.ok ? 'healthy' : 'unhealthy';
    } catch {
      return 'unhealthy';
    }
  }
  
  private async getPerformanceMetrics(): Promise<{
    errorRate: number;
    responseTime: number;
    throughput: number;
  }> {
    // Collect from monitoring service
    return {
      errorRate: 0.001,
      responseTime: 150,
      throughput: 1000
    };
  }
  
  private shouldAlert(metrics: DeploymentMetrics): boolean {
    return (
      metrics.healthStatus === 'unhealthy' ||
      metrics.errorRate > 0.01 ||
      metrics.responseTime > 2000
    );
  }
  
  private async sendAlert(metrics: DeploymentMetrics): Promise<void> {
    const alertData = {
      message: 'Deployment monitoring alert',
      metrics,
      timestamp: Date.now()
    };
    
    // Send to alerting service
    await fetch('/api/alerts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(alertData)
    });
  }
}

export const deploymentMonitor = new DeploymentMonitor();
```

---

## Summary

This Deployment Integration Guide provides comprehensive deployment strategies for DafnckMachine v3.1, ensuring reliable and efficient application deployment through CI/CD pipeline setup, build optimization, environment configuration, deployment strategies, and production monitoring.

Key deployment areas include:

1. **CI/CD Pipeline Setup**: GitHub Actions workflow and deployment automation
2. **Build Optimization**: Vite configuration and bundle optimization
3. **Environment Configuration**: Environment-specific settings and validation
4. **Deployment Strategies**: Blue-green and canary deployment implementations
5. **Production Monitoring**: Deployment monitoring and alerting systems

The guide ensures reliable deployment processes while maintaining high availability and performance standards. 