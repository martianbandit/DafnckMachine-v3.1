# Unit Testing Implementation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Component_Architecture_Design.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Component_Architecture_Design.md)

## Overview

This guide provides comprehensive unit testing implementation strategies for DafnckMachine v3.1, covering component testing, utility functions, custom hooks, and snapshot testing using Jest, React Testing Library, and modern testing practices.

## Table of Contents

1. [Testing Framework Setup](#testing-framework-setup)
2. [Component Testing](#component-testing)
3. [Hook Testing](#hook-testing)
4. [Utility Function Testing](#utility-function-testing)
5. [Snapshot Testing](#snapshot-testing)
6. [Mocking Strategies](#mocking-strategies)
7. [Test Coverage](#test-coverage)
8. [Best Practices](#best-practices)

## Testing Framework Setup

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy'
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/test/**/*',
    '!src/**/*.stories.{ts,tsx}'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}'
  ]
};
```

### Test Setup

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { configure } from '@testing-library/react';

// Configure testing library
configure({ testIdAttribute: 'data-testid' });

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  observe() { return null; }
  disconnect() { return null; }
  unobserve() { return null; }
};

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor() {}
  observe() { return null; }
  disconnect() { return null; }
  unobserve() { return null; }
};

// Mock matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### Test Utilities

```typescript
// src/test/utils.tsx
import React from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

// Test providers wrapper
const AllTheProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });

  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        {children}
      </BrowserRouter>
    </QueryClientProvider>
  );
};

// Custom render function
const customRender = (
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllTheProviders, ...options });

// Re-export everything
export * from '@testing-library/react';
export { customRender as render };

// Test data factories
export const createMockUser = (overrides = {}) => ({
  id: '1',
  name: 'Test User',
  email: 'test@example.com',
  role: 'user',
  ...overrides
});

export const createMockPost = (overrides = {}) => ({
  id: '1',
  title: 'Test Post',
  content: 'Test content',
  authorId: '1',
  createdAt: new Date().toISOString(),
  ...overrides
});
```

## Component Testing

### Basic Component Testing

```typescript
// Button.test.tsx
import { render, screen, fireEvent } from '@/test/utils';
import { Button } from '@/components/ui/Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });

  test('handles click events', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('applies correct variant classes', () => {
    render(<Button variant="primary">Primary Button</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('bg-blue-600');
  });

  test('disables button when loading', () => {
    render(<Button loading>Loading Button</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
    expect(button).toHaveAttribute('aria-disabled', 'true');
  });

  test('shows loading state', () => {
    render(<Button loading loadingText="Processing...">Submit</Button>);
    
    expect(screen.getByText('Processing...')).toBeInTheDocument();
    expect(screen.queryByText('Submit')).not.toBeInTheDocument();
  });
});
```

### Form Component Testing

```typescript
// ContactForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@/test/utils';
import { ContactForm } from '@/components/forms/ContactForm';

describe('ContactForm Component', () => {
  test('renders all form fields', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/message/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  test('validates required fields', async () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    const submitButton = screen.getByRole('button', { name: /submit/i });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText(/name is required/i)).toBeInTheDocument();
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    });
  });

  test('validates email format', async () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    fireEvent.change(emailInput, { target: { value: 'invalid-email' } });
    fireEvent.blur(emailInput);
    
    await waitFor(() => {
      expect(screen.getByText(/invalid email format/i)).toBeInTheDocument();
    });
  });

  test('submits form with valid data', async () => {
    const mockSubmit = jest.fn();
    render(<ContactForm onSubmit={mockSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/name/i), {
      target: { value: 'John Doe' }
    });
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'john@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/message/i), {
      target: { value: 'Hello world' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        email: 'john@example.com',
        message: 'Hello world'
      });
    });
  });
});
```

### Modal Component Testing

```typescript
// Modal.test.tsx
import { render, screen, fireEvent } from '@/test/utils';
import { Modal } from '@/components/ui/Modal';

describe('Modal Component', () => {
  test('renders when open', () => {
    render(
      <Modal isOpen={true} onClose={jest.fn()} title="Test Modal">
        <p>Modal content</p>
      </Modal>
    );
    
    expect(screen.getByRole('dialog')).toBeInTheDocument();
    expect(screen.getByText('Test Modal')).toBeInTheDocument();
    expect(screen.getByText('Modal content')).toBeInTheDocument();
  });

  test('does not render when closed', () => {
    render(
      <Modal isOpen={false} onClose={jest.fn()} title="Test Modal">
        <p>Modal content</p>
      </Modal>
    );
    
    expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
  });

  test('calls onClose when close button clicked', () => {
    const mockClose = jest.fn();
    render(
      <Modal isOpen={true} onClose={mockClose} title="Test Modal">
        <p>Modal content</p>
      </Modal>
    );
    
    const closeButton = screen.getByRole('button', { name: /close/i });
    fireEvent.click(closeButton);
    
    expect(mockClose).toHaveBeenCalledTimes(1);
  });

  test('calls onClose when escape key pressed', () => {
    const mockClose = jest.fn();
    render(
      <Modal isOpen={true} onClose={mockClose} title="Test Modal">
        <p>Modal content</p>
      </Modal>
    );
    
    fireEvent.keyDown(document, { key: 'Escape' });
    
    expect(mockClose).toHaveBeenCalledTimes(1);
  });

  test('traps focus within modal', () => {
    render(
      <Modal isOpen={true} onClose={jest.fn()} title="Test Modal">
        <button>First Button</button>
        <button>Second Button</button>
      </Modal>
    );
    
    const firstButton = screen.getByText('First Button');
    const secondButton = screen.getByText('Second Button');
    
    // Focus should start on first focusable element
    expect(document.activeElement).toBe(firstButton);
    
    // Tab should move to second button
    fireEvent.keyDown(firstButton, { key: 'Tab' });
    expect(document.activeElement).toBe(secondButton);
  });
});
```

## Hook Testing

### Custom Hook Testing

```typescript
// useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from '@/hooks/useCounter';

describe('useCounter Hook', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  test('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  test('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  test('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  test('resets count', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

### API Hook Testing

```typescript
// useApi.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useUsers } from '@/hooks/useUsers';
import { api } from '@/lib/api';

// Mock the API
jest.mock('@/lib/api');
const mockApi = api as jest.Mocked<typeof api>;

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } }
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useUsers Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('fetches users successfully', async () => {
    const mockUsers = [
      { id: '1', name: 'User 1', email: 'user1@example.com' },
      { id: '2', name: 'User 2', email: 'user2@example.com' }
    ];
    
    mockApi.getUsers.mockResolvedValue(mockUsers);
    
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper()
    });
    
    expect(result.current.isLoading).toBe(true);
    
    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });
    
    expect(result.current.data).toEqual(mockUsers);
    expect(result.current.error).toBeNull();
  });

  test('handles fetch error', async () => {
    const mockError = new Error('Failed to fetch users');
    mockApi.getUsers.mockRejectedValue(mockError);
    
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper()
    });
    
    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });
    
    expect(result.current.data).toBeUndefined();
    expect(result.current.error).toEqual(mockError);
  });
});
```

## Utility Function Testing

### Pure Function Testing

```typescript
// utils.test.ts
import { 
  formatCurrency, 
  validateEmail, 
  debounce, 
  deepMerge 
} from '@/lib/utils';

describe('Utility Functions', () => {
  describe('formatCurrency', () => {
    test('formats positive numbers', () => {
      expect(formatCurrency(1234.56)).toBe('$1,234.56');
      expect(formatCurrency(0)).toBe('$0.00');
    });

    test('formats negative numbers', () => {
      expect(formatCurrency(-1234.56)).toBe('-$1,234.56');
    });

    test('handles different currencies', () => {
      expect(formatCurrency(1234.56, 'EUR')).toBe('€1,234.56');
    });
  });

  describe('validateEmail', () => {
    test('validates correct email formats', () => {
      expect(validateEmail('test@example.com')).toBe(true);
      expect(validateEmail('user.name+tag@domain.co.uk')).toBe(true);
    });

    test('rejects invalid email formats', () => {
      expect(validateEmail('invalid-email')).toBe(false);
      expect(validateEmail('test@')).toBe(false);
      expect(validateEmail('@example.com')).toBe(false);
    });
  });

  describe('debounce', () => {
    jest.useFakeTimers();

    test('delays function execution', () => {
      const mockFn = jest.fn();
      const debouncedFn = debounce(mockFn, 1000);

      debouncedFn();
      expect(mockFn).not.toHaveBeenCalled();

      jest.advanceTimersByTime(1000);
      expect(mockFn).toHaveBeenCalledTimes(1);
    });

    test('cancels previous calls', () => {
      const mockFn = jest.fn();
      const debouncedFn = debounce(mockFn, 1000);

      debouncedFn();
      debouncedFn();
      debouncedFn();

      jest.advanceTimersByTime(1000);
      expect(mockFn).toHaveBeenCalledTimes(1);
    });
  });

  describe('deepMerge', () => {
    test('merges objects deeply', () => {
      const obj1 = { a: 1, b: { c: 2 } };
      const obj2 = { b: { d: 3 }, e: 4 };
      
      const result = deepMerge(obj1, obj2);
      
      expect(result).toEqual({
        a: 1,
        b: { c: 2, d: 3 },
        e: 4
      });
    });

    test('handles arrays', () => {
      const obj1 = { items: [1, 2] };
      const obj2 = { items: [3, 4] };
      
      const result = deepMerge(obj1, obj2);
      
      expect(result.items).toEqual([3, 4]);
    });
  });
});
```

## Snapshot Testing

### Component Snapshots

```typescript
// Button.snapshot.test.tsx
import { render } from '@/test/utils';
import { Button } from '@/components/ui/Button';

describe('Button Snapshots', () => {
  test('renders primary button', () => {
    const { container } = render(
      <Button variant="primary">Primary Button</Button>
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });

  test('renders secondary button', () => {
    const { container } = render(
      <Button variant="secondary">Secondary Button</Button>
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });

  test('renders loading button', () => {
    const { container } = render(
      <Button loading loadingText="Loading...">Submit</Button>
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });

  test('renders disabled button', () => {
    const { container } = render(
      <Button disabled>Disabled Button</Button>
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

### Inline Snapshots

```typescript
// Card.test.tsx
import { render } from '@/test/utils';
import { Card } from '@/components/ui/Card';

describe('Card Component', () => {
  test('renders basic card structure', () => {
    const { container } = render(
      <Card title="Test Card" description="Test description">
        <p>Card content</p>
      </Card>
    );
    
    expect(container.firstChild).toMatchInlineSnapshot(`
      <div
        class="bg-white rounded-lg shadow-md p-6"
      >
        <div
          class="mb-4"
        >
          <h3
            class="text-lg font-semibold mb-2"
          >
            Test Card
          </h3>
          <p
            class="text-gray-600"
          >
            Test description
          </p>
        </div>
        <div>
          <p>
            Card content
          </p>
        </div>
      </div>
    `);
  });
});
```

## Mocking Strategies

### API Mocking

```typescript
// __mocks__/api.ts
export const api = {
  getUsers: jest.fn(),
  createUser: jest.fn(),
  updateUser: jest.fn(),
  deleteUser: jest.fn(),
  
  getPosts: jest.fn(),
  createPost: jest.fn(),
  updatePost: jest.fn(),
  deletePost: jest.fn()
};

// Test file using mocks
import { api } from '@/lib/api';
import { createMockUser } from '@/test/utils';

jest.mock('@/lib/api');
const mockApi = api as jest.Mocked<typeof api>;

describe('User Service', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('creates user successfully', async () => {
    const mockUser = createMockUser();
    mockApi.createUser.mockResolvedValue(mockUser);

    const result = await userService.createUser({
      name: 'Test User',
      email: 'test@example.com'
    });

    expect(result).toEqual(mockUser);
    expect(mockApi.createUser).toHaveBeenCalledWith({
      name: 'Test User',
      email: 'test@example.com'
    });
  });
});
```

### Module Mocking

```typescript
// Mock external dependencies
jest.mock('react-router-dom', () => ({
  ...jest.requireActual('react-router-dom'),
  useNavigate: () => jest.fn(),
  useParams: () => ({ id: '1' })
}));

jest.mock('@/hooks/useAuth', () => ({
  useAuth: () => ({
    user: { id: '1', name: 'Test User' },
    isAuthenticated: true,
    login: jest.fn(),
    logout: jest.fn()
  })
}));
```

## Test Coverage

### Coverage Configuration

```typescript
// Coverage reporting setup
const coverageConfig = {
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/test/**/*',
    '!src/**/*.stories.{ts,tsx}',
    '!src/main.tsx',
    '!src/vite-env.d.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  },
  coverageReporters: ['text', 'lcov', 'html']
};
```

### Coverage Analysis

```bash
# Generate coverage report
npm run test:coverage

# View coverage in browser
open coverage/lcov-report/index.html

# Coverage commands
npm run test:coverage:watch
npm run test:coverage:ci
```

## Best Practices

### Testing Guidelines

```typescript
// Testing best practices
export const testingBestPractices = {
  // Test structure
  structure: {
    arrange: 'Setup test data and conditions',
    act: 'Execute the code under test',
    assert: 'Verify the expected outcome'
  },
  
  // Test naming
  naming: {
    pattern: 'should [expected behavior] when [condition]',
    examples: [
      'should display error message when form is invalid',
      'should call onSubmit when form is submitted',
      'should disable button when loading'
    ]
  },
  
  // Test isolation
  isolation: {
    independent: 'Each test should be independent',
    cleanup: 'Clean up after each test',
    mocking: 'Mock external dependencies'
  },
  
  // Test coverage
  coverage: {
    target: '80% minimum coverage',
    focus: 'Critical business logic',
    avoid: 'Testing implementation details'
  }
} as const;
```

---

## Summary

This Unit Testing Implementation guide provides comprehensive testing strategies for DafnckMachine v3.1, ensuring high-quality, reliable code through thorough component testing, hook testing, utility function testing, and snapshot testing. The implementation follows modern testing practices with Jest and React Testing Library to maintain code quality and prevent regressions. 