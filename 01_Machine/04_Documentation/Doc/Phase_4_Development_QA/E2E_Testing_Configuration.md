# E2E Testing Configuration - DafnckMachine v3.1

## Overview
Comprehensive End-to-End (E2E) testing configuration for the DafnckMachine v3.1 project using Playwright, covering test setup, organization, execution, and CI/CD integration.

## E2E Testing Strategy

### Testing Philosophy
- **User-Centric**: Tests simulate real user interactions and workflows
- **Critical Path Coverage**: Focus on essential user journeys and business flows
- **Cross-Browser Compatibility**: Ensure functionality across different browsers
- **Performance Validation**: Monitor page load times and user experience metrics
- **Visual Regression**: Detect unintended UI changes

### Test Pyramid Integration
- **E2E Tests**: 10% of total test suite
- **Focus Areas**: Critical user workflows, integration points, browser-specific features
- **Execution**: Automated in CI/CD pipeline and on-demand for releases

## Playwright Configuration

### Installation and Setup

```bash
# Install Playwright
npm install -D @playwright/test

# Install browsers
npx playwright install

# Install system dependencies (Linux)
npx playwright install-deps
```

### Playwright Configuration File

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Test directory
  testDir: './tests/e2e',
  
  // Run tests in files in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only in the source code
  forbidOnly: !!process.env.CI,
  
  // Retry on CI only
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter to use
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
    process.env.CI ? ['github'] : ['list']
  ],
  
  // Shared settings for all the projects below
  use: {
    // Base URL to use in actions like `await page.goto('/')`
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    
    // Collect trace when retrying the failed test
    trace: 'on-first-retry',
    
    // Record video on failure
    video: 'retain-on-failure',
    
    // Take screenshot on failure
    screenshot: 'only-on-failure',
    
    // Global timeout for each action
    actionTimeout: 10000,
    
    // Global timeout for navigation
    navigationTimeout: 30000,
  },

  // Configure projects for major browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Run your local dev server before starting the tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

### Environment Configuration

```typescript
// tests/e2e/config/test-config.ts
export const testConfig = {
  // Base URLs for different environments
  baseUrls: {
    development: 'http://localhost:3000',
    staging: 'https://staging.dafnckmachine.com',
    production: 'https://dafnckmachine.com'
  },
  
  // API endpoints
  apiUrls: {
    development: 'http://localhost:4000/api',
    staging: 'https://api-staging.dafnckmachine.com',
    production: 'https://api.dafnckmachine.com'
  },
  
  // Test user credentials
  testUsers: {
    admin: {
      email: process.env.TEST_ADMIN_EMAIL || 'admin@test.com',
      password: process.env.TEST_ADMIN_PASSWORD || 'testpassword123'
    },
    user: {
      email: process.env.TEST_USER_EMAIL || 'user@test.com',
      password: process.env.TEST_USER_PASSWORD || 'testpassword123'
    },
    viewer: {
      email: process.env.TEST_VIEWER_EMAIL || 'viewer@test.com',
      password: process.env.TEST_VIEWER_PASSWORD || 'testpassword123'
    }
  },
  
  // Timeouts
  timeouts: {
    short: 5000,
    medium: 15000,
    long: 30000,
    navigation: 30000
  },
  
  // Feature flags for testing
  features: {
    enableAnalytics: false,
    enableNotifications: true,
    enableRealTimeUpdates: true
  }
};
```

## Test Organization Structure

### Directory Structure

```
tests/e2e/
├── config/
│   ├── test-config.ts          # Test configuration
│   └── global-setup.ts         # Global test setup
├── fixtures/
│   ├── auth.fixture.ts         # Authentication fixtures
│   ├── data.fixture.ts         # Test data fixtures
│   └── page.fixture.ts         # Page object fixtures
├── pages/
│   ├── base.page.ts            # Base page object
│   ├── login.page.ts           # Login page object
│   ├── dashboard.page.ts       # Dashboard page object
│   ├── projects.page.ts        # Projects page object
│   └── users.page.ts           # Users page object
├── tests/
│   ├── auth/
│   │   ├── login.spec.ts       # Login functionality tests
│   │   ├── registration.spec.ts # Registration tests
│   │   └── password-reset.spec.ts # Password reset tests
│   ├── projects/
│   │   ├── project-creation.spec.ts # Project creation tests
│   │   ├── project-management.spec.ts # Project management tests
│   │   └── project-collaboration.spec.ts # Collaboration tests
│   ├── users/
│   │   ├── user-management.spec.ts # User management tests
│   │   └── user-profile.spec.ts # User profile tests
│   └── workflows/
│       ├── complete-project-workflow.spec.ts # End-to-end workflows
│       └── user-onboarding.spec.ts # User onboarding flow
├── utils/
│   ├── test-helpers.ts         # Test utility functions
│   ├── data-generators.ts      # Test data generators
│   └── assertions.ts           # Custom assertions
└── reports/                    # Test reports and artifacts
```

## Page Object Model Implementation

### Base Page Object

```typescript
// tests/e2e/pages/base.page.ts
import { Page, Locator, expect } from '@playwright/test';
import { testConfig } from '../config/test-config';

export abstract class BasePage {
  protected page: Page;
  protected baseUrl: string;

  constructor(page: Page) {
    this.page = page;
    this.baseUrl = testConfig.baseUrls[process.env.NODE_ENV || 'development'];
  }

  // Navigation methods
  async goto(path: string = ''): Promise<void> {
    await this.page.goto(`${this.baseUrl}${path}`);
  }

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  // Common element interactions
  async clickElement(selector: string): Promise<void> {
    await this.page.click(selector);
  }

  async fillInput(selector: string, value: string): Promise<void> {
    await this.page.fill(selector, value);
  }

  async selectOption(selector: string, value: string): Promise<void> {
    await this.page.selectOption(selector, value);
  }

  // Wait methods
  async waitForElement(selector: string, timeout?: number): Promise<Locator> {
    return this.page.waitForSelector(selector, { timeout });
  }

  async waitForText(text: string, timeout?: number): Promise<void> {
    await this.page.waitForFunction(
      (searchText) => document.body.innerText.includes(searchText),
      text,
      { timeout }
    );
  }

  // Assertion helpers
  async expectElementVisible(selector: string): Promise<void> {
    await expect(this.page.locator(selector)).toBeVisible();
  }

  async expectElementHidden(selector: string): Promise<void> {
    await expect(this.page.locator(selector)).toBeHidden();
  }

  async expectTextContent(selector: string, text: string): Promise<void> {
    await expect(this.page.locator(selector)).toContainText(text);
  }

  // Screenshot and debugging
  async takeScreenshot(name: string): Promise<void> {
    await this.page.screenshot({ 
      path: `test-results/screenshots/${name}.png`,
      fullPage: true 
    });
  }
}
```

### Specific Page Objects

```typescript
// tests/e2e/pages/login.page.ts
import { Page } from '@playwright/test';
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  // Selectors
  private readonly emailInput = '[data-testid="email-input"]';
  private readonly passwordInput = '[data-testid="password-input"]';
  private readonly loginButton = '[data-testid="login-button"]';
  private readonly errorMessage = '[data-testid="error-message"]';
  private readonly forgotPasswordLink = '[data-testid="forgot-password-link"]';

  constructor(page: Page) {
    super(page);
  }

  // Page actions
  async navigateToLogin(): Promise<void> {
    await this.goto('/login');
    await this.waitForPageLoad();
  }

  async enterCredentials(email: string, password: string): Promise<void> {
    await this.fillInput(this.emailInput, email);
    await this.fillInput(this.passwordInput, password);
  }

  async clickLogin(): Promise<void> {
    await this.clickElement(this.loginButton);
  }

  async login(email: string, password: string): Promise<void> {
    await this.navigateToLogin();
    await this.enterCredentials(email, password);
    await this.clickLogin();
  }

  async clickForgotPassword(): Promise<void> {
    await this.clickElement(this.forgotPasswordLink);
  }

  // Assertions
  async expectLoginSuccess(): Promise<void> {
    await this.page.waitForURL('**/dashboard');
  }

  async expectLoginError(message?: string): Promise<void> {
    await this.expectElementVisible(this.errorMessage);
    if (message) {
      await this.expectTextContent(this.errorMessage, message);
    }
  }

  async expectOnLoginPage(): Promise<void> {
    await this.expectElementVisible(this.loginButton);
    await this.expectElementVisible(this.emailInput);
    await this.expectElementVisible(this.passwordInput);
  }
}
```

```typescript
// tests/e2e/pages/dashboard.page.ts
import { Page } from '@playwright/test';
import { BasePage } from './base.page';

export class DashboardPage extends BasePage {
  // Selectors
  private readonly welcomeMessage = '[data-testid="welcome-message"]';
  private readonly projectsSection = '[data-testid="projects-section"]';
  private readonly createProjectButton = '[data-testid="create-project-button"]';
  private readonly userMenu = '[data-testid="user-menu"]';
  private readonly logoutButton = '[data-testid="logout-button"]';
  private readonly notificationBell = '[data-testid="notification-bell"]';

  constructor(page: Page) {
    super(page);
  }

  // Navigation
  async navigateToDashboard(): Promise<void> {
    await this.goto('/dashboard');
    await this.waitForPageLoad();
  }

  // Page actions
  async createNewProject(): Promise<void> {
    await this.clickElement(this.createProjectButton);
  }

  async openUserMenu(): Promise<void> {
    await this.clickElement(this.userMenu);
  }

  async logout(): Promise<void> {
    await this.openUserMenu();
    await this.clickElement(this.logoutButton);
  }

  async checkNotifications(): Promise<void> {
    await this.clickElement(this.notificationBell);
  }

  // Assertions
  async expectDashboardLoaded(): Promise<void> {
    await this.expectElementVisible(this.welcomeMessage);
    await this.expectElementVisible(this.projectsSection);
  }

  async expectWelcomeMessage(userName: string): Promise<void> {
    await this.expectTextContent(this.welcomeMessage, `Welcome, ${userName}`);
  }
}
```

## Test Implementation Examples

### Authentication Tests

```typescript
// tests/e2e/tests/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/login.page';
import { DashboardPage } from '../../pages/dashboard.page';
import { testConfig } from '../../config/test-config';

test.describe('User Authentication', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
  });

  test('should login with valid credentials', async ({ page }) => {
    // Arrange
    const { email, password } = testConfig.testUsers.user;

    // Act
    await loginPage.login(email, password);

    // Assert
    await loginPage.expectLoginSuccess();
    await dashboardPage.expectDashboardLoaded();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    // Arrange
    const invalidCredentials = {
      email: 'invalid@example.com',
      password: 'wrongpassword'
    };

    // Act
    await loginPage.login(invalidCredentials.email, invalidCredentials.password);

    // Assert
    await loginPage.expectLoginError('Invalid email or password');
  });

  test('should redirect to forgot password page', async ({ page }) => {
    // Arrange
    await loginPage.navigateToLogin();

    // Act
    await loginPage.clickForgotPassword();

    // Assert
    await page.waitForURL('**/forgot-password');
  });

  test('should maintain session after page refresh', async ({ page }) => {
    // Arrange
    const { email, password } = testConfig.testUsers.user;
    await loginPage.login(email, password);
    await dashboardPage.expectDashboardLoaded();

    // Act
    await page.reload();

    // Assert
    await dashboardPage.expectDashboardLoaded();
  });
});
```

### Project Management Tests

```typescript
// tests/e2e/tests/projects/project-creation.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/login.page';
import { DashboardPage } from '../../pages/dashboard.page';
import { ProjectsPage } from '../../pages/projects.page';
import { testConfig } from '../../config/test-config';
import { generateTestData } from '../../utils/data-generators';

test.describe('Project Creation', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;
  let projectsPage: ProjectsPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    projectsPage = new ProjectsPage(page);

    // Login before each test
    const { email, password } = testConfig.testUsers.user;
    await loginPage.login(email, password);
    await dashboardPage.expectDashboardLoaded();
  });

  test('should create a new project successfully', async ({ page }) => {
    // Arrange
    const projectData = generateTestData.project();

    // Act
    await dashboardPage.createNewProject();
    await projectsPage.fillProjectForm(projectData);
    await projectsPage.submitProject();

    // Assert
    await projectsPage.expectProjectCreated(projectData.name);
    await projectsPage.expectProjectInList(projectData.name);
  });

  test('should validate required fields', async ({ page }) => {
    // Act
    await dashboardPage.createNewProject();
    await projectsPage.submitProject();

    // Assert
    await projectsPage.expectValidationError('Project name is required');
  });

  test('should create project with team members', async ({ page }) => {
    // Arrange
    const projectData = generateTestData.project();
    const teamMembers = ['user1@example.com', 'user2@example.com'];

    // Act
    await dashboardPage.createNewProject();
    await projectsPage.fillProjectForm(projectData);
    await projectsPage.addTeamMembers(teamMembers);
    await projectsPage.submitProject();

    // Assert
    await projectsPage.expectProjectCreated(projectData.name);
    await projectsPage.expectTeamMembersAdded(teamMembers);
  });
});
```

### Complete Workflow Tests

```typescript
// tests/e2e/tests/workflows/complete-project-workflow.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/login.page';
import { DashboardPage } from '../../pages/dashboard.page';
import { ProjectsPage } from '../../pages/projects.page';
import { TasksPage } from '../../pages/tasks.page';
import { testConfig } from '../../config/test-config';
import { generateTestData } from '../../utils/data-generators';

test.describe('Complete Project Workflow', () => {
  test('should complete full project lifecycle', async ({ page }) => {
    // Initialize page objects
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);
    const projectsPage = new ProjectsPage(page);
    const tasksPage = new TasksPage(page);

    // Test data
    const { email, password } = testConfig.testUsers.user;
    const projectData = generateTestData.project();
    const taskData = generateTestData.task();

    // Step 1: Login
    await loginPage.login(email, password);
    await dashboardPage.expectDashboardLoaded();

    // Step 2: Create Project
    await dashboardPage.createNewProject();
    await projectsPage.fillProjectForm(projectData);
    await projectsPage.submitProject();
    await projectsPage.expectProjectCreated(projectData.name);

    // Step 3: Navigate to Project
    await projectsPage.openProject(projectData.name);
    await projectsPage.expectProjectDetailsLoaded();

    // Step 4: Create Task
    await projectsPage.navigateToTasks();
    await tasksPage.createNewTask();
    await tasksPage.fillTaskForm(taskData);
    await tasksPage.submitTask();
    await tasksPage.expectTaskCreated(taskData.title);

    // Step 5: Update Task Status
    await tasksPage.updateTaskStatus(taskData.title, 'In Progress');
    await tasksPage.expectTaskStatus(taskData.title, 'In Progress');

    // Step 6: Complete Task
    await tasksPage.updateTaskStatus(taskData.title, 'Completed');
    await tasksPage.expectTaskStatus(taskData.title, 'Completed');

    // Step 7: Verify Project Progress
    await projectsPage.navigateToOverview();
    await projectsPage.expectProjectProgress(100); // 1 task completed out of 1

    // Step 8: Archive Project
    await projectsPage.archiveProject();
    await projectsPage.expectProjectArchived();
  });
});
```

## Test Utilities and Helpers

### Data Generators

```typescript
// tests/e2e/utils/data-generators.ts
import { faker } from '@faker-js/faker';

export const generateTestData = {
  user: () => ({
    firstName: faker.person.firstName(),
    lastName: faker.person.lastName(),
    email: faker.internet.email(),
    password: 'TestPassword123!',
    role: 'user'
  }),

  project: () => ({
    name: `Test Project ${faker.string.alphanumeric(6)}`,
    description: faker.lorem.paragraph(),
    status: 'active',
    priority: faker.helpers.arrayElement(['low', 'medium', 'high']),
    dueDate: faker.date.future().toISOString().split('T')[0]
  }),

  task: () => ({
    title: `Test Task ${faker.string.alphanumeric(6)}`,
    description: faker.lorem.sentence(),
    priority: faker.helpers.arrayElement(['low', 'medium', 'high']),
    estimatedHours: faker.number.int({ min: 1, max: 40 }),
    dueDate: faker.date.future().toISOString().split('T')[0]
  }),

  uniqueEmail: () => `test-${Date.now()}-${faker.string.alphanumeric(6)}@example.com`,
  
  uniqueProjectName: () => `Project-${Date.now()}-${faker.string.alphanumeric(6)}`
};
```

### Custom Assertions

```typescript
// tests/e2e/utils/assertions.ts
import { expect, Page } from '@playwright/test';

export class CustomAssertions {
  constructor(private page: Page) {}

  async expectPageTitle(expectedTitle: string): Promise<void> {
    await expect(this.page).toHaveTitle(expectedTitle);
  }

  async expectUrl(expectedUrl: string | RegExp): Promise<void> {
    await expect(this.page).toHaveURL(expectedUrl);
  }

  async expectElementCount(selector: string, count: number): Promise<void> {
    await expect(this.page.locator(selector)).toHaveCount(count);
  }

  async expectElementAttribute(
    selector: string, 
    attribute: string, 
    value: string
  ): Promise<void> {
    await expect(this.page.locator(selector)).toHaveAttribute(attribute, value);
  }

  async expectElementClass(selector: string, className: string): Promise<void> {
    await expect(this.page.locator(selector)).toHaveClass(new RegExp(className));
  }

  async expectApiResponse(
    apiCall: () => Promise<any>,
    expectedStatus: number,
    expectedData?: any
  ): Promise<void> {
    const response = await apiCall();
    expect(response.status()).toBe(expectedStatus);
    
    if (expectedData) {
      const responseData = await response.json();
      expect(responseData).toMatchObject(expectedData);
    }
  }

  async expectPerformanceMetric(
    metricName: string,
    threshold: number
  ): Promise<void> {
    const metrics = await this.page.evaluate(() => {
      return JSON.parse(JSON.stringify(performance.getEntriesByType('navigation')));
    });
    
    const metric = metrics[0][metricName];
    expect(metric).toBeLessThan(threshold);
  }
}
```

## CI/CD Integration

### GitHub Actions Workflow

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * *' # Run daily at 2 AM

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]
        
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright browsers
        run: npx playwright install --with-deps ${{ matrix.browser }}
        
      - name: Start application
        run: |
          npm run build
          npm run start &
          npx wait-on http://localhost:3000
          
      - name: Run E2E tests
        run: npx playwright test --project=${{ matrix.browser }}
        env:
          BASE_URL: http://localhost:3000
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          TEST_ADMIN_EMAIL: ${{ secrets.TEST_ADMIN_EMAIL }}
          TEST_ADMIN_PASSWORD: ${{ secrets.TEST_ADMIN_PASSWORD }}
          
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report-${{ matrix.browser }}
          path: playwright-report/
          retention-days: 30
          
      - name: Upload test videos
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: test-videos-${{ matrix.browser }}
          path: test-results/
          retention-days: 7
```

### Test Execution Scripts

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:chromium": "playwright test --project=chromium",
    "test:e2e:firefox": "playwright test --project=firefox",
    "test:e2e:webkit": "playwright test --project=webkit",
    "test:e2e:mobile": "playwright test --project='Mobile Chrome' --project='Mobile Safari'",
    "test:e2e:report": "playwright show-report",
    "test:e2e:install": "playwright install",
    "test:e2e:codegen": "playwright codegen localhost:3000"
  }
}
```

## Performance and Visual Testing

### Performance Testing Integration

```typescript
// tests/e2e/tests/performance/page-performance.spec.ts
import { test, expect } from '@playwright/test';
import { CustomAssertions } from '../../utils/assertions';

test.describe('Performance Tests', () => {
  test('should load dashboard within performance thresholds', async ({ page }) => {
    const assertions = new CustomAssertions(page);
    
    // Navigate to dashboard
    await page.goto('/dashboard');
    
    // Check performance metrics
    await assertions.expectPerformanceMetric('loadEventEnd', 3000); // 3 seconds
    await assertions.expectPerformanceMetric('domContentLoadedEventEnd', 2000); // 2 seconds
    
    // Check Core Web Vitals
    const vitals = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          resolve(entries.map(entry => ({
            name: entry.name,
            value: entry.value
          })));
        }).observe({ entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift'] });
      });
    });
    
    // Assert Core Web Vitals thresholds
    // LCP should be under 2.5 seconds
    // FID should be under 100ms
    // CLS should be under 0.1
  });
});
```

### Visual Regression Testing

```typescript
// tests/e2e/tests/visual/visual-regression.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression Tests', () => {
  test('should match dashboard screenshot', async ({ page }) => {
    await page.goto('/dashboard');
    await page.waitForLoadState('networkidle');
    
    // Take screenshot and compare with baseline
    await expect(page).toHaveScreenshot('dashboard.png');
  });

  test('should match login page screenshot', async ({ page }) => {
    await page.goto('/login');
    await page.waitForLoadState('networkidle');
    
    await expect(page).toHaveScreenshot('login-page.png');
  });

  test('should match project creation modal', async ({ page }) => {
    // Setup: Login and navigate to projects
    await page.goto('/dashboard');
    await page.click('[data-testid="create-project-button"]');
    await page.waitForSelector('[data-testid="project-modal"]');
    
    // Screenshot the modal
    await expect(page.locator('[data-testid="project-modal"]')).toHaveScreenshot('project-modal.png');
  });
});
```

## Best Practices and Guidelines

### Test Writing Best Practices

1. **Use Data Test IDs**: Always use `data-testid` attributes for reliable element selection
2. **Page Object Pattern**: Organize tests using page objects for maintainability
3. **Independent Tests**: Each test should be independent and able to run in isolation
4. **Descriptive Names**: Use clear, descriptive test names that explain the expected behavior
5. **Proper Assertions**: Use appropriate assertions that clearly indicate what is being tested

### Performance Optimization

```typescript
// Optimize test execution
test.describe.configure({ mode: 'parallel' });

// Use beforeAll for expensive setup
test.beforeAll(async ({ browser }) => {
  // Expensive setup that can be shared
});

// Use page.route for API mocking
test('should handle API errors gracefully', async ({ page }) => {
  await page.route('**/api/users', route => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Internal Server Error' })
    });
  });
  
  await page.goto('/users');
  await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
});
```

### Debugging and Troubleshooting

```typescript
// Debug helpers
test('debug test', async ({ page }) => {
  // Pause execution for debugging
  await page.pause();
  
  // Take screenshot for debugging
  await page.screenshot({ path: 'debug-screenshot.png' });
  
  // Log page content
  console.log(await page.content());
  
  // Log console messages
  page.on('console', msg => console.log('PAGE LOG:', msg.text()));
  
  // Log network requests
  page.on('request', request => console.log('REQUEST:', request.url()));
  page.on('response', response => console.log('RESPONSE:', response.url(), response.status()));
});
```

## Related Documentation

- [Test Strategy Framework](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Test_Strategy_Framework.md)
- [Unit Testing Guidelines](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Unit_Testing_Guidelines.md)
- [Integration Testing Plan](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Integration_Testing_Plan.md)
- [Performance Optimization Strategies](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Strategies.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: test-orchestrator-agent
- **Related Workflows**: 15_Automated_Testing.md 