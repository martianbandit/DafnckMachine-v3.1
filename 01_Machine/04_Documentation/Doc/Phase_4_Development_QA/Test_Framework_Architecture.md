# Test Framework Architecture - DafnckMachine v3.1

## Overview
Comprehensive test framework architecture for DafnckMachine v3.1, providing a robust, scalable, and maintainable testing infrastructure that supports unit testing, integration testing, end-to-end testing, performance testing, and security testing with automated execution and reporting.

## Testing Architecture Overview

### Testing Pyramid Structure
```
                    🔺 E2E Tests (10%)
                   /                \
                  /  Integration     \
                 /   Tests (20%)      \
                /                      \
               /________________________\
              Unit Tests (70%)
```

### Framework Components
1. **Test Execution Engine**: Jest, Mocha, Playwright
2. **Assertion Libraries**: Jest matchers, Chai, custom assertions
3. **Mocking Framework**: Jest mocks, Sinon.js, MSW
4. **Test Data Management**: Factories, fixtures, database seeding
5. **Reporting System**: Coverage reports, test results, analytics
6. **CI/CD Integration**: GitHub Actions, pipeline automation

## Core Testing Frameworks

### Unit Testing Framework (Jest)
```javascript
// jest.config.js
module.exports = {
  // Test environment configuration
  testEnvironment: 'node',
  
  // Test file patterns
  testMatch: [
    '<rootDir>/tests/unit/**/*.test.js',
    '<rootDir>/src/**/__tests__/**/*.test.js'
  ],
  
  // Setup files
  setupFilesAfterEnv: [
    '<rootDir>/tests/setup/jest.setup.js'
  ],
  
  // Coverage configuration
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
    '!src/config/**',
    '!src/migrations/**',
    '!src/**/*.test.js'
  ],
  
  // Coverage thresholds
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/services/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    },
    './src/domain/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95
    }
  },
  
  // Test timeout
  testTimeout: 10000,
  
  // Module name mapping
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@tests/(.*)$': '<rootDir>/tests/$1'
  },
  
  // Transform configuration
  transform: {
    '^.+\\.js$': 'babel-jest'
  },
  
  // Clear mocks between tests
  clearMocks: true,
  restoreMocks: true,
  
  // Verbose output
  verbose: true
};
```

### Integration Testing Framework
```javascript
// tests/integration/jest.config.js
module.exports = {
  ...require('../../jest.config.js'),
  
  testMatch: [
    '<rootDir>/tests/integration/**/*.test.js'
  ],
  
  setupFilesAfterEnv: [
    '<rootDir>/tests/setup/integration.setup.js'
  ],
  
  // Integration test specific configuration
  testEnvironment: 'node',
  testTimeout: 30000,
  
  // Global setup and teardown
  globalSetup: '<rootDir>/tests/setup/global-setup.js',
  globalTeardown: '<rootDir>/tests/setup/global-teardown.js',
  
  // Run tests serially for database consistency
  maxWorkers: 1
};
```

### End-to-End Testing Framework (Playwright)
```javascript
// playwright.config.js
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  // Test directory
  testDir: './tests/e2e',
  
  // Timeout configuration
  timeout: 30000,
  expect: {
    timeout: 5000
  },
  
  // Test execution configuration
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter configuration
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }]
  ],
  
  // Global setup
  globalSetup: require.resolve('./tests/setup/e2e-global-setup.js'),
  globalTeardown: require.resolve('./tests/setup/e2e-global-teardown.js'),
  
  // Test configuration
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 30000
  },
  
  // Browser projects
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] }
    },
    {
      name: 'mobile-safari',
      use: { ...devices['iPhone 12'] }
    }
  ],
  
  // Web server configuration
  webServer: {
    command: 'npm run start:test',
    port: 3000,
    reuseExistingServer: !process.env.CI
  }
});
```

## Test Organization Structure

### Directory Structure
```
tests/
├── unit/                     # Unit tests
│   ├── services/            # Service layer tests
│   ├── domain/              # Domain entity tests
│   ├── repositories/        # Repository tests
│   └── utils/               # Utility function tests
├── integration/             # Integration tests
│   ├── api/                 # API endpoint tests
│   ├── database/            # Database integration tests
│   └── external/            # External service tests
├── e2e/                     # End-to-end tests
│   ├── user-journeys/       # Complete user workflows
│   ├── admin/               # Admin functionality tests
│   └── mobile/              # Mobile-specific tests
├── performance/             # Performance tests
│   ├── load/                # Load testing
│   ├── stress/              # Stress testing
│   └── benchmarks/          # Performance benchmarks
├── security/                # Security tests
│   ├── auth/                # Authentication tests
│   ├── authorization/       # Authorization tests
│   └── vulnerabilities/     # Security vulnerability tests
├── fixtures/                # Test data fixtures
│   ├── users.json          # User test data
│   ├── projects.json       # Project test data
│   └── tasks.json          # Task test data
├── helpers/                 # Test helper utilities
│   ├── database.js         # Database test helpers
│   ├── auth.js             # Authentication helpers
│   └── factories.js        # Test data factories
└── setup/                   # Test setup and configuration
    ├── jest.setup.js       # Jest global setup
    ├── integration.setup.js # Integration test setup
    ├── e2e-global-setup.js # E2E global setup
    └── global-teardown.js  # Global cleanup
```

## Test Data Management

### Test Data Factories
```javascript
// tests/helpers/factories.js
const { faker } = require('@faker-js/faker');

/**
 * Factory for creating test user data
 */
class UserFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      email: faker.internet.email(),
      firstName: faker.name.firstName(),
      lastName: faker.name.lastName(),
      password: 'password123',
      role: 'user',
      plan: 'free',
      status: 'active',
      emailVerified: true,
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  static createAdmin(overrides = {}) {
    return this.create({
      role: 'admin',
      plan: 'enterprise',
      ...overrides
    });
  }

  static createPremium(overrides = {}) {
    return this.create({
      plan: 'pro',
      ...overrides
    });
  }
}

/**
 * Factory for creating test project data
 */
class ProjectFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      userId: faker.datatype.uuid(),
      name: faker.company.name(),
      description: faker.lorem.paragraph(),
      status: 'active',
      settings: {
        notifications: true,
        autoArchive: false,
        taskAutoAssignment: false
      },
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  static createForUser(userId, overrides = {}) {
    return this.create({
      userId,
      ...overrides
    });
  }
}

/**
 * Factory for creating test task data
 */
class TaskFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      projectId: faker.datatype.uuid(),
      title: faker.lorem.sentence(),
      description: faker.lorem.paragraph(),
      status: 'pending',
      priority: 'medium',
      assignedTo: null,
      dueDate: faker.date.future(),
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  static createForProject(projectId, overrides = {}) {
    return this.create({
      projectId,
      ...overrides
    });
  }
}

module.exports = {
  UserFactory,
  ProjectFactory,
  TaskFactory
};
```

### Database Test Helpers
```javascript
// tests/helpers/database.js
const prisma = require('../../src/lib/prisma');
const { UserFactory, ProjectFactory, TaskFactory } = require('./factories');

/**
 * Database test helper utilities
 */
class DatabaseTestHelper {
  /**
   * Sets up test database with clean state
   */
  static async setup() {
    // Enable test mode
    process.env.NODE_ENV = 'test';
    
    // Run migrations
    await this.runMigrations();
    
    // Seed initial test data
    await this.seedTestData();
  }

  /**
   * Cleans up test database
   */
  static async cleanup() {
    // Clean up in reverse order of dependencies
    await prisma.task.deleteMany();
    await prisma.project.deleteMany();
    await prisma.user.deleteMany();
    
    // Disconnect from database
    await prisma.$disconnect();
  }

  /**
   * Resets database to clean state
   */
  static async reset() {
    await this.cleanup();
    await this.seedTestData();
  }

  /**
   * Runs database migrations
   */
  static async runMigrations() {
    // Run Prisma migrations
    const { execSync } = require('child_process');
    execSync('npx prisma migrate deploy', { 
      env: { ...process.env, DATABASE_URL: process.env.TEST_DATABASE_URL }
    });
  }

  /**
   * Seeds test database with initial data
   */
  static async seedTestData() {
    // Create test users
    const testUsers = UserFactory.createMany(5);
    await prisma.user.createMany({ data: testUsers });

    // Create test projects
    const testProjects = ProjectFactory.createMany(10, {
      userId: testUsers[0].id
    });
    await prisma.project.createMany({ data: testProjects });

    // Create test tasks
    const testTasks = TaskFactory.createMany(20, {
      projectId: testProjects[0].id
    });
    await prisma.task.createMany({ data: testTasks });
  }

  /**
   * Creates a test user and returns it
   */
  static async createTestUser(overrides = {}) {
    const userData = UserFactory.create(overrides);
    return await prisma.user.create({ data: userData });
  }

  /**
   * Creates a test project for a user
   */
  static async createTestProject(userId, overrides = {}) {
    const projectData = ProjectFactory.createForUser(userId, overrides);
    return await prisma.project.create({ data: projectData });
  }

  /**
   * Creates a test task for a project
   */
  static async createTestTask(projectId, overrides = {}) {
    const taskData = TaskFactory.createForProject(projectId, overrides);
    return await prisma.task.create({ data: taskData });
  }

  /**
   * Truncates all tables
   */
  static async truncateAll() {
    const tablenames = await prisma.$queryRaw`
      SELECT tablename FROM pg_tables WHERE schemaname='public'
    `;
    
    for (const { tablename } of tablenames) {
      if (tablename !== '_prisma_migrations') {
        await prisma.$executeRawUnsafe(`TRUNCATE TABLE "public"."${tablename}" CASCADE;`);
      }
    }
  }
}

module.exports = DatabaseTestHelper;
```

## Test Execution Strategies

### Parallel Test Execution
```javascript
// tests/setup/parallel-execution.js
const os = require('os');

/**
 * Configures parallel test execution based on environment
 */
class ParallelTestExecutor {
  static getWorkerCount() {
    const cpuCount = os.cpus().length;
    const isCI = process.env.CI === 'true';
    
    if (isCI) {
      // Conservative approach for CI environments
      return Math.min(2, cpuCount);
    }
    
    // Use more workers locally for faster execution
    return Math.max(1, Math.floor(cpuCount * 0.75));
  }

  static getTestTimeout() {
    const isCI = process.env.CI === 'true';
    return isCI ? 30000 : 10000; // Longer timeout in CI
  }

  static shouldRunInBand() {
    // Run tests serially for integration tests to avoid database conflicts
    return process.env.TEST_TYPE === 'integration';
  }
}

module.exports = ParallelTestExecutor;
```

### Test Isolation Strategy
```javascript
// tests/setup/isolation.js
/**
 * Ensures test isolation and prevents test interference
 */
class TestIsolationManager {
  constructor() {
    this.testDatabases = new Map();
    this.testPorts = new Set();
  }

  /**
   * Creates isolated test environment for each test suite
   */
  async createIsolatedEnvironment(testSuite) {
    const environment = {
      databaseUrl: await this.createIsolatedDatabase(testSuite),
      port: await this.allocatePort(),
      tempDir: await this.createTempDirectory(testSuite)
    };

    return environment;
  }

  /**
   * Creates isolated database for test suite
   */
  async createIsolatedDatabase(testSuite) {
    const dbName = `test_${testSuite}_${Date.now()}`;
    const dbUrl = `postgresql://test:test@localhost:5432/${dbName}`;
    
    // Create database
    await this.createDatabase(dbName);
    
    // Store for cleanup
    this.testDatabases.set(testSuite, dbName);
    
    return dbUrl;
  }

  /**
   * Allocates available port for test server
   */
  async allocatePort() {
    const startPort = 3001;
    const maxPort = 3100;
    
    for (let port = startPort; port <= maxPort; port++) {
      if (!this.testPorts.has(port) && await this.isPortAvailable(port)) {
        this.testPorts.add(port);
        return port;
      }
    }
    
    throw new Error('No available ports for test server');
  }

  /**
   * Cleans up isolated environment
   */
  async cleanupEnvironment(testSuite) {
    // Drop test database
    const dbName = this.testDatabases.get(testSuite);
    if (dbName) {
      await this.dropDatabase(dbName);
      this.testDatabases.delete(testSuite);
    }

    // Release port
    // Port cleanup handled automatically when process ends
  }

  /**
   * Checks if port is available
   */
  async isPortAvailable(port) {
    return new Promise((resolve) => {
      const server = require('net').createServer();
      server.listen(port, () => {
        server.close(() => resolve(true));
      });
      server.on('error', () => resolve(false));
    });
  }

  /**
   * Creates database
   */
  async createDatabase(dbName) {
    const { Client } = require('pg');
    const client = new Client({
      host: 'localhost',
      port: 5432,
      user: 'test',
      password: 'test',
      database: 'postgres'
    });

    await client.connect();
    await client.query(`CREATE DATABASE "${dbName}"`);
    await client.end();
  }

  /**
   * Drops database
   */
  async dropDatabase(dbName) {
    const { Client } = require('pg');
    const client = new Client({
      host: 'localhost',
      port: 5432,
      user: 'test',
      password: 'test',
      database: 'postgres'
    });

    await client.connect();
    await client.query(`DROP DATABASE IF EXISTS "${dbName}"`);
    await client.end();
  }
}

module.exports = TestIsolationManager;
```

## Mock and Stub Management

### Service Mocking Strategy
```javascript
// tests/helpers/mocks.js
/**
 * Centralized mock management for consistent test behavior
 */
class MockManager {
  constructor() {
    this.mocks = new Map();
    this.originalImplementations = new Map();
  }

  /**
   * Creates mock for external service
   */
  mockExternalService(serviceName, mockImplementation) {
    const originalService = require(`../../src/services/${serviceName}`);
    
    // Store original implementation
    this.originalImplementations.set(serviceName, originalService);
    
    // Create mock
    const mockService = jest.createMockFromModule(`../../src/services/${serviceName}`);
    Object.assign(mockService, mockImplementation);
    
    // Store mock
    this.mocks.set(serviceName, mockService);
    
    return mockService;
  }

  /**
   * Creates database mock
   */
  mockDatabase() {
    const mockPrisma = {
      user: {
        create: jest.fn(),
        findUnique: jest.fn(),
        findMany: jest.fn(),
        update: jest.fn(),
        delete: jest.fn(),
        count: jest.fn()
      },
      project: {
        create: jest.fn(),
        findUnique: jest.fn(),
        findMany: jest.fn(),
        update: jest.fn(),
        delete: jest.fn(),
        count: jest.fn()
      },
      task: {
        create: jest.fn(),
        findUnique: jest.fn(),
        findMany: jest.fn(),
        update: jest.fn(),
        delete: jest.fn(),
        count: jest.fn()
      },
      $transaction: jest.fn(),
      $disconnect: jest.fn()
    };

    this.mocks.set('prisma', mockPrisma);
    return mockPrisma;
  }

  /**
   * Creates HTTP client mock
   */
  mockHttpClient() {
    const mockAxios = {
      get: jest.fn(),
      post: jest.fn(),
      put: jest.fn(),
      delete: jest.fn(),
      patch: jest.fn(),
      create: jest.fn(() => mockAxios),
      defaults: {
        headers: {}
      },
      interceptors: {
        request: { use: jest.fn() },
        response: { use: jest.fn() }
      }
    };

    this.mocks.set('axios', mockAxios);
    return mockAxios;
  }

  /**
   * Restores all mocks to original implementations
   */
  restoreAll() {
    for (const [serviceName, originalImplementation] of this.originalImplementations) {
      jest.doMock(`../../src/services/${serviceName}`, () => originalImplementation);
    }
    
    this.mocks.clear();
    this.originalImplementations.clear();
    jest.restoreAllMocks();
  }

  /**
   * Gets mock by name
   */
  getMock(name) {
    return this.mocks.get(name);
  }
}

// Global mock manager instance
const mockManager = new MockManager();

module.exports = mockManager;
```

## Test Reporting and Analytics

### Custom Test Reporter
```javascript
// tests/reporters/custom-reporter.js
/**
 * Custom test reporter for enhanced test analytics
 */
class CustomTestReporter {
  constructor(globalConfig, options) {
    this.globalConfig = globalConfig;
    this.options = options;
    this.testResults = [];
    this.startTime = Date.now();
  }

  onRunStart(results, options) {
    console.log('🚀 Starting DafnckMachine v3.1 Test Suite');
    console.log(`Running ${results.numTotalTestSuites} test suites`);
  }

  onTestSuiteStart(test) {
    console.log(`📝 Starting test suite: ${test.path}`);
  }

  onTestSuiteResult(test, testResult) {
    const { testResults, perfStats } = testResult;
    const passed = testResults.filter(t => t.status === 'passed').length;
    const failed = testResults.filter(t => t.status === 'failed').length;
    const skipped = testResults.filter(t => t.status === 'skipped').length;

    console.log(`✅ ${passed} passed, ❌ ${failed} failed, ⏭️ ${skipped} skipped`);
    console.log(`⏱️ Duration: ${perfStats.runtime}ms`);

    // Store results for analytics
    this.testResults.push({
      suiteName: test.path,
      passed,
      failed,
      skipped,
      duration: perfStats.runtime,
      timestamp: new Date()
    });
  }

  onRunComplete(contexts, results) {
    const duration = Date.now() - this.startTime;
    const { numPassedTests, numFailedTests, numTotalTests } = results;

    console.log('\n📊 Test Suite Summary');
    console.log('=====================');
    console.log(`Total Tests: ${numTotalTests}`);
    console.log(`Passed: ${numPassedTests}`);
    console.log(`Failed: ${numFailedTests}`);
    console.log(`Success Rate: ${((numPassedTests / numTotalTests) * 100).toFixed(2)}%`);
    console.log(`Total Duration: ${duration}ms`);

    // Generate detailed report
    this.generateDetailedReport(results);
  }

  generateDetailedReport(results) {
    const report = {
      summary: {
        totalTests: results.numTotalTests,
        passed: results.numPassedTests,
        failed: results.numFailedTests,
        successRate: (results.numPassedTests / results.numTotalTests) * 100,
        duration: Date.now() - this.startTime
      },
      suites: this.testResults,
      timestamp: new Date(),
      environment: {
        nodeVersion: process.version,
        platform: process.platform,
        ci: process.env.CI === 'true'
      }
    };

    // Write report to file
    const fs = require('fs');
    const path = require('path');
    const reportPath = path.join(process.cwd(), 'test-results', 'detailed-report.json');
    
    // Ensure directory exists
    fs.mkdirSync(path.dirname(reportPath), { recursive: true });
    
    fs.writeFileSync(reportPath, JSON.stringify(report, null, 2));
    console.log(`📄 Detailed report saved to: ${reportPath}`);
  }
}

module.exports = CustomTestReporter;
```

## Performance Testing Integration

### Load Testing Configuration
```javascript
// tests/performance/load-test.config.js
module.exports = {
  scenarios: {
    // API load testing
    api_load: {
      executor: 'ramping-vus',
      startVUs: 1,
      stages: [
        { duration: '30s', target: 10 },
        { duration: '1m', target: 50 },
        { duration: '2m', target: 100 },
        { duration: '30s', target: 0 }
      ],
      gracefulRampDown: '30s'
    },
    
    // Stress testing
    stress_test: {
      executor: 'ramping-vus',
      startVUs: 1,
      stages: [
        { duration: '1m', target: 100 },
        { duration: '2m', target: 200 },
        { duration: '1m', target: 300 },
        { duration: '2m', target: 300 },
        { duration: '1m', target: 0 }
      ]
    },
    
    // Spike testing
    spike_test: {
      executor: 'ramping-vus',
      startVUs: 1,
      stages: [
        { duration: '30s', target: 50 },
        { duration: '10s', target: 500 },
        { duration: '30s', target: 50 },
        { duration: '30s', target: 0 }
      ]
    }
  },
  
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.1'],    // Error rate under 10%
    http_reqs: ['rate>100']           // Request rate over 100 RPS
  }
};
```

This comprehensive test framework architecture provides a robust foundation for automated testing in DafnckMachine v3.1, ensuring high quality, reliability, and maintainability.

---

**Last Updated**: 2025-01-27  
**Version**: 3.1.0  
**Related Documents**: 
- [Backend_Testing_Strategy.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Backend_Testing_Strategy.md)
- [Unit_Testing_Implementation.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Unit_Testing_Implementation.md)
- [E2E_Testing_Implementation.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/E2E_Testing_Implementation.md) 