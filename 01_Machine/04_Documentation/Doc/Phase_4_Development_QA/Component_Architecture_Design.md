# Component Architecture Design - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Component Architecture Design Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document defines the component architecture design for DafnckMachine v3.1, establishing patterns, hierarchies, and best practices for building scalable, maintainable, and reusable React components with TypeScript.

## Architecture Principles

### 1. Component Hierarchy
- **Atomic Design**: Components organized by complexity levels
- **Composition over Inheritance**: Favor component composition
- **Single Responsibility**: Each component has one clear purpose
- **Reusability**: Components designed for multiple contexts

### 2. Design Patterns
- **Container/Presentational**: Separate logic from presentation
- **Compound Components**: Related components working together
- **Render Props**: Flexible component behavior sharing
- **Higher-Order Components**: Cross-cutting concerns

### 3. Type Safety
- **Strict TypeScript**: Comprehensive type definitions
- **Interface Contracts**: Clear component APIs
- **Generic Components**: Type-safe reusable components
- **Runtime Validation**: PropTypes for development

## Component Hierarchy

### Atomic Level (Level 1)

#### Basic Elements
```typescript
// src/components/atoms/Button/Button.tsx
import React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/utils/cn';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
  loading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, loading, leftIcon, rightIcon, children, disabled, ...props }, ref) => {
    const isDisabled = disabled || loading;

    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        disabled={isDisabled}
        {...props}
      >
        {loading && (
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
          </svg>
        )}
        {leftIcon && !loading && <span className="mr-2">{leftIcon}</span>}
        {children}
        {rightIcon && <span className="ml-2">{rightIcon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

#### Input Components
```typescript
// src/components/atoms/Input/Input.tsx
import React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/utils/cn';

const inputVariants = cva(
  'flex w-full rounded-md border border-input bg-transparent px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50',
  {
    variants: {
      variant: {
        default: '',
        error: 'border-destructive focus-visible:ring-destructive',
        success: 'border-green-500 focus-visible:ring-green-500',
      },
      size: {
        default: 'h-10',
        sm: 'h-9',
        lg: 'h-11',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface InputProps
  extends React.InputHTMLAttributes<HTMLInputElement>,
    VariantProps<typeof inputVariants> {
  label?: string;
  error?: string;
  helperText?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ className, variant, size, label, error, helperText, leftIcon, rightIcon, ...props }, ref) => {
    const inputVariant = error ? 'error' : variant;

    return (
      <div className="space-y-2">
        {label && (
          <label className="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70">
            {label}
          </label>
        )}
        <div className="relative">
          {leftIcon && (
            <div className="absolute left-3 top-1/2 transform -translate-y-1/2 text-muted-foreground">
              {leftIcon}
            </div>
          )}
          <input
            className={cn(
              inputVariants({ variant: inputVariant, size, className }),
              leftIcon && 'pl-10',
              rightIcon && 'pr-10'
            )}
            ref={ref}
            {...props}
          />
          {rightIcon && (
            <div className="absolute right-3 top-1/2 transform -translate-y-1/2 text-muted-foreground">
              {rightIcon}
            </div>
          )}
        </div>
        {error && <p className="text-sm text-destructive">{error}</p>}
        {helperText && !error && <p className="text-sm text-muted-foreground">{helperText}</p>}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

### Molecular Level (Level 2)

#### Form Components
```typescript
// src/components/molecules/FormField/FormField.tsx
import React from 'react';
import { Input, InputProps } from '@/components/atoms/Input';
import { Label } from '@/components/atoms/Label';

export interface FormFieldProps extends Omit<InputProps, 'label'> {
  name: string;
  label: string;
  required?: boolean;
  description?: string;
}

export const FormField: React.FC<FormFieldProps> = ({
  name,
  label,
  required,
  description,
  error,
  ...inputProps
}) => {
  return (
    <div className="space-y-2">
      <Label htmlFor={name} className="text-sm font-medium">
        {label}
        {required && <span className="text-destructive ml-1">*</span>}
      </Label>
      <Input
        id={name}
        name={name}
        error={error}
        {...inputProps}
      />
      {description && !error && (
        <p className="text-sm text-muted-foreground">{description}</p>
      )}
    </div>
  );
};
```

#### Search Components
```typescript
// src/components/molecules/SearchBox/SearchBox.tsx
import React, { useState, useCallback } from 'react';
import { Input } from '@/components/atoms/Input';
import { Button } from '@/components/atoms/Button';
import { SearchIcon, XIcon } from 'lucide-react';
import { useDebounce } from '@/hooks/useDebounce';

export interface SearchBoxProps {
  placeholder?: string;
  onSearch: (query: string) => void;
  onClear?: () => void;
  debounceMs?: number;
  className?: string;
}

export const SearchBox: React.FC<SearchBoxProps> = ({
  placeholder = 'Search...',
  onSearch,
  onClear,
  debounceMs = 300,
  className,
}) => {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, debounceMs);

  React.useEffect(() => {
    onSearch(debouncedQuery);
  }, [debouncedQuery, onSearch]);

  const handleClear = useCallback(() => {
    setQuery('');
    onClear?.();
  }, [onClear]);

  return (
    <div className={className}>
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
        leftIcon={<SearchIcon className="h-4 w-4" />}
        rightIcon={
          query && (
            <Button
              variant="ghost"
              size="icon"
              onClick={handleClear}
              className="h-6 w-6"
            >
              <XIcon className="h-4 w-4" />
            </Button>
          )
        }
      />
    </div>
  );
};
```

### Organism Level (Level 3)

#### Data Table Component
```typescript
// src/components/organisms/DataTable/DataTable.tsx
import React from 'react';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/atoms/Table';
import { Button } from '@/components/atoms/Button';
import { SearchBox } from '@/components/molecules/SearchBox';
import { Pagination } from '@/components/molecules/Pagination';

export interface Column<T> {
  key: keyof T;
  header: string;
  render?: (value: any, row: T) => React.ReactNode;
  sortable?: boolean;
  width?: string;
}

export interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  loading?: boolean;
  searchable?: boolean;
  pagination?: {
    page: number;
    pageSize: number;
    total: number;
    onPageChange: (page: number) => void;
  };
  onSearch?: (query: string) => void;
  onSort?: (key: keyof T, direction: 'asc' | 'desc') => void;
  emptyMessage?: string;
}

export function DataTable<T extends Record<string, any>>({
  data,
  columns,
  loading,
  searchable,
  pagination,
  onSearch,
  onSort,
  emptyMessage = 'No data available',
}: DataTableProps<T>) {
  const [sortConfig, setSortConfig] = React.useState<{
    key: keyof T;
    direction: 'asc' | 'desc';
  } | null>(null);

  const handleSort = (key: keyof T) => {
    const direction = sortConfig?.key === key && sortConfig.direction === 'asc' ? 'desc' : 'asc';
    setSortConfig({ key, direction });
    onSort?.(key, direction);
  };

  if (loading) {
    return (
      <div className="space-y-4">
        {searchable && <div className="h-10 bg-muted animate-pulse rounded" />}
        <div className="space-y-2">
          {Array.from({ length: 5 }).map((_, i) => (
            <div key={i} className="h-12 bg-muted animate-pulse rounded" />
          ))}
        </div>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {searchable && onSearch && (
        <SearchBox onSearch={onSearch} className="max-w-sm" />
      )}
      
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            <TableRow>
              {columns.map((column) => (
                <TableHead
                  key={String(column.key)}
                  style={{ width: column.width }}
                  className={column.sortable ? 'cursor-pointer hover:bg-muted' : ''}
                  onClick={() => column.sortable && handleSort(column.key)}
                >
                  <div className="flex items-center space-x-2">
                    <span>{column.header}</span>
                    {column.sortable && sortConfig?.key === column.key && (
                      <span className="text-xs">
                        {sortConfig.direction === 'asc' ? '↑' : '↓'}
                      </span>
                    )}
                  </div>
                </TableHead>
              ))}
            </TableRow>
          </TableHeader>
          <TableBody>
            {data.length === 0 ? (
              <TableRow>
                <TableCell colSpan={columns.length} className="text-center py-8">
                  {emptyMessage}
                </TableCell>
              </TableRow>
            ) : (
              data.map((row, index) => (
                <TableRow key={index}>
                  {columns.map((column) => (
                    <TableCell key={String(column.key)}>
                      {column.render
                        ? column.render(row[column.key], row)
                        : String(row[column.key] || '')}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </div>

      {pagination && (
        <Pagination
          currentPage={pagination.page}
          totalPages={Math.ceil(pagination.total / pagination.pageSize)}
          onPageChange={pagination.onPageChange}
        />
      )}
    </div>
  );
}
```

#### Form Component
```typescript
// src/components/organisms/Form/Form.tsx
import React from 'react';
import { useForm, FormProvider, FieldValues, DefaultValues } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/atoms/Button';

export interface FormProps<T extends FieldValues> {
  schema: z.ZodSchema<T>;
  defaultValues?: DefaultValues<T>;
  onSubmit: (data: T) => void | Promise<void>;
  children: React.ReactNode;
  className?: string;
  submitText?: string;
  loading?: boolean;
}

export function Form<T extends FieldValues>({
  schema,
  defaultValues,
  onSubmit,
  children,
  className,
  submitText = 'Submit',
  loading,
}: FormProps<T>) {
  const methods = useForm<T>({
    resolver: zodResolver(schema),
    defaultValues,
  });

  const handleSubmit = async (data: T) => {
    try {
      await onSubmit(data);
    } catch (error) {
      console.error('Form submission error:', error);
    }
  };

  return (
    <FormProvider {...methods}>
      <form
        onSubmit={methods.handleSubmit(handleSubmit)}
        className={className}
      >
        <div className="space-y-6">
          {children}
          <Button
            type="submit"
            loading={loading}
            disabled={loading}
            className="w-full"
          >
            {submitText}
          </Button>
        </div>
      </form>
    </FormProvider>
  );
}
```

### Template Level (Level 4)

#### Page Layout Template
```typescript
// src/components/templates/PageLayout/PageLayout.tsx
import React from 'react';
import { Header } from '@/components/organisms/Header';
import { Sidebar } from '@/components/organisms/Sidebar';
import { Footer } from '@/components/organisms/Footer';
import { Breadcrumbs } from '@/components/molecules/Breadcrumbs';

export interface PageLayoutProps {
  children: React.ReactNode;
  title?: string;
  breadcrumbs?: Array<{ label: string; href?: string }>;
  sidebar?: boolean;
  className?: string;
}

export const PageLayout: React.FC<PageLayoutProps> = ({
  children,
  title,
  breadcrumbs,
  sidebar = true,
  className,
}) => {
  return (
    <div className="min-h-screen bg-background">
      <Header />
      
      <div className="flex">
        {sidebar && <Sidebar />}
        
        <main className={`flex-1 ${sidebar ? 'ml-64' : ''} ${className}`}>
          <div className="container mx-auto px-6 py-8">
            {breadcrumbs && (
              <Breadcrumbs items={breadcrumbs} className="mb-6" />
            )}
            
            {title && (
              <h1 className="text-3xl font-bold text-foreground mb-8">
                {title}
              </h1>
            )}
            
            {children}
          </div>
        </main>
      </div>
      
      <Footer />
    </div>
  );
};
```

## Component Patterns

### Compound Components Pattern

```typescript
// src/components/organisms/Accordion/Accordion.tsx
import React, { createContext, useContext, useState } from 'react';

interface AccordionContextType {
  openItems: string[];
  toggleItem: (value: string) => void;
  multiple?: boolean;
}

const AccordionContext = createContext<AccordionContextType | undefined>(undefined);

export interface AccordionProps {
  children: React.ReactNode;
  multiple?: boolean;
  defaultValue?: string | string[];
}

export const Accordion: React.FC<AccordionProps> & {
  Item: typeof AccordionItem;
  Trigger: typeof AccordionTrigger;
  Content: typeof AccordionContent;
} = ({ children, multiple = false, defaultValue = [] }) => {
  const [openItems, setOpenItems] = useState<string[]>(
    Array.isArray(defaultValue) ? defaultValue : [defaultValue]
  );

  const toggleItem = (value: string) => {
    setOpenItems(prev => {
      if (multiple) {
        return prev.includes(value)
          ? prev.filter(item => item !== value)
          : [...prev, value];
      } else {
        return prev.includes(value) ? [] : [value];
      }
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggleItem, multiple }}>
      <div className="space-y-2">{children}</div>
    </AccordionContext.Provider>
  );
};

const AccordionItem: React.FC<{ value: string; children: React.ReactNode }> = ({
  value,
  children,
}) => {
  return <div data-value={value}>{children}</div>;
};

const AccordionTrigger: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionTrigger must be used within Accordion');

  const value = React.useContext(AccordionItemContext);
  const isOpen = context.openItems.includes(value);

  return (
    <button
      onClick={() => context.toggleItem(value)}
      className="flex w-full items-center justify-between py-4 font-medium transition-all hover:underline"
    >
      {children}
      <span className={`transition-transform ${isOpen ? 'rotate-180' : ''}`}>
        ↓
      </span>
    </button>
  );
};

const AccordionContent: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionContent must be used within Accordion');

  const value = React.useContext(AccordionItemContext);
  const isOpen = context.openItems.includes(value);

  return (
    <div
      className={`overflow-hidden transition-all ${
        isOpen ? 'max-h-screen pb-4' : 'max-h-0'
      }`}
    >
      {children}
    </div>
  );
};

Accordion.Item = AccordionItem;
Accordion.Trigger = AccordionTrigger;
Accordion.Content = AccordionContent;
```

### Render Props Pattern

```typescript
// src/components/utilities/DataFetcher/DataFetcher.tsx
import React from 'react';
import { useQuery } from '@tanstack/react-query';

export interface DataFetcherProps<T> {
  queryKey: string[];
  queryFn: () => Promise<T>;
  children: (props: {
    data: T | undefined;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

export function DataFetcher<T>({ queryKey, queryFn, children }: DataFetcherProps<T>) {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey,
    queryFn,
  });

  return (
    <>
      {children({
        data,
        loading: isLoading,
        error: error as Error | null,
        refetch,
      })}
    </>
  );
}

// Usage
<DataFetcher
  queryKey={['users']}
  queryFn={() => fetchUsers()}
>
  {({ data, loading, error }) => {
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;
    return <UserList users={data} />;
  }}
</DataFetcher>
```

### Higher-Order Component Pattern

```typescript
// src/components/hoc/withAuth.tsx
import React from 'react';
import { useAuth } from '@/hooks/useAuth';
import { Navigate } from 'react-router-dom';

export interface WithAuthOptions {
  requiredRoles?: string[];
  redirectTo?: string;
}

export function withAuth<P extends object>(
  Component: React.ComponentType<P>,
  options: WithAuthOptions = {}
) {
  const { requiredRoles = [], redirectTo = '/login' } = options;

  return function AuthenticatedComponent(props: P) {
    const { isAuthenticated, user, isLoading } = useAuth();

    if (isLoading) {
      return <div>Loading...</div>;
    }

    if (!isAuthenticated) {
      return <Navigate to={redirectTo} replace />;
    }

    if (requiredRoles.length > 0 && user) {
      const hasRequiredRole = requiredRoles.some(role =>
        user.roles.includes(role)
      );
      
      if (!hasRequiredRole) {
        return <Navigate to="/unauthorized" replace />;
      }
    }

    return <Component {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard, {
  requiredRoles: ['admin', 'user'],
});
```

## Component Composition

### Flexible Component APIs

```typescript
// src/components/molecules/Card/Card.tsx
import React from 'react';
import { cn } from '@/utils/cn';

export interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: 'default' | 'outlined' | 'elevated';
  padding?: 'none' | 'sm' | 'md' | 'lg';
}

export const Card: React.FC<CardProps> & {
  Header: typeof CardHeader;
  Body: typeof CardBody;
  Footer: typeof CardFooter;
} = ({ 
  children, 
  variant = 'default', 
  padding = 'md',
  className,
  ...props 
}) => {
  return (
    <div
      className={cn(
        'rounded-lg border bg-card text-card-foreground shadow-sm',
        {
          'border-border': variant === 'outlined',
          'shadow-lg': variant === 'elevated',
          'p-0': padding === 'none',
          'p-4': padding === 'sm',
          'p-6': padding === 'md',
          'p-8': padding === 'lg',
        },
        className
      )}
      {...props}
    >
      {children}
    </div>
  );
};

const CardHeader: React.FC<React.HTMLAttributes<HTMLDivElement>> = ({
  children,
  className,
  ...props
}) => (
  <div className={cn('flex flex-col space-y-1.5 p-6', className)} {...props}>
    {children}
  </div>
);

const CardBody: React.FC<React.HTMLAttributes<HTMLDivElement>> = ({
  children,
  className,
  ...props
}) => (
  <div className={cn('p-6 pt-0', className)} {...props}>
    {children}
  </div>
);

const CardFooter: React.FC<React.HTMLAttributes<HTMLDivElement>> = ({
  children,
  className,
  ...props
}) => (
  <div className={cn('flex items-center p-6 pt-0', className)} {...props}>
    {children}
  </div>
);

Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;
```

## Type Safety Patterns

### Generic Component Types

```typescript
// src/components/molecules/Select/Select.tsx
import React from 'react';

export interface Option<T = string> {
  value: T;
  label: string;
  disabled?: boolean;
}

export interface SelectProps<T> {
  options: Option<T>[];
  value?: T;
  onChange: (value: T) => void;
  placeholder?: string;
  disabled?: boolean;
  multiple?: boolean;
  searchable?: boolean;
}

export function Select<T = string>({
  options,
  value,
  onChange,
  placeholder = 'Select an option...',
  disabled = false,
  multiple = false,
  searchable = false,
}: SelectProps<T>) {
  // Implementation details...
  return (
    <div>
      {/* Select implementation */}
    </div>
  );
}

// Usage with type safety
const stringSelect = (
  <Select<string>
    options={[
      { value: 'option1', label: 'Option 1' },
      { value: 'option2', label: 'Option 2' },
    ]}
    onChange={(value: string) => console.log(value)}
  />
);

const numberSelect = (
  <Select<number>
    options={[
      { value: 1, label: 'One' },
      { value: 2, label: 'Two' },
    ]}
    onChange={(value: number) => console.log(value)}
  />
);
```

## Performance Optimization

### Memoization Patterns

```typescript
// src/components/organisms/VirtualizedList/VirtualizedList.tsx
import React, { memo, useMemo } from 'react';
import { FixedSizeList as List } from 'react-window';

export interface VirtualizedListProps<T> {
  items: T[];
  height: number;
  itemHeight: number;
  renderItem: (props: { index: number; style: React.CSSProperties; data: T }) => React.ReactNode;
  className?: string;
}

export const VirtualizedList = memo(<T,>({
  items,
  height,
  itemHeight,
  renderItem,
  className,
}: VirtualizedListProps<T>) => {
  const ItemRenderer = useMemo(() => {
    return ({ index, style }: { index: number; style: React.CSSProperties }) => (
      <div style={style}>
        {renderItem({ index, style, data: items[index] })}
      </div>
    );
  }, [items, renderItem]);

  return (
    <List
      className={className}
      height={height}
      itemCount={items.length}
      itemSize={itemHeight}
      itemData={items}
    >
      {ItemRenderer}
    </List>
  );
}) as <T>(props: VirtualizedListProps<T>) => JSX.Element;
```

## Testing Patterns

### Component Testing Utilities

```typescript
// src/test/componentTestUtils.tsx
import React from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  queryClient?: QueryClient;
  initialEntries?: string[];
}

export function renderWithProviders(
  ui: React.ReactElement,
  {
    queryClient = createTestQueryClient(),
    initialEntries = ['/'],
    ...renderOptions
  }: CustomRenderOptions = {}
) {
  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        <BrowserRouter>
          {children}
        </BrowserRouter>
      </QueryClientProvider>
    );
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
    queryClient,
  };
}

// Component test example
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '../Button';

describe('Button', () => {
  it('renders with correct text', () => {
    renderWithProviders(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    renderWithProviders(<Button onClick={handleClick}>Click me</Button>);
    
    await userEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## Best Practices

### 1. Component Design
- **Single Responsibility**: Each component has one clear purpose
- **Composition over Inheritance**: Build complex UIs through composition
- **Prop Interface Design**: Clear, minimal, and flexible APIs
- **Default Props**: Sensible defaults for optional props

### 2. Performance
- **Memoization**: Use React.memo for expensive components
- **Lazy Loading**: Load components on demand
- **Virtual Scrolling**: For large lists and tables
- **Bundle Splitting**: Separate component bundles

### 3. Accessibility
- **Semantic HTML**: Use appropriate HTML elements
- **ARIA Labels**: Provide accessible names and descriptions
- **Keyboard Navigation**: Support keyboard interactions
- **Focus Management**: Proper focus handling

### 4. Maintainability
- **Consistent Naming**: Follow naming conventions
- **Documentation**: Document component APIs
- **Testing**: Comprehensive component testing
- **Type Safety**: Strong TypeScript typing

This comprehensive component architecture design ensures scalable, maintainable, and reusable components for DafnckMachine v3.1. 