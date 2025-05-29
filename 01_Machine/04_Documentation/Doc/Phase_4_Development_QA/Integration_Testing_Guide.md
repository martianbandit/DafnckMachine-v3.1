# Integration Testing Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Unit_Testing_Implementation.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Unit_Testing_Implementation.md), [API_Integration_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Integration_Guide.md)

## Overview

This guide provides comprehensive integration testing strategies for DafnckMachine v3.1, covering API integration testing with MSW, user flow testing, cross-browser testing with Playwright, end-to-end testing scenarios, performance testing, visual regression testing, and CI/CD automation.

## Table of Contents

1. [Integration Testing Strategy](#integration-testing-strategy)
2. [API Integration Testing](#api-integration-testing)
3. [User Flow Testing](#user-flow-testing)
4. [Cross-Browser Testing](#cross-browser-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Performance Testing](#performance-testing)
7. [Visual Regression Testing](#visual-regression-testing)
8. [CI/CD Integration](#cicd-integration)

## Integration Testing Strategy

### Testing Pyramid

```typescript
// Integration testing strategy
export const integrationTestingStrategy = {
  // Testing levels
  levels: {
    unit: '70% - Individual components and functions',
    integration: '20% - Component interactions and API integration',
    e2e: '10% - Complete user workflows'
  },
  
  // Integration test types
  types: {
    apiIntegration: 'Backend API communication',
    componentIntegration: 'Component interaction testing',
    userFlows: 'Multi-step user scenarios',
    crossBrowser: 'Browser compatibility testing',
    performance: 'Load and performance testing',
    visual: 'Visual regression testing'
  },
  
  // Test environments
  environments: {
    development: 'Local development testing',
    staging: 'Pre-production testing',
    production: 'Production monitoring'
  }
} as const;
```

### Test Configuration

```typescript
// Integration test configuration
export const integrationTestConfig = {
  // Test timeouts
  timeouts: {
    default: 30000,
    api: 10000,
    navigation: 5000,
    animation: 2000
  },
  
  // Test data
  testData: {
    users: 'test-users.json',
    products: 'test-products.json',
    orders: 'test-orders.json'
  },
  
  // Environment variables
  env: {
    apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3001',
    testUser: process.env.TEST_USER_EMAIL || 'test@example.com',
    testPassword: process.env.TEST_USER_PASSWORD || 'password123'
  }
} as const;
```

## API Integration Testing

### MSW Setup

```typescript
// src/test/mocks/handlers.ts
import { rest } from 'msw';
import { setupServer } from 'msw/node';

// API response types
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: Record<string, string[]>;
}

// Mock data
const mockUsers = [
  { id: '1', name: 'John Doe', email: 'john@example.com', role: 'admin' },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com', role: 'user' }
];

const mockPosts = [
  { id: '1', title: 'First Post', content: 'Content 1', authorId: '1' },
  { id: '2', title: 'Second Post', content: 'Content 2', authorId: '2' }
];

// API handlers
export const handlers = [
  // Authentication endpoints
  rest.post('/api/auth/login', (req, res, ctx) => {
    const { email, password } = req.body as any;
    
    if (email === 'test@example.com' && password === 'password123') {
      return res(
        ctx.json<ApiResponse<any>>({
          data: {
            user: mockUsers[0],
            token: 'mock-jwt-token',
            refreshToken: 'mock-refresh-token'
          },
          success: true
        })
      );
    }
    
    return res(
      ctx.status(401),
      ctx.json<ApiResponse<null>>({
        data: null,
        success: false,
        message: 'Invalid credentials'
      })
    );
  }),

  rest.post('/api/auth/logout', (req, res, ctx) => {
    return res(
      ctx.json<ApiResponse<null>>({
        data: null,
        success: true
      })
    );
  }),

  // User endpoints
  rest.get('/api/users', (req, res, ctx) => {
    const page = req.url.searchParams.get('page') || '1';
    const limit = req.url.searchParams.get('limit') || '10';
    
    return res(
      ctx.json<ApiResponse<any>>({
        data: {
          users: mockUsers,
          pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            total: mockUsers.length,
            totalPages: 1
          }
        },
        success: true
      })
    );
  }),

  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    const user = mockUsers.find(u => u.id === id);
    
    if (!user) {
      return res(
        ctx.status(404),
        ctx.json<ApiResponse<null>>({
          data: null,
          success: false,
          message: 'User not found'
        })
      );
    }
    
    return res(
      ctx.json<ApiResponse<any>>({
        data: user,
        success: true
      })
    );
  }),

  rest.post('/api/users', (req, res, ctx) => {
    const userData = req.body as any;
    const newUser = {
      id: String(mockUsers.length + 1),
      ...userData
    };
    
    mockUsers.push(newUser);
    
    return res(
      ctx.status(201),
      ctx.json<ApiResponse<any>>({
        data: newUser,
        success: true
      })
    );
  }),

  // Posts endpoints
  rest.get('/api/posts', (req, res, ctx) => {
    return res(
      ctx.json<ApiResponse<any>>({
        data: mockPosts,
        success: true
      })
    );
  }),

  // Error simulation
  rest.get('/api/error', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json<ApiResponse<null>>({
        data: null,
        success: false,
        message: 'Internal server error'
      })
    );
  })
];

// Setup MSW server
export const server = setupServer(...handlers);
```

### API Integration Tests

```typescript
// src/test/integration/api.test.tsx
import { render, screen, fireEvent, waitFor } from '@/test/utils';
import { server } from '@/test/mocks/handlers';
import { rest } from 'msw';
import { LoginForm } from '@/components/auth/LoginForm';
import { UsersList } from '@/components/users/UsersList';

describe('API Integration Tests', () => {
  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());

  describe('Authentication Flow', () => {
    test('successful login', async () => {
      const mockOnSuccess = jest.fn();
      
      render(<LoginForm onSuccess={mockOnSuccess} />);
      
      fireEvent.change(screen.getByLabelText(/email/i), {
        target: { value: 'test@example.com' }
      });
      fireEvent.change(screen.getByLabelText(/password/i), {
        target: { value: 'password123' }
      });
      
      fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
      
      await waitFor(() => {
        expect(mockOnSuccess).toHaveBeenCalledWith({
          user: expect.objectContaining({
            email: 'test@example.com'
          }),
          token: 'mock-jwt-token'
        });
      });
    });

    test('failed login with invalid credentials', async () => {
      render(<LoginForm onSuccess={jest.fn()} />);
      
      fireEvent.change(screen.getByLabelText(/email/i), {
        target: { value: 'wrong@example.com' }
      });
      fireEvent.change(screen.getByLabelText(/password/i), {
        target: { value: 'wrongpassword' }
      });
      
      fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
      
      await waitFor(() => {
        expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
      });
    });
  });

  describe('Users API Integration', () => {
    test('loads and displays users', async () => {
      render(<UsersList />);
      
      expect(screen.getByText(/loading/i)).toBeInTheDocument();
      
      await waitFor(() => {
        expect(screen.getByText('John Doe')).toBeInTheDocument();
        expect(screen.getByText('Jane Smith')).toBeInTheDocument();
      });
    });

    test('handles API errors gracefully', async () => {
      // Override handler to return error
      server.use(
        rest.get('/api/users', (req, res, ctx) => {
          return res(
            ctx.status(500),
            ctx.json({
              data: null,
              success: false,
              message: 'Server error'
            })
          );
        })
      );
      
      render(<UsersList />);
      
      await waitFor(() => {
        expect(screen.getByText(/error loading users/i)).toBeInTheDocument();
      });
    });

    test('retries failed requests', async () => {
      let callCount = 0;
      
      server.use(
        rest.get('/api/users', (req, res, ctx) => {
          callCount++;
          
          if (callCount < 3) {
            return res(ctx.status(500));
          }
          
          return res(
            ctx.json({
              data: { users: [], pagination: { page: 1, limit: 10, total: 0, totalPages: 0 } },
              success: true
            })
          );
        })
      );
      
      render(<UsersList />);
      
      await waitFor(() => {
        expect(screen.queryByText(/error/i)).not.toBeInTheDocument();
      }, { timeout: 10000 });
      
      expect(callCount).toBe(3);
    });
  });

  describe('Real-time Updates', () => {
    test('handles WebSocket messages', async () => {
      // Mock WebSocket
      const mockWebSocket = {
        send: jest.fn(),
        close: jest.fn(),
        addEventListener: jest.fn(),
        removeEventListener: jest.fn()
      };
      
      global.WebSocket = jest.fn(() => mockWebSocket) as any;
      
      render(<UsersList />);
      
      // Simulate WebSocket message
      const messageHandler = mockWebSocket.addEventListener.mock.calls
        .find(call => call[0] === 'message')?.[1];
      
      if (messageHandler) {
        messageHandler({
          data: JSON.stringify({
            type: 'user:updated',
            payload: { id: '1', name: 'John Updated' }
          })
        });
      }
      
      await waitFor(() => {
        expect(screen.getByText('John Updated')).toBeInTheDocument();
      });
    });
  });
});
```

## User Flow Testing

### Multi-Step User Scenarios

```typescript
// src/test/integration/userFlows.test.tsx
import { render, screen, fireEvent, waitFor } from '@/test/utils';
import { App } from '@/App';
import { server } from '@/test/mocks/handlers';

describe('User Flow Integration Tests', () => {
  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());

  test('complete user registration and login flow', async () => {
    render(<App />);
    
    // Navigate to registration
    fireEvent.click(screen.getByText(/sign up/i));
    
    await waitFor(() => {
      expect(screen.getByText(/create account/i)).toBeInTheDocument();
    });
    
    // Fill registration form
    fireEvent.change(screen.getByLabelText(/first name/i), {
      target: { value: 'John' }
    });
    fireEvent.change(screen.getByLabelText(/last name/i), {
      target: { value: 'Doe' }
    });
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'john@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /create account/i }));
    
    // Verify redirect to verification page
    await waitFor(() => {
      expect(screen.getByText(/verify your email/i)).toBeInTheDocument();
    });
    
    // Navigate to login
    fireEvent.click(screen.getByText(/sign in/i));
    
    // Login with new account
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'john@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    // Verify successful login and dashboard access
    await waitFor(() => {
      expect(screen.getByText(/dashboard/i)).toBeInTheDocument();
    });
  });

  test('user profile update flow', async () => {
    // Start with authenticated user
    render(<App />);
    
    // Navigate to profile
    fireEvent.click(screen.getByText(/profile/i));
    
    await waitFor(() => {
      expect(screen.getByText(/edit profile/i)).toBeInTheDocument();
    });
    
    // Update profile information
    fireEvent.change(screen.getByLabelText(/name/i), {
      target: { value: 'John Updated' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /save changes/i }));
    
    // Verify success message
    await waitFor(() => {
      expect(screen.getByText(/profile updated successfully/i)).toBeInTheDocument();
    });
    
    // Verify updated information is displayed
    expect(screen.getByDisplayValue('John Updated')).toBeInTheDocument();
  });

  test('shopping cart and checkout flow', async () => {
    render(<App />);
    
    // Browse products
    fireEvent.click(screen.getByText(/products/i));
    
    await waitFor(() => {
      expect(screen.getByText(/product catalog/i)).toBeInTheDocument();
    });
    
    // Add product to cart
    const addToCartButtons = screen.getAllByText(/add to cart/i);
    fireEvent.click(addToCartButtons[0]);
    
    // Verify cart update
    await waitFor(() => {
      expect(screen.getByText(/1/)).toBeInTheDocument(); // Cart count
    });
    
    // Go to cart
    fireEvent.click(screen.getByText(/cart/i));
    
    await waitFor(() => {
      expect(screen.getByText(/shopping cart/i)).toBeInTheDocument();
    });
    
    // Proceed to checkout
    fireEvent.click(screen.getByText(/checkout/i));
    
    // Fill checkout form
    fireEvent.change(screen.getByLabelText(/shipping address/i), {
      target: { value: '123 Main St' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /place order/i }));
    
    // Verify order confirmation
    await waitFor(() => {
      expect(screen.getByText(/order confirmed/i)).toBeInTheDocument();
    });
  });
});
```

## Cross-Browser Testing

### Playwright Setup

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './src/test/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },

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

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Cross-Browser Tests

```typescript
// src/test/e2e/crossBrowser.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Cross-Browser Compatibility', () => {
  test('login works across all browsers', async ({ page }) => {
    await page.goto('/auth/login');
    
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    
    await page.click('[data-testid="login-button"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('responsive navigation works on mobile', async ({ page }) => {
    await page.goto('/');
    
    // Test mobile menu
    await page.click('[data-testid="mobile-menu-button"]');
    await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    
    await page.click('[data-testid="mobile-menu-close"]');
    await expect(page.locator('[data-testid="mobile-menu"]')).not.toBeVisible();
  });

  test('form validation works consistently', async ({ page }) => {
    await page.goto('/contact');
    
    // Submit empty form
    await page.click('[data-testid="submit-button"]');
    
    // Check validation messages
    await expect(page.locator('[data-testid="name-error"]')).toContainText('Name is required');
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Email is required');
  });

  test('animations and transitions work smoothly', async ({ page }) => {
    await page.goto('/');
    
    // Test modal animation
    await page.click('[data-testid="open-modal-button"]');
    
    // Wait for animation to complete
    await page.waitForTimeout(500);
    
    await expect(page.locator('[data-testid="modal"]')).toBeVisible();
    
    // Test close animation
    await page.click('[data-testid="close-modal-button"]');
    await page.waitForTimeout(500);
    
    await expect(page.locator('[data-testid="modal"]')).not.toBeVisible();
  });
});
```

## End-to-End Testing

### E2E Test Scenarios

```typescript
// src/test/e2e/userJourneys.spec.ts
import { test, expect } from '@playwright/test';

test.describe('End-to-End User Journeys', () => {
  test.beforeEach(async ({ page }) => {
    // Setup test data
    await page.goto('/test-setup');
    await page.evaluate(() => {
      localStorage.setItem('test-mode', 'true');
    });
  });

  test('admin user management workflow', async ({ page }) => {
    // Login as admin
    await page.goto('/auth/login');
    await page.fill('[data-testid="email-input"]', 'admin@example.com');
    await page.fill('[data-testid="password-input"]', 'admin123');
    await page.click('[data-testid="login-button"]');
    
    // Navigate to users management
    await page.click('[data-testid="admin-menu"]');
    await page.click('[data-testid="users-link"]');
    
    await expect(page).toHaveURL('/admin/users');
    
    // Create new user
    await page.click('[data-testid="create-user-button"]');
    
    await page.fill('[data-testid="user-name-input"]', 'Test User');
    await page.fill('[data-testid="user-email-input"]', 'testuser@example.com');
    await page.selectOption('[data-testid="user-role-select"]', 'user');
    
    await page.click('[data-testid="save-user-button"]');
    
    // Verify user creation
    await expect(page.locator('[data-testid="success-message"]')).toContainText('User created successfully');
    await expect(page.locator('text=Test User')).toBeVisible();
    
    // Edit user
    await page.click('[data-testid="edit-user-button"]:has-text("Test User")');
    await page.fill('[data-testid="user-name-input"]', 'Updated Test User');
    await page.click('[data-testid="save-user-button"]');
    
    // Verify user update
    await expect(page.locator('text=Updated Test User')).toBeVisible();
    
    // Delete user
    await page.click('[data-testid="delete-user-button"]:has-text("Updated Test User")');
    await page.click('[data-testid="confirm-delete-button"]');
    
    // Verify user deletion
    await expect(page.locator('text=Updated Test User')).not.toBeVisible();
  });

  test('content creation and publishing workflow', async ({ page }) => {
    // Login as content creator
    await page.goto('/auth/login');
    await page.fill('[data-testid="email-input"]', 'creator@example.com');
    await page.fill('[data-testid="password-input"]', 'creator123');
    await page.click('[data-testid="login-button"]');
    
    // Navigate to content creation
    await page.click('[data-testid="create-content-button"]');
    
    // Fill content form
    await page.fill('[data-testid="title-input"]', 'Test Article');
    await page.fill('[data-testid="content-editor"]', 'This is test content for the article.');
    
    // Add tags
    await page.fill('[data-testid="tags-input"]', 'test, article');
    await page.press('[data-testid="tags-input"]', 'Enter');
    
    // Upload featured image
    await page.setInputFiles('[data-testid="image-upload"]', 'test-image.jpg');
    
    // Save as draft
    await page.click('[data-testid="save-draft-button"]');
    await expect(page.locator('[data-testid="draft-saved-message"]')).toBeVisible();
    
    // Preview content
    await page.click('[data-testid="preview-button"]');
    await expect(page.locator('text=Test Article')).toBeVisible();
    
    // Publish content
    await page.click('[data-testid="publish-button"]');
    await page.click('[data-testid="confirm-publish-button"]');
    
    // Verify publication
    await expect(page.locator('[data-testid="published-message"]')).toContainText('Article published successfully');
    
    // Check public view
    await page.goto('/articles/test-article');
    await expect(page.locator('h1:has-text("Test Article")')).toBeVisible();
  });

  test('e-commerce purchase flow', async ({ page }) => {
    // Browse products as guest
    await page.goto('/products');
    
    // Filter products
    await page.selectOption('[data-testid="category-filter"]', 'electronics');
    await page.fill('[data-testid="price-min"]', '100');
    await page.fill('[data-testid="price-max"]', '500');
    await page.click('[data-testid="apply-filters"]');
    
    // Select product
    await page.click('[data-testid="product-card"]:first-child');
    
    // Add to cart
    await page.selectOption('[data-testid="quantity-select"]', '2');
    await page.click('[data-testid="add-to-cart-button"]');
    
    // Verify cart update
    await expect(page.locator('[data-testid="cart-count"]')).toContainText('2');
    
    // Go to cart
    await page.click('[data-testid="cart-button"]');
    
    // Update quantity
    await page.fill('[data-testid="quantity-input"]', '3');
    await page.click('[data-testid="update-cart-button"]');
    
    // Proceed to checkout
    await page.click('[data-testid="checkout-button"]');
    
    // Guest checkout
    await page.click('[data-testid="guest-checkout-button"]');
    
    // Fill shipping information
    await page.fill('[data-testid="first-name"]', 'John');
    await page.fill('[data-testid="last-name"]', 'Doe');
    await page.fill('[data-testid="email"]', 'john@example.com');
    await page.fill('[data-testid="address"]', '123 Main St');
    await page.fill('[data-testid="city"]', 'Anytown');
    await page.fill('[data-testid="zip"]', '12345');
    
    // Select shipping method
    await page.click('[data-testid="shipping-standard"]');
    
    // Fill payment information
    await page.fill('[data-testid="card-number"]', '4111111111111111');
    await page.fill('[data-testid="expiry"]', '12/25');
    await page.fill('[data-testid="cvv"]', '123');
    
    // Place order
    await page.click('[data-testid="place-order-button"]');
    
    // Verify order confirmation
    await expect(page.locator('[data-testid="order-confirmation"]')).toBeVisible();
    await expect(page.locator('[data-testid="order-number"]')).toContainText(/ORD-\d+/);
  });
});
```

## Performance Testing

### Performance Test Setup

```typescript
// src/test/performance/performance.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('page load performance', async ({ page }) => {
    // Start performance monitoring
    await page.goto('/');
    
    // Measure Core Web Vitals
    const metrics = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          const vitals = {};
          
          entries.forEach((entry) => {
            if (entry.name === 'largest-contentful-paint') {
              vitals.lcp = entry.value;
            }
            if (entry.name === 'first-input-delay') {
              vitals.fid = entry.value;
            }
            if (entry.name === 'cumulative-layout-shift') {
              vitals.cls = entry.value;
            }
          });
          
          resolve(vitals);
        }).observe({ entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift'] });
      });
    });
    
    // Assert performance thresholds
    expect(metrics.lcp).toBeLessThan(2500); // LCP < 2.5s
    expect(metrics.fid).toBeLessThan(100);  // FID < 100ms
    expect(metrics.cls).toBeLessThan(0.1);  // CLS < 0.1
  });

  test('bundle size performance', async ({ page }) => {
    // Navigate to page and measure bundle sizes
    const response = await page.goto('/');
    
    // Get all network requests
    const requests = [];
    page.on('response', (response) => {
      if (response.url().includes('.js') || response.url().includes('.css')) {
        requests.push({
          url: response.url(),
          size: response.headers()['content-length']
        });
      }
    });
    
    await page.waitForLoadState('networkidle');
    
    // Calculate total bundle size
    const totalSize = requests.reduce((sum, req) => sum + parseInt(req.size || '0'), 0);
    
    // Assert bundle size threshold (e.g., < 1MB)
    expect(totalSize).toBeLessThan(1024 * 1024);
  });

  test('API response time performance', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Measure API response times
    const apiTimes = [];
    
    page.on('response', (response) => {
      if (response.url().includes('/api/')) {
        const timing = response.timing();
        apiTimes.push(timing.responseEnd - timing.requestStart);
      }
    });
    
    await page.waitForLoadState('networkidle');
    
    // Assert API response times
    const avgResponseTime = apiTimes.reduce((sum, time) => sum + time, 0) / apiTimes.length;
    expect(avgResponseTime).toBeLessThan(500); // < 500ms average
  });
});
```

## Visual Regression Testing

### Visual Testing Setup

```typescript
// src/test/visual/visual.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression Tests', () => {
  test('homepage visual consistency', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    // Take full page screenshot
    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
      threshold: 0.2
    });
  });

  test('login form visual consistency', async ({ page }) => {
    await page.goto('/auth/login');
    
    // Take component screenshot
    await expect(page.locator('[data-testid="login-form"]')).toHaveScreenshot('login-form.png');
  });

  test('responsive design visual tests', async ({ page }) => {
    await page.goto('/');
    
    // Test different viewport sizes
    const viewports = [
      { width: 320, height: 568 },  // Mobile
      { width: 768, height: 1024 }, // Tablet
      { width: 1920, height: 1080 } // Desktop
    ];
    
    for (const viewport of viewports) {
      await page.setViewportSize(viewport);
      await page.waitForTimeout(500); // Allow layout to settle
      
      await expect(page).toHaveScreenshot(`homepage-${viewport.width}x${viewport.height}.png`, {
        fullPage: true
      });
    }
  });

  test('dark mode visual consistency', async ({ page }) => {
    await page.goto('/');
    
    // Enable dark mode
    await page.click('[data-testid="theme-toggle"]');
    await page.waitForTimeout(500);
    
    await expect(page).toHaveScreenshot('homepage-dark.png', {
      fullPage: true
    });
  });

  test('component states visual tests', async ({ page }) => {
    await page.goto('/components');
    
    // Test button states
    await expect(page.locator('[data-testid="button-default"]')).toHaveScreenshot('button-default.png');
    
    await page.hover('[data-testid="button-hover"]');
    await expect(page.locator('[data-testid="button-hover"]')).toHaveScreenshot('button-hover.png');
    
    await page.click('[data-testid="button-active"]');
    await expect(page.locator('[data-testid="button-active"]')).toHaveScreenshot('button-active.png');
  });
});
```

## CI/CD Integration

### GitHub Actions Workflow

```yaml
# .github/workflows/integration-tests.yml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Start test server
        run: |
          npm run build
          npm run preview &
          npx wait-on http://localhost:4173
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: test-results
          path: test-results/
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  visual-regression:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run visual tests
        run: npm run test:visual
      
      - name: Upload visual diff artifacts
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: visual-diffs
          path: test-results/

  performance-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
      
      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli@0.12.x
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

### Test Scripts Configuration

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:integration": "jest --config jest.integration.config.js",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:visual": "playwright test --config playwright.visual.config.ts",
    "test:performance": "playwright test --config playwright.performance.config.ts",
    "test:all": "npm run test && npm run test:integration && npm run test:e2e"
  }
}
```

---

## Summary

This Integration Testing Guide provides comprehensive testing strategies for DafnckMachine v3.1, ensuring robust application quality through API integration testing, user flow validation, cross-browser compatibility, end-to-end scenarios, performance monitoring, visual regression testing, and automated CI/CD integration.

Key testing areas include:

1. **API Integration**: MSW-based API mocking and testing
2. **User Flows**: Multi-step scenario validation
3. **Cross-Browser**: Playwright-based compatibility testing
4. **E2E Testing**: Complete user journey validation
5. **Performance**: Core Web Vitals and load testing
6. **Visual Regression**: Screenshot-based UI consistency
7. **CI/CD Integration**: Automated testing pipelines

The guide ensures comprehensive test coverage while maintaining fast feedback loops and reliable quality assurance processes. 