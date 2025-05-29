# Development Workflow Integration - DafnckMachine v3.1

## Document Information
- **Document Type**: Development Workflow Integration
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Status**: Active
- **Owner**: Development Orchestrator Agent

## Overview
This document defines the comprehensive integration strategy for TaskMaster with development tools, processes, and workflows for DafnckMachine v3.1, ensuring seamless coordination between task management, code development, version control, CI/CD pipelines, and quality assurance processes.

## Integration Architecture

### Workflow Integration Framework
```
┌─────────────────────────────────────────────────────────────┐
│                    TaskMaster Core System                   │
├─────────────────────────────────────────────────────────────┤
│                    Development Tool Layer                   │
├─────────────────────────────────────────────────────────────┤
│                    Version Control Integration              │
├─────────────────────────────────────────────────────────────┤
│                    CI/CD Pipeline Integration               │
├─────────────────────────────────────────────────────────────┤
│                    Quality Assurance Integration            │
└─────────────────────────────────────────────────────────────┘
```

### Integration Components
1. **IDE Integration**: Cursor, VS Code, and development environment integration
2. **Version Control**: Git workflow integration with branch management
3. **CI/CD Pipeline**: Automated build, test, and deployment integration
4. **Code Quality**: Linting, formatting, and code review integration
5. **Testing Framework**: Automated testing and quality assurance integration
6. **Project Management**: Issue tracking and project coordination integration

## IDE Integration

### Cursor Integration
```json
{
  "cursorIntegration": {
    "mcpConfiguration": {
      "server": "taskmaster-ai",
      "tools": [
        "mcp_taskmaster-ai_initialize_project",
        "mcp_taskmaster-ai_get_tasks",
        "mcp_taskmaster-ai_add_task",
        "mcp_taskmaster-ai_set_task_status",
        "mcp_taskmaster-ai_expand_task",
        "mcp_taskmaster-ai_update_task"
      ],
      "shortcuts": {
        "viewTasks": "Ctrl+Shift+T",
        "nextTask": "Ctrl+Shift+N",
        "updateStatus": "Ctrl+Shift+U",
        "addTask": "Ctrl+Shift+A"
      }
    },
    "workspaceSettings": {
      "taskmaster.autoSync": true,
      "taskmaster.showProgress": true,
      "taskmaster.enableNotifications": true,
      "taskmaster.branchNaming": "task-{id}-{title-slug}"
    }
  }
}
```

### VS Code Extension Configuration
```json
{
  "vsCodeExtension": {
    "name": "TaskMaster Integration",
    "version": "1.0.0",
    "features": [
      "Task list sidebar",
      "Status bar progress indicator",
      "Command palette integration",
      "Git branch synchronization",
      "Automated task updates"
    ],
    "commands": [
      {
        "command": "taskmaster.viewTasks",
        "title": "View Tasks",
        "category": "TaskMaster"
      },
      {
        "command": "taskmaster.nextTask",
        "title": "Get Next Task",
        "category": "TaskMaster"
      },
      {
        "command": "taskmaster.createBranch",
        "title": "Create Branch for Task",
        "category": "TaskMaster"
      }
    ]
  }
}
```

### Development Environment Setup
```bash
#!/bin/bash
# setup-dev-integration.sh

echo "🛠️ Setting up TaskMaster development integration..."

# Install TaskMaster globally
npm install -g task-master-ai

# Setup workspace configuration
mkdir -p .vscode
cat > .vscode/settings.json << EOF
{
  "taskmaster.enabled": true,
  "taskmaster.projectRoot": "\${workspaceFolder}",
  "taskmaster.autoSync": true,
  "taskmaster.showInStatusBar": true,
  "files.associations": {
    "*.taskmaster": "json"
  }
}
EOF

# Setup tasks.json for VS Code
cat > .vscode/tasks.json << EOF
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "TaskMaster: List Tasks",
      "type": "shell",
      "command": "task-master list",
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "TaskMaster: Next Task",
      "type": "shell",
      "command": "task-master next",
      "group": "build"
    },
    {
      "label": "TaskMaster: Validate Dependencies",
      "type": "shell",
      "command": "task-master validate-dependencies",
      "group": "test"
    }
  ]
}
EOF

echo "✅ Development integration setup complete!"
```

## Version Control Integration

### Git Workflow Integration
```bash
#!/bin/bash
# git-taskmaster-integration.sh

# Setup git hooks for TaskMaster integration
mkdir -p .git/hooks

# Pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
set -e

echo "🔍 TaskMaster pre-commit validation..."

# Validate task dependencies
task-master validate-dependencies

# Update task files
task-master generate

# Stage updated task files
git add tasks/

# Check for task ID in branch name
BRANCH_NAME=$(git branch --show-current)
if [[ $BRANCH_NAME =~ task-([0-9]+) ]]; then
    TASK_ID="${BASH_REMATCH[1]}"
    echo "📝 Updating task $TASK_ID status to in-progress"
    task-master set-status --id="$TASK_ID" --status=in-progress
fi

echo "✅ Pre-commit validation complete"
EOF

# Post-commit hook
cat > .git/hooks/post-commit << 'EOF'
#!/bin/bash

# Extract task ID from commit message
COMMIT_MSG=$(git log -1 --pretty=%B)
if [[ $COMMIT_MSG =~ \[task-([0-9]+)\] ]]; then
    TASK_ID="${BASH_REMATCH[1]}"
    echo "📊 Updating task $TASK_ID with commit information"
    task-master update-subtask --id="$TASK_ID.1" --prompt="Commit: $COMMIT_MSG"
fi
EOF

# Pre-push hook
cat > .git/hooks/pre-push << 'EOF'
#!/bin/bash

# Run final validation before push
echo "🚀 Running pre-push TaskMaster validation..."
task-master validate-dependencies

# Check if pushing to main branch
BRANCH_NAME=$(git branch --show-current)
if [[ $BRANCH_NAME == "main" ]]; then
    echo "⚠️ Pushing to main branch - ensure all tasks are complete"
    PENDING_TASKS=$(task-master list --status=pending | wc -l)
    if [ $PENDING_TASKS -gt 0 ]; then
        echo "❌ Cannot push to main with pending tasks"
        exit 1
    fi
fi

echo "✅ Pre-push validation complete"
EOF

# Make hooks executable
chmod +x .git/hooks/*

echo "🔗 Git hooks configured for TaskMaster integration"
```

### Branch Management Strategy
```javascript
const branchManagement = {
  createTaskBranch(taskId) {
    const task = taskmaster.getTask(taskId);
    const branchName = this.generateBranchName(task);
    
    // Create and checkout new branch
    exec(`git checkout -b ${branchName}`);
    
    // Update task status
    taskmaster.setTaskStatus(taskId, 'in-progress');
    
    // Add initial commit
    exec(`git commit --allow-empty -m "[task-${taskId}] Start work on ${task.title}"`);
    
    return branchName;
  },

  generateBranchName(task) {
    const prefix = this.getBranchPrefix(task.category);
    const slug = task.title.toLowerCase()
      .replace(/[^a-z0-9\s]/g, '')
      .replace(/\s+/g, '-')
      .substring(0, 30);
    
    return `${prefix}/task-${task.id}-${slug}`;
  },

  getBranchPrefix(category) {
    const prefixes = {
      'feature': 'feature',
      'bugfix': 'fix',
      'hotfix': 'hotfix',
      'documentation': 'docs',
      'testing': 'test',
      'infrastructure': 'infra'
    };
    
    return prefixes[category] || 'feature';
  },

  mergeBranch(taskId) {
    const task = taskmaster.getTask(taskId);
    const branchName = this.generateBranchName(task);
    
    // Ensure all quality gates pass
    if (!this.validateQualityGates(taskId)) {
      throw new Error('Quality gates not met for task completion');
    }
    
    // Merge to main
    exec(`git checkout main`);
    exec(`git merge ${branchName} --no-ff`);
    
    // Update task status
    taskmaster.setTaskStatus(taskId, 'done');
    
    // Clean up branch
    exec(`git branch -d ${branchName}`);
    
    return true;
  }
};
```

### Commit Message Standards
```json
{
  "commitMessageStandards": {
    "format": "[task-{id}] {type}({scope}): {description}",
    "types": {
      "feat": "New feature implementation",
      "fix": "Bug fix",
      "docs": "Documentation changes",
      "style": "Code style changes (formatting, etc.)",
      "refactor": "Code refactoring",
      "test": "Adding or updating tests",
      "chore": "Maintenance tasks"
    },
    "examples": [
      "[task-5] feat(auth): implement JWT authentication system",
      "[task-12] fix(api): resolve user validation error",
      "[task-8] docs(readme): update installation instructions",
      "[task-15] test(auth): add unit tests for login functionality"
    ],
    "validation": {
      "maxLength": 72,
      "requireTaskId": true,
      "requireType": true,
      "requireDescription": true
    }
  }
}
```

## CI/CD Pipeline Integration

### GitHub Actions Integration
```yaml
# .github/workflows/taskmaster-ci.yml
name: TaskMaster CI/CD Integration

on:
  push:
    branches: [main, develop, 'feature/**', 'fix/**']
  pull_request:
    branches: [main, develop]

env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

jobs:
  taskmaster-validation:
    runs-on: ubuntu-latest
    name: TaskMaster Validation
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install TaskMaster
        run: npm install -g task-master-ai
        
      - name: Validate task dependencies
        run: task-master validate-dependencies
        
      - name: Generate task files
        run: task-master generate
        
      - name: Check for task updates
        run: |
          if [ -n "$(git status --porcelain tasks/)" ]; then
            echo "Task files updated during CI"
            git config --local user.email "action@github.com"
            git config --local user.name "GitHub Action"
            git add tasks/
            git commit -m "Auto-update task files [skip ci]"
            git push
          fi

  task-status-update:
    runs-on: ubuntu-latest
    name: Update Task Status
    needs: taskmaster-validation
    if: github.event_name == 'push' || github.event_name == 'pull_request'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Install TaskMaster
        run: npm install -g task-master-ai
        
      - name: Extract task ID from branch
        id: extract-task
        run: |
          BRANCH_NAME="${{ github.head_ref || github.ref_name }}"
          echo "Branch name: $BRANCH_NAME"
          
          if [[ $BRANCH_NAME =~ task-([0-9]+) ]]; then
            TASK_ID="${BASH_REMATCH[1]}"
            echo "task-id=$TASK_ID" >> $GITHUB_OUTPUT
            echo "Found task ID: $TASK_ID"
          else
            echo "No task ID found in branch name"
          fi
          
      - name: Update task status for PR
        if: github.event_name == 'pull_request' && steps.extract-task.outputs.task-id != ''
        run: |
          TASK_ID="${{ steps.extract-task.outputs.task-id }}"
          echo "Setting task $TASK_ID to review status"
          task-master set-status --id="$TASK_ID" --status=review
          
      - name: Update task status for merge
        if: github.event_name == 'push' && github.ref == 'refs/heads/main' && steps.extract-task.outputs.task-id != ''
        run: |
          TASK_ID="${{ steps.extract-task.outputs.task-id }}"
          echo "Setting task $TASK_ID to done status"
          task-master set-status --id="$TASK_ID" --status=done
          task-master update-subtask --id="$TASK_ID.1" --prompt="Completed and merged to main branch on $(date)"

  quality-gates:
    runs-on: ubuntu-latest
    name: Quality Gates Validation
    needs: taskmaster-validation
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run linting
        run: npm run lint
        
      - name: Run type checking
        run: npm run type-check
        
      - name: Run unit tests
        run: npm run test:unit -- --coverage
        
      - name: Run integration tests
        run: npm run test:integration
        
      - name: Check code coverage
        run: |
          COVERAGE=$(npm run test:coverage --silent | grep -o '[0-9]*\.[0-9]*%' | head -1 | sed 's/%//')
          echo "Code coverage: $COVERAGE%"
          
          if (( $(echo "$COVERAGE < 85" | bc -l) )); then
            echo "❌ Code coverage below threshold (85%)"
            exit 1
          fi
          
      - name: Security audit
        run: npm audit --audit-level moderate
        
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

### Jenkins Pipeline Integration
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        ANTHROPIC_API_KEY = credentials('anthropic-api-key')
        OPENAI_API_KEY = credentials('openai-api-key')
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    // Install TaskMaster
                    sh 'npm install -g task-master-ai'
                    
                    // Extract task ID from branch name
                    def branchName = env.BRANCH_NAME
                    def taskIdMatcher = branchName =~ /task-(\d+)/
                    if (taskIdMatcher) {
                        env.TASK_ID = taskIdMatcher[0][1]
                        echo "Found task ID: ${env.TASK_ID}"
                    }
                }
            }
        }
        
        stage('TaskMaster Validation') {
            steps {
                sh 'task-master validate-dependencies'
                sh 'task-master generate'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit -- --coverage'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('Linting') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }
        
        stage('Quality Gates') {
            steps {
                script {
                    // Check code coverage
                    def coverage = sh(
                        script: "npm run test:coverage --silent | grep -o '[0-9]*\\.[0-9]*%' | head -1 | sed 's/%//'",
                        returnStdout: true
                    ).trim()
                    
                    if (coverage.toFloat() < 85) {
                        error("Code coverage ${coverage}% below threshold (85%)")
                    }
                    
                    // Update task status
                    if (env.TASK_ID) {
                        sh "task-master set-status --id=${env.TASK_ID} --status=review"
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Deploy application
                    sh 'npm run deploy'
                    
                    // Mark task as complete
                    if (env.TASK_ID) {
                        sh "task-master set-status --id=${env.TASK_ID} --status=done"
                        sh "task-master update-subtask --id=${env.TASK_ID}.1 --prompt='Deployed to production on \$(date)'"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Archive test results
            publishTestResults testResultsPattern: 'test-results.xml'
            
            // Archive coverage reports
            publishCoverage adapters: [
                coberturaAdapter('coverage/cobertura-coverage.xml')
            ]
        }
        
        failure {
            script {
                // Update task status on failure
                if (env.TASK_ID) {
                    sh "task-master update-subtask --id=${env.TASK_ID}.1 --prompt='Build failed: ${env.BUILD_URL}'"
                }
            }
        }
    }
}
```

## Code Quality Integration

### ESLint Configuration
```json
{
  "eslintConfig": {
    "extends": [
      "eslint:recommended",
      "@typescript-eslint/recommended",
      "prettier"
    ],
    "plugins": [
      "@typescript-eslint",
      "taskmaster"
    ],
    "rules": {
      "taskmaster/require-task-id": "error",
      "taskmaster/valid-commit-format": "error",
      "@typescript-eslint/no-unused-vars": "error",
      "@typescript-eslint/explicit-function-return-type": "warn"
    },
    "overrides": [
      {
        "files": ["tasks/**/*.md"],
        "rules": {
          "taskmaster/validate-task-structure": "error"
        }
      }
    ]
  }
}
```

### Prettier Configuration
```json
{
  "prettierConfig": {
    "semi": true,
    "trailingComma": "es5",
    "singleQuote": true,
    "printWidth": 80,
    "tabWidth": 2,
    "useTabs": false,
    "plugins": ["prettier-plugin-taskmaster"]
  }
}
```

### Husky Git Hooks
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged && task-master validate-dependencies",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS && taskmaster-commit-validator",
      "pre-push": "npm run test && task-master generate"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{md,json}": [
      "prettier --write",
      "git add"
    ],
    "tasks/**/*.md": [
      "taskmaster-task-validator",
      "git add"
    ]
  }
}
```

## Testing Framework Integration

### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/index.ts',
    '!src/**/*.d.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 85,
      functions: 85,
      lines: 85,
      statements: 85,
    },
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}',
  ],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  reporters: [
    'default',
    ['jest-junit', { outputDirectory: 'test-results', outputName: 'junit.xml' }],
    ['jest-taskmaster-reporter', { taskFile: 'tasks/tasks.json' }],
  ],
};
```

### TaskMaster Test Reporter
```javascript
// jest-taskmaster-reporter.js
class TaskMasterReporter {
  constructor(globalConfig, options) {
    this.globalConfig = globalConfig;
    this.options = options;
  }

  onRunComplete(contexts, results) {
    const taskFile = this.options.taskFile;
    const testResults = this.processTestResults(results);
    
    // Update task status based on test results
    this.updateTaskStatus(testResults);
    
    // Generate test report
    this.generateTestReport(testResults);
  }

  processTestResults(results) {
    return {
      numTotalTests: results.numTotalTests,
      numPassedTests: results.numPassedTests,
      numFailedTests: results.numFailedTests,
      testSuites: results.testResults.map(suite => ({
        testFilePath: suite.testFilePath,
        numPassingTests: suite.numPassingTests,
        numFailingTests: suite.numFailingTests,
        failureMessage: suite.failureMessage,
      })),
    };
  }

  updateTaskStatus(testResults) {
    if (testResults.numFailedTests === 0) {
      // All tests passed - update relevant tasks
      exec('task-master update-task --id=current --prompt="All tests passing"');
    } else {
      // Tests failed - create or update failure tasks
      exec(`task-master add-task --prompt="Fix ${testResults.numFailedTests} failing tests" --priority=high`);
    }
  }
}

module.exports = TaskMasterReporter;
```

## Project Management Integration

### Jira Integration
```javascript
const jiraIntegration = {
  async syncTaskToJira(taskId) {
    const task = await taskmaster.getTask(taskId);
    const jiraIssue = {
      fields: {
        project: { key: 'DAFNCK' },
        summary: task.title,
        description: task.description,
        issuetype: { name: this.mapTaskTypeToJiraIssueType(task.category) },
        priority: { name: this.mapPriorityToJira(task.priority) },
        assignee: { name: this.mapAssigneeToJira(task.assignee) },
        customfield_10001: taskId, // TaskMaster ID field
      },
    };

    const response = await this.jiraClient.issues.createIssue(jiraIssue);
    
    // Update task with Jira issue key
    await taskmaster.updateTask(taskId, {
      jiraKey: response.key,
      jiraUrl: `${this.jiraBaseUrl}/browse/${response.key}`,
    });

    return response;
  },

  async syncJiraToTask(jiraKey) {
    const jiraIssue = await this.jiraClient.issues.getIssue({ issueKey: jiraKey });
    const taskId = jiraIssue.fields.customfield_10001;

    if (taskId) {
      const statusMapping = {
        'To Do': 'pending',
        'In Progress': 'in-progress',
        'In Review': 'review',
        'Done': 'done',
      };

      const newStatus = statusMapping[jiraIssue.fields.status.name];
      if (newStatus) {
        await taskmaster.setTaskStatus(taskId, newStatus);
      }
    }
  },

  setupWebhooks() {
    // Setup Jira webhook to sync status changes
    const webhookConfig = {
      name: 'TaskMaster Sync',
      url: `${this.webhookBaseUrl}/jira-webhook`,
      events: ['jira:issue_updated', 'jira:issue_created'],
      filters: {
        'issue-related-events-section': 'project = DAFNCK',
      },
    };

    return this.jiraClient.webhooks.createWebhook(webhookConfig);
  },
};
```

### Slack Integration
```javascript
const slackIntegration = {
  async sendTaskNotification(taskId, event) {
    const task = await taskmaster.getTask(taskId);
    const message = this.formatTaskMessage(task, event);
    
    await this.slackClient.chat.postMessage({
      channel: '#development',
      text: message.text,
      blocks: message.blocks,
    });
  },

  formatTaskMessage(task, event) {
    const eventMessages = {
      created: `🆕 New task created: *${task.title}*`,
      updated: `📝 Task updated: *${task.title}*`,
      completed: `✅ Task completed: *${task.title}*`,
      blocked: `🚫 Task blocked: *${task.title}*`,
    };

    return {
      text: eventMessages[event],
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: eventMessages[event],
          },
        },
        {
          type: 'section',
          fields: [
            {
              type: 'mrkdwn',
              text: `*ID:* ${task.id}`,
            },
            {
              type: 'mrkdwn',
              text: `*Priority:* ${task.priority}`,
            },
            {
              type: 'mrkdwn',
              text: `*Status:* ${task.status}`,
            },
            {
              type: 'mrkdwn',
              text: `*Assignee:* ${task.assignee}`,
            },
          ],
        },
        {
          type: 'actions',
          elements: [
            {
              type: 'button',
              text: {
                type: 'plain_text',
                text: 'View Task',
              },
              url: `${this.taskMasterUrl}/tasks/${task.id}`,
            },
          ],
        },
      ],
    };
  },

  setupSlashCommands() {
    // Setup Slack slash commands for TaskMaster
    const commands = [
      {
        command: '/tm-list',
        description: 'List current tasks',
        handler: this.handleListCommand,
      },
      {
        command: '/tm-next',
        description: 'Get next task to work on',
        handler: this.handleNextCommand,
      },
      {
        command: '/tm-status',
        description: 'Update task status',
        handler: this.handleStatusCommand,
      },
    ];

    return commands;
  },
};
```

## Monitoring and Analytics

### Performance Monitoring
```javascript
const performanceMonitoring = {
  async trackWorkflowMetrics() {
    const metrics = {
      taskCompletionRate: await this.calculateCompletionRate(),
      averageTaskDuration: await this.calculateAverageDuration(),
      blockedTaskCount: await this.getBlockedTaskCount(),
      codeQualityScore: await this.getCodeQualityScore(),
      deploymentFrequency: await this.getDeploymentFrequency(),
    };

    await this.sendMetricsToMonitoring(metrics);
    return metrics;
  },

  async generateWorkflowReport() {
    const report = {
      timestamp: new Date().toISOString(),
      period: 'weekly',
      metrics: await this.trackWorkflowMetrics(),
      trends: await this.calculateTrends(),
      recommendations: await this.generateRecommendations(),
    };

    await this.saveReport(report);
    await this.sendReportNotification(report);
    
    return report;
  },

  setupDashboard() {
    // Configure monitoring dashboard
    const dashboardConfig = {
      name: 'TaskMaster Development Workflow',
      panels: [
        {
          title: 'Task Completion Rate',
          type: 'stat',
          targets: ['taskmaster.completion_rate'],
        },
        {
          title: 'Active Tasks by Status',
          type: 'pie',
          targets: ['taskmaster.tasks_by_status'],
        },
        {
          title: 'Code Quality Trends',
          type: 'graph',
          targets: ['taskmaster.code_quality_score'],
        },
        {
          title: 'Deployment Frequency',
          type: 'stat',
          targets: ['taskmaster.deployment_frequency'],
        },
      ],
    };

    return dashboardConfig;
  },
};
```

### Automated Reporting
```bash
#!/bin/bash
# generate-workflow-report.sh

echo "📊 Generating TaskMaster workflow report..."

# Collect metrics
TOTAL_TASKS=$(task-master list | wc -l)
COMPLETED_TASKS=$(task-master list --status=done | wc -l)
IN_PROGRESS_TASKS=$(task-master list --status=in-progress | wc -l)
BLOCKED_TASKS=$(task-master list --status=blocked | wc -l)

# Calculate completion rate
COMPLETION_RATE=$(echo "scale=2; $COMPLETED_TASKS * 100 / $TOTAL_TASKS" | bc)

# Generate report
REPORT_FILE="reports/workflow-report-$(date +%Y%m%d).md"
mkdir -p reports

cat > "$REPORT_FILE" << EOF
# TaskMaster Workflow Report - $(date +"%Y-%m-%d")

## Summary
- **Total Tasks**: $TOTAL_TASKS
- **Completed**: $COMPLETED_TASKS
- **In Progress**: $IN_PROGRESS_TASKS
- **Blocked**: $BLOCKED_TASKS
- **Completion Rate**: $COMPLETION_RATE%

## Task Breakdown
$(task-master list --with-subtasks)

## Recent Activity
$(git log --oneline --since="1 week ago" | head -10)

## Quality Metrics
- **Test Coverage**: $(npm run test:coverage --silent | grep -o '[0-9]*\.[0-9]*%' | head -1)
- **Code Quality**: $(npm run lint --silent | grep -c "error" || echo "0") errors

## Recommendations
$(task-master complexity-report | tail -5)

---
Generated on $(date) by TaskMaster Workflow Integration
EOF

echo "📋 Report generated: $REPORT_FILE"

# Send to Slack if webhook configured
if [[ -n "$SLACK_WEBHOOK_URL" ]]; then
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"📊 Weekly TaskMaster Report\n\`\`\`$(cat $REPORT_FILE)\`\`\`\"}" \
        "$SLACK_WEBHOOK_URL"
fi
```

---

**Implementation Status**: Ready for Deployment  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-10 