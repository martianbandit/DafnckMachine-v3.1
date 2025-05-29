# Page Implementation Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Page Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This guide provides comprehensive documentation for implementing pages in DafnckMachine v3.1, covering page structure, routing patterns, data management, SEO optimization, and performance considerations.

## Page Architecture

### Page Structure Hierarchy
```
src/pages/
├── auth/                    # Authentication pages
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   └── ForgotPasswordPage.tsx
├── dashboard/               # Dashboard pages
│   ├── DashboardPage.tsx
│   ├── AnalyticsPage.tsx
│   └── SettingsPage.tsx
├── public/                  # Public pages
│   ├── HomePage.tsx
│   ├── AboutPage.tsx
│   └── ContactPage.tsx
├── error/                   # Error pages
│   ├── NotFoundPage.tsx
│   └── ErrorBoundaryPage.tsx
└── shared/                  # Shared page components
    ├── layouts/
    └── templates/
```

### Page Component Pattern

```typescript
// src/pages/shared/types.ts
export interface PageProps {
  title?: string;
  description?: string;
  keywords?: string[];
  canonical?: string;
  noIndex?: boolean;
  className?: string;
}

export interface PageMetadata {
  title: string;
  description: string;
  keywords: string[];
  ogImage?: string;
  canonical?: string;
  noIndex?: boolean;
  structuredData?: Record<string, any>;
}

export interface PageState {
  loading: boolean;
  error: string | null;
  data: any;
}
```

### Base Page Component

```typescript
// src/pages/shared/BasePage.tsx
import React, { useEffect } from 'react';
import { Helmet } from 'react-helmet-async';
import { useLocation } from 'react-router-dom';
import { PageProps, PageMetadata } from './types';
import { ErrorBoundary } from '@/components/molecules/ErrorBoundary';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';
import { cn } from '@/utils/cn';

interface BasePageProps extends PageProps {
  children: React.ReactNode;
  metadata?: PageMetadata;
  loading?: boolean;
  error?: string | null;
  layout?: 'default' | 'auth' | 'dashboard' | 'minimal';
  breadcrumbs?: Array<{ label: string; href?: string }>;
}

export const BasePage: React.FC<BasePageProps> = ({
  children,
  metadata,
  loading = false,
  error = null,
  layout = 'default',
  breadcrumbs,
  className,
}) => {
  const location = useLocation();

  // Update page metadata
  useEffect(() => {
    if (metadata?.title) {
      document.title = `${metadata.title} | DafnckMachine`;
    }
  }, [metadata?.title]);

  // Generate structured data
  const generateStructuredData = () => {
    if (!metadata?.structuredData) return null;

    return (
      <script type="application/ld+json">
        {JSON.stringify(metadata.structuredData)}
      </script>
    );
  };

  if (loading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <LoadingSpinner size="lg" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-center">
          <h1 className="text-2xl font-bold text-destructive mb-2">Error</h1>
          <p className="text-muted-foreground">{error}</p>
        </div>
      </div>
    );
  }

  return (
    <ErrorBoundary>
      {/* SEO Metadata */}
      <Helmet>
        {metadata?.title && <title>{metadata.title} | DafnckMachine</title>}
        {metadata?.description && <meta name="description" content={metadata.description} />}
        {metadata?.keywords && <meta name="keywords" content={metadata.keywords.join(', ')} />}
        {metadata?.canonical && <link rel="canonical" href={metadata.canonical} />}
        {metadata?.noIndex && <meta name="robots" content="noindex, nofollow" />}
        
        {/* Open Graph */}
        {metadata?.title && <meta property="og:title" content={metadata.title} />}
        {metadata?.description && <meta property="og:description" content={metadata.description} />}
        {metadata?.ogImage && <meta property="og:image" content={metadata.ogImage} />}
        <meta property="og:url" content={window.location.href} />
        <meta property="og:type" content="website" />
        
        {/* Twitter Card */}
        <meta name="twitter:card" content="summary_large_image" />
        {metadata?.title && <meta name="twitter:title" content={metadata.title} />}
        {metadata?.description && <meta name="twitter:description" content={metadata.description} />}
        {metadata?.ogImage && <meta name="twitter:image" content={metadata.ogImage} />}
        
        {/* Structured Data */}
        {generateStructuredData()}
      </Helmet>

      {/* Page Content */}
      <div className={cn('min-h-screen', className)}>
        {/* Breadcrumbs */}
        {breadcrumbs && breadcrumbs.length > 0 && (
          <nav className="bg-muted/30 border-b px-6 py-3">
            <ol className="flex items-center space-x-2 text-sm">
              {breadcrumbs.map((crumb, index) => (
                <li key={index} className="flex items-center">
                  {index > 0 && <span className="mx-2 text-muted-foreground">/</span>}
                  {crumb.href ? (
                    <a href={crumb.href} className="text-primary hover:underline">
                      {crumb.label}
                    </a>
                  ) : (
                    <span className="text-muted-foreground">{crumb.label}</span>
                  )}
                </li>
              ))}
            </ol>
          </nav>
        )}

        {/* Main Content */}
        <main role="main">
          {children}
        </main>
      </div>
    </ErrorBoundary>
  );
};
```

## Page Implementation Patterns

### 1. Dashboard Page Pattern

```typescript
// src/pages/dashboard/DashboardPage.tsx
import React, { useState, useEffect } from 'react';
import { BasePage } from '@/pages/shared/BasePage';
import { Dashboard } from '@/components/complex/Dashboard/Dashboard';
import { useAuth } from '@/hooks/useAuth';
import { useDashboardData } from '@/hooks/useDashboardData';
import { DashboardLayout } from '@/layouts/DashboardLayout';

export const DashboardPage: React.FC = () => {
  const { user } = useAuth();
  const { data, loading, error, refetch } = useDashboardData();

  const metadata = {
    title: 'Dashboard',
    description: 'Your personal dashboard with key metrics and insights',
    keywords: ['dashboard', 'analytics', 'metrics'],
    noIndex: true, // Private page
  };

  const breadcrumbs = [
    { label: 'Home', href: '/' },
    { label: 'Dashboard' },
  ];

  const dashboardWidgets = [
    {
      id: 'metrics',
      title: 'Key Metrics',
      type: 'metric' as const,
      size: 'md' as const,
      data: {
        value: data?.totalUsers || 0,
        change: data?.userGrowth || 0,
        icon: '👥',
      },
    },
    {
      id: 'revenue',
      title: 'Revenue',
      type: 'metric' as const,
      size: 'md' as const,
      data: {
        value: `$${data?.revenue || 0}`,
        change: data?.revenueGrowth || 0,
        icon: '💰',
      },
    },
    {
      id: 'tasks',
      title: 'Task Progress',
      type: 'progress' as const,
      size: 'lg' as const,
      data: {
        label: 'Completed Tasks',
        current: data?.completedTasks || 0,
        total: data?.totalTasks || 0,
      },
    },
    {
      id: 'recent-activity',
      title: 'Recent Activity',
      type: 'list' as const,
      size: 'lg' as const,
      data: {
        items: data?.recentActivity || [],
      },
    },
  ];

  return (
    <DashboardLayout>
      <BasePage
        metadata={metadata}
        loading={loading}
        error={error}
        breadcrumbs={breadcrumbs}
        className="p-6"
      >
        <div className="space-y-6">
          <div className="flex items-center justify-between">
            <div>
              <h1 className="text-3xl font-bold">Welcome back, {user?.name}!</h1>
              <p className="text-muted-foreground">
                Here's what's happening with your projects today.
              </p>
            </div>
            <button
              onClick={refetch}
              className="px-4 py-2 bg-primary text-primary-foreground rounded-md hover:bg-primary/90"
            >
              Refresh
            </button>
          </div>

          <Dashboard
            widgets={dashboardWidgets}
            layout="grid"
          />
        </div>
      </BasePage>
    </DashboardLayout>
  );
};
```

### 2. Form Page Pattern

```typescript
// src/pages/auth/RegisterPage.tsx
import React, { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import { BasePage } from '@/pages/shared/BasePage';
import { MultiStepForm } from '@/components/complex/MultiStepForm/MultiStepForm';
import { AuthLayout } from '@/layouts/AuthLayout';
import { useAuth } from '@/hooks/useAuth';
import { validateEmail, validatePassword } from '@/utils/validation';

// Step Components
const PersonalInfoStep: React.FC<any> = ({ data, onDataChange, isValid }) => {
  const [formData, setFormData] = useState(data);

  const handleChange = (field: string, value: string) => {
    const newData = { ...formData, [field]: value };
    setFormData(newData);
    onDataChange(newData);
  };

  return (
    <div className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-2">First Name</label>
        <input
          type="text"
          value={formData.firstName || ''}
          onChange={(e) => handleChange('firstName', e.target.value)}
          className="w-full px-3 py-2 border rounded-md"
          required
        />
      </div>
      <div>
        <label className="block text-sm font-medium mb-2">Last Name</label>
        <input
          type="text"
          value={formData.lastName || ''}
          onChange={(e) => handleChange('lastName', e.target.value)}
          className="w-full px-3 py-2 border rounded-md"
          required
        />
      </div>
      <div>
        <label className="block text-sm font-medium mb-2">Email</label>
        <input
          type="email"
          value={formData.email || ''}
          onChange={(e) => handleChange('email', e.target.value)}
          className="w-full px-3 py-2 border rounded-md"
          required
        />
      </div>
    </div>
  );
};

const AccountSetupStep: React.FC<any> = ({ data, onDataChange }) => {
  const [formData, setFormData] = useState(data);

  const handleChange = (field: string, value: string) => {
    const newData = { ...formData, [field]: value };
    setFormData(newData);
    onDataChange(newData);
  };

  return (
    <div className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-2">Password</label>
        <input
          type="password"
          value={formData.password || ''}
          onChange={(e) => handleChange('password', e.target.value)}
          className="w-full px-3 py-2 border rounded-md"
          required
        />
      </div>
      <div>
        <label className="block text-sm font-medium mb-2">Confirm Password</label>
        <input
          type="password"
          value={formData.confirmPassword || ''}
          onChange={(e) => handleChange('confirmPassword', e.target.value)}
          className="w-full px-3 py-2 border rounded-md"
          required
        />
      </div>
    </div>
  );
};

export const RegisterPage: React.FC = () => {
  const navigate = useNavigate();
  const { register } = useAuth();
  const [isSubmitting, setIsSubmitting] = useState(false);

  const metadata = {
    title: 'Create Account',
    description: 'Join DafnckMachine and start building amazing projects',
    keywords: ['register', 'signup', 'account', 'join'],
  };

  const formSteps = [
    {
      id: 'personal',
      title: 'Personal Information',
      description: 'Tell us about yourself',
      component: PersonalInfoStep,
      validation: (data: any) => {
        return data.firstName && data.lastName && validateEmail(data.email);
      },
    },
    {
      id: 'account',
      title: 'Account Setup',
      description: 'Create your secure account',
      component: AccountSetupStep,
      validation: (data: any) => {
        return validatePassword(data.password) && data.password === data.confirmPassword;
      },
    },
  ];

  const handleComplete = async (formData: any) => {
    setIsSubmitting(true);
    try {
      await register({
        firstName: formData.personal.firstName,
        lastName: formData.personal.lastName,
        email: formData.personal.email,
        password: formData.account.password,
      });
      navigate('/dashboard');
    } catch (error) {
      console.error('Registration failed:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <AuthLayout>
      <BasePage metadata={metadata} className="min-h-screen flex items-center justify-center">
        <div className="w-full max-w-md space-y-6">
          <div className="text-center">
            <h1 className="text-2xl font-bold">Create your account</h1>
            <p className="text-muted-foreground mt-2">
              Already have an account?{' '}
              <Link to="/auth/login" className="text-primary hover:underline">
                Sign in
              </Link>
            </p>
          </div>

          <MultiStepForm
            steps={formSteps}
            onComplete={handleComplete}
            onCancel={() => navigate('/')}
            showProgress={true}
            showStepNumbers={true}
          />
        </div>
      </BasePage>
    </AuthLayout>
  );
};
```

### 3. Data List Page Pattern

```typescript
// src/pages/admin/UsersPage.tsx
import React, { useState, useMemo } from 'react';
import { BasePage } from '@/pages/shared/BasePage';
import { DataTable } from '@/components/complex/DataTable/DataTable';
import { Button } from '@/components/atoms/Button';
import { Badge } from '@/components/atoms/Badge';
import { AdminLayout } from '@/layouts/AdminLayout';
import { useUsers } from '@/hooks/useUsers';
import { User } from '@/types/user';

export const UsersPage: React.FC = () => {
  const { data: users, loading, error, refetch } = useUsers();
  const [selectedUsers, setSelectedUsers] = useState<User[]>([]);

  const metadata = {
    title: 'User Management',
    description: 'Manage users and their permissions',
    keywords: ['users', 'management', 'admin'],
    noIndex: true,
  };

  const breadcrumbs = [
    { label: 'Admin', href: '/admin' },
    { label: 'Users' },
  ];

  const columns = useMemo(() => [
    {
      id: 'name',
      header: 'Name',
      accessorKey: 'name' as keyof User,
      sortable: true,
      filterable: true,
    },
    {
      id: 'email',
      header: 'Email',
      accessorKey: 'email' as keyof User,
      sortable: true,
      filterable: true,
    },
    {
      id: 'role',
      header: 'Role',
      accessorKey: 'role' as keyof User,
      cell: ({ value }: { value: string }) => (
        <Badge variant={value === 'admin' ? 'destructive' : 'default'}>
          {value}
        </Badge>
      ),
      sortable: true,
      filterable: true,
    },
    {
      id: 'status',
      header: 'Status',
      accessorKey: 'status' as keyof User,
      cell: ({ value }: { value: string }) => (
        <Badge variant={value === 'active' ? 'success' : 'secondary'}>
          {value}
        </Badge>
      ),
      sortable: true,
      filterable: true,
    },
    {
      id: 'createdAt',
      header: 'Created',
      accessorKey: 'createdAt' as keyof User,
      cell: ({ value }: { value: string }) => (
        new Date(value).toLocaleDateString()
      ),
      sortable: true,
    },
    {
      id: 'actions',
      header: 'Actions',
      cell: ({ row }: { row: User }) => (
        <div className="flex space-x-2">
          <Button size="sm" variant="outline">
            Edit
          </Button>
          <Button size="sm" variant="destructive">
            Delete
          </Button>
        </div>
      ),
    },
  ], []);

  const handleSelectionChange = (selected: User[]) => {
    setSelectedUsers(selected);
  };

  const handleBulkAction = (action: string) => {
    console.log(`Performing ${action} on`, selectedUsers);
  };

  return (
    <AdminLayout>
      <BasePage
        metadata={metadata}
        loading={loading}
        error={error}
        breadcrumbs={breadcrumbs}
        className="p-6"
      >
        <div className="space-y-6">
          <div className="flex items-center justify-between">
            <div>
              <h1 className="text-3xl font-bold">Users</h1>
              <p className="text-muted-foreground">
                Manage user accounts and permissions
              </p>
            </div>
            <div className="flex space-x-2">
              <Button variant="outline" onClick={refetch}>
                Refresh
              </Button>
              <Button>
                Add User
              </Button>
            </div>
          </div>

          {selectedUsers.length > 0 && (
            <div className="flex items-center space-x-2 p-4 bg-muted rounded-md">
              <span className="text-sm font-medium">
                {selectedUsers.length} user(s) selected
              </span>
              <Button size="sm" onClick={() => handleBulkAction('activate')}>
                Activate
              </Button>
              <Button size="sm" onClick={() => handleBulkAction('deactivate')}>
                Deactivate
              </Button>
              <Button size="sm" variant="destructive" onClick={() => handleBulkAction('delete')}>
                Delete
              </Button>
            </div>
          )}

          <DataTable
            data={users || []}
            columns={columns}
            loading={loading}
            error={error}
            pagination={{
              pageSize: 20,
              showSizeSelector: true,
              showInfo: true,
            }}
            selection={{
              enabled: true,
              onSelectionChange: handleSelectionChange,
            }}
            sorting={{
              enabled: true,
              defaultSort: [{ id: 'createdAt', desc: true }],
            }}
            filtering={{
              enabled: true,
              globalFilter: true,
              columnFilters: true,
            }}
            actions={{
              onRowClick: (user) => console.log('Row clicked:', user),
              onRowDoubleClick: (user) => console.log('Row double-clicked:', user),
            }}
          />
        </div>
      </BasePage>
    </AdminLayout>
  );
};
```

## SEO and Performance Optimization

### SEO Best Practices

```typescript
// src/utils/seo.ts
export interface SEOConfig {
  title: string;
  description: string;
  keywords: string[];
  canonical?: string;
  ogImage?: string;
  noIndex?: boolean;
  structuredData?: Record<string, any>;
}

export const generatePageMetadata = (config: SEOConfig) => {
  return {
    title: `${config.title} | DafnckMachine`,
    description: config.description,
    keywords: config.keywords,
    canonical: config.canonical || window.location.href,
    ogImage: config.ogImage || '/images/og-default.jpg',
    noIndex: config.noIndex || false,
    structuredData: config.structuredData,
  };
};

export const generateStructuredData = (type: string, data: any) => {
  const baseStructure = {
    '@context': 'https://schema.org',
    '@type': type,
    ...data,
  };

  return baseStructure;
};

// Example usage for different page types
export const generateBreadcrumbStructuredData = (breadcrumbs: Array<{ label: string; href?: string }>) => {
  return generateStructuredData('BreadcrumbList', {
    itemListElement: breadcrumbs.map((crumb, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: crumb.label,
      item: crumb.href ? `${window.location.origin}${crumb.href}` : undefined,
    })),
  });
};
```

### Performance Optimization

```typescript
// src/hooks/usePagePerformance.ts
import { useEffect } from 'react';

export const usePagePerformance = (pageName: string) => {
  useEffect(() => {
    // Mark page load start
    performance.mark(`${pageName}-start`);

    // Track Core Web Vitals
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'largest-contentful-paint') {
          console.log('LCP:', entry.startTime);
        }
        if (entry.entryType === 'first-input') {
          console.log('FID:', entry.processingStart - entry.startTime);
        }
        if (entry.entryType === 'layout-shift') {
          console.log('CLS:', entry.value);
        }
      }
    });

    observer.observe({ entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift'] });

    return () => {
      // Mark page load end
      performance.mark(`${pageName}-end`);
      performance.measure(pageName, `${pageName}-start`, `${pageName}-end`);
      
      observer.disconnect();
    };
  }, [pageName]);
};
```

## Error Handling

### Error Boundary Implementation

```typescript
// src/pages/error/ErrorBoundaryPage.tsx
import React from 'react';
import { BasePage } from '@/pages/shared/BasePage';
import { Button } from '@/components/atoms/Button';
import { AlertTriangle, RefreshCw, Home } from 'lucide-react';

interface ErrorBoundaryPageProps {
  error?: Error;
  resetError?: () => void;
}

export const ErrorBoundaryPage: React.FC<ErrorBoundaryPageProps> = ({
  error,
  resetError,
}) => {
  const metadata = {
    title: 'Something went wrong',
    description: 'An unexpected error occurred',
    noIndex: true,
  };

  return (
    <BasePage metadata={metadata}>
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="text-center space-y-6 max-w-md">
          <div className="flex justify-center">
            <AlertTriangle className="h-16 w-16 text-destructive" />
          </div>
          
          <div>
            <h1 className="text-2xl font-bold mb-2">Something went wrong</h1>
            <p className="text-muted-foreground">
              We're sorry, but something unexpected happened. Please try again.
            </p>
          </div>

          {error && process.env.NODE_ENV === 'development' && (
            <div className="text-left bg-muted p-4 rounded-md">
              <h3 className="font-medium mb-2">Error Details:</h3>
              <pre className="text-sm text-destructive overflow-auto">
                {error.message}
              </pre>
            </div>
          )}

          <div className="flex justify-center space-x-4">
            {resetError && (
              <Button onClick={resetError} className="flex items-center space-x-2">
                <RefreshCw className="h-4 w-4" />
                <span>Try Again</span>
              </Button>
            )}
            <Button variant="outline" onClick={() => window.location.href = '/'}>
              <Home className="h-4 w-4 mr-2" />
              Go Home
            </Button>
          </div>
        </div>
      </div>
    </BasePage>
  );
};
```

### 404 Not Found Page

```typescript
// src/pages/error/NotFoundPage.tsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BasePage } from '@/pages/shared/BasePage';
import { Button } from '@/components/atoms/Button';
import { Search, Home, ArrowLeft } from 'lucide-react';

export const NotFoundPage: React.FC = () => {
  const metadata = {
    title: 'Page Not Found',
    description: 'The page you are looking for does not exist',
    noIndex: true,
  };

  return (
    <BasePage metadata={metadata}>
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="text-center space-y-6 max-w-md">
          <div>
            <h1 className="text-6xl font-bold text-muted-foreground mb-4">404</h1>
            <h2 className="text-2xl font-bold mb-2">Page Not Found</h2>
            <p className="text-muted-foreground">
              The page you are looking for might have been removed, had its name changed, 
              or is temporarily unavailable.
            </p>
          </div>

          <div className="flex flex-col sm:flex-row justify-center gap-4">
            <Button asChild>
              <Link to="/" className="flex items-center space-x-2">
                <Home className="h-4 w-4" />
                <span>Go Home</span>
              </Link>
            </Button>
            <Button variant="outline" onClick={() => window.history.back()}>
              <ArrowLeft className="h-4 w-4 mr-2" />
              Go Back
            </Button>
          </div>

          <div className="pt-6 border-t">
            <p className="text-sm text-muted-foreground mb-4">
              Looking for something specific?
            </p>
            <div className="flex">
              <input
                type="text"
                placeholder="Search..."
                className="flex-1 px-3 py-2 border rounded-l-md focus:outline-none focus:ring-2 focus:ring-primary"
              />
              <Button className="rounded-l-none">
                <Search className="h-4 w-4" />
              </Button>
            </div>
          </div>
        </div>
      </div>
    </BasePage>
  );
};
```

## Testing Strategies

### Page Testing Utilities

```typescript
// src/utils/testing/pageTestUtils.tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { HelmetProvider } from 'react-helmet-async';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

export const createTestQueryClient = () => {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  });
};

export const renderWithProviders = (
  ui: React.ReactElement,
  options: {
    queryClient?: QueryClient;
    initialEntries?: string[];
  } = {}
) => {
  const { queryClient = createTestQueryClient(), initialEntries = ['/'] } = options;

  const Wrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
    <HelmetProvider>
      <QueryClientProvider client={queryClient}>
        <BrowserRouter>
          {children}
        </BrowserRouter>
      </QueryClientProvider>
    </HelmetProvider>
  );

  return render(ui, { wrapper: Wrapper });
};

export const expectPageMetadata = (title: string, description?: string) => {
  expect(document.title).toContain(title);
  
  if (description) {
    const metaDescription = document.querySelector('meta[name="description"]');
    expect(metaDescription?.getAttribute('content')).toBe(description);
  }
};
```

### Example Page Tests

```typescript
// src/pages/__tests__/DashboardPage.test.tsx
import React from 'react';
import { screen, waitFor } from '@testing-library/react';
import { DashboardPage } from '../dashboard/DashboardPage';
import { renderWithProviders, expectPageMetadata } from '@/utils/testing/pageTestUtils';

// Mock hooks
jest.mock('@/hooks/useAuth', () => ({
  useAuth: () => ({
    user: { name: 'John Doe', email: 'john@example.com' },
    isAuthenticated: true,
  }),
}));

jest.mock('@/hooks/useDashboardData', () => ({
  useDashboardData: () => ({
    data: {
      totalUsers: 1250,
      userGrowth: 12.5,
      revenue: 45000,
      revenueGrowth: 8.3,
      completedTasks: 75,
      totalTasks: 100,
      recentActivity: [
        { title: 'New user registered', subtitle: '2 minutes ago' },
        { title: 'Task completed', subtitle: '5 minutes ago' },
      ],
    },
    loading: false,
    error: null,
    refetch: jest.fn(),
  }),
}));

describe('DashboardPage', () => {
  it('renders dashboard with user greeting', async () => {
    renderWithProviders(<DashboardPage />);

    await waitFor(() => {
      expect(screen.getByText('Welcome back, John Doe!')).toBeInTheDocument();
    });
  });

  it('sets correct page metadata', async () => {
    renderWithProviders(<DashboardPage />);

    await waitFor(() => {
      expectPageMetadata('Dashboard', 'Your personal dashboard with key metrics and insights');
    });
  });

  it('displays dashboard widgets', async () => {
    renderWithProviders(<DashboardPage />);

    await waitFor(() => {
      expect(screen.getByText('Key Metrics')).toBeInTheDocument();
      expect(screen.getByText('Revenue')).toBeInTheDocument();
      expect(screen.getByText('Task Progress')).toBeInTheDocument();
      expect(screen.getByText('Recent Activity')).toBeInTheDocument();
    });
  });

  it('shows loading state', async () => {
    // Mock loading state
    jest.doMock('@/hooks/useDashboardData', () => ({
      useDashboardData: () => ({
        data: null,
        loading: true,
        error: null,
        refetch: jest.fn(),
      }),
    }));

    renderWithProviders(<DashboardPage />);

    expect(screen.getByRole('status')).toBeInTheDocument(); // Loading spinner
  });
});
```

## Best Practices

### 1. Page Structure
- **Consistent Layout**: Use base page component for consistent structure
- **SEO Optimization**: Implement proper metadata and structured data
- **Error Handling**: Comprehensive error boundaries and fallback states
- **Performance**: Optimize loading states and data fetching

### 2. Data Management
- **Loading States**: Show appropriate loading indicators
- **Error States**: Handle and display errors gracefully
- **Caching**: Implement proper data caching strategies
- **Real-time Updates**: Handle real-time data updates when needed

### 3. User Experience
- **Navigation**: Clear breadcrumbs and navigation patterns
- **Responsive Design**: Mobile-first responsive implementation
- **Accessibility**: Full keyboard navigation and screen reader support
- **Performance**: Optimize Core Web Vitals and loading performance

### 4. Testing
- **Unit Tests**: Test page components and logic
- **Integration Tests**: Test page interactions and data flow
- **E2E Tests**: Test complete user workflows
- **Performance Tests**: Monitor and test page performance

This comprehensive page implementation guide provides the foundation for building consistent, performant, and user-friendly pages in DafnckMachine v3.1. 