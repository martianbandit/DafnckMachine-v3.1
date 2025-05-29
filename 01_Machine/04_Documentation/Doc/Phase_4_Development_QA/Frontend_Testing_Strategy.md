# Frontend Testing Strategy - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Testing Strategy Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active

## Overview

This document outlines the comprehensive testing strategy for DafnckMachine v3.1 frontend development, covering unit testing, integration testing, end-to-end testing, and quality assurance practices to ensure robust, reliable, and maintainable code.

## Testing Philosophy

### 1. Testing Pyramid
- **Unit Tests (70%)**: Fast, isolated tests for individual components and functions
- **Integration Tests (20%)**: Tests for component interactions and API integrations
- **End-to-End Tests (10%)**: Full user journey tests across the entire application

### 2. Testing Principles
- **Test-Driven Development (TDD)**: Write tests before implementation when possible
- **Behavior-Driven Testing**: Focus on user behavior and business requirements
- **Continuous Testing**: Automated testing in CI/CD pipeline
- **Quality Gates**: Minimum coverage thresholds and quality metrics

## Testing Framework Stack

### Core Testing Libraries
```json
{
  "dependencies": {
    "@testing-library/react": "^13.4.0",
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/user-event": "^14.4.3",
    "jest": "^29.5.0",
    "cypress": "^12.17.0",
    "msw": "^1.2.2",
    "@storybook/react": "^7.0.0",
    "chromatic": "^6.19.9"
  }
}
```

### Testing Tools Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/reportWebVitals.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}',
  ],
};
```

## Unit Testing Strategy

### Component Testing Patterns
```typescript
// components/__tests__/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '../Button';

describe('Button Component', () => {
  const user = userEvent.setup();

  it('renders with correct text', () => {
    render(<Button variant="primary">Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('applies correct variant class', () => {
    render(<Button variant="secondary">Test</Button>);
    expect(screen.getByRole('button')).toHaveClass('button--secondary');
  });

  it('handles click events', async () => {
    const handleClick = jest.fn();
    render(<Button variant="primary" onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('shows loading state', () => {
    render(<Button variant="primary" loading>Loading</Button>);
    const button = screen.getByRole('button');
    
    expect(button).toBeDisabled();
    expect(button).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByText('Loading')).toBeInTheDocument();
  });

  it('supports keyboard navigation', async () => {
    const handleClick = jest.fn();
    render(<Button variant="primary" onClick={handleClick}>Press me</Button>);
    
    const button = screen.getByRole('button');
    button.focus();
    
    await user.keyboard('{Enter}');
    expect(handleClick).toHaveBeenCalledTimes(1);
    
    await user.keyboard(' ');
    expect(handleClick).toHaveBeenCalledTimes(2);
  });
});
```

### Hook Testing
```typescript
// hooks/__tests__/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { useAuth } from '../useAuth';
import { authSlice } from '../../store/slices/authSlice';

const createMockStore = (initialState = {}) => {
  return configureStore({
    reducer: {
      auth: authSlice.reducer,
    },
    preloadedState: initialState,
  });
};

const wrapper = ({ children, store }: any) => (
  <Provider store={store}>{children}</Provider>
);

describe('useAuth Hook', () => {
  it('returns initial auth state', () => {
    const store = createMockStore();
    const { result } = renderHook(() => useAuth(), {
      wrapper: ({ children }) => wrapper({ children, store }),
    });

    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBeNull();
  });

  it('handles login success', async () => {
    const store = createMockStore();
    const { result } = renderHook(() => useAuth(), {
      wrapper: ({ children }) => wrapper({ children, store }),
    });

    const mockUser = { id: '1', email: 'test@example.com', name: 'Test User' };
    
    // Mock the login API call
    jest.spyOn(authService, 'login').mockResolvedValue({
      user: mockUser,
      token: 'mock-token',
      refreshToken: 'mock-refresh-token',
    });

    await act(async () => {
      await result.current.login({ email: 'test@example.com', password: 'password' });
    });

    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.user).toEqual(mockUser);
  });

  it('checks permissions correctly', () => {
    const store = createMockStore({
      auth: {
        permissions: [{ name: 'admin' }, { name: 'user' }],
      },
    });

    const { result } = renderHook(() => useAuth(), {
      wrapper: ({ children }) => wrapper({ children, store }),
    });

    expect(result.current.hasPermission('admin')).toBe(true);
    expect(result.current.hasPermission('moderator')).toBe(false);
  });
});
```

### Utility Function Testing
```typescript
// utils/__tests__/validation.test.ts
import { validateEmail, validatePassword, validateForm } from '../validation';

describe('Validation Utilities', () => {
  describe('validateEmail', () => {
    it('validates correct email formats', () => {
      expect(validateEmail('test@example.com')).toBe(true);
      expect(validateEmail('user.name+tag@domain.co.uk')).toBe(true);
    });

    it('rejects invalid email formats', () => {
      expect(validateEmail('invalid-email')).toBe(false);
      expect(validateEmail('test@')).toBe(false);
      expect(validateEmail('@example.com')).toBe(false);
    });
  });

  describe('validatePassword', () => {
    it('validates strong passwords', () => {
      expect(validatePassword('StrongPass123!')).toBe(true);
    });

    it('rejects weak passwords', () => {
      expect(validatePassword('weak')).toBe(false);
      expect(validatePassword('12345678')).toBe(false);
    });
  });

  describe('validateForm', () => {
    it('validates complete form data', () => {
      const formData = {
        email: 'test@example.com',
        password: 'StrongPass123!',
        confirmPassword: 'StrongPass123!',
      };

      const result = validateForm(formData);
      expect(result.isValid).toBe(true);
      expect(result.errors).toEqual({});
    });

    it('returns errors for invalid form data', () => {
      const formData = {
        email: 'invalid-email',
        password: 'weak',
        confirmPassword: 'different',
      };

      const result = validateForm(formData);
      expect(result.isValid).toBe(false);
      expect(result.errors.email).toBeDefined();
      expect(result.errors.password).toBeDefined();
      expect(result.errors.confirmPassword).toBeDefined();
    });
  });
});
```

## Integration Testing

### API Integration Testing
```typescript
// services/__tests__/apiService.test.ts
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { apiService } from '../apiService';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: '1', name: 'John Doe', email: 'john@example.com' },
        { id: '2', name: 'Jane Smith', email: 'jane@example.com' },
      ])
    );
  }),

  rest.post('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({ id: '3', name: 'New User', email: 'new@example.com' })
    );
  }),

  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    if (id === '404') {
      return res(ctx.status(404), ctx.json({ error: 'User not found' }));
    }
    return res(
      ctx.json({ id, name: 'Test User', email: 'test@example.com' })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('API Service', () => {
  it('fetches users successfully', async () => {
    const users = await apiService.getUsers();
    expect(users).toHaveLength(2);
    expect(users[0]).toEqual({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    });
  });

  it('creates user successfully', async () => {
    const newUser = { name: 'New User', email: 'new@example.com' };
    const createdUser = await apiService.createUser(newUser);
    
    expect(createdUser).toEqual({
      id: '3',
      name: 'New User',
      email: 'new@example.com',
    });
  });

  it('handles API errors', async () => {
    await expect(apiService.getUserById('404')).rejects.toThrow('User not found');
  });

  it('includes authentication headers', async () => {
    const token = 'mock-token';
    apiService.setToken(token);

    server.use(
      rest.get('/api/profile', (req, res, ctx) => {
        const authHeader = req.headers.get('authorization');
        if (authHeader !== `Bearer ${token}`) {
          return res(ctx.status(401), ctx.json({ error: 'Unauthorized' }));
        }
        return res(ctx.json({ id: '1', name: 'Authenticated User' }));
      })
    );

    const profile = await apiService.getProfile();
    expect(profile.name).toBe('Authenticated User');
  });
});
```

### Component Integration Testing
```typescript
// components/__tests__/UserForm.integration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { UserForm } from '../UserForm';
import { TestProviders } from '../../utils/testUtils';

const server = setupServer(
  rest.post('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({ id: '1', name: 'Test User', email: 'test@example.com' })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserForm Integration', () => {
  const user = userEvent.setup();

  it('submits form successfully', async () => {
    const onSuccess = jest.fn();
    
    render(
      <TestProviders>
        <UserForm onSuccess={onSuccess} />
      </TestProviders>
    );

    // Fill out the form
    await user.type(screen.getByLabelText(/name/i), 'Test User');
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'StrongPass123!');

    // Submit the form
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // Wait for success
    await waitFor(() => {
      expect(onSuccess).toHaveBeenCalledWith({
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
      });
    });
  });

  it('displays validation errors', async () => {
    render(
      <TestProviders>
        <UserForm />
      </TestProviders>
    );

    // Submit empty form
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // Check for validation errors
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/password is required/i)).toBeInTheDocument();
  });

  it('handles API errors', async () => {
    server.use(
      rest.post('/api/users', (req, res, ctx) => {
        return res(
          ctx.status(400),
          ctx.json({ error: 'Email already exists' })
        );
      })
    );

    render(
      <TestProviders>
        <UserForm />
      </TestProviders>
    );

    await user.type(screen.getByLabelText(/name/i), 'Test User');
    await user.type(screen.getByLabelText(/email/i), 'existing@example.com');
    await user.type(screen.getByLabelText(/password/i), 'StrongPass123!');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(screen.getByText(/email already exists/i)).toBeInTheDocument();
    });
  });
});
```

## End-to-End Testing

### Cypress Configuration
```typescript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    supportFile: 'cypress/support/e2e.ts',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
  },
  component: {
    devServer: {
      framework: 'create-react-app',
      bundler: 'webpack',
    },
    supportFile: 'cypress/support/component.ts',
    specPattern: 'src/**/*.cy.{js,jsx,ts,tsx}',
  },
});
```

### E2E Test Examples
```typescript
// cypress/e2e/auth.cy.ts
describe('Authentication Flow', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('allows user to login', () => {
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="submit-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-menu"]').should('be.visible');
    cy.get('[data-testid="welcome-message"]').should('contain', 'Welcome back');
  });

  it('shows error for invalid credentials', () => {
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="submit-button"]').click();

    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
  });

  it('allows user to logout', () => {
    // Login first
    cy.login('test@example.com', 'password123');
    
    cy.get('[data-testid="user-menu"]').click();
    cy.get('[data-testid="logout-button"]').click();

    cy.url().should('eq', Cypress.config().baseUrl + '/');
    cy.get('[data-testid="login-button"]').should('be.visible');
  });
});

// cypress/e2e/dashboard.cy.ts
describe('Dashboard Functionality', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password123');
    cy.visit('/dashboard');
  });

  it('displays user dashboard', () => {
    cy.get('[data-testid="dashboard-title"]').should('contain', 'Dashboard');
    cy.get('[data-testid="stats-cards"]').should('be.visible');
    cy.get('[data-testid="recent-activity"]').should('be.visible');
  });

  it('allows navigation between sections', () => {
    cy.get('[data-testid="nav-projects"]').click();
    cy.url().should('include', '/projects');
    cy.get('[data-testid="projects-list"]').should('be.visible');

    cy.get('[data-testid="nav-tasks"]').click();
    cy.url().should('include', '/tasks');
    cy.get('[data-testid="tasks-list"]').should('be.visible');
  });

  it('handles responsive design', () => {
    cy.viewport('iphone-x');
    cy.get('[data-testid="mobile-menu-toggle"]').should('be.visible');
    cy.get('[data-testid="sidebar"]').should('not.be.visible');

    cy.get('[data-testid="mobile-menu-toggle"]').click();
    cy.get('[data-testid="mobile-nav"]').should('be.visible');
  });
});
```

### Custom Cypress Commands
```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      logout(): Chainable<void>;
      createProject(name: string, description?: string): Chainable<void>;
      deleteProject(projectId: string): Chainable<void>;
    }
  }
}

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(email);
    cy.get('[data-testid="password-input"]').type(password);
    cy.get('[data-testid="submit-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('logout', () => {
  cy.get('[data-testid="user-menu"]').click();
  cy.get('[data-testid="logout-button"]').click();
  cy.url().should('eq', Cypress.config().baseUrl + '/');
});

Cypress.Commands.add('createProject', (name: string, description?: string) => {
  cy.get('[data-testid="create-project-button"]').click();
  cy.get('[data-testid="project-name-input"]').type(name);
  if (description) {
    cy.get('[data-testid="project-description-input"]').type(description);
  }
  cy.get('[data-testid="save-project-button"]').click();
  cy.get('[data-testid="success-message"]').should('be.visible');
});
```

## Visual Testing

### Storybook Configuration
```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',
    '@storybook/addon-viewport',
    '@storybook/addon-docs',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  typescript: {
    check: false,
    reactDocgen: 'react-docgen-typescript',
    reactDocgenTypescriptOptions: {
      shouldExtractLiteralValuesFromEnum: true,
      propFilter: (prop) => (prop.parent ? !/node_modules/.test(prop.parent.fileName) : true),
    },
  },
};

export default config;
```

### Component Stories
```typescript
// components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A versatile button component with multiple variants and states.',
      },
    },
  },
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['primary', 'secondary', 'danger', 'ghost'],
    },
    size: {
      control: { type: 'select' },
      options: ['small', 'medium', 'large'],
    },
    loading: {
      control: { type: 'boolean' },
    },
    disabled: {
      control: { type: 'boolean' },
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Loading: Story = {
  args: {
    variant: 'primary',
    loading: true,
    children: 'Loading...',
  },
};

export const Disabled: Story = {
  args: {
    variant: 'primary',
    disabled: true,
    children: 'Disabled Button',
  },
};

export const AllSizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
      <Button variant="primary" size="small">Small</Button>
      <Button variant="primary" size="medium">Medium</Button>
      <Button variant="primary" size="large">Large</Button>
    </div>
  ),
};
```

### Chromatic Visual Testing
```typescript
// .github/workflows/chromatic.yml
name: 'Chromatic'

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  chromatic-deployment:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Publish to Chromatic
        uses: chromaui/action@v1
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          buildScriptName: 'build-storybook'
          exitZeroOnChanges: true
```

## Accessibility Testing

### Automated A11y Testing
```typescript
// components/__tests__/Button.a11y.test.tsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from '../Button';

expect.extend(toHaveNoViolations);

describe('Button Accessibility', () => {
  it('should not have accessibility violations', async () => {
    const { container } = render(
      <Button variant="primary">Accessible Button</Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('should have proper ARIA attributes when loading', async () => {
    const { container } = render(
      <Button variant="primary" loading>Loading Button</Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('should be keyboard accessible', () => {
    render(<Button variant="primary">Keyboard Button</Button>);
    const button = screen.getByRole('button');
    
    expect(button).toHaveAttribute('tabindex', '0');
    expect(button).not.toHaveAttribute('aria-hidden', 'true');
  });
});
```

### Cypress A11y Testing
```typescript
// cypress/e2e/accessibility.cy.ts
describe('Accessibility Tests', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.injectAxe();
  });

  it('should not have accessibility violations on homepage', () => {
    cy.checkA11y();
  });

  it('should not have accessibility violations on login page', () => {
    cy.visit('/login');
    cy.checkA11y();
  });

  it('should support keyboard navigation', () => {
    cy.get('body').tab();
    cy.focused().should('have.attr', 'data-testid', 'skip-link');
    
    cy.focused().tab();
    cy.focused().should('have.attr', 'data-testid', 'main-nav');
  });

  it('should have proper focus management in modals', () => {
    cy.get('[data-testid="open-modal-button"]').click();
    cy.focused().should('have.attr', 'data-testid', 'modal-close-button');
    
    cy.get('[data-testid="modal-close-button"]').click();
    cy.focused().should('have.attr', 'data-testid', 'open-modal-button');
  });
});
```

## Performance Testing

### Performance Metrics Testing
```typescript
// cypress/e2e/performance.cy.ts
describe('Performance Tests', () => {
  it('should load homepage within performance budget', () => {
    cy.visit('/', {
      onBeforeLoad: (win) => {
        win.performance.mark('start');
      },
      onLoad: (win) => {
        win.performance.mark('end');
        win.performance.measure('pageLoad', 'start', 'end');
        
        const measure = win.performance.getEntriesByName('pageLoad')[0];
        expect(measure.duration).to.be.lessThan(3000); // 3 seconds
      },
    });
  });

  it('should have good Core Web Vitals', () => {
    cy.visit('/');
    
    // Check Largest Contentful Paint
    cy.window().then((win) => {
      new win.PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lcp = entries[entries.length - 1];
        expect(lcp.startTime).to.be.lessThan(2500); // 2.5 seconds
      }).observe({ entryTypes: ['largest-contentful-paint'] });
    });
  });
});
```

### Bundle Size Testing
```javascript
// scripts/bundle-size-test.js
const { execSync } = require('child_process');
const fs = require('fs');

const buildOutput = execSync('npm run build', { encoding: 'utf-8' });
const bundleAnalysis = execSync('npx webpack-bundle-analyzer build/static/js/*.js --json', { encoding: 'utf-8' });

const analysis = JSON.parse(bundleAnalysis);
const mainBundleSize = analysis.assets.find(asset => asset.name.includes('main')).size;

const maxBundleSize = 250 * 1024; // 250KB
if (mainBundleSize > maxBundleSize) {
  console.error(`Bundle size ${mainBundleSize} exceeds maximum ${maxBundleSize}`);
  process.exit(1);
}

console.log(`Bundle size ${mainBundleSize} is within limits`);
```

## Test Data Management

### Test Fixtures
```typescript
// cypress/fixtures/users.json
{
  "validUser": {
    "email": "test@example.com",
    "password": "StrongPass123!",
    "name": "Test User"
  },
  "adminUser": {
    "email": "admin@example.com",
    "password": "AdminPass123!",
    "name": "Admin User",
    "role": "admin"
  },
  "invalidUser": {
    "email": "invalid-email",
    "password": "weak"
  }
}
```

### Mock Data Factories
```typescript
// utils/testFactories.ts
import { faker } from '@faker-js/faker';

export const createMockUser = (overrides: Partial<User> = {}): User => ({
  id: faker.string.uuid(),
  email: faker.internet.email(),
  name: faker.person.fullName(),
  avatar: faker.image.avatar(),
  role: 'user',
  permissions: [],
  preferences: {},
  createdAt: faker.date.past().toISOString(),
  updatedAt: faker.date.recent().toISOString(),
  ...overrides,
});

export const createMockProject = (overrides: Partial<Project> = {}): Project => ({
  id: faker.string.uuid(),
  name: faker.company.name(),
  description: faker.lorem.paragraph(),
  status: 'active',
  ownerId: faker.string.uuid(),
  createdAt: faker.date.past().toISOString(),
  updatedAt: faker.date.recent().toISOString(),
  ...overrides,
});

export const createMockTask = (overrides: Partial<Task> = {}): Task => ({
  id: faker.string.uuid(),
  title: faker.lorem.sentence(),
  description: faker.lorem.paragraph(),
  status: 'pending',
  priority: 'medium',
  assigneeId: faker.string.uuid(),
  projectId: faker.string.uuid(),
  createdAt: faker.date.past().toISOString(),
  updatedAt: faker.date.recent().toISOString(),
  ...overrides,
});
```

## CI/CD Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
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
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
      
      - name: Start application
        run: npm start &
        
      - name: Wait for application
        run: npx wait-on http://localhost:3000
      
      - name: Run Cypress tests
        uses: cypress-io/github-action@v5
        with:
          browser: chrome
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}

  accessibility-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build Storybook
        run: npm run build-storybook
      
      - name: Run accessibility tests
        run: npm run test:a11y
```

## Quality Gates

### Coverage Requirements
```javascript
// jest.config.js - Coverage thresholds
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/utils/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
  },
};
```

### Quality Metrics
- **Unit Test Coverage**: Minimum 80% overall, 90% for components
- **Integration Test Coverage**: All critical user flows
- **E2E Test Coverage**: All major user journeys
- **Accessibility Score**: 100% WCAG AA compliance
- **Performance Budget**: LCP < 2.5s, FID < 100ms, CLS < 0.1

## Best Practices

### Testing Guidelines
1. **Write Tests First**: Follow TDD when possible
2. **Test Behavior, Not Implementation**: Focus on what the component does
3. **Use Descriptive Test Names**: Clear, specific test descriptions
4. **Arrange, Act, Assert**: Structure tests clearly
5. **Mock External Dependencies**: Isolate units under test
6. **Test Edge Cases**: Include error conditions and boundary cases

### Maintenance Practices
- Regular test review and refactoring
- Update tests when requirements change
- Monitor test performance and execution time
- Keep test data fresh and relevant
- Document complex test scenarios

---

**Document Status**: Active  
**Next Review**: 2025-04-27  
**Maintained By**: Frontend Development Team 