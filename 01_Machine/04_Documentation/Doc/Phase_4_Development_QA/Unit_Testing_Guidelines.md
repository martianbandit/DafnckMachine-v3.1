# Unit Testing Guidelines - DafnckMachine v3.1

## Overview
Comprehensive unit testing guidelines for the DafnckMachine v3.1 project, covering testing frameworks, best practices, patterns, and standards for writing effective unit tests.

## Testing Framework Setup

### Primary Testing Stack

#### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@/components/(.*)$': '<rootDir>/src/components/$1',
    '^@/services/(.*)$': '<rootDir>/src/services/$1',
    '^@/utils/(.*)$': '<rootDir>/src/utils/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/test/**/*',
    '!src/**/*.stories.tsx'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 90,
      statements: 90
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}'
  ],
  transform: {
    '^.+\\.(ts|tsx)$': 'ts-jest'
  },
  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx', 'json'],
  testTimeout: 10000
};
```

#### Test Setup Configuration
```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { configure } from '@testing-library/react';
import { server } from './mocks/server';

// Configure React Testing Library
configure({
  testIdAttribute: 'data-testid',
  asyncUtilTimeout: 5000
});

// Mock IntersectionObserver
global.IntersectionObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn()
}));

// Mock ResizeObserver
global.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn()
}));

// Setup MSW
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Mock console methods in tests
const originalError = console.error;
beforeAll(() => {
  console.error = (...args: any[]) => {
    if (
      typeof args[0] === 'string' &&
      args[0].includes('Warning: ReactDOM.render is no longer supported')
    ) {
      return;
    }
    originalError.call(console, ...args);
  };
});

afterAll(() => {
  console.error = originalError;
});
```

### React Testing Library Setup
```typescript
// src/test/utils.tsx
import React, { ReactElement } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from '@/contexts/ThemeContext';
import { AuthProvider } from '@/contexts/AuthContext';

// Create a custom render function that includes providers
const AllTheProviders = ({ children }: { children: React.ReactNode }) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        cacheTime: 0
      }
    }
  });

  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <ThemeProvider>
          <AuthProvider>
            {children}
          </AuthProvider>
        </ThemeProvider>
      </BrowserRouter>
    </QueryClientProvider>
  );
};

const customRender = (
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllTheProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

## Unit Testing Patterns

### 1. Component Testing

#### Basic Component Test
```typescript
// src/components/Button/Button.test.tsx
import { render, screen, fireEvent } from '@/test/utils';
import { Button } from './Button';

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies correct CSS classes based on variant', () => {
    render(<Button variant="primary">Primary Button</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('btn-primary');
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled Button</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  it('shows loading state correctly', () => {
    render(<Button loading>Loading Button</Button>);
    
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });
});
```

#### Form Component Testing
```typescript
// src/components/LoginForm/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@/test/utils';
import { LoginForm } from './LoginForm';

describe('LoginForm Component', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders all form fields', () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /sign in/i })).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
      expect(screen.getByText(/password is required/i)).toBeInTheDocument();
    });
    
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('validates email format', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'invalid-email' }
    });
    fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/invalid email format/i)).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    fireEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });
});
```

### 2. Hook Testing

#### Custom Hook Testing
```typescript
// src/hooks/useAuth/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useAuth } from './useAuth';
import { authService } from '@/services/authService';

// Mock the auth service
jest.mock('@/services/authService');
const mockAuthService = authService as jest.Mocked<typeof authService>;

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useAuth Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('returns initial state correctly', () => {
    const { result } = renderHook(() => useAuth(), {
      wrapper: createWrapper()
    });
    
    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.isLoading).toBe(false);
  });

  it('handles login successfully', async () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    mockAuthService.login.mockResolvedValue({ user: mockUser, token: 'token' });
    
    const { result } = renderHook(() => useAuth(), {
      wrapper: createWrapper()
    });
    
    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });
    
    expect(result.current.user).toEqual(mockUser);
    expect(result.current.isAuthenticated).toBe(true);
    expect(mockAuthService.login).toHaveBeenCalledWith('test@example.com', 'password');
  });

  it('handles login failure', async () => {
    const error = new Error('Invalid credentials');
    mockAuthService.login.mockRejectedValue(error);
    
    const { result } = renderHook(() => useAuth(), {
      wrapper: createWrapper()
    });
    
    await act(async () => {
      try {
        await result.current.login('test@example.com', 'wrong-password');
      } catch (e) {
        // Expected to throw
      }
    });
    
    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });

  it('handles logout correctly', async () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    mockAuthService.login.mockResolvedValue({ user: mockUser, token: 'token' });
    mockAuthService.logout.mockResolvedValue(undefined);
    
    const { result } = renderHook(() => useAuth(), {
      wrapper: createWrapper()
    });
    
    // Login first
    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });
    
    // Then logout
    await act(async () => {
      await result.current.logout();
    });
    
    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(mockAuthService.logout).toHaveBeenCalled();
  });
});
```

### 3. Service/Utility Testing

#### Service Function Testing
```typescript
// src/services/userService/userService.test.ts
import { userService } from './userService';
import { apiClient } from '@/lib/apiClient';

// Mock the API client
jest.mock('@/lib/apiClient');
const mockApiClient = apiClient as jest.Mocked<typeof apiClient>;

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getUser', () => {
    it('returns user data for valid ID', async () => {
      const mockUser = { id: '1', email: 'test@example.com', name: 'Test User' };
      mockApiClient.get.mockResolvedValue({ data: mockUser });
      
      const result = await userService.getUser('1');
      
      expect(result).toEqual(mockUser);
      expect(mockApiClient.get).toHaveBeenCalledWith('/users/1');
    });

    it('throws error for invalid ID', async () => {
      mockApiClient.get.mockRejectedValue(new Error('User not found'));
      
      await expect(userService.getUser('invalid')).rejects.toThrow('User not found');
    });
  });

  describe('updateUser', () => {
    it('updates user successfully', async () => {
      const userId = '1';
      const updateData = { name: 'Updated Name' };
      const updatedUser = { id: userId, email: 'test@example.com', name: 'Updated Name' };
      
      mockApiClient.put.mockResolvedValue({ data: updatedUser });
      
      const result = await userService.updateUser(userId, updateData);
      
      expect(result).toEqual(updatedUser);
      expect(mockApiClient.put).toHaveBeenCalledWith(`/users/${userId}`, updateData);
    });

    it('validates required fields', async () => {
      await expect(userService.updateUser('1', {})).rejects.toThrow('No update data provided');
    });
  });

  describe('deleteUser', () => {
    it('deletes user successfully', async () => {
      mockApiClient.delete.mockResolvedValue({ data: { success: true } });
      
      const result = await userService.deleteUser('1');
      
      expect(result).toBe(true);
      expect(mockApiClient.delete).toHaveBeenCalledWith('/users/1');
    });
  });
});
```

#### Utility Function Testing
```typescript
// src/utils/validation/validation.test.ts
import { validateEmail, validatePassword, validateRequired } from './validation';

describe('Validation Utils', () => {
  describe('validateEmail', () => {
    it('returns true for valid email addresses', () => {
      const validEmails = [
        'test@example.com',
        'user.name@domain.co.uk',
        'user+tag@example.org'
      ];
      
      validEmails.forEach(email => {
        expect(validateEmail(email)).toBe(true);
      });
    });

    it('returns false for invalid email addresses', () => {
      const invalidEmails = [
        'invalid-email',
        '@example.com',
        'user@',
        'user..name@example.com',
        ''
      ];
      
      invalidEmails.forEach(email => {
        expect(validateEmail(email)).toBe(false);
      });
    });
  });

  describe('validatePassword', () => {
    it('returns true for strong passwords', () => {
      const strongPasswords = [
        'StrongPass123!',
        'MySecure@Password1',
        'Complex#Pass99'
      ];
      
      strongPasswords.forEach(password => {
        expect(validatePassword(password)).toBe(true);
      });
    });

    it('returns false for weak passwords', () => {
      const weakPasswords = [
        'weak',
        '12345678',
        'password',
        'PASSWORD',
        'Pass123' // Too short
      ];
      
      weakPasswords.forEach(password => {
        expect(validatePassword(password)).toBe(false);
      });
    });
  });

  describe('validateRequired', () => {
    it('returns true for non-empty values', () => {
      expect(validateRequired('value')).toBe(true);
      expect(validateRequired('0')).toBe(true);
      expect(validateRequired(0)).toBe(true);
      expect(validateRequired(false)).toBe(true);
    });

    it('returns false for empty values', () => {
      expect(validateRequired('')).toBe(false);
      expect(validateRequired(null)).toBe(false);
      expect(validateRequired(undefined)).toBe(false);
      expect(validateRequired('   ')).toBe(false); // Whitespace only
    });
  });
});
```

## Mocking Strategies

### 1. API Mocking with MSW

#### MSW Setup
```typescript
// src/test/mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  // Auth endpoints
  rest.post('/api/auth/login', (req, res, ctx) => {
    const { email, password } = req.body as any;
    
    if (email === 'test@example.com' && password === 'password') {
      return res(
        ctx.status(200),
        ctx.json({
          user: { id: '1', email: 'test@example.com', name: 'Test User' },
          token: 'mock-jwt-token'
        })
      );
    }
    
    return res(
      ctx.status(401),
      ctx.json({ error: 'Invalid credentials' })
    );
  }),

  rest.post('/api/auth/logout', (req, res, ctx) => {
    return res(ctx.status(200), ctx.json({ success: true }));
  }),

  // User endpoints
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    
    if (id === '1') {
      return res(
        ctx.status(200),
        ctx.json({ id: '1', email: 'test@example.com', name: 'Test User' })
      );
    }
    
    return res(
      ctx.status(404),
      ctx.json({ error: 'User not found' })
    );
  }),

  rest.put('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    const updateData = req.body as any;
    
    return res(
      ctx.status(200),
      ctx.json({ id, ...updateData })
    );
  })
];
```

```typescript
// src/test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### 2. Module Mocking

#### External Library Mocking
```typescript
// Mock React Router
jest.mock('react-router-dom', () => ({
  ...jest.requireActual('react-router-dom'),
  useNavigate: () => jest.fn(),
  useLocation: () => ({ pathname: '/test' }),
  useParams: () => ({ id: '1' })
}));

// Mock date-fns
jest.mock('date-fns', () => ({
  format: jest.fn((date, formatStr) => '2024-01-15'),
  parseISO: jest.fn((dateStr) => new Date('2024-01-15')),
  isValid: jest.fn(() => true)
}));

// Mock file upload
jest.mock('@/utils/fileUpload', () => ({
  uploadFile: jest.fn().mockResolvedValue({
    url: 'https://example.com/uploaded-file.jpg',
    id: 'file-123'
  })
}));
```

#### Partial Module Mocking
```typescript
// Mock only specific functions from a module
jest.mock('@/services/apiService', () => ({
  ...jest.requireActual('@/services/apiService'),
  uploadImage: jest.fn().mockResolvedValue({ url: 'mock-url' }),
  deleteImage: jest.fn().mockResolvedValue(true)
}));
```

### 3. Component Mocking

#### Complex Component Mocking
```typescript
// Mock heavy components for faster tests
jest.mock('@/components/Chart', () => {
  return function MockChart({ data, title }: any) {
    return (
      <div data-testid="mock-chart">
        <h3>{title}</h3>
        <div>Chart with {data.length} data points</div>
      </div>
    );
  };
});

// Mock third-party components
jest.mock('react-select', () => {
  return function MockSelect({ options, onChange, value, placeholder }: any) {
    return (
      <select
        data-testid="mock-select"
        value={value?.value || ''}
        onChange={(e) => {
          const selectedOption = options.find((opt: any) => opt.value === e.target.value);
          onChange(selectedOption);
        }}
      >
        <option value="">{placeholder}</option>
        {options.map((option: any) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
    );
  };
});
```

## Test Organization

### 1. File Structure
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── Button.stories.tsx
│   └── LoginForm/
│       ├── LoginForm.tsx
│       ├── LoginForm.test.tsx
│       └── __tests__/
│           ├── LoginForm.integration.test.tsx
│           └── LoginForm.accessibility.test.tsx
├── hooks/
│   ├── useAuth/
│   │   ├── useAuth.ts
│   │   └── useAuth.test.ts
├── services/
│   ├── userService/
│   │   ├── userService.ts
│   │   └── userService.test.ts
├── utils/
│   ├── validation/
│   │   ├── validation.ts
│   │   └── validation.test.ts
└── test/
    ├── setup.ts
    ├── utils.tsx
    └── mocks/
        ├── handlers.ts
        └── server.ts
```

### 2. Test Naming Conventions

#### Describe Blocks
```typescript
// Feature-based grouping
describe('UserProfile Component', () => {
  describe('when user is authenticated', () => {
    describe('and has complete profile', () => {
      it('displays all user information', () => {
        // Test implementation
      });
    });
  });
});

// Behavior-based grouping
describe('UserService', () => {
  describe('getUser()', () => {
    it('returns user data for valid ID', () => {
      // Test implementation
    });
    
    it('throws error for invalid ID', () => {
      // Test implementation
    });
  });
});
```

#### Test Names
```typescript
// Good test names - describe behavior and expected outcome
it('displays error message when login fails')
it('redirects to dashboard after successful login')
it('disables submit button while form is submitting')
it('validates email format before submission')

// Avoid implementation details
it('calls handleSubmit function') // ❌ Too implementation-focused
it('submits form with user credentials') // ✅ Behavior-focused
```

## Best Practices

### 1. Test Independence
```typescript
// ✅ Good - Each test is independent
describe('UserList Component', () => {
  beforeEach(() => {
    // Reset state before each test
    jest.clearAllMocks();
  });

  it('displays loading state initially', () => {
    render(<UserList />);
    expect(screen.getByTestId('loading')).toBeInTheDocument();
  });

  it('displays users after loading', async () => {
    render(<UserList />);
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
});

// ❌ Bad - Tests depend on each other
describe('UserList Component', () => {
  let component: RenderResult;

  it('renders component', () => {
    component = render(<UserList />);
    expect(component).toBeDefined();
  });

  it('displays users', () => {
    // This test depends on the previous test
    expect(component.getByText('John Doe')).toBeInTheDocument();
  });
});
```

### 2. Arrange-Act-Assert Pattern
```typescript
it('updates user profile successfully', async () => {
  // Arrange
  const user = { id: '1', name: 'John Doe', email: 'john@example.com' };
  const updatedData = { name: 'Jane Doe' };
  mockApiClient.put.mockResolvedValue({ data: { ...user, ...updatedData } });

  // Act
  const result = await userService.updateUser(user.id, updatedData);

  // Assert
  expect(result).toEqual({ ...user, ...updatedData });
  expect(mockApiClient.put).toHaveBeenCalledWith('/users/1', updatedData);
});
```

### 3. Test Data Management
```typescript
// Use factories for consistent test data
const createMockUser = (overrides = {}) => ({
  id: '1',
  email: 'test@example.com',
  name: 'Test User',
  createdAt: '2024-01-15T00:00:00Z',
  ...overrides
});

// Use descriptive test data
it('displays user with special characters in name', () => {
  const userWithSpecialChars = createMockUser({
    name: 'José María O\'Connor-Smith'
  });
  
  render(<UserProfile user={userWithSpecialChars} />);
  expect(screen.getByText('José María O\'Connor-Smith')).toBeInTheDocument();
});
```

### 4. Async Testing
```typescript
// ✅ Good - Proper async testing
it('loads and displays user data', async () => {
  render(<UserProfile userId="1" />);
  
  // Wait for loading to complete
  await waitFor(() => {
    expect(screen.queryByTestId('loading')).not.toBeInTheDocument();
  });
  
  // Assert final state
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});

// ✅ Good - Testing error states
it('displays error message when user loading fails', async () => {
  mockApiClient.get.mockRejectedValue(new Error('Network error'));
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText(/error loading user/i)).toBeInTheDocument();
  });
});

// ❌ Bad - Not waiting for async operations
it('displays user data', () => {
  render(<UserProfile userId="1" />);
  expect(screen.getByText('John Doe')).toBeInTheDocument(); // May fail
});
```

### 5. Error Testing
```typescript
// Test error boundaries
it('displays error fallback when component throws', () => {
  const ThrowError = () => {
    throw new Error('Component error');
  };
  
  render(
    <ErrorBoundary>
      <ThrowError />
    </ErrorBoundary>
  );
  
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
});

// Test error handling in services
it('handles network errors gracefully', async () => {
  mockApiClient.get.mockRejectedValue(new Error('Network error'));
  
  await expect(userService.getUser('1')).rejects.toThrow('Network error');
});
```

## Performance Testing

### 1. Component Performance
```typescript
// Test component rendering performance
it('renders large list efficiently', () => {
  const largeUserList = Array.from({ length: 1000 }, (_, i) => 
    createMockUser({ id: i.toString(), name: `User ${i}` })
  );
  
  const startTime = performance.now();
  render(<UserList users={largeUserList} />);
  const endTime = performance.now();
  
  expect(endTime - startTime).toBeLessThan(100); // Should render in under 100ms
});

// Test memory leaks
it('cleans up event listeners on unmount', () => {
  const addEventListenerSpy = jest.spyOn(window, 'addEventListener');
  const removeEventListenerSpy = jest.spyOn(window, 'removeEventListener');
  
  const { unmount } = render(<ComponentWithEventListeners />);
  
  expect(addEventListenerSpy).toHaveBeenCalled();
  
  unmount();
  
  expect(removeEventListenerSpy).toHaveBeenCalledTimes(
    addEventListenerSpy.mock.calls.length
  );
});
```

## Accessibility Testing

### 1. Basic Accessibility Tests
```typescript
// Test keyboard navigation
it('supports keyboard navigation', () => {
  render(<NavigationMenu />);
  
  const firstItem = screen.getByRole('menuitem', { name: /home/i });
  firstItem.focus();
  
  fireEvent.keyDown(firstItem, { key: 'ArrowDown' });
  
  const secondItem = screen.getByRole('menuitem', { name: /about/i });
  expect(secondItem).toHaveFocus();
});

// Test ARIA attributes
it('has correct ARIA attributes', () => {
  render(<Modal isOpen={true} title="Test Modal" />);
  
  const modal = screen.getByRole('dialog');
  expect(modal).toHaveAttribute('aria-labelledby');
  expect(modal).toHaveAttribute('aria-modal', 'true');
});

// Test screen reader support
it('provides proper labels for screen readers', () => {
  render(<SearchInput />);
  
  const input = screen.getByLabelText(/search/i);
  expect(input).toHaveAttribute('aria-describedby');
  
  const description = screen.getByText(/enter search terms/i);
  expect(description).toHaveAttribute('id', input.getAttribute('aria-describedby'));
});
```

## Code Coverage

### 1. Coverage Configuration
```javascript
// jest.config.js - Coverage settings
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/test/**/*',
    '!src/**/*.stories.tsx',
    '!src/types/**/*'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 90,
      statements: 90
    },
    './src/components/': {
      branches: 85,
      functions: 90,
      lines: 95,
      statements: 95
    },
    './src/services/': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95
    }
  },
  coverageReporters: ['text', 'lcov', 'html', 'json-summary']
};
```

### 2. Coverage Analysis
```bash
# Generate coverage report
npm run test:coverage

# View coverage in browser
open coverage/lcov-report/index.html

# Check coverage thresholds
npm run test:coverage:check
```

## Continuous Integration

### 1. GitHub Actions Configuration
```yaml
# .github/workflows/test.yml
name: Unit Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
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
        run: npm run test:unit
      
      - name: Generate coverage report
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
      
      - name: Comment coverage on PR
        if: github.event_name == 'pull_request'
        uses: romeovs/lcov-reporter-action@v0.3.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          lcov-file: ./coverage/lcov.info
```

## Related Documentation

- [Test Strategy Framework](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Test_Strategy_Framework.md)
- [Integration Testing Plan](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Integration_Testing_Plan.md)
- [E2E Testing Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/E2E_Testing_Configuration.md)
- [Code Documentation Standards](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Code_Documentation_Standards.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: qa-team, development-team
- **Related Workflows**: 15_Automated_Testing.md 