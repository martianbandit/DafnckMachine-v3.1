# Integration Testing Framework
## DafnckMachine v3.1 - Comprehensive Testing Strategy & Quality Assurance

### Document Overview
This document defines the comprehensive integration testing framework for DafnckMachine v3.1, establishing testing strategies, automation frameworks, quality assurance processes, and continuous testing practices to ensure robust, reliable, and high-quality software delivery across the AI agent orchestration platform.

### Testing Philosophy

#### Core Testing Principles
1. **Shift-Left Testing**: Early testing integration in development lifecycle
2. **Test Automation**: Automated testing for consistency and efficiency
3. **Risk-Based Testing**: Focus on high-risk areas and critical paths
4. **Continuous Testing**: Testing integrated into CI/CD pipelines
5. **Quality Gates**: Defined quality criteria for release decisions

#### Testing Pyramid Strategy
```typescript
// Testing Pyramid Implementation
interface TestingPyramid {
  unitTests: {
    percentage: 70;
    scope: "Individual functions and components";
    tools: ["Jest", "Mocha", "PyTest"];
    coverage: 90;
  };
  integrationTests: {
    percentage: 20;
    scope: "Service interactions and API contracts";
    tools: ["Supertest", "TestContainers", "Postman"];
    coverage: 80;
  };
  e2eTests: {
    percentage: 10;
    scope: "Complete user workflows";
    tools: ["Playwright", "Cypress", "Selenium"];
    coverage: 60;
  };
}
```

### Integration Testing Strategy

#### 1. Service Integration Testing
```typescript
// Service Integration Test Configuration
interface ServiceIntegrationTests {
  apiTesting: {
    framework: "Supertest";
    testTypes: [
      "Contract testing",
      "API endpoint validation",
      "Request/response validation",
      "Error handling",
      "Authentication/authorization"
    ];
    coverage: {
      endpoints: 100;
      statusCodes: 100;
      errorScenarios: 90;
    };
  };
  
  databaseIntegration: {
    framework: "TestContainers";
    testTypes: [
      "CRUD operations",
      "Transaction handling",
      "Data consistency",
      "Migration testing",
      "Performance testing"
    ];
    setup: {
      containerImage: "postgres:15";
      testData: "fixtures/test-data.sql";
      cleanup: "automatic";
    };
  };
  
  messageQueueTesting: {
    framework: "TestContainers";
    testTypes: [
      "Message publishing",
      "Message consumption",
      "Dead letter queues",
      "Message ordering",
      "Retry mechanisms"
    ];
    setup: {
      containerImage: "apache/kafka:latest";
      topics: ["test-events", "test-commands"];
      cleanup: "automatic";
    };
  };
}

// Example Service Integration Test
describe('Agent Orchestration Service Integration', () => {
  let app: Application;
  let database: PostgreSqlContainer;
  let redis: RedisContainer;
  
  beforeAll(async () => {
    // Start test containers
    database = await new PostgreSqlContainer()
      .withDatabase('test_db')
      .withUsername('test_user')
      .withPassword('test_pass')
      .start();
    
    redis = await new RedisContainer()
      .withExposedPorts(6379)
      .start();
    
    // Initialize application with test configuration
    app = await createTestApp({
      database: {
        url: database.getConnectionUri()
      },
      redis: {
        url: `redis://${redis.getHost()}:${redis.getMappedPort(6379)}`
      }
    });
  });
  
  afterAll(async () => {
    await database.stop();
    await redis.stop();
    await app.close();
  });
  
  describe('Agent Management', () => {
    it('should create agent and persist to database', async () => {
      const agentData = {
        name: 'test-agent',
        type: 'orchestrator',
        configuration: { maxTasks: 10 }
      };
      
      const response = await request(app)
        .post('/api/agents')
        .send(agentData)
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(String),
        name: 'test-agent',
        status: 'active'
      });
      
      // Verify database persistence
      const agent = await database.query(
        'SELECT * FROM agents WHERE id = $1',
        [response.body.id]
      );
      expect(agent.rows).toHaveLength(1);
    });
    
    it('should handle agent creation with invalid data', async () => {
      const invalidData = { name: '' };
      
      const response = await request(app)
        .post('/api/agents')
        .send(invalidData)
        .expect(400);
      
      expect(response.body).toMatchObject({
        error: 'Validation failed',
        details: expect.arrayContaining([
          expect.objectContaining({
            field: 'name',
            message: 'Name is required'
          })
        ])
      });
    });
  });
});
```

#### 2. Contract Testing
```typescript
// Pact Contract Testing Configuration
interface ContractTestingFramework {
  provider: "Pact";
  contracts: {
    userService: {
      consumers: ["web-app", "mobile-app"];
      interactions: [
        {
          description: "Get user profile",
          request: {
            method: "GET",
            path: "/api/users/123",
            headers: { "Authorization": "Bearer token" }
          },
          response: {
            status: 200,
            headers: { "Content-Type": "application/json" },
            body: {
              id: "123",
              email: "user@example.com",
              name: "John Doe"
            }
          }
        }
      ];
    };
  };
}

// Example Pact Consumer Test
import { Pact } from '@pact-foundation/pact';

describe('User Service Contract', () => {
  const provider = new Pact({
    consumer: 'web-app',
    provider: 'user-service',
    port: 1234,
    log: path.resolve(process.cwd(), 'logs', 'pact.log'),
    dir: path.resolve(process.cwd(), 'pacts'),
    logLevel: 'INFO'
  });
  
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());
  afterEach(() => provider.verify());
  
  it('should get user profile', async () => {
    await provider.addInteraction({
      state: 'user exists',
      uponReceiving: 'a request for user profile',
      withRequest: {
        method: 'GET',
        path: '/api/users/123',
        headers: {
          'Authorization': 'Bearer valid-token'
        }
      },
      willRespondWith: {
        status: 200,
        headers: {
          'Content-Type': 'application/json'
        },
        body: {
          id: '123',
          email: 'user@example.com',
          name: 'John Doe',
          createdAt: Matchers.iso8601DateTime()
        }
      }
    });
    
    const userService = new UserService('http://localhost:1234');
    const user = await userService.getProfile('123', 'valid-token');
    
    expect(user).toMatchObject({
      id: '123',
      email: 'user@example.com',
      name: 'John Doe'
    });
  });
});
```

#### 3. End-to-End Testing
```typescript
// E2E Testing Configuration
interface E2ETestingFramework {
  framework: "Playwright";
  browsers: ["chromium", "firefox", "webkit"];
  environments: ["staging", "production"];
  testTypes: [
    "User journey testing",
    "Cross-browser compatibility",
    "Performance testing",
    "Accessibility testing",
    "Visual regression testing"
  ];
}

// Example E2E Test
import { test, expect } from '@playwright/test';

test.describe('Agent Management Workflow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'admin@dafnckmachine.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('should create and configure new agent', async ({ page }) => {
    // Navigate to agent creation
    await page.click('[data-testid="create-agent-button"]');
    await expect(page).toHaveURL('/agents/create');
    
    // Fill agent details
    await page.fill('[data-testid="agent-name"]', 'Test Agent');
    await page.selectOption('[data-testid="agent-type"]', 'orchestrator');
    await page.fill('[data-testid="max-tasks"]', '10');
    
    // Submit form
    await page.click('[data-testid="create-button"]');
    
    // Verify success
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('Agent created successfully');
    
    // Verify agent appears in list
    await page.goto('/agents');
    await expect(page.locator('[data-testid="agent-list"]'))
      .toContainText('Test Agent');
  });
  
  test('should handle agent workflow execution', async ({ page }) => {
    // Create workflow
    await page.goto('/workflows/create');
    await page.fill('[data-testid="workflow-name"]', 'Test Workflow');
    
    // Add tasks
    await page.click('[data-testid="add-task-button"]');
    await page.fill('[data-testid="task-name"]', 'Initial Task');
    await page.selectOption('[data-testid="task-type"]', 'data-processing');
    
    // Save workflow
    await page.click('[data-testid="save-workflow"]');
    
    // Execute workflow
    await page.click('[data-testid="execute-workflow"]');
    
    // Monitor execution
    await expect(page.locator('[data-testid="workflow-status"]'))
      .toContainText('Running');
    
    // Wait for completion
    await page.waitForSelector('[data-testid="workflow-status"]:has-text("Completed")', {
      timeout: 30000
    });
    
    // Verify results
    await expect(page.locator('[data-testid="execution-results"]'))
      .toBeVisible();
  });
});
```

### Test Automation Framework

#### 1. CI/CD Integration
```yaml
# GitHub Actions Workflow
name: Integration Tests
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
        env:
          CI: true
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
          CI: true
      
      - name: Run contract tests
        run: npm run test:contract
        env:
          CI: true
      
      - name: Publish Pact contracts
        run: npm run pact:publish
        env:
          PACT_BROKER_BASE_URL: ${{ secrets.PACT_BROKER_URL }}
          PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: integration
          name: integration-tests

  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Start application
        run: |
          npm run build
          npm run start:test &
          npx wait-on http://localhost:3000
      
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          BASE_URL: http://localhost:3000
      
      - name: Upload E2E artifacts
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

#### 2. Test Data Management
```typescript
// Test Data Factory
interface TestDataFactory {
  users: {
    admin: () => UserData;
    regular: () => UserData;
    inactive: () => UserData;
  };
  agents: {
    orchestrator: () => AgentData;
    worker: () => AgentData;
    analyzer: () => AgentData;
  };
  workflows: {
    simple: () => WorkflowData;
    complex: () => WorkflowData;
    parallel: () => WorkflowData;
  };
}

// Example Test Data Factory Implementation
export class TestDataFactory {
  static createUser(overrides: Partial<UserData> = {}): UserData {
    return {
      id: faker.string.uuid(),
      email: faker.internet.email(),
      name: faker.person.fullName(),
      role: 'user',
      status: 'active',
      createdAt: new Date(),
      ...overrides
    };
  }
  
  static createAgent(overrides: Partial<AgentData> = {}): AgentData {
    return {
      id: faker.string.uuid(),
      name: faker.company.name(),
      type: 'orchestrator',
      status: 'active',
      configuration: {
        maxTasks: faker.number.int({ min: 1, max: 20 }),
        timeout: faker.number.int({ min: 30, max: 300 })
      },
      createdAt: new Date(),
      ...overrides
    };
  }
  
  static createWorkflow(overrides: Partial<WorkflowData> = {}): WorkflowData {
    return {
      id: faker.string.uuid(),
      name: faker.lorem.words(3),
      description: faker.lorem.sentence(),
      status: 'draft',
      tasks: [
        {
          id: faker.string.uuid(),
          name: 'Initial Task',
          type: 'data-processing',
          order: 1
        }
      ],
      createdAt: new Date(),
      ...overrides
    };
  }
}

// Database Seeding
export class DatabaseSeeder {
  static async seedTestData(database: Database): Promise<void> {
    // Create test users
    const users = [
      TestDataFactory.createUser({ role: 'admin', email: 'admin@test.com' }),
      TestDataFactory.createUser({ role: 'user', email: 'user@test.com' })
    ];
    
    for (const user of users) {
      await database.query(
        'INSERT INTO users (id, email, name, role, status) VALUES ($1, $2, $3, $4, $5)',
        [user.id, user.email, user.name, user.role, user.status]
      );
    }
    
    // Create test agents
    const agents = [
      TestDataFactory.createAgent({ type: 'orchestrator' }),
      TestDataFactory.createAgent({ type: 'worker' })
    ];
    
    for (const agent of agents) {
      await database.query(
        'INSERT INTO agents (id, name, type, status, configuration) VALUES ($1, $2, $3, $4, $5)',
        [agent.id, agent.name, agent.type, agent.status, JSON.stringify(agent.configuration)]
      );
    }
  }
  
  static async cleanupTestData(database: Database): Promise<void> {
    await database.query('DELETE FROM workflow_executions');
    await database.query('DELETE FROM workflows');
    await database.query('DELETE FROM agents');
    await database.query('DELETE FROM users');
  }
}
```

### Performance Testing

#### 1. Load Testing Configuration
```typescript
// K6 Load Testing Scripts
interface LoadTestingConfiguration {
  scenarios: {
    userRegistration: {
      executor: "ramping-vus";
      startVUs: 0;
      stages: [
        { duration: "2m", target: 10 },
        { duration: "5m", target: 50 },
        { duration: "2m", target: 0 }
      ];
    };
    agentOrchestration: {
      executor: "constant-vus";
      vus: 20;
      duration: "10m";
    };
    workflowExecution: {
      executor: "per-vu-iterations";
      vus: 10;
      iterations: 100;
    };
  };
  thresholds: {
    http_req_duration: ["p(95)<500"];
    http_req_failed: ["rate<0.1"];
    http_reqs: ["rate>100"];
  };
}

// Example K6 Load Test
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

export let errorRate = new Rate('errors');

export let options = {
  stages: [
    { duration: '2m', target: 10 },
    { duration: '5m', target: 50 },
    { duration: '2m', target: 0 }
  ],
  thresholds: {
    errors: ['rate<0.1'],
    http_req_duration: ['p(95)<500']
  }
};

export default function() {
  // Login
  let loginResponse = http.post('http://api.dafnckmachine.com/auth/login', {
    email: 'test@example.com',
    password: 'password123'
  });
  
  check(loginResponse, {
    'login successful': (r) => r.status === 200,
    'token received': (r) => r.json('token') !== undefined
  }) || errorRate.add(1);
  
  let token = loginResponse.json('token');
  
  // Create agent
  let agentResponse = http.post(
    'http://api.dafnckmachine.com/api/agents',
    JSON.stringify({
      name: `Agent-${__VU}-${__ITER}`,
      type: 'orchestrator',
      configuration: { maxTasks: 10 }
    }),
    {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    }
  );
  
  check(agentResponse, {
    'agent created': (r) => r.status === 201,
    'agent has id': (r) => r.json('id') !== undefined
  }) || errorRate.add(1);
  
  sleep(1);
}
```

#### 2. Stress Testing
```javascript
// Artillery.js Stress Test Configuration
module.exports = {
  config: {
    target: 'http://api.dafnckmachine.com',
    phases: [
      { duration: 60, arrivalRate: 10 },
      { duration: 120, arrivalRate: 50 },
      { duration: 60, arrivalRate: 100 },
      { duration: 180, arrivalRate: 200 }
    ],
    payload: {
      path: './test-data.csv',
      fields: ['email', 'password']
    }
  },
  scenarios: [
    {
      name: 'User Authentication Flow',
      weight: 40,
      flow: [
        {
          post: {
            url: '/auth/login',
            json: {
              email: '{{ email }}',
              password: '{{ password }}'
            },
            capture: {
              json: '$.token',
              as: 'authToken'
            }
          }
        },
        {
          get: {
            url: '/api/user/profile',
            headers: {
              'Authorization': 'Bearer {{ authToken }}'
            }
          }
        }
      ]
    },
    {
      name: 'Agent Operations',
      weight: 60,
      flow: [
        {
          post: {
            url: '/api/agents',
            json: {
              name: 'Load Test Agent {{ $randomString() }}',
              type: 'orchestrator'
            },
            headers: {
              'Authorization': 'Bearer {{ authToken }}'
            },
            capture: {
              json: '$.id',
              as: 'agentId'
            }
          }
        },
        {
          get: {
            url: '/api/agents/{{ agentId }}',
            headers: {
              'Authorization': 'Bearer {{ authToken }}'
            }
          }
        }
      ]
    }
  ]
};
```

### Quality Gates and Metrics

#### 1. Test Coverage Requirements
```typescript
// Coverage Configuration
interface CoverageRequirements {
  overall: {
    statements: 85;
    branches: 80;
    functions: 90;
    lines: 85;
  };
  critical: {
    statements: 95;
    branches: 90;
    functions: 100;
    lines: 95;
  };
  excludes: [
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/test/**",
    "**/coverage/**"
  ];
}

// Jest Coverage Configuration
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      statements: 85,
      branches: 80,
      functions: 90,
      lines: 85
    },
    './src/core/': {
      statements: 95,
      branches: 90,
      functions: 100,
      lines: 95
    }
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.test.{ts,tsx}',
    '!src/**/*.spec.{ts,tsx}',
    '!src/test/**'
  ]
};
```

#### 2. Quality Metrics Dashboard
```typescript
// Quality Metrics Collection
interface QualityMetrics {
  testMetrics: {
    totalTests: number;
    passedTests: number;
    failedTests: number;
    skippedTests: number;
    testCoverage: number;
    executionTime: number;
  };
  
  performanceMetrics: {
    responseTime: {
      p50: number;
      p95: number;
      p99: number;
    };
    throughput: number;
    errorRate: number;
    availability: number;
  };
  
  codeQualityMetrics: {
    codeComplexity: number;
    duplicateCode: number;
    technicalDebt: number;
    securityVulnerabilities: number;
  };
}

// Metrics Collection Service
export class MetricsCollector {
  static async collectTestMetrics(): Promise<TestMetrics> {
    const testResults = await this.runTestSuite();
    
    return {
      totalTests: testResults.numTotalTests,
      passedTests: testResults.numPassedTests,
      failedTests: testResults.numFailedTests,
      skippedTests: testResults.numPendingTests,
      testCoverage: testResults.coverageMap.getCoverageSummary().toJSON().statements.pct,
      executionTime: testResults.testResults.reduce((acc, result) => acc + result.perfStats.runtime, 0)
    };
  }
  
  static async collectPerformanceMetrics(): Promise<PerformanceMetrics> {
    const loadTestResults = await this.runLoadTests();
    
    return {
      responseTime: {
        p50: loadTestResults.http_req_duration.p50,
        p95: loadTestResults.http_req_duration.p95,
        p99: loadTestResults.http_req_duration.p99
      },
      throughput: loadTestResults.http_reqs.rate,
      errorRate: loadTestResults.http_req_failed.rate,
      availability: (1 - loadTestResults.http_req_failed.rate) * 100
    };
  }
}
```

### Test Environment Management

#### 1. Environment Configuration
```typescript
// Test Environment Setup
interface TestEnvironmentConfig {
  environments: {
    unit: {
      database: "in-memory";
      services: "mocked";
      external: "stubbed";
    };
    integration: {
      database: "testcontainers";
      services: "real";
      external: "stubbed";
    };
    e2e: {
      database: "dedicated-test-db";
      services: "real";
      external: "staging";
    };
    staging: {
      database: "staging-db";
      services: "real";
      external: "real";
    };
  };
}

// Environment Manager
export class TestEnvironmentManager {
  static async setupEnvironment(type: EnvironmentType): Promise<TestEnvironment> {
    switch (type) {
      case 'unit':
        return this.setupUnitTestEnvironment();
      case 'integration':
        return this.setupIntegrationTestEnvironment();
      case 'e2e':
        return this.setupE2ETestEnvironment();
      default:
        throw new Error(`Unknown environment type: ${type}`);
    }
  }
  
  private static async setupIntegrationTestEnvironment(): Promise<TestEnvironment> {
    const containers = await this.startTestContainers();
    const database = await this.initializeDatabase(containers.postgres);
    const redis = await this.initializeRedis(containers.redis);
    
    return {
      database,
      redis,
      cleanup: async () => {
        await containers.postgres.stop();
        await containers.redis.stop();
      }
    };
  }
}
```

#### 2. Test Data Isolation
```typescript
// Test Data Isolation Strategy
interface TestDataIsolation {
  strategy: "database-per-test" | "transaction-rollback" | "cleanup-hooks";
  implementation: {
    beforeEach: () => Promise<void>;
    afterEach: () => Promise<void>;
    beforeAll: () => Promise<void>;
    afterAll: () => Promise<void>;
  };
}

// Database Transaction Rollback Strategy
export class TransactionTestStrategy {
  private transaction: DatabaseTransaction;
  
  async beforeEach(): Promise<void> {
    this.transaction = await database.beginTransaction();
  }
  
  async afterEach(): Promise<void> {
    await this.transaction.rollback();
  }
  
  getDatabase(): Database {
    return this.transaction;
  }
}

// Example Usage
describe('User Service Integration Tests', () => {
  const testStrategy = new TransactionTestStrategy();
  
  beforeEach(async () => {
    await testStrategy.beforeEach();
  });
  
  afterEach(async () => {
    await testStrategy.afterEach();
  });
  
  it('should create user', async () => {
    const database = testStrategy.getDatabase();
    const userService = new UserService(database);
    
    const user = await userService.create({
      email: 'test@example.com',
      name: 'Test User'
    });
    
    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });
});
```

### Continuous Testing Pipeline

#### 1. Pipeline Configuration
```yaml
# Azure DevOps Pipeline
trigger:
  branches:
    include:
      - main
      - develop
      - feature/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: test-variables
  - name: NODE_VERSION
    value: '18.x'

stages:
  - stage: UnitTests
    displayName: 'Unit Tests'
    jobs:
      - job: RunUnitTests
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(NODE_VERSION)
          
          - task: Npm@1
            inputs:
              command: 'ci'
          
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run test:unit'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/unit/junit.xml'
              testRunTitle: 'Unit Tests'
          
          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: 'coverage/cobertura-coverage.xml'

  - stage: IntegrationTests
    displayName: 'Integration Tests'
    dependsOn: UnitTests
    jobs:
      - job: RunIntegrationTests
        services:
          postgres:
            image: postgres:15
            env:
              POSTGRES_PASSWORD: postgres
              POSTGRES_DB: test_db
        
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(NODE_VERSION)
          
          - task: Npm@1
            inputs:
              command: 'ci'
          
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run test:integration'
            env:
              DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/integration/junit.xml'
              testRunTitle: 'Integration Tests'

  - stage: E2ETests
    displayName: 'End-to-End Tests'
    dependsOn: IntegrationTests
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - job: RunE2ETests
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(NODE_VERSION)
          
          - task: Npm@1
            inputs:
              command: 'ci'
          
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run build'
          
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run start:test'
            env:
              NODE_ENV: test
          
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run test:e2e'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/e2e/junit.xml'
              testRunTitle: 'E2E Tests'
          
          - task: PublishBuildArtifacts@1
            condition: failed()
            inputs:
              pathToPublish: 'test-results/e2e/screenshots'
              artifactName: 'e2e-screenshots'
```

### Test Reporting and Analytics

#### 1. Test Results Dashboard
```typescript
// Test Results Aggregation
interface TestResultsDashboard {
  overview: {
    totalTests: number;
    passRate: number;
    failRate: number;
    coverage: number;
    trend: TestTrend[];
  };
  
  breakdown: {
    byType: TestTypeBreakdown;
    byModule: ModuleBreakdown;
    byEnvironment: EnvironmentBreakdown;
  };
  
  performance: {
    executionTime: number;
    slowestTests: TestExecution[];
    flakiestTests: TestFlakiness[];
  };
}

// Test Analytics Service
export class TestAnalyticsService {
  static async generateDashboard(): Promise<TestResultsDashboard> {
    const testResults = await this.aggregateTestResults();
    const coverage = await this.getCoverageData();
    const performance = await this.getPerformanceMetrics();
    
    return {
      overview: {
        totalTests: testResults.total,
        passRate: (testResults.passed / testResults.total) * 100,
        failRate: (testResults.failed / testResults.total) * 100,
        coverage: coverage.overall,
        trend: await this.getTrendData()
      },
      breakdown: {
        byType: this.groupByType(testResults),
        byModule: this.groupByModule(testResults),
        byEnvironment: this.groupByEnvironment(testResults)
      },
      performance: {
        executionTime: performance.totalTime,
        slowestTests: performance.slowest,
        flakiestTests: await this.getFlakiestTests()
      }
    };
  }
}
```

### Implementation Timeline

#### Phase 1: Foundation (Weeks 1-2)
- Test framework setup
- Unit testing infrastructure
- Basic CI/CD integration

#### Phase 2: Integration Testing (Weeks 3-4)
- Service integration tests
- Contract testing implementation
- Test data management

#### Phase 3: E2E and Performance (Weeks 5-6)
- End-to-end test suite
- Performance testing framework
- Load testing implementation

#### Phase 4: Quality Gates (Weeks 7-8)
- Quality metrics dashboard
- Test reporting system
- Continuous improvement process

### Conclusion

This integration testing framework provides comprehensive quality assurance for DafnckMachine v3.1, featuring:

- **Multi-layered testing strategy** with unit, integration, and E2E tests
- **Automated testing pipeline** integrated with CI/CD
- **Performance and load testing** for scalability validation
- **Quality gates and metrics** for release decisions
- **Test environment management** for consistent testing
- **Comprehensive reporting** for continuous improvement

The framework ensures high-quality software delivery while maintaining development velocity and system reliability. 