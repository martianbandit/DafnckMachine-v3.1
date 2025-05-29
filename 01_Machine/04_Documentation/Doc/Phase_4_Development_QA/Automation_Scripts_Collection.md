# Automation Scripts Collection - DafnckMachine v3.1

## Document Information
- **Document Type**: Automation Scripts Collection
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Status**: Active
- **Owner**: DevOps Agent

## Overview
This document provides a comprehensive collection of automation scripts for DafnckMachine v3.1 development, including TaskMaster automation, CI/CD integrations, workflow automation, and utility scripts to streamline development processes and ensure consistent execution of repetitive tasks.

## TaskMaster Automation Scripts

### Task Automation Scripts

#### Auto Task Creation Script
```bash
#!/bin/bash
# auto-create-tasks.sh - Automatically create tasks from commit messages

set -e

COMMIT_MESSAGE="$1"
BRANCH_NAME="$2"
PROJECT_ROOT="${3:-$(pwd)}"

# Extract task information from commit message
if [[ $COMMIT_MESSAGE =~ ^(feat|fix|docs|style|refactor|test|chore)\(([^)]+)\):\ (.+)$ ]]; then
    TYPE="${BASH_REMATCH[1]}"
    SCOPE="${BASH_REMATCH[2]}"
    DESCRIPTION="${BASH_REMATCH[3]}"
    
    # Map commit types to task priorities
    case $TYPE in
        "feat") PRIORITY="high" ;;
        "fix") PRIORITY="high" ;;
        "docs") PRIORITY="low" ;;
        "test") PRIORITY="medium" ;;
        *) PRIORITY="medium" ;;
    esac
    
    # Create task using TaskMaster
    task-master add-task \
        --prompt="$TYPE($SCOPE): $DESCRIPTION" \
        --priority="$PRIORITY" \
        --research
        
    echo "✅ Task created for commit: $COMMIT_MESSAGE"
else
    echo "ℹ️ Commit message doesn't match conventional format, skipping task creation"
fi
```

#### Task Status Sync Script
```bash
#!/bin/bash
# sync-task-status.sh - Sync task status with Git branches

set -e

PROJECT_ROOT="${1:-$(pwd)}"
BRANCH_NAME=$(git branch --show-current)

# Extract task ID from branch name (e.g., feature/task-15-user-auth)
if [[ $BRANCH_NAME =~ task-([0-9]+) ]]; then
    TASK_ID="${BASH_REMATCH[1]}"
    
    # Check if branch has been merged
    if git merge-base --is-ancestor HEAD origin/main 2>/dev/null; then
        echo "🎉 Branch merged, marking task $TASK_ID as done"
        task-master set-status --id="$TASK_ID" --status=done
    else
        echo "🔄 Setting task $TASK_ID to in-progress"
        task-master set-status --id="$TASK_ID" --status=in-progress
    fi
else
    echo "ℹ️ Branch name doesn't contain task ID, skipping status sync"
fi
```

#### Daily Task Report Script
```bash
#!/bin/bash
# daily-task-report.sh - Generate daily task progress report

set -e

PROJECT_ROOT="${1:-$(pwd)}"
REPORT_DATE=$(date +"%Y-%m-%d")
REPORT_FILE="reports/daily-report-$REPORT_DATE.md"

mkdir -p reports

cat > "$REPORT_FILE" << EOF
# Daily Task Report - $REPORT_DATE

## Summary
$(task-master list --status=done | wc -l) tasks completed
$(task-master list --status=in-progress | wc -l) tasks in progress
$(task-master list --status=pending | wc -l) tasks pending

## Completed Tasks
$(task-master list --status=done)

## In Progress Tasks
$(task-master list --status=in-progress)

## Next Tasks
$(task-master next)

## Generated at: $(date)
EOF

echo "📊 Daily report generated: $REPORT_FILE"

# Send to Slack if webhook is configured
if [[ -n "$SLACK_WEBHOOK_URL" ]]; then
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"📊 Daily Task Report for $REPORT_DATE\n\`\`\`$(cat $REPORT_FILE)\`\`\`\"}" \
        "$SLACK_WEBHOOK_URL"
fi
```

### Quality Automation Scripts

#### Pre-commit Quality Check
```bash
#!/bin/bash
# pre-commit-quality.sh - Run quality checks before commit

set -e

echo "🔍 Running pre-commit quality checks..."

# Run linting
echo "📝 Checking code style..."
npm run lint || {
    echo "❌ Linting failed. Please fix issues before committing."
    exit 1
}

# Run type checking
echo "🔧 Checking TypeScript types..."
npm run type-check || {
    echo "❌ Type checking failed. Please fix type errors before committing."
    exit 1
}

# Run unit tests
echo "🧪 Running unit tests..."
npm run test:unit || {
    echo "❌ Unit tests failed. Please fix failing tests before committing."
    exit 1
}

# Check test coverage
echo "📊 Checking test coverage..."
COVERAGE=$(npm run test:coverage --silent | grep -o '[0-9]*\.[0-9]*%' | head -1 | sed 's/%//')
THRESHOLD=85

if (( $(echo "$COVERAGE < $THRESHOLD" | bc -l) )); then
    echo "❌ Test coverage ($COVERAGE%) is below threshold ($THRESHOLD%)"
    exit 1
fi

# Security scan
echo "🔒 Running security scan..."
npm audit --audit-level moderate || {
    echo "❌ Security vulnerabilities found. Please fix before committing."
    exit 1
}

echo "✅ All quality checks passed!"
```

#### Automated Testing Script
```bash
#!/bin/bash
# run-automated-tests.sh - Comprehensive test execution

set -e

PROJECT_ROOT="${1:-$(pwd)}"
TEST_TYPE="${2:-all}"

echo "🚀 Starting automated test suite..."

case $TEST_TYPE in
    "unit")
        echo "🧪 Running unit tests..."
        npm run test:unit -- --coverage --watchAll=false
        ;;
    "integration")
        echo "🔗 Running integration tests..."
        npm run test:integration
        ;;
    "e2e")
        echo "🌐 Running end-to-end tests..."
        npm run test:e2e
        ;;
    "performance")
        echo "⚡ Running performance tests..."
        npm run test:performance
        ;;
    "security")
        echo "🔒 Running security tests..."
        npm run test:security
        ;;
    "all")
        echo "🎯 Running all tests..."
        npm run test:unit -- --coverage --watchAll=false
        npm run test:integration
        npm run test:e2e
        npm run test:performance
        npm run test:security
        ;;
    *)
        echo "❌ Unknown test type: $TEST_TYPE"
        echo "Available types: unit, integration, e2e, performance, security, all"
        exit 1
        ;;
esac

echo "✅ Test suite completed successfully!"
```

## CI/CD Integration Scripts

### GitHub Actions Integration

#### TaskMaster GitHub Action
```yaml
# .github/actions/taskmaster/action.yml
name: 'TaskMaster Integration'
description: 'Integrate TaskMaster with GitHub workflows'
inputs:
  command:
    description: 'TaskMaster command to execute'
    required: true
  task-id:
    description: 'Task ID for status updates'
    required: false
  anthropic-api-key:
    description: 'Anthropic API key'
    required: true

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install TaskMaster
      shell: bash
      run: npm install -g task-master-ai
      
    - name: Execute TaskMaster Command
      shell: bash
      run: |
        export ANTHROPIC_API_KEY="${{ inputs.anthropic-api-key }}"
        task-master ${{ inputs.command }}
        
    - name: Update Task Status
      if: inputs.task-id != ''
      shell: bash
      run: |
        export ANTHROPIC_API_KEY="${{ inputs.anthropic-api-key }}"
        case "${{ github.event_name }}" in
          "pull_request")
            task-master set-status --id="${{ inputs.task-id }}" --status=in-progress
            ;;
          "push")
            if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
              task-master set-status --id="${{ inputs.task-id }}" --status=done
            fi
            ;;
        esac
```

#### Workflow Templates
```yaml
# .github/workflows/taskmaster-integration.yml
name: TaskMaster Integration

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  taskmaster-sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Extract Task ID from Branch
        id: extract-task
        run: |
          BRANCH_NAME="${{ github.head_ref || github.ref_name }}"
          if [[ $BRANCH_NAME =~ task-([0-9]+) ]]; then
            echo "task-id=${BASH_REMATCH[1]}" >> $GITHUB_OUTPUT
          fi
          
      - name: TaskMaster Integration
        uses: ./.github/actions/taskmaster
        with:
          command: 'validate-dependencies'
          task-id: ${{ steps.extract-task.outputs.task-id }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          
      - name: Generate Task Files
        uses: ./.github/actions/taskmaster
        with:
          command: 'generate'
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Deployment Automation Scripts

#### Automated Deployment Script
```bash
#!/bin/bash
# deploy.sh - Automated deployment with TaskMaster integration

set -e

ENVIRONMENT="${1:-staging}"
VERSION="${2:-latest}"
PROJECT_ROOT="${3:-$(pwd)}"

echo "🚀 Starting deployment to $ENVIRONMENT..."

# Pre-deployment checks
echo "🔍 Running pre-deployment checks..."
./scripts/pre-commit-quality.sh

# Update deployment task status
if [[ -n "$DEPLOYMENT_TASK_ID" ]]; then
    task-master set-status --id="$DEPLOYMENT_TASK_ID" --status=in-progress
fi

# Build application
echo "🏗️ Building application..."
npm run build

# Run deployment tests
echo "🧪 Running deployment tests..."
npm run test:deployment

# Deploy based on environment
case $ENVIRONMENT in
    "staging")
        echo "📦 Deploying to staging..."
        ./scripts/deploy-staging.sh "$VERSION"
        ;;
    "production")
        echo "🌟 Deploying to production..."
        ./scripts/deploy-production.sh "$VERSION"
        ;;
    *)
        echo "❌ Unknown environment: $ENVIRONMENT"
        exit 1
        ;;
esac

# Post-deployment verification
echo "✅ Running post-deployment verification..."
./scripts/verify-deployment.sh "$ENVIRONMENT"

# Update task status on success
if [[ -n "$DEPLOYMENT_TASK_ID" ]]; then
    task-master set-status --id="$DEPLOYMENT_TASK_ID" --status=done
    task-master update-subtask --id="$DEPLOYMENT_TASK_ID.1" --prompt="Deployment to $ENVIRONMENT completed successfully at $(date)"
fi

echo "🎉 Deployment to $ENVIRONMENT completed successfully!"
```

#### Rollback Script
```bash
#!/bin/bash
# rollback.sh - Automated rollback with TaskMaster tracking

set -e

ENVIRONMENT="${1:-staging}"
VERSION="${2}"
REASON="${3:-Manual rollback}"

if [[ -z "$VERSION" ]]; then
    echo "❌ Version is required for rollback"
    echo "Usage: $0 <environment> <version> [reason]"
    exit 1
fi

echo "⏪ Starting rollback to version $VERSION in $ENVIRONMENT..."

# Create rollback task
ROLLBACK_TASK_ID=$(task-master add-task \
    --prompt="Emergency rollback to version $VERSION in $ENVIRONMENT. Reason: $REASON" \
    --priority=high \
    | grep -o 'Task [0-9]*' | grep -o '[0-9]*')

echo "📝 Created rollback task: $ROLLBACK_TASK_ID"

# Execute rollback
case $ENVIRONMENT in
    "staging")
        ./scripts/rollback-staging.sh "$VERSION"
        ;;
    "production")
        ./scripts/rollback-production.sh "$VERSION"
        ;;
    *)
        echo "❌ Unknown environment: $ENVIRONMENT"
        task-master set-status --id="$ROLLBACK_TASK_ID" --status=cancelled
        exit 1
        ;;
esac

# Verify rollback
./scripts/verify-deployment.sh "$ENVIRONMENT"

# Update task status
task-master set-status --id="$ROLLBACK_TASK_ID" --status=done
task-master update-subtask --id="$ROLLBACK_TASK_ID.1" --prompt="Rollback completed successfully. Version $VERSION is now active in $ENVIRONMENT."

echo "✅ Rollback completed successfully!"
```

## Development Workflow Scripts

### Environment Setup Scripts

#### Development Environment Setup
```bash
#!/bin/bash
# setup-dev-env.sh - Setup development environment

set -e

echo "🛠️ Setting up development environment..."

# Check prerequisites
echo "🔍 Checking prerequisites..."
command -v node >/dev/null 2>&1 || { echo "❌ Node.js is required but not installed."; exit 1; }
command -v npm >/dev/null 2>&1 || { echo "❌ npm is required but not installed."; exit 1; }
command -v git >/dev/null 2>&1 || { echo "❌ Git is required but not installed."; exit 1; }

# Install dependencies
echo "📦 Installing dependencies..."
npm install

# Install TaskMaster globally
echo "🎯 Installing TaskMaster..."
npm install -g task-master-ai

# Setup git hooks
echo "🪝 Setting up git hooks..."
mkdir -p .git/hooks
cp scripts/pre-commit-quality.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

# Initialize TaskMaster
echo "📋 Initializing TaskMaster..."
if [[ ! -f ".taskmasterconfig" ]]; then
    task-master init --yes
fi

# Setup environment variables
echo "🔧 Setting up environment variables..."
if [[ ! -f ".env" ]]; then
    cp .env.example .env
    echo "⚠️ Please update .env file with your API keys"
fi

# Create initial tasks if none exist
if [[ ! -f "tasks/tasks.json" ]]; then
    echo "📝 Creating initial tasks..."
    if [[ -f "scripts/prd.txt" ]]; then
        task-master parse-prd scripts/prd.txt
    fi
fi

echo "✅ Development environment setup complete!"
echo "📖 Next steps:"
echo "  1. Update .env file with your API keys"
echo "  2. Run 'npm run dev' to start development server"
echo "  3. Run 'task-master list' to see available tasks"
```

#### Database Setup Script
```bash
#!/bin/bash
# setup-database.sh - Database setup and migration

set -e

ENVIRONMENT="${1:-development}"

echo "🗄️ Setting up database for $ENVIRONMENT..."

# Load environment variables
if [[ -f ".env.$ENVIRONMENT" ]]; then
    source ".env.$ENVIRONMENT"
elif [[ -f ".env" ]]; then
    source ".env"
fi

# Check database connection
echo "🔍 Checking database connection..."
npm run db:check || {
    echo "❌ Database connection failed"
    exit 1
}

# Run migrations
echo "📈 Running database migrations..."
npm run db:migrate

# Seed database if development
if [[ "$ENVIRONMENT" == "development" ]]; then
    echo "🌱 Seeding database with test data..."
    npm run db:seed
fi

# Create TaskMaster task for database setup
task-master add-task \
    --prompt="Database setup completed for $ENVIRONMENT environment" \
    --priority=low \
    --status=done

echo "✅ Database setup complete!"
```

### Monitoring and Alerting Scripts

#### Health Check Script
```bash
#!/bin/bash
# health-check.sh - Application health monitoring

set -e

ENVIRONMENT="${1:-production}"
ALERT_WEBHOOK="${2:-$SLACK_WEBHOOK_URL}"

echo "🩺 Running health check for $ENVIRONMENT..."

# Define endpoints to check
ENDPOINTS=(
    "/api/health"
    "/api/tasks"
    "/api/auth/status"
)

# Get base URL based on environment
case $ENVIRONMENT in
    "production")
        BASE_URL="https://api.dafnck.com"
        ;;
    "staging")
        BASE_URL="https://staging-api.dafnck.com"
        ;;
    "development")
        BASE_URL="http://localhost:3000"
        ;;
    *)
        echo "❌ Unknown environment: $ENVIRONMENT"
        exit 1
        ;;
esac

FAILED_CHECKS=()

# Check each endpoint
for endpoint in "${ENDPOINTS[@]}"; do
    echo "🔍 Checking $BASE_URL$endpoint..."
    
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL$endpoint" || echo "000")
    
    if [[ "$HTTP_CODE" -ge 200 && "$HTTP_CODE" -lt 300 ]]; then
        echo "✅ $endpoint - OK ($HTTP_CODE)"
    else
        echo "❌ $endpoint - FAILED ($HTTP_CODE)"
        FAILED_CHECKS+=("$endpoint")
    fi
done

# Report results
if [[ ${#FAILED_CHECKS[@]} -eq 0 ]]; then
    echo "✅ All health checks passed!"
    
    # Create success task
    task-master add-task \
        --prompt="Health check passed for $ENVIRONMENT environment" \
        --priority=low \
        --status=done
else
    echo "❌ Health check failed for endpoints: ${FAILED_CHECKS[*]}"
    
    # Create failure task
    FAILURE_TASK_ID=$(task-master add-task \
        --prompt="Health check failed for $ENVIRONMENT. Failed endpoints: ${FAILED_CHECKS[*]}" \
        --priority=high \
        | grep -o 'Task [0-9]*' | grep -o '[0-9]*')
    
    # Send alert if webhook is configured
    if [[ -n "$ALERT_WEBHOOK" ]]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"🚨 Health check failed for $ENVIRONMENT\\nFailed endpoints: ${FAILED_CHECKS[*]}\\nTask created: $FAILURE_TASK_ID\"}" \
            "$ALERT_WEBHOOK"
    fi
    
    exit 1
fi
```

#### Performance Monitoring Script
```bash
#!/bin/bash
# performance-monitor.sh - Monitor application performance

set -e

ENVIRONMENT="${1:-production}"
DURATION="${2:-60}"  # seconds

echo "⚡ Monitoring performance for $ENVIRONMENT (${DURATION}s)..."

# Get base URL
case $ENVIRONMENT in
    "production")
        BASE_URL="https://api.dafnck.com"
        ;;
    "staging")
        BASE_URL="https://staging-api.dafnck.com"
        ;;
    *)
        BASE_URL="http://localhost:3000"
        ;;
esac

# Run performance test
REPORT_FILE="reports/performance-$(date +%Y%m%d-%H%M%S).json"
mkdir -p reports

k6 run --duration="${DURATION}s" \
    --vus=10 \
    --out json="$REPORT_FILE" \
    - <<EOF
import http from 'k6/http';
import { check } from 'k6';

export default function() {
    const response = http.get('$BASE_URL/api/tasks');
    check(response, {
        'status is 200': (r) => r.status === 200,
        'response time < 500ms': (r) => r.timings.duration < 500,
    });
}
EOF

# Parse results
AVG_RESPONSE_TIME=$(jq -r '.metrics.http_req_duration.avg' "$REPORT_FILE")
ERROR_RATE=$(jq -r '.metrics.http_req_failed.rate' "$REPORT_FILE")

echo "📊 Performance Results:"
echo "  Average Response Time: ${AVG_RESPONSE_TIME}ms"
echo "  Error Rate: ${ERROR_RATE}%"

# Check thresholds
if (( $(echo "$AVG_RESPONSE_TIME > 1000" | bc -l) )); then
    echo "⚠️ Response time exceeds threshold (1000ms)"
    task-master add-task \
        --prompt="Performance degradation detected in $ENVIRONMENT. Average response time: ${AVG_RESPONSE_TIME}ms" \
        --priority=high
fi

if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
    echo "⚠️ Error rate exceeds threshold (5%)"
    task-master add-task \
        --prompt="High error rate detected in $ENVIRONMENT. Error rate: ${ERROR_RATE}%" \
        --priority=high
fi
```

## Utility Scripts

### Backup and Recovery Scripts

#### Backup Script
```bash
#!/bin/bash
# backup.sh - Backup TaskMaster data and configuration

set -e

BACKUP_DIR="${1:-backups}"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_NAME="taskmaster_backup_$TIMESTAMP"

echo "💾 Creating backup: $BACKUP_NAME..."

mkdir -p "$BACKUP_DIR"

# Create backup directory
BACKUP_PATH="$BACKUP_DIR/$BACKUP_NAME"
mkdir -p "$BACKUP_PATH"

# Backup TaskMaster files
echo "📋 Backing up TaskMaster data..."
cp -r tasks/ "$BACKUP_PATH/" 2>/dev/null || echo "No tasks directory found"
cp .taskmasterconfig "$BACKUP_PATH/" 2>/dev/null || echo "No .taskmasterconfig found"
cp -r scripts/ "$BACKUP_PATH/" 2>/dev/null || echo "No scripts directory found"
cp -r reports/ "$BACKUP_PATH/" 2>/dev/null || echo "No reports directory found"

# Backup environment configuration
echo "🔧 Backing up configuration..."
cp .env.example "$BACKUP_PATH/" 2>/dev/null || echo "No .env.example found"

# Create backup manifest
cat > "$BACKUP_PATH/manifest.json" << EOF
{
  "backup_name": "$BACKUP_NAME",
  "timestamp": "$TIMESTAMP",
  "created_at": "$(date -Iseconds)",
  "project_root": "$(pwd)",
  "git_commit": "$(git rev-parse HEAD 2>/dev/null || echo 'unknown')",
  "git_branch": "$(git branch --show-current 2>/dev/null || echo 'unknown')"
}
EOF

# Compress backup
echo "🗜️ Compressing backup..."
tar -czf "$BACKUP_PATH.tar.gz" -C "$BACKUP_DIR" "$BACKUP_NAME"
rm -rf "$BACKUP_PATH"

echo "✅ Backup created: $BACKUP_PATH.tar.gz"

# Create TaskMaster task for backup
task-master add-task \
    --prompt="Backup created: $BACKUP_NAME" \
    --priority=low \
    --status=done
```

#### Restore Script
```bash
#!/bin/bash
# restore.sh - Restore TaskMaster data from backup

set -e

BACKUP_FILE="$1"

if [[ -z "$BACKUP_FILE" ]]; then
    echo "❌ Backup file is required"
    echo "Usage: $0 <backup_file.tar.gz>"
    exit 1
fi

if [[ ! -f "$BACKUP_FILE" ]]; then
    echo "❌ Backup file not found: $BACKUP_FILE"
    exit 1
fi

echo "🔄 Restoring from backup: $BACKUP_FILE..."

# Create temporary directory
TEMP_DIR=$(mktemp -d)
trap "rm -rf $TEMP_DIR" EXIT

# Extract backup
echo "📦 Extracting backup..."
tar -xzf "$BACKUP_FILE" -C "$TEMP_DIR"

# Find backup directory
BACKUP_DIR=$(find "$TEMP_DIR" -name "taskmaster_backup_*" -type d | head -1)

if [[ -z "$BACKUP_DIR" ]]; then
    echo "❌ Invalid backup file format"
    exit 1
fi

# Show backup info
if [[ -f "$BACKUP_DIR/manifest.json" ]]; then
    echo "📋 Backup Information:"
    jq . "$BACKUP_DIR/manifest.json"
fi

# Confirm restore
read -p "⚠️ This will overwrite existing TaskMaster data. Continue? (y/N): " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Restore cancelled"
    exit 1
fi

# Backup current state
echo "💾 Backing up current state..."
./scripts/backup.sh "backups/pre-restore"

# Restore files
echo "🔄 Restoring files..."
[[ -d "$BACKUP_DIR/tasks" ]] && cp -r "$BACKUP_DIR/tasks" .
[[ -f "$BACKUP_DIR/.taskmasterconfig" ]] && cp "$BACKUP_DIR/.taskmasterconfig" .
[[ -d "$BACKUP_DIR/scripts" ]] && cp -r "$BACKUP_DIR/scripts" .
[[ -d "$BACKUP_DIR/reports" ]] && cp -r "$BACKUP_DIR/reports" .

echo "✅ Restore completed successfully!"

# Create TaskMaster task for restore
task-master add-task \
    --prompt="Restored from backup: $(basename $BACKUP_FILE)" \
    --priority=medium \
    --status=done
```

### Maintenance Scripts

#### Cleanup Script
```bash
#!/bin/bash
# cleanup.sh - Clean up temporary files and old data

set -e

echo "🧹 Starting cleanup..."

# Clean old logs
echo "📝 Cleaning old logs..."
find logs/ -name "*.log" -mtime +30 -delete 2>/dev/null || echo "No old logs found"

# Clean old reports
echo "📊 Cleaning old reports..."
find reports/ -name "*.json" -mtime +90 -delete 2>/dev/null || echo "No old reports found"

# Clean old backups
echo "💾 Cleaning old backups..."
find backups/ -name "*.tar.gz" -mtime +180 -delete 2>/dev/null || echo "No old backups found"

# Clean node modules cache
echo "📦 Cleaning npm cache..."
npm cache clean --force

# Clean temporary files
echo "🗑️ Cleaning temporary files..."
rm -rf tmp/ .tmp/ 2>/dev/null || echo "No temp directories found"

echo "✅ Cleanup completed!"
```

---

**Implementation Status**: Ready for Deployment  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-10 