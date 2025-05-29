# TaskMaster Configuration Templates - DafnckMachine v3.1

## Document Information
- **Document Type**: Configuration Templates
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Status**: Active
- **Owner**: Task Planning Agent

## Overview
This document provides comprehensive configuration templates for TaskMaster implementation, including setup configurations, environment templates, integration patterns, and customization options to ensure consistent and optimal TaskMaster deployment across different environments and use cases.

## Core Configuration Templates

### Basic TaskMaster Configuration
```json
{
  "taskmasterConfig": {
    "version": "3.1.0",
    "projectName": "DafnckMachine",
    "projectDescription": "AI-powered task management and orchestration system",
    "projectVersion": "3.1.0",
    "authorName": "Development Team",
    "createdAt": "2025-01-27T00:00:00Z",
    "lastUpdated": "2025-01-27T00:00:00Z",
    
    "models": {
      "main": "claude-3-5-sonnet-20241022",
      "research": "claude-3-5-sonnet-20241022",
      "fallback": "gpt-4o"
    },
    
    "parameters": {
      "maxTokens": 4000,
      "temperature": 0.7,
      "topP": 0.9,
      "frequencyPenalty": 0.0,
      "presencePenalty": 0.0
    },
    
    "logging": {
      "level": "info",
      "enableFileLogging": true,
      "logDirectory": "./logs",
      "maxLogFiles": 10,
      "maxLogSize": "10MB"
    },
    
    "defaults": {
      "defaultSubtasks": 5,
      "defaultPriority": "medium",
      "defaultStatus": "pending",
      "autoGenerate": true,
      "enableNotifications": true
    }
  }
}
```

### Environment-Specific Templates

#### Development Environment
```json
{
  "environment": "development",
  "taskmasterConfig": {
    "models": {
      "main": "claude-3-5-sonnet-20241022",
      "research": "claude-3-5-sonnet-20241022",
      "fallback": "gpt-4o"
    },
    "parameters": {
      "maxTokens": 2000,
      "temperature": 0.8,
      "enableDebugMode": true,
      "verboseLogging": true
    },
    "features": {
      "enableExperimentalFeatures": true,
      "enableMockData": true,
      "enableTestMode": true,
      "skipValidation": false
    },
    "integrations": {
      "enableLocalMCP": true,
      "enableHotReload": true,
      "enableDevTools": true
    }
  },
  "environmentVariables": {
    "TASKMASTER_LOG_LEVEL": "debug",
    "TASKMASTER_ENABLE_MOCK": "true",
    "TASKMASTER_DEV_MODE": "true"
  }
}
```

#### Staging Environment
```json
{
  "environment": "staging",
  "taskmasterConfig": {
    "models": {
      "main": "claude-3-5-sonnet-20241022",
      "research": "claude-3-5-sonnet-20241022",
      "fallback": "gpt-4o"
    },
    "parameters": {
      "maxTokens": 3000,
      "temperature": 0.7,
      "enableDebugMode": false,
      "verboseLogging": false
    },
    "features": {
      "enableExperimentalFeatures": false,
      "enableMockData": false,
      "enableTestMode": false,
      "skipValidation": false
    },
    "integrations": {
      "enablePerformanceMonitoring": true,
      "enableErrorTracking": true,
      "enableAnalytics": true
    }
  },
  "environmentVariables": {
    "TASKMASTER_LOG_LEVEL": "info",
    "TASKMASTER_ENABLE_MONITORING": "true",
    "TASKMASTER_STAGING_MODE": "true"
  }
}
```

#### Production Environment
```json
{
  "environment": "production",
  "taskmasterConfig": {
    "models": {
      "main": "claude-3-5-sonnet-20241022",
      "research": "claude-3-5-sonnet-20241022",
      "fallback": "gpt-4o"
    },
    "parameters": {
      "maxTokens": 4000,
      "temperature": 0.6,
      "enableDebugMode": false,
      "verboseLogging": false
    },
    "features": {
      "enableExperimentalFeatures": false,
      "enableMockData": false,
      "enableTestMode": false,
      "skipValidation": false
    },
    "security": {
      "enableEncryption": true,
      "enableAuditLogging": true,
      "enableRateLimiting": true,
      "maxRequestsPerMinute": 100
    },
    "performance": {
      "enableCaching": true,
      "cacheTimeout": 3600,
      "enableCompression": true,
      "maxConcurrentTasks": 50
    }
  },
  "environmentVariables": {
    "TASKMASTER_LOG_LEVEL": "warn",
    "TASKMASTER_PRODUCTION_MODE": "true",
    "TASKMASTER_ENABLE_SECURITY": "true"
  }
}
```

## MCP Integration Templates

### Cursor MCP Configuration
```json
{
  "mcpServers": {
    "taskmaster-ai": {
      "command": "npx",
      "args": ["-y", "task-master-ai", "mcp"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
        "OPENAI_API_KEY": "${OPENAI_API_KEY}",
        "PERPLEXITY_API_KEY": "${PERPLEXITY_API_KEY}",
        "GOOGLE_API_KEY": "${GOOGLE_API_KEY}",
        "MISTRAL_API_KEY": "${MISTRAL_API_KEY}",
        "XAI_API_KEY": "${XAI_API_KEY}",
        "OPENROUTER_API_KEY": "${OPENROUTER_API_KEY}"
      }
    }
  },
  "rules": [
    "Use TaskMaster MCP tools for task management operations",
    "Prefer MCP tools over CLI commands for better integration",
    "Always use projectRoot parameter with absolute paths",
    "Enable research mode for complex task analysis"
  ]
}
```

### Local MCP Server Configuration
```json
{
  "mcpServers": {
    "taskmaster-local": {
      "command": "node",
      "args": ["./node_modules/task-master-ai/dist/mcp/index.js"],
      "cwd": "/path/to/project",
      "env": {
        "TASKMASTER_PROJECT_ROOT": "/path/to/project",
        "TASKMASTER_CONFIG_PATH": "./.taskmasterconfig",
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
        "OPENAI_API_KEY": "${OPENAI_API_KEY}"
      }
    }
  }
}
```

## Task Configuration Templates

### Task Structure Template
```json
{
  "taskTemplate": {
    "id": "{{taskId}}",
    "title": "{{taskTitle}}",
    "description": "{{taskDescription}}",
    "status": "pending",
    "priority": "medium",
    "dependencies": [],
    "tags": [],
    "assignee": null,
    "estimatedHours": null,
    "actualHours": null,
    "createdAt": "{{timestamp}}",
    "updatedAt": "{{timestamp}}",
    "dueDate": null,
    "details": "{{taskDetails}}",
    "testStrategy": "{{testStrategy}}",
    "acceptanceCriteria": [],
    "subtasks": [],
    "metadata": {
      "complexity": null,
      "riskLevel": "low",
      "category": "development",
      "component": null
    }
  }
}
```

### Subtask Template
```json
{
  "subtaskTemplate": {
    "id": "{{parentId}}.{{subtaskIndex}}",
    "title": "{{subtaskTitle}}",
    "description": "{{subtaskDescription}}",
    "status": "pending",
    "dependencies": [],
    "estimatedHours": null,
    "actualHours": null,
    "details": "{{subtaskDetails}}",
    "parentId": "{{parentId}}",
    "order": "{{subtaskIndex}}",
    "createdAt": "{{timestamp}}",
    "updatedAt": "{{timestamp}}"
  }
}
```

### Task Categories Template
```json
{
  "taskCategories": {
    "development": {
      "color": "#4CAF50",
      "icon": "code",
      "defaultPriority": "medium",
      "estimatedHours": 8,
      "requiredSkills": ["programming", "testing"]
    },
    "design": {
      "color": "#FF9800",
      "icon": "palette",
      "defaultPriority": "medium",
      "estimatedHours": 6,
      "requiredSkills": ["design", "ux"]
    },
    "testing": {
      "color": "#2196F3",
      "icon": "bug_report",
      "defaultPriority": "high",
      "estimatedHours": 4,
      "requiredSkills": ["testing", "qa"]
    },
    "documentation": {
      "color": "#9C27B0",
      "icon": "description",
      "defaultPriority": "low",
      "estimatedHours": 3,
      "requiredSkills": ["writing", "technical"]
    },
    "infrastructure": {
      "color": "#607D8B",
      "icon": "cloud",
      "defaultPriority": "high",
      "estimatedHours": 6,
      "requiredSkills": ["devops", "infrastructure"]
    }
  }
}
```

## Workflow Configuration Templates

### Sprint Configuration
```json
{
  "sprintConfig": {
    "sprintDuration": 14,
    "sprintStartDay": "monday",
    "planningMeetingDuration": 2,
    "reviewMeetingDuration": 1,
    "retrospectiveDuration": 1,
    "dailyStandupDuration": 0.25,
    "capacityBuffer": 0.2,
    "velocityCalculation": "average_last_3_sprints",
    "autoTaskAssignment": true,
    "enableBurndownChart": true,
    "enableVelocityTracking": true
  }
}
```

### Quality Gates Configuration
```json
{
  "qualityGates": {
    "taskCompletion": {
      "requiredCriteria": [
        "code_review_approved",
        "tests_passing",
        "documentation_updated",
        "acceptance_criteria_met"
      ],
      "optionalCriteria": [
        "performance_benchmarks_met",
        "security_scan_clean"
      ],
      "blockingCriteria": [
        "critical_bugs_resolved",
        "security_vulnerabilities_fixed"
      ]
    },
    "sprintCompletion": {
      "requiredCriteria": [
        "all_tasks_completed",
        "integration_tests_passing",
        "deployment_successful"
      ],
      "qualityThresholds": {
        "testCoverage": 85,
        "codeQuality": 8.0,
        "performanceScore": 90
      }
    }
  }
}
```

### Notification Configuration
```json
{
  "notifications": {
    "channels": {
      "slack": {
        "enabled": true,
        "webhookUrl": "${SLACK_WEBHOOK_URL}",
        "defaultChannel": "#development",
        "mentionUsers": true
      },
      "email": {
        "enabled": true,
        "smtpServer": "${SMTP_SERVER}",
        "smtpPort": 587,
        "username": "${SMTP_USERNAME}",
        "password": "${SMTP_PASSWORD}"
      },
      "webhook": {
        "enabled": false,
        "url": "${WEBHOOK_URL}",
        "headers": {
          "Authorization": "Bearer ${WEBHOOK_TOKEN}"
        }
      }
    },
    "events": {
      "taskCreated": ["slack"],
      "taskCompleted": ["slack", "email"],
      "taskOverdue": ["slack", "email"],
      "sprintStarted": ["slack"],
      "sprintCompleted": ["slack", "email"],
      "qualityGateFailed": ["slack", "email"],
      "criticalError": ["slack", "email", "webhook"]
    },
    "templates": {
      "taskCreated": "New task created: {{task.title}} ({{task.id}})",
      "taskCompleted": "Task completed: {{task.title}} by {{task.assignee}}",
      "taskOverdue": "⚠️ Task overdue: {{task.title}} (due: {{task.dueDate}})",
      "sprintStarted": "🚀 Sprint {{sprint.number}} started",
      "sprintCompleted": "✅ Sprint {{sprint.number}} completed",
      "qualityGateFailed": "❌ Quality gate failed for {{task.title}}",
      "criticalError": "🚨 Critical error in TaskMaster: {{error.message}}"
    }
  }
}
```

## Integration Templates

### CI/CD Integration
```yaml
# GitHub Actions Template
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
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install TaskMaster
        run: npm install -g task-master-ai
        
      - name: Sync Tasks
        run: |
          task-master validate-dependencies
          task-master generate
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          
      - name: Update Task Status
        run: |
          task-master set-status --id=${{ github.event.number }} --status=in-progress
        if: github.event_name == 'pull_request'
        
      - name: Complete Task on Merge
        run: |
          task-master set-status --id=${{ github.event.number }} --status=done
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

### Jira Integration Template
```json
{
  "jiraIntegration": {
    "enabled": true,
    "serverUrl": "${JIRA_SERVER_URL}",
    "username": "${JIRA_USERNAME}",
    "apiToken": "${JIRA_API_TOKEN}",
    "projectKey": "DAFNCK",
    "issueTypeMapping": {
      "development": "Story",
      "bug": "Bug",
      "testing": "Test",
      "documentation": "Task"
    },
    "statusMapping": {
      "pending": "To Do",
      "in-progress": "In Progress",
      "review": "In Review",
      "done": "Done",
      "cancelled": "Cancelled"
    },
    "fieldMapping": {
      "title": "summary",
      "description": "description",
      "priority": "priority",
      "assignee": "assignee",
      "dueDate": "duedate"
    },
    "syncSettings": {
      "bidirectionalSync": true,
      "syncInterval": 300,
      "conflictResolution": "taskmaster_wins"
    }
  }
}
```

### Trello Integration Template
```json
{
  "trelloIntegration": {
    "enabled": false,
    "apiKey": "${TRELLO_API_KEY}",
    "token": "${TRELLO_TOKEN}",
    "boardId": "${TRELLO_BOARD_ID}",
    "listMapping": {
      "pending": "To Do",
      "in-progress": "Doing",
      "review": "Review",
      "done": "Done"
    },
    "labelMapping": {
      "high": "red",
      "medium": "yellow",
      "low": "green"
    },
    "syncSettings": {
      "createCards": true,
      "updateCards": true,
      "deleteCards": false,
      "syncComments": true
    }
  }
}
```

## Monitoring and Analytics Templates

### Performance Monitoring
```json
{
  "performanceMonitoring": {
    "enabled": true,
    "metrics": {
      "taskCompletionRate": {
        "enabled": true,
        "threshold": 0.85,
        "alertOnThreshold": true
      },
      "averageTaskDuration": {
        "enabled": true,
        "threshold": 8,
        "unit": "hours"
      },
      "sprintVelocity": {
        "enabled": true,
        "trackingPeriod": "last_5_sprints"
      },
      "qualityMetrics": {
        "enabled": true,
        "testCoverage": 85,
        "codeQuality": 8.0,
        "bugRate": 0.05
      }
    },
    "reporting": {
      "frequency": "daily",
      "recipients": ["team-lead@company.com"],
      "format": "json",
      "includeCharts": true
    }
  }
}
```

### Analytics Configuration
```json
{
  "analytics": {
    "enabled": true,
    "provider": "google_analytics",
    "trackingId": "${GA_TRACKING_ID}",
    "events": {
      "taskCreated": true,
      "taskCompleted": true,
      "taskUpdated": false,
      "sprintStarted": true,
      "sprintCompleted": true,
      "userActions": true
    },
    "customDimensions": {
      "taskCategory": 1,
      "taskPriority": 2,
      "assignee": 3,
      "sprintNumber": 4
    },
    "goals": {
      "taskCompletion": {
        "type": "destination",
        "value": "/task/completed"
      },
      "sprintCompletion": {
        "type": "event",
        "category": "sprint",
        "action": "completed"
      }
    }
  }
}
```

## Security Configuration Templates

### Security Settings
```json
{
  "security": {
    "authentication": {
      "enabled": true,
      "provider": "oauth2",
      "clientId": "${OAUTH_CLIENT_ID}",
      "clientSecret": "${OAUTH_CLIENT_SECRET}",
      "redirectUri": "${OAUTH_REDIRECT_URI}",
      "scopes": ["read", "write"]
    },
    "authorization": {
      "enabled": true,
      "roleBasedAccess": true,
      "roles": {
        "admin": ["read", "write", "delete", "admin"],
        "developer": ["read", "write"],
        "viewer": ["read"]
      }
    },
    "encryption": {
      "enabled": true,
      "algorithm": "AES-256-GCM",
      "keyRotation": true,
      "keyRotationInterval": 2592000
    },
    "auditLogging": {
      "enabled": true,
      "logLevel": "info",
      "retentionPeriod": 31536000,
      "events": ["login", "logout", "task_create", "task_update", "task_delete"]
    }
  }
}
```

### API Security Template
```json
{
  "apiSecurity": {
    "rateLimiting": {
      "enabled": true,
      "windowMs": 900000,
      "maxRequests": 100,
      "skipSuccessfulRequests": false
    },
    "cors": {
      "enabled": true,
      "origin": ["https://app.dafnck.com", "https://staging.dafnck.com"],
      "methods": ["GET", "POST", "PUT", "DELETE"],
      "allowedHeaders": ["Content-Type", "Authorization"]
    },
    "helmet": {
      "enabled": true,
      "contentSecurityPolicy": true,
      "hsts": true,
      "noSniff": true,
      "xssFilter": true
    },
    "validation": {
      "enabled": true,
      "strictMode": true,
      "sanitizeInput": true,
      "maxPayloadSize": "10mb"
    }
  }
}
```

## Backup and Recovery Templates

### Backup Configuration
```json
{
  "backup": {
    "enabled": true,
    "schedule": "0 2 * * *",
    "retention": {
      "daily": 7,
      "weekly": 4,
      "monthly": 12
    },
    "storage": {
      "provider": "aws_s3",
      "bucket": "${BACKUP_BUCKET}",
      "region": "${AWS_REGION}",
      "encryption": true
    },
    "includes": [
      "tasks.json",
      ".taskmasterconfig",
      "logs/",
      "reports/"
    ],
    "excludes": [
      "node_modules/",
      "tmp/",
      "*.log"
    ]
  }
}
```

### Disaster Recovery Template
```json
{
  "disasterRecovery": {
    "enabled": true,
    "rpo": 3600,
    "rto": 1800,
    "backupLocations": [
      "primary_datacenter",
      "secondary_datacenter",
      "cloud_storage"
    ],
    "failoverProcedure": {
      "automatic": true,
      "healthCheckInterval": 60,
      "failoverThreshold": 3,
      "notificationChannels": ["email", "slack"]
    },
    "recoveryProcedure": {
      "steps": [
        "validate_backup_integrity",
        "restore_configuration",
        "restore_data",
        "verify_functionality",
        "notify_stakeholders"
      ],
      "rollbackPlan": true
    }
  }
}
```

---

**Implementation Status**: Ready for Deployment  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-10 