# User Interface Components
## DafnckMachine v3.1 - Component Library & Implementation Specifications

### Document Overview
This document provides comprehensive specifications for all user interface components in DafnckMachine v3.1, including detailed implementation guidelines, accessibility requirements, and usage patterns for consistent component development across all platforms.

### Component Architecture

#### Component Hierarchy
```
DafnckMachine UI Components
├── Foundation
│   ├── Design Tokens
│   ├── Typography
│   ├── Color System
│   └── Spacing System
├── Layout Components
│   ├── Container
│   ├── Grid
│   ├── Stack
│   └── Flex
├── Form Components
│   ├── Input
│   ├── Textarea
│   ├── Select
│   ├── Checkbox
│   ├── Radio
│   ├── Switch
│   └── Button
├── Navigation Components
│   ├── Header
│   ├── Sidebar
│   ├── Breadcrumb
│   ├── Pagination
│   └── Tabs
├── Data Display
│   ├── Table
│   ├── List
│   ├── Card
│   ├── Badge
│   └── Avatar
├── Feedback Components
│   ├── Alert
│   ├── Toast
│   ├── Modal
│   ├── Tooltip
│   └── Progress
└── Interactive Components
    ├── Dropdown
    ├── Accordion
    ├── Carousel
    └── Menu
```

### Foundation Components

#### Design Tokens Integration
```typescript
// Token usage in components
interface ComponentTokens {
  color: {
    primary: string;
    secondary: string;
    accent: string;
    neutral: Record<string, string>;
    semantic: Record<string, string>;
  };
  spacing: Record<string, string>;
  typography: {
    fontFamily: Record<string, string>;
    fontSize: Record<string, string>;
    fontWeight: Record<string, string>;
    lineHeight: Record<string, string>;
  };
  borderRadius: Record<string, string>;
  shadow: Record<string, string>;
  transition: Record<string, string>;
}
```

### Form Components

#### Button Component
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'tertiary' | 'danger';
  size: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
  onClick?: (event: React.MouseEvent) => void;
  children: React.ReactNode;
  type?: 'button' | 'submit' | 'reset';
  'aria-label'?: string;
}

// Implementation Example
const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  fullWidth = false,
  icon,
  iconPosition = 'left',
  onClick,
  children,
  type = 'button',
  'aria-label': ariaLabel,
  ...props
}) => {
  const baseClasses = 'dm-button';
  const variantClasses = `dm-button--${variant}`;
  const sizeClasses = `dm-button--${size}`;
  const stateClasses = [
    disabled && 'dm-button--disabled',
    loading && 'dm-button--loading',
    fullWidth && 'dm-button--full-width'
  ].filter(Boolean).join(' ');

  return (
    <button
      className={`${baseClasses} ${variantClasses} ${sizeClasses} ${stateClasses}`}
      disabled={disabled || loading}
      onClick={onClick}
      type={type}
      aria-label={ariaLabel}
      aria-busy={loading}
      {...props}
    >
      {icon && iconPosition === 'left' && (
        <span className="dm-button__icon dm-button__icon--left">
          {icon}
        </span>
      )}
      <span className="dm-button__content">
        {children}
      </span>
      {icon && iconPosition === 'right' && (
        <span className="dm-button__icon dm-button__icon--right">
          {icon}
        </span>
      )}
    </button>
  );
};
```

#### Input Component
```typescript
interface InputProps {
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url';
  value?: string;
  defaultValue?: string;
  placeholder?: string;
  disabled?: boolean;
  required?: boolean;
  error?: boolean;
  errorMessage?: string;
  helperText?: string;
  label?: string;
  id?: string;
  name?: string;
  autoComplete?: string;
  onChange?: (event: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur?: (event: React.FocusEvent<HTMLInputElement>) => void;
  onFocus?: (event: React.FocusEvent<HTMLInputElement>) => void;
  'aria-describedby'?: string;
}

const Input: React.FC<InputProps> = ({
  type = 'text',
  value,
  defaultValue,
  placeholder,
  disabled = false,
  required = false,
  error = false,
  errorMessage,
  helperText,
  label,
  id,
  name,
  autoComplete,
  onChange,
  onBlur,
  onFocus,
  'aria-describedby': ariaDescribedBy,
  ...props
}) => {
  const inputId = id || `input-${Math.random().toString(36).substr(2, 9)}`;
  const errorId = `${inputId}-error`;
  const helperId = `${inputId}-helper`;
  
  const describedBy = [
    ariaDescribedBy,
    errorMessage && errorId,
    helperText && helperId
  ].filter(Boolean).join(' ');

  return (
    <div className="dm-form-group">
      {label && (
        <label htmlFor={inputId} className="dm-label">
          {label}
          {required && <span className="dm-label__required">*</span>}
        </label>
      )}
      <input
        id={inputId}
        name={name}
        type={type}
        value={value}
        defaultValue={defaultValue}
        placeholder={placeholder}
        disabled={disabled}
        required={required}
        autoComplete={autoComplete}
        className={`dm-input ${error ? 'dm-input--error' : ''}`}
        onChange={onChange}
        onBlur={onBlur}
        onFocus={onFocus}
        aria-describedby={describedBy || undefined}
        aria-invalid={error}
        {...props}
      />
      {errorMessage && (
        <span id={errorId} className="dm-error-message" role="alert">
          {errorMessage}
        </span>
      )}
      {helperText && !errorMessage && (
        <span id={helperId} className="dm-helper-text">
          {helperText}
        </span>
      )}
    </div>
  );
};
```

### Navigation Components

#### Header Component
```typescript
interface HeaderProps {
  logo?: React.ReactNode;
  navigation?: NavigationItem[];
  actions?: React.ReactNode;
  sticky?: boolean;
  transparent?: boolean;
  className?: string;
}

interface NavigationItem {
  label: string;
  href?: string;
  onClick?: () => void;
  active?: boolean;
  children?: NavigationItem[];
}

const Header: React.FC<HeaderProps> = ({
  logo,
  navigation = [],
  actions,
  sticky = false,
  transparent = false,
  className = '',
  ...props
}) => {
  const headerClasses = [
    'dm-header',
    sticky && 'dm-header--sticky',
    transparent && 'dm-header--transparent',
    className
  ].filter(Boolean).join(' ');

  return (
    <header className={headerClasses} {...props}>
      <div className="dm-container">
        <div className="dm-header__content">
          {logo && (
            <div className="dm-header__logo">
              {logo}
            </div>
          )}
          
          {navigation.length > 0 && (
            <nav className="dm-header__navigation" role="navigation">
              <ul className="dm-nav-list">
                {navigation.map((item, index) => (
                  <li key={index} className="dm-nav-item">
                    {item.href ? (
                      <a
                        href={item.href}
                        className={`dm-nav-link ${item.active ? 'dm-nav-link--active' : ''}`}
                        aria-current={item.active ? 'page' : undefined}
                      >
                        {item.label}
                      </a>
                    ) : (
                      <button
                        onClick={item.onClick}
                        className={`dm-nav-link ${item.active ? 'dm-nav-link--active' : ''}`}
                        aria-current={item.active ? 'page' : undefined}
                      >
                        {item.label}
                      </button>
                    )}
                  </li>
                ))}
              </ul>
            </nav>
          )}
          
          {actions && (
            <div className="dm-header__actions">
              {actions}
            </div>
          )}
        </div>
      </div>
    </header>
  );
};
```

### Data Display Components

#### Card Component
```typescript
interface CardProps {
  variant?: 'default' | 'outlined' | 'elevated';
  padding?: 'none' | 'small' | 'medium' | 'large';
  header?: React.ReactNode;
  footer?: React.ReactNode;
  children: React.ReactNode;
  className?: string;
  onClick?: () => void;
  hoverable?: boolean;
}

const Card: React.FC<CardProps> = ({
  variant = 'default',
  padding = 'medium',
  header,
  footer,
  children,
  className = '',
  onClick,
  hoverable = false,
  ...props
}) => {
  const cardClasses = [
    'dm-card',
    `dm-card--${variant}`,
    `dm-card--padding-${padding}`,
    hoverable && 'dm-card--hoverable',
    onClick && 'dm-card--clickable',
    className
  ].filter(Boolean).join(' ');

  const CardElement = onClick ? 'button' : 'div';

  return (
    <CardElement
      className={cardClasses}
      onClick={onClick}
      {...props}
    >
      {header && (
        <div className="dm-card__header">
          {header}
        </div>
      )}
      
      <div className="dm-card__body">
        {children}
      </div>
      
      {footer && (
        <div className="dm-card__footer">
          {footer}
        </div>
      )}
    </CardElement>
  );
};
```

### Feedback Components

#### Alert Component
```typescript
interface AlertProps {
  variant: 'success' | 'warning' | 'error' | 'info';
  title?: string;
  children: React.ReactNode;
  dismissible?: boolean;
  onDismiss?: () => void;
  icon?: React.ReactNode;
  className?: string;
}

const Alert: React.FC<AlertProps> = ({
  variant,
  title,
  children,
  dismissible = false,
  onDismiss,
  icon,
  className = '',
  ...props
}) => {
  const alertClasses = [
    'dm-alert',
    `dm-alert--${variant}`,
    className
  ].filter(Boolean).join(' ');

  return (
    <div
      className={alertClasses}
      role="alert"
      {...props}
    >
      {icon && (
        <div className="dm-alert__icon">
          {icon}
        </div>
      )}
      
      <div className="dm-alert__content">
        {title && (
          <div className="dm-alert__title">
            {title}
          </div>
        )}
        <div className="dm-alert__message">
          {children}
        </div>
      </div>
      
      {dismissible && (
        <button
          className="dm-alert__dismiss"
          onClick={onDismiss}
          aria-label="Dismiss alert"
        >
          <span aria-hidden="true">&times;</span>
        </button>
      )}
    </div>
  );
};
```

#### Modal Component
```typescript
interface ModalProps {
  open: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
  size?: 'small' | 'medium' | 'large' | 'fullscreen';
  closeOnOverlayClick?: boolean;
  closeOnEscape?: boolean;
  className?: string;
}

const Modal: React.FC<ModalProps> = ({
  open,
  onClose,
  title,
  children,
  footer,
  size = 'medium',
  closeOnOverlayClick = true,
  closeOnEscape = true,
  className = '',
  ...props
}) => {
  const modalClasses = [
    'dm-modal',
    `dm-modal--${size}`,
    className
  ].filter(Boolean).join(' ');

  // Handle escape key
  React.useEffect(() => {
    if (!closeOnEscape) return;
    
    const handleEscape = (event: KeyboardEvent) => {
      if (event.key === 'Escape') {
        onClose();
      }
    };

    if (open) {
      document.addEventListener('keydown', handleEscape);
    }

    return () => {
      document.removeEventListener('keydown', handleEscape);
    };
  }, [open, closeOnEscape, onClose]);

  // Handle body scroll lock
  React.useEffect(() => {
    if (open) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = '';
    }

    return () => {
      document.body.style.overflow = '';
    };
  }, [open]);

  if (!open) return null;

  return (
    <div className="dm-modal-overlay" role="dialog" aria-modal="true">
      <div
        className="dm-modal-backdrop"
        onClick={closeOnOverlayClick ? onClose : undefined}
      />
      <div className={modalClasses} {...props}>
        {title && (
          <div className="dm-modal__header">
            <h2 className="dm-modal__title">{title}</h2>
            <button
              className="dm-modal__close"
              onClick={onClose}
              aria-label="Close modal"
            >
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
        )}
        
        <div className="dm-modal__body">
          {children}
        </div>
        
        {footer && (
          <div className="dm-modal__footer">
            {footer}
          </div>
        )}
      </div>
    </div>
  );
};
```

### Component Styling

#### CSS Architecture
```scss
// Component base styles
.dm-component {
  // Base component styles using design tokens
  font-family: var(--font-primary);
  color: var(--color-text-primary);
  
  // Component variants
  &--variant {
    // Variant-specific styles
  }
  
  // Component states
  &:hover {
    // Hover styles
  }
  
  &:focus {
    // Focus styles for accessibility
    outline: 2px solid var(--color-border-focus);
    outline-offset: 2px;
  }
  
  &:disabled {
    // Disabled styles
    opacity: var(--opacity-50);
    cursor: not-allowed;
  }
  
  // Component elements
  &__element {
    // Element-specific styles
  }
  
  // Component modifiers
  &--modifier {
    // Modifier-specific styles
  }
}
```

### Accessibility Standards

#### ARIA Implementation
```typescript
// Accessibility props interface
interface AccessibilityProps {
  'aria-label'?: string;
  'aria-labelledby'?: string;
  'aria-describedby'?: string;
  'aria-expanded'?: boolean;
  'aria-selected'?: boolean;
  'aria-checked'?: boolean;
  'aria-disabled'?: boolean;
  'aria-hidden'?: boolean;
  'aria-current'?: 'page' | 'step' | 'location' | 'date' | 'time' | boolean;
  role?: string;
  tabIndex?: number;
}

// Focus management utilities
export const focusManagement = {
  trapFocus: (container: HTMLElement) => {
    // Implementation for focus trapping
  },
  restoreFocus: (element: HTMLElement) => {
    // Implementation for focus restoration
  },
  getFirstFocusableElement: (container: HTMLElement) => {
    // Implementation to find first focusable element
  },
  getLastFocusableElement: (container: HTMLElement) => {
    // Implementation to find last focusable element
  }
};
```

#### Keyboard Navigation
```typescript
// Keyboard event handlers
export const keyboardHandlers = {
  handleArrowNavigation: (
    event: KeyboardEvent,
    items: HTMLElement[],
    currentIndex: number
  ) => {
    switch (event.key) {
      case 'ArrowDown':
      case 'ArrowRight':
        event.preventDefault();
        const nextIndex = (currentIndex + 1) % items.length;
        items[nextIndex]?.focus();
        break;
      case 'ArrowUp':
      case 'ArrowLeft':
        event.preventDefault();
        const prevIndex = currentIndex === 0 ? items.length - 1 : currentIndex - 1;
        items[prevIndex]?.focus();
        break;
      case 'Home':
        event.preventDefault();
        items[0]?.focus();
        break;
      case 'End':
        event.preventDefault();
        items[items.length - 1]?.focus();
        break;
    }
  }
};
```

### Testing Guidelines

#### Component Testing
```typescript
// Test utilities for components
export const testUtils = {
  renderWithTheme: (component: React.ReactElement) => {
    // Render component with design system theme
  },
  
  testAccessibility: async (component: React.ReactElement) => {
    // Run accessibility tests using axe-core
  },
  
  testKeyboardNavigation: (component: React.ReactElement) => {
    // Test keyboard navigation patterns
  },
  
  testResponsiveDesign: (component: React.ReactElement) => {
    // Test component across different viewport sizes
  }
};

// Example component test
describe('Button Component', () => {
  it('should render with correct accessibility attributes', () => {
    const { getByRole } = render(
      <Button aria-label="Submit form">
        Submit
      </Button>
    );
    
    const button = getByRole('button');
    expect(button).toHaveAttribute('aria-label', 'Submit form');
  });
  
  it('should handle keyboard navigation', () => {
    const handleClick = jest.fn();
    const { getByRole } = render(
      <Button onClick={handleClick}>
        Click me
      </Button>
    );
    
    const button = getByRole('button');
    fireEvent.keyDown(button, { key: 'Enter' });
    expect(handleClick).toHaveBeenCalled();
  });
});
```

### Performance Optimization

#### Component Optimization
```typescript
// Performance optimization techniques
export const optimizations = {
  // Memoization for expensive calculations
  useMemoizedValue: <T>(value: T, deps: React.DependencyList): T => {
    return React.useMemo(() => value, deps);
  },
  
  // Callback memoization
  useMemoizedCallback: <T extends (...args: any[]) => any>(
    callback: T,
    deps: React.DependencyList
  ): T => {
    return React.useCallback(callback, deps);
  },
  
  // Component memoization
  memoizeComponent: <P extends object>(
    Component: React.ComponentType<P>
  ): React.ComponentType<P> => {
    return React.memo(Component);
  }
};

// Lazy loading for large components
export const LazyComponent = React.lazy(() => import('./HeavyComponent'));

// Usage with Suspense
const ComponentWithSuspense: React.FC = () => (
  <React.Suspense fallback={<div>Loading...</div>}>
    <LazyComponent />
  </React.Suspense>
);
```

### Documentation Standards

#### Component Documentation
```typescript
/**
 * Button component for user interactions
 * 
 * @example
 * ```tsx
 * <Button variant="primary" size="large" onClick={handleClick}>
 *   Click me
 * </Button>
 * ```
 * 
 * @param variant - Visual style variant
 * @param size - Size of the button
 * @param disabled - Whether the button is disabled
 * @param loading - Whether the button is in loading state
 * @param onClick - Click event handler
 * @param children - Button content
 */
export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'tertiary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: (event: React.MouseEvent) => void;
  children: React.ReactNode;
}
```

---

**Document Status**: Active  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-27  
**Owner**: @ui-designer-agent, @design-system-agent 