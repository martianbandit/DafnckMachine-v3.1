# Routing Implementation Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Routing Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This guide provides comprehensive documentation for implementing routing in DafnckMachine v3.1 using React Router v6, covering route configuration, navigation patterns, authentication guards, lazy loading, and advanced routing features.

## Router Architecture

### Core Router Setup

```typescript
// src/router/index.tsx
import React, { Suspense } from 'react';
import { createBrowserRouter, RouterProvider, Outlet } from 'react-router-dom';
import { HelmetProvider } from 'react-helmet-async';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/providers/AuthProvider';
import { ThemeProvider } from '@/providers/ThemeProvider';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';
import { ErrorBoundary } from '@/components/molecules/ErrorBoundary';
import { ProtectedRoute } from './ProtectedRoute';
import { PublicRoute } from './PublicRoute';

// Lazy load pages for code splitting
const HomePage = React.lazy(() => import('@/pages/public/HomePage'));
const AboutPage = React.lazy(() => import('@/pages/public/AboutPage'));
const ContactPage = React.lazy(() => import('@/pages/public/ContactPage'));
const LoginPage = React.lazy(() => import('@/pages/auth/LoginPage'));
const RegisterPage = React.lazy(() => import('@/pages/auth/RegisterPage'));
const DashboardPage = React.lazy(() => import('@/pages/dashboard/DashboardPage'));
const ProfilePage = React.lazy(() => import('@/pages/dashboard/ProfilePage'));
const SettingsPage = React.lazy(() => import('@/pages/dashboard/SettingsPage'));
const AdminPage = React.lazy(() => import('@/pages/admin/AdminPage'));
const UsersPage = React.lazy(() => import('@/pages/admin/UsersPage'));
const NotFoundPage = React.lazy(() => import('@/pages/error/NotFoundPage'));

// Root layout component
const RootLayout: React.FC = () => {
  return (
    <HelmetProvider>
      <QueryClientProvider client={queryClient}>
        <AuthProvider>
          <ThemeProvider>
            <ErrorBoundary>
              <Suspense fallback={<LoadingSpinner size="lg" />}>
                <Outlet />
              </Suspense>
            </ErrorBoundary>
          </ThemeProvider>
        </AuthProvider>
      </QueryClientProvider>
    </HelmetProvider>
  );
};

// Create router configuration
export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <NotFoundPage />,
    children: [
      // Public routes
      {
        path: '',
        element: <PublicRoute />,
        children: [
          { index: true, element: <HomePage /> },
          { path: 'about', element: <AboutPage /> },
          { path: 'contact', element: <ContactPage /> },
        ],
      },
      
      // Authentication routes
      {
        path: 'auth',
        element: <PublicRoute redirectTo="/dashboard" />,
        children: [
          { path: 'login', element: <LoginPage /> },
          { path: 'register', element: <RegisterPage /> },
          { path: 'forgot-password', element: <ForgotPasswordPage /> },
          { path: 'reset-password/:token', element: <ResetPasswordPage /> },
        ],
      },
      
      // Protected dashboard routes
      {
        path: 'dashboard',
        element: <ProtectedRoute />,
        children: [
          { index: true, element: <DashboardPage /> },
          { path: 'profile', element: <ProfilePage /> },
          { path: 'settings', element: <SettingsPage /> },
          { path: 'projects', element: <ProjectsPage /> },
          { path: 'projects/:id', element: <ProjectDetailPage /> },
          { path: 'projects/:id/edit', element: <ProjectEditPage /> },
        ],
      },
      
      // Admin routes with role-based access
      {
        path: 'admin',
        element: <ProtectedRoute requiredRole="admin" />,
        children: [
          { index: true, element: <AdminPage /> },
          { path: 'users', element: <UsersPage /> },
          { path: 'users/:id', element: <UserDetailPage /> },
          { path: 'settings', element: <AdminSettingsPage /> },
          { path: 'analytics', element: <AdminAnalyticsPage /> },
        ],
      },
      
      // API documentation routes
      {
        path: 'docs',
        element: <DocsLayout />,
        children: [
          { index: true, element: <DocsHomePage /> },
          { path: 'api', element: <ApiDocsPage /> },
          { path: 'guides', element: <GuidesPage /> },
          { path: 'guides/:slug', element: <GuideDetailPage /> },
        ],
      },
      
      // Catch-all route for 404
      { path: '*', element: <NotFoundPage /> },
    ],
  },
]);

// Router provider component
export const AppRouter: React.FC = () => {
  return <RouterProvider router={router} />;
};
```

### Route Protection Components

```typescript
// src/router/ProtectedRoute.tsx
import React from 'react';
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';
import { DashboardLayout } from '@/layouts/DashboardLayout';
import { AdminLayout } from '@/layouts/AdminLayout';

interface ProtectedRouteProps {
  requiredRole?: string;
  requiredPermissions?: string[];
  layout?: 'dashboard' | 'admin' | 'minimal';
  redirectTo?: string;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  requiredRole,
  requiredPermissions = [],
  layout = 'dashboard',
  redirectTo = '/auth/login',
}) => {
  const { user, isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  // Show loading spinner while checking authentication
  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <LoadingSpinner size="lg" />
      </div>
    );
  }

  // Redirect to login if not authenticated
  if (!isAuthenticated) {
    return <Navigate to={redirectTo} state={{ from: location }} replace />;
  }

  // Check role-based access
  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/dashboard" replace />;
  }

  // Check permission-based access
  if (requiredPermissions.length > 0) {
    const hasPermissions = requiredPermissions.every(permission =>
      user?.permissions?.includes(permission)
    );
    
    if (!hasPermissions) {
      return <Navigate to="/dashboard" replace />;
    }
  }

  // Render with appropriate layout
  const renderWithLayout = () => {
    switch (layout) {
      case 'admin':
        return (
          <AdminLayout>
            <Outlet />
          </AdminLayout>
        );
      case 'minimal':
        return <Outlet />;
      case 'dashboard':
      default:
        return (
          <DashboardLayout>
            <Outlet />
          </DashboardLayout>
        );
    }
  };

  return renderWithLayout();
};
```

```typescript
// src/router/PublicRoute.tsx
import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { PublicLayout } from '@/layouts/PublicLayout';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';

interface PublicRouteProps {
  redirectTo?: string;
  layout?: 'public' | 'auth' | 'minimal';
}

export const PublicRoute: React.FC<PublicRouteProps> = ({
  redirectTo,
  layout = 'public',
}) => {
  const { isAuthenticated, isLoading } = useAuth();

  // Show loading spinner while checking authentication
  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <LoadingSpinner size="lg" />
      </div>
    );
  }

  // Redirect authenticated users if specified
  if (isAuthenticated && redirectTo) {
    return <Navigate to={redirectTo} replace />;
  }

  // Render with appropriate layout
  const renderWithLayout = () => {
    switch (layout) {
      case 'auth':
        return (
          <AuthLayout>
            <Outlet />
          </AuthLayout>
        );
      case 'minimal':
        return <Outlet />;
      case 'public':
      default:
        return (
          <PublicLayout>
            <Outlet />
          </PublicLayout>
        );
    }
  };

  return renderWithLayout();
};
```

## Navigation Components

### Navigation Hook

```typescript
// src/hooks/useNavigation.ts
import { useNavigate, useLocation, useParams, useSearchParams } from 'react-router-dom';
import { useCallback } from 'react';

export interface NavigationOptions {
  replace?: boolean;
  state?: any;
  preserveQuery?: boolean;
}

export const useNavigation = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const params = useParams();
  const [searchParams, setSearchParams] = useSearchParams();

  // Enhanced navigation with options
  const navigateTo = useCallback((
    to: string,
    options: NavigationOptions = {}
  ) => {
    const { replace = false, state, preserveQuery = false } = options;
    
    let targetPath = to;
    
    // Preserve query parameters if requested
    if (preserveQuery && searchParams.toString()) {
      const separator = to.includes('?') ? '&' : '?';
      targetPath = `${to}${separator}${searchParams.toString()}`;
    }
    
    navigate(targetPath, { replace, state });
  }, [navigate, searchParams]);

  // Go back with fallback
  const goBack = useCallback((fallback = '/') => {
    if (window.history.length > 1) {
      navigate(-1);
    } else {
      navigate(fallback);
    }
  }, [navigate]);

  // Update query parameters
  const updateQuery = useCallback((
    updates: Record<string, string | null>,
    options: { replace?: boolean } = {}
  ) => {
    const newParams = new URLSearchParams(searchParams);
    
    Object.entries(updates).forEach(([key, value]) => {
      if (value === null) {
        newParams.delete(key);
      } else {
        newParams.set(key, value);
      }
    });
    
    setSearchParams(newParams, { replace: options.replace });
  }, [searchParams, setSearchParams]);

  // Get query parameter value
  const getQuery = useCallback((key: string, defaultValue?: string) => {
    return searchParams.get(key) || defaultValue;
  }, [searchParams]);

  // Check if current path matches
  const isCurrentPath = useCallback((path: string) => {
    return location.pathname === path;
  }, [location.pathname]);

  // Check if current path starts with
  const isCurrentSection = useCallback((section: string) => {
    return location.pathname.startsWith(section);
  }, [location.pathname]);

  return {
    // Navigation functions
    navigateTo,
    goBack,
    
    // Query parameter functions
    updateQuery,
    getQuery,
    
    // Path checking functions
    isCurrentPath,
    isCurrentSection,
    
    // Router hooks
    location,
    params,
    searchParams,
  };
};
```

### Breadcrumb Component

```typescript
// src/components/navigation/Breadcrumb.tsx
import React from 'react';
import { Link, useLocation } from 'react-router-dom';
import { ChevronRight, Home } from 'lucide-react';
import { cn } from '@/utils/cn';

export interface BreadcrumbItem {
  label: string;
  href?: string;
  icon?: React.ReactNode;
}

interface BreadcrumbProps {
  items?: BreadcrumbItem[];
  showHome?: boolean;
  separator?: React.ReactNode;
  className?: string;
}

export const Breadcrumb: React.FC<BreadcrumbProps> = ({
  items,
  showHome = true,
  separator = <ChevronRight className="h-4 w-4" />,
  className,
}) => {
  const location = useLocation();

  // Auto-generate breadcrumbs from current path if items not provided
  const breadcrumbItems = items || generateBreadcrumbsFromPath(location.pathname);

  const allItems = showHome
    ? [{ label: 'Home', href: '/', icon: <Home className="h-4 w-4" /> }, ...breadcrumbItems]
    : breadcrumbItems;

  if (allItems.length <= 1) {
    return null;
  }

  return (
    <nav aria-label="Breadcrumb" className={cn('flex items-center space-x-2', className)}>
      <ol className="flex items-center space-x-2">
        {allItems.map((item, index) => {
          const isLast = index === allItems.length - 1;
          
          return (
            <li key={index} className="flex items-center">
              {index > 0 && (
                <span className="mx-2 text-muted-foreground">
                  {separator}
                </span>
              )}
              
              {item.href && !isLast ? (
                <Link
                  to={item.href}
                  className="flex items-center space-x-1 text-sm text-primary hover:text-primary/80 transition-colors"
                >
                  {item.icon}
                  <span>{item.label}</span>
                </Link>
              ) : (
                <span className="flex items-center space-x-1 text-sm text-muted-foreground">
                  {item.icon}
                  <span>{item.label}</span>
                </span>
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
};

// Helper function to generate breadcrumbs from path
const generateBreadcrumbsFromPath = (pathname: string): BreadcrumbItem[] => {
  const segments = pathname.split('/').filter(Boolean);
  const breadcrumbs: BreadcrumbItem[] = [];
  
  let currentPath = '';
  
  segments.forEach((segment, index) => {
    currentPath += `/${segment}`;
    
    // Convert segment to readable label
    const label = segment
      .split('-')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');
    
    // Don't link to the current page
    const href = index === segments.length - 1 ? undefined : currentPath;
    
    breadcrumbs.push({ label, href });
  });
  
  return breadcrumbs;
};
```

### Navigation Menu Component

```typescript
// src/components/navigation/NavigationMenu.tsx
import React from 'react';
import { Link, useLocation } from 'react-router-dom';
import { cn } from '@/utils/cn';

export interface NavigationItem {
  id: string;
  label: string;
  href: string;
  icon?: React.ReactNode;
  badge?: {
    text: string;
    variant?: 'default' | 'destructive' | 'success' | 'warning';
  };
  children?: NavigationItem[];
  requiredRole?: string;
  requiredPermissions?: string[];
}

interface NavigationMenuProps {
  items: NavigationItem[];
  orientation?: 'horizontal' | 'vertical';
  variant?: 'default' | 'pills' | 'underline';
  className?: string;
}

export const NavigationMenu: React.FC<NavigationMenuProps> = ({
  items,
  orientation = 'horizontal',
  variant = 'default',
  className,
}) => {
  const location = useLocation();

  const isActiveItem = (item: NavigationItem): boolean => {
    if (item.href === location.pathname) return true;
    if (item.children) {
      return item.children.some(child => isActiveItem(child));
    }
    return false;
  };

  const renderNavigationItem = (item: NavigationItem, level = 0) => {
    const isActive = isActiveItem(item);
    
    const itemClasses = cn(
      'flex items-center space-x-2 px-3 py-2 text-sm font-medium rounded-md transition-colors',
      {
        // Active states
        'bg-primary text-primary-foreground': isActive && variant === 'default',
        'bg-primary/10 text-primary': isActive && variant === 'pills',
        'border-b-2 border-primary text-primary': isActive && variant === 'underline',
        
        // Inactive states
        'text-muted-foreground hover:text-foreground hover:bg-accent': !isActive,
        
        // Indentation for nested items
        'ml-4': level > 0,
      }
    );

    return (
      <div key={item.id}>
        <Link to={item.href} className={itemClasses}>
          {item.icon && <span className="flex-shrink-0">{item.icon}</span>}
          <span>{item.label}</span>
          {item.badge && (
            <span className={cn(
              'inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium',
              {
                'bg-primary/20 text-primary': item.badge.variant === 'default',
                'bg-destructive/20 text-destructive': item.badge.variant === 'destructive',
                'bg-green-500/20 text-green-700': item.badge.variant === 'success',
                'bg-yellow-500/20 text-yellow-700': item.badge.variant === 'warning',
              }
            )}>
              {item.badge.text}
            </span>
          )}
        </Link>
        
        {item.children && (
          <div className="mt-1 space-y-1">
            {item.children.map(child => renderNavigationItem(child, level + 1))}
          </div>
        )}
      </div>
    );
  };

  return (
    <nav className={cn(
      'space-y-1',
      {
        'flex space-y-0 space-x-1': orientation === 'horizontal',
        'flex-col': orientation === 'vertical',
      },
      className
    )}>
      {items.map(item => renderNavigationItem(item))}
    </nav>
  );
};
```

## Advanced Routing Features

### Route-based Code Splitting

```typescript
// src/utils/routeUtils.tsx
import React, { Suspense } from 'react';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';

// Higher-order component for lazy loading with custom loading component
export const withLazyLoading = <P extends object>(
  Component: React.LazyExoticComponent<React.ComponentType<P>>,
  LoadingComponent: React.ComponentType = () => <LoadingSpinner size="lg" />
) => {
  return (props: P) => (
    <Suspense fallback={<LoadingComponent />}>
      <Component {...props} />
    </Suspense>
  );
};

// Route preloading utility
export const preloadRoute = (routeImport: () => Promise<any>) => {
  const componentImport = routeImport();
  return componentImport;
};

// Preload routes on hover or focus
export const useRoutePreloading = () => {
  const preloadOnHover = (routeImport: () => Promise<any>) => {
    return {
      onMouseEnter: () => preloadRoute(routeImport),
      onFocus: () => preloadRoute(routeImport),
    };
  };

  return { preloadOnHover };
};
```

### Route Transitions

```typescript
// src/components/navigation/RouteTransition.tsx
import React from 'react';
import { useLocation } from 'react-router-dom';
import { AnimatePresence, motion } from 'framer-motion';

interface RouteTransitionProps {
  children: React.ReactNode;
  mode?: 'fade' | 'slide' | 'scale';
  duration?: number;
}

export const RouteTransition: React.FC<RouteTransitionProps> = ({
  children,
  mode = 'fade',
  duration = 0.3,
}) => {
  const location = useLocation();

  const variants = {
    fade: {
      initial: { opacity: 0 },
      animate: { opacity: 1 },
      exit: { opacity: 0 },
    },
    slide: {
      initial: { x: 300, opacity: 0 },
      animate: { x: 0, opacity: 1 },
      exit: { x: -300, opacity: 0 },
    },
    scale: {
      initial: { scale: 0.8, opacity: 0 },
      animate: { scale: 1, opacity: 1 },
      exit: { scale: 1.2, opacity: 0 },
    },
  };

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial="initial"
        animate="animate"
        exit="exit"
        variants={variants[mode]}
        transition={{ duration }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### Route Analytics

```typescript
// src/hooks/useRouteAnalytics.ts
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

export const useRouteAnalytics = () => {
  const location = useLocation();

  useEffect(() => {
    // Track page view
    if (typeof window !== 'undefined' && window.gtag) {
      window.gtag('config', 'GA_MEASUREMENT_ID', {
        page_path: location.pathname + location.search,
      });
    }

    // Track custom analytics
    if (typeof window !== 'undefined' && window.analytics) {
      window.analytics.page(location.pathname);
    }

    // Performance tracking
    const navigationStart = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    if (navigationStart) {
      const loadTime = navigationStart.loadEventEnd - navigationStart.loadEventStart;
      console.log(`Page load time: ${loadTime}ms`);
    }
  }, [location]);
};
```

## Route Configuration Patterns

### Dynamic Route Configuration

```typescript
// src/config/routes.ts
import { RouteObject } from 'react-router-dom';
import { UserRole } from '@/types/auth';

export interface ExtendedRouteObject extends Omit<RouteObject, 'children'> {
  requiredRole?: UserRole;
  requiredPermissions?: string[];
  title?: string;
  description?: string;
  children?: ExtendedRouteObject[];
}

export const routeConfig: ExtendedRouteObject[] = [
  {
    path: '/',
    title: 'Home',
    description: 'Welcome to DafnckMachine',
    element: <HomePage />,
  },
  {
    path: '/dashboard',
    title: 'Dashboard',
    description: 'Your personal dashboard',
    element: <ProtectedRoute />,
    children: [
      {
        index: true,
        title: 'Dashboard Overview',
        element: <DashboardPage />,
      },
      {
        path: 'projects',
        title: 'Projects',
        element: <ProjectsPage />,
      },
      {
        path: 'projects/:id',
        title: 'Project Details',
        element: <ProjectDetailPage />,
      },
    ],
  },
  {
    path: '/admin',
    title: 'Admin Panel',
    description: 'Administrative functions',
    requiredRole: 'admin',
    element: <ProtectedRoute requiredRole="admin" />,
    children: [
      {
        index: true,
        title: 'Admin Dashboard',
        element: <AdminPage />,
      },
      {
        path: 'users',
        title: 'User Management',
        requiredPermissions: ['users:read'],
        element: <UsersPage />,
      },
    ],
  },
];

// Generate router from configuration
export const generateRouter = (routes: ExtendedRouteObject[]): RouteObject[] => {
  return routes.map(route => ({
    ...route,
    children: route.children ? generateRouter(route.children) : undefined,
  }));
};
```

### Route Metadata Management

```typescript
// src/hooks/useRouteMetadata.ts
import { useEffect } from 'react';
import { useLocation, useMatches } from 'react-router-dom';

export const useRouteMetadata = () => {
  const location = useLocation();
  const matches = useMatches();

  useEffect(() => {
    // Find the deepest match with metadata
    const currentMatch = matches
      .reverse()
      .find(match => match.data?.title || match.data?.description);

    if (currentMatch?.data) {
      // Update document title
      if (currentMatch.data.title) {
        document.title = `${currentMatch.data.title} | DafnckMachine`;
      }

      // Update meta description
      if (currentMatch.data.description) {
        let metaDescription = document.querySelector('meta[name="description"]');
        if (!metaDescription) {
          metaDescription = document.createElement('meta');
          metaDescription.setAttribute('name', 'description');
          document.head.appendChild(metaDescription);
        }
        metaDescription.setAttribute('content', currentMatch.data.description);
      }
    }
  }, [location, matches]);
};
```

## Testing Strategies

### Route Testing Utilities

```typescript
// src/utils/testing/routeTestUtils.tsx
import React from 'react';
import { render } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/providers/AuthProvider';

export const renderWithRouter = (
  ui: React.ReactElement,
  options: {
    initialEntries?: string[];
    initialIndex?: number;
    queryClient?: QueryClient;
    user?: any;
  } = {}
) => {
  const {
    initialEntries = ['/'],
    initialIndex = 0,
    queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    }),
    user = null,
  } = options;

  const Wrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      <AuthProvider initialUser={user}>
        <MemoryRouter initialEntries={initialEntries} initialIndex={initialIndex}>
          {children}
        </MemoryRouter>
      </AuthProvider>
    </QueryClientProvider>
  );

  return render(ui, { wrapper: Wrapper });
};
```

### Route Tests Examples

```typescript
// src/router/__tests__/ProtectedRoute.test.tsx
import React from 'react';
import { screen } from '@testing-library/react';
import { ProtectedRoute } from '../ProtectedRoute';
import { renderWithRouter } from '@/utils/testing/routeTestUtils';

describe('ProtectedRoute', () => {
  it('redirects to login when not authenticated', () => {
    renderWithRouter(
      <ProtectedRoute />,
      { initialEntries: ['/dashboard'] }
    );

    expect(window.location.pathname).toBe('/auth/login');
  });

  it('renders children when authenticated', () => {
    const user = { id: '1', name: 'John Doe', role: 'user' };
    
    renderWithRouter(
      <ProtectedRoute>
        <div>Protected Content</div>
      </ProtectedRoute>,
      { user, initialEntries: ['/dashboard'] }
    );

    expect(screen.getByText('Protected Content')).toBeInTheDocument();
  });

  it('redirects when user lacks required role', () => {
    const user = { id: '1', name: 'John Doe', role: 'user' };
    
    renderWithRouter(
      <ProtectedRoute requiredRole="admin">
        <div>Admin Content</div>
      </ProtectedRoute>,
      { user, initialEntries: ['/admin'] }
    );

    expect(window.location.pathname).toBe('/dashboard');
  });
});
```

## Best Practices

### 1. Route Organization
- **Hierarchical Structure**: Organize routes in a logical hierarchy
- **Code Splitting**: Implement lazy loading for better performance
- **Route Guards**: Use consistent authentication and authorization patterns
- **Metadata Management**: Implement proper SEO and metadata handling

### 2. Navigation Patterns
- **Consistent Navigation**: Use standardized navigation components
- **Breadcrumbs**: Implement breadcrumbs for complex hierarchies
- **Route Preloading**: Preload routes on user interaction
- **Transition Effects**: Add smooth transitions between routes

### 3. Performance Optimization
- **Lazy Loading**: Split code at route boundaries
- **Route Preloading**: Preload critical routes
- **Bundle Analysis**: Monitor and optimize bundle sizes
- **Caching**: Implement proper route-level caching

### 4. Security Considerations
- **Route Protection**: Implement proper authentication guards
- **Role-based Access**: Control access based on user roles
- **Permission Checks**: Validate permissions at route level
- **Secure Redirects**: Prevent open redirect vulnerabilities

This comprehensive routing implementation guide provides the foundation for building a robust, secure, and performant routing system in DafnckMachine v3.1. 