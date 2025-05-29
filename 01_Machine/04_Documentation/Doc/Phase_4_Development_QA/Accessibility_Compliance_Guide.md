# Accessibility Compliance Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Accessibility Compliance Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Author**: DafnckMachine Development Team
- **Status**: Active Implementation Guide

## Table of Contents
1. [Accessibility Overview](#accessibility-overview)
2. [WCAG 2.1 Compliance](#wcag-21-compliance)
3. [ARIA Implementation](#aria-implementation)
4. [Keyboard Navigation](#keyboard-navigation)
5. [Screen Reader Support](#screen-reader-support)
6. [Color and Contrast](#color-and-contrast)
7. [Focus Management](#focus-management)
8. [Form Accessibility](#form-accessibility)
9. [Media Accessibility](#media-accessibility)
10. [Testing and Validation](#testing-and-validation)

## Accessibility Overview

### Accessibility Philosophy
DafnckMachine is committed to creating inclusive digital experiences that are accessible to all users, regardless of their abilities, disabilities, or assistive technologies.

### Core Principles
- **Perceivable**: Information must be presentable in ways users can perceive
- **Operable**: Interface components must be operable by all users
- **Understandable**: Information and UI operation must be understandable
- **Robust**: Content must be robust enough for various assistive technologies

### Compliance Standards
- **WCAG 2.1 Level AA**: Primary compliance target
- **Section 508**: US Federal accessibility requirements
- **EN 301 549**: European accessibility standard
- **ADA**: Americans with Disabilities Act compliance

### Accessibility Features
- Keyboard navigation support
- Screen reader compatibility
- High contrast mode support
- Reduced motion preferences
- Focus indicators
- Alternative text for images
- Semantic HTML structure

## WCAG 2.1 Compliance

### Level A Requirements

#### 1.1 Text Alternatives
```typescript
// Image with alternative text
const AccessibleImage: React.FC<{
  src: string;
  alt: string;
  decorative?: boolean;
}> = ({ src, alt, decorative = false }) => (
  <img 
    src={src} 
    alt={decorative ? '' : alt}
    role={decorative ? 'presentation' : undefined}
  />
);

// Icon with accessible label
const AccessibleIcon: React.FC<{
  icon: string;
  label: string;
  decorative?: boolean;
}> = ({ icon, label, decorative = false }) => (
  <span 
    className={`icon-${icon}`}
    aria-label={decorative ? undefined : label}
    aria-hidden={decorative}
    role={decorative ? 'presentation' : 'img'}
  />
);
```

#### 1.2 Time-based Media
```typescript
// Video with captions and transcripts
const AccessibleVideo: React.FC<{
  src: string;
  captions?: string;
  transcript?: string;
  autoplay?: boolean;
}> = ({ src, captions, transcript, autoplay = false }) => (
  <div className="video-container">
    <video 
      controls
      autoPlay={autoplay}
      muted={autoplay} // Autoplay requires muted
    >
      <source src={src} type="video/mp4" />
      {captions && <track kind="captions" src={captions} srcLang="en" default />}
      Your browser does not support the video tag.
    </video>
    {transcript && (
      <details className="transcript">
        <summary>Video Transcript</summary>
        <div dangerouslySetInnerHTML={{ __html: transcript }} />
      </details>
    )}
  </div>
);
```

#### 1.3 Adaptable Content
```typescript
// Semantic HTML structure
const AccessibleLayout: React.FC = () => (
  <div className="app-layout">
    <header role="banner">
      <nav role="navigation" aria-label="Main navigation">
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/about">About</a></li>
          <li><a href="/contact">Contact</a></li>
        </ul>
      </nav>
    </header>
    
    <main role="main">
      <h1>Page Title</h1>
      <section aria-labelledby="content-heading">
        <h2 id="content-heading">Content Section</h2>
        <p>Content goes here...</p>
      </section>
    </main>
    
    <aside role="complementary" aria-label="Sidebar">
      <h2>Related Information</h2>
      <p>Sidebar content...</p>
    </aside>
    
    <footer role="contentinfo">
      <p>&copy; 2025 DafnckMachine</p>
    </footer>
  </div>
);
```

#### 1.4 Distinguishable Content
```scss
// High contrast and color considerations
.button {
  // Ensure sufficient color contrast (4.5:1 for normal text)
  background-color: #0066cc;
  color: #ffffff;
  border: 2px solid transparent;
  
  // Don't rely solely on color for information
  &.error {
    background-color: #d32f2f;
    border-color: #b71c1c;
    
    &::before {
      content: '⚠ ';
      font-weight: bold;
    }
  }
  
  // Focus indicator
  &:focus {
    outline: 3px solid #ffbf47;
    outline-offset: 2px;
  }
  
  // High contrast mode support
  @media (prefers-contrast: high) {
    border-color: currentColor;
  }
}

// Responsive text sizing
.text {
  font-size: 1rem;
  line-height: 1.5;
  
  // Support for user zoom up to 200%
  @media (min-width: 1200px) {
    font-size: 1.125rem;
  }
}
```

### Level AA Requirements

#### 2.1 Keyboard Accessible
```typescript
// Keyboard navigation component
const KeyboardNavigable: React.FC<{
  children: React.ReactNode;
  onKeyDown?: (event: React.KeyboardEvent) => void;
}> = ({ children, onKeyDown }) => {
  const handleKeyDown = (event: React.KeyboardEvent) => {
    switch (event.key) {
      case 'Enter':
      case ' ':
        // Handle activation
        event.preventDefault();
        break;
      case 'Escape':
        // Handle escape
        event.preventDefault();
        break;
      case 'ArrowDown':
      case 'ArrowUp':
      case 'ArrowLeft':
      case 'ArrowRight':
        // Handle arrow navigation
        event.preventDefault();
        break;
    }
    
    onKeyDown?.(event);
  };

  return (
    <div 
      tabIndex={0}
      onKeyDown={handleKeyDown}
      role="button"
    >
      {children}
    </div>
  );
};

// Skip link implementation
const SkipLink: React.FC = () => (
  <a 
    href="#main-content"
    className="skip-link"
    onFocus={(e) => e.target.scrollIntoView()}
  >
    Skip to main content
  </a>
);
```

#### 2.4 Navigable
```typescript
// Page title management
const usePageTitle = (title: string) => {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = `${title} - DafnckMachine`;
    
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
};

// Breadcrumb navigation
const Breadcrumb: React.FC<{
  items: Array<{ label: string; href?: string }>;
}> = ({ items }) => (
  <nav aria-label="Breadcrumb">
    <ol className="breadcrumb">
      {items.map((item, index) => (
        <li key={index} className="breadcrumb-item">
          {item.href ? (
            <a href={item.href}>{item.label}</a>
          ) : (
            <span aria-current="page">{item.label}</span>
          )}
          {index < items.length - 1 && (
            <span aria-hidden="true" className="separator">/</span>
          )}
        </li>
      ))}
    </ol>
  </nav>
);
```

#### 3.1 Readable
```typescript
// Language declaration
const LanguageProvider: React.FC<{
  lang: string;
  children: React.ReactNode;
}> = ({ lang, children }) => {
  useEffect(() => {
    document.documentElement.lang = lang;
  }, [lang]);

  return <>{children}</>;
};

// Abbreviation with expansion
const Abbreviation: React.FC<{
  children: string;
  expansion: string;
}> = ({ children, expansion }) => (
  <abbr title={expansion}>{children}</abbr>
);
```

#### 3.2 Predictable
```typescript
// Consistent navigation
const Navigation: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <nav role="navigation" aria-label="Main navigation">
      <button
        aria-expanded={isOpen}
        aria-controls="nav-menu"
        onClick={() => setIsOpen(!isOpen)}
        className="nav-toggle"
      >
        Menu
      </button>
      
      <ul id="nav-menu" className={`nav-menu ${isOpen ? 'open' : ''}`}>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/services">Services</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  );
};
```

#### 3.3 Input Assistance
```typescript
// Form with error handling
const AccessibleForm: React.FC = () => {
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [values, setValues] = useState({ email: '', password: '' });

  const validateForm = () => {
    const newErrors: Record<string, string> = {};
    
    if (!values.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!values.password) {
      newErrors.password = 'Password is required';
    } else if (values.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); validateForm(); }}>
      <div className="form-group">
        <label htmlFor="email">
          Email Address <span aria-label="required">*</span>
        </label>
        <input
          id="email"
          type="email"
          value={values.email}
          onChange={(e) => setValues({ ...values, email: e.target.value })}
          aria-describedby={errors.email ? 'email-error' : 'email-help'}
          aria-invalid={!!errors.email}
          required
        />
        <div id="email-help" className="help-text">
          We'll never share your email with anyone else.
        </div>
        {errors.email && (
          <div id="email-error" className="error-message" role="alert">
            {errors.email}
          </div>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="password">
          Password <span aria-label="required">*</span>
        </label>
        <input
          id="password"
          type="password"
          value={values.password}
          onChange={(e) => setValues({ ...values, password: e.target.value })}
          aria-describedby={errors.password ? 'password-error' : 'password-help'}
          aria-invalid={!!errors.password}
          required
        />
        <div id="password-help" className="help-text">
          Password must be at least 8 characters long.
        </div>
        {errors.password && (
          <div id="password-error" className="error-message" role="alert">
            {errors.password}
          </div>
        )}
      </div>
      
      <button type="submit">Sign In</button>
    </form>
  );
};
```

## ARIA Implementation

### ARIA Landmarks
```typescript
// Landmark roles for page structure
const PageLayout: React.FC = () => (
  <div className="page-layout">
    <header role="banner">
      <h1>Site Title</h1>
      <nav role="navigation" aria-label="Main">
        {/* Navigation items */}
      </nav>
    </header>
    
    <main role="main" id="main-content">
      <h1>Page Title</h1>
      {/* Main content */}
    </main>
    
    <aside role="complementary" aria-labelledby="sidebar-title">
      <h2 id="sidebar-title">Related Links</h2>
      {/* Sidebar content */}
    </aside>
    
    <footer role="contentinfo">
      {/* Footer content */}
    </footer>
  </div>
);
```

### ARIA States and Properties
```typescript
// Expandable content with ARIA
const ExpandableSection: React.FC<{
  title: string;
  children: React.ReactNode;
  defaultExpanded?: boolean;
}> = ({ title, children, defaultExpanded = false }) => {
  const [isExpanded, setIsExpanded] = useState(defaultExpanded);
  const contentId = useId();
  const buttonId = useId();

  return (
    <div className="expandable-section">
      <button
        id={buttonId}
        aria-expanded={isExpanded}
        aria-controls={contentId}
        onClick={() => setIsExpanded(!isExpanded)}
        className="expand-button"
      >
        {title}
        <span aria-hidden="true">{isExpanded ? '−' : '+'}</span>
      </button>
      
      <div
        id={contentId}
        role="region"
        aria-labelledby={buttonId}
        className={`expandable-content ${isExpanded ? 'expanded' : 'collapsed'}`}
        hidden={!isExpanded}
      >
        {children}
      </div>
    </div>
  );
};

// Tab interface with ARIA
const TabPanel: React.FC<{
  tabs: Array<{ label: string; content: React.ReactNode }>;
}> = ({ tabs }) => {
  const [activeTab, setActiveTab] = useState(0);
  const tabListId = useId();

  return (
    <div className="tab-panel">
      <div role="tablist" aria-labelledby={tabListId}>
        <h2 id={tabListId} className="sr-only">Tab Navigation</h2>
        {tabs.map((tab, index) => (
          <button
            key={index}
            role="tab"
            aria-selected={activeTab === index}
            aria-controls={`panel-${index}`}
            id={`tab-${index}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => setActiveTab(index)}
            onKeyDown={(e) => {
              if (e.key === 'ArrowRight') {
                setActiveTab((activeTab + 1) % tabs.length);
              } else if (e.key === 'ArrowLeft') {
                setActiveTab((activeTab - 1 + tabs.length) % tabs.length);
              }
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={index}
          role="tabpanel"
          id={`panel-${index}`}
          aria-labelledby={`tab-${index}`}
          hidden={activeTab !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
};
```

### ARIA Live Regions
```typescript
// Live region for dynamic content updates
const LiveRegion: React.FC<{
  message: string;
  politeness?: 'polite' | 'assertive' | 'off';
  atomic?: boolean;
}> = ({ message, politeness = 'polite', atomic = true }) => (
  <div
    aria-live={politeness}
    aria-atomic={atomic}
    className="sr-only"
  >
    {message}
  </div>
);

// Status updates component
const StatusUpdates: React.FC = () => {
  const [status, setStatus] = useState('');
  const [loading, setLoading] = useState(false);

  const handleAction = async () => {
    setLoading(true);
    setStatus('Processing your request...');
    
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      setStatus('Request completed successfully');
    } catch (error) {
      setStatus('An error occurred. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <button onClick={handleAction} disabled={loading}>
        {loading ? 'Processing...' : 'Submit Request'}
      </button>
      
      <LiveRegion message={status} politeness="assertive" />
      
      {loading && (
        <div role="status" aria-label="Loading">
          <span className="spinner" aria-hidden="true"></span>
          <span className="sr-only">Processing request...</span>
        </div>
      )}
    </div>
  );
};
```

## Keyboard Navigation

### Focus Management
```typescript
// Focus trap for modals
const useFocusTrap = (isActive: boolean) => {
  const containerRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isActive || !containerRef.current) return;

    const container = containerRef.current;
    const focusableElements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0] as HTMLElement;
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

    const handleTabKey = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;

      if (e.shiftKey) {
        if (document.activeElement === firstElement) {
          lastElement.focus();
          e.preventDefault();
        }
      } else {
        if (document.activeElement === lastElement) {
          firstElement.focus();
          e.preventDefault();
        }
      }
    };

    container.addEventListener('keydown', handleTabKey);
    firstElement?.focus();

    return () => {
      container.removeEventListener('keydown', handleTabKey);
    };
  }, [isActive]);

  return containerRef;
};

// Modal with focus management
const Modal: React.FC<{
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}> = ({ isOpen, onClose, title, children }) => {
  const modalRef = useFocusTrap(isOpen);
  const previousFocusRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement as HTMLElement;
    } else if (previousFocusRef.current) {
      previousFocusRef.current.focus();
    }
  }, [isOpen]);

  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      document.body.style.overflow = 'hidden';
    }

    return () => {
      document.removeEventListener('keydown', handleEscape);
      document.body.style.overflow = '';
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onClick={(e) => e.stopPropagation()}
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close modal"
            className="close-button"
          >
            ×
          </button>
        </header>
        
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
};
```

### Keyboard Shortcuts
```typescript
// Global keyboard shortcuts
const useKeyboardShortcuts = () => {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      // Skip if user is typing in an input
      if (e.target instanceof HTMLInputElement || 
          e.target instanceof HTMLTextAreaElement) {
        return;
      }

      // Alt + M: Main navigation
      if (e.altKey && e.key === 'm') {
        e.preventDefault();
        const mainNav = document.querySelector('[role="navigation"]') as HTMLElement;
        mainNav?.focus();
      }

      // Alt + S: Search
      if (e.altKey && e.key === 's') {
        e.preventDefault();
        const searchInput = document.querySelector('#search') as HTMLElement;
        searchInput?.focus();
      }

      // Alt + C: Main content
      if (e.altKey && e.key === 'c') {
        e.preventDefault();
        const mainContent = document.querySelector('#main-content') as HTMLElement;
        mainContent?.focus();
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, []);
};

// Keyboard shortcut help
const KeyboardShortcuts: React.FC = () => (
  <div className="keyboard-shortcuts">
    <h3>Keyboard Shortcuts</h3>
    <dl>
      <dt>Alt + M</dt>
      <dd>Navigate to main menu</dd>
      
      <dt>Alt + S</dt>
      <dd>Focus search input</dd>
      
      <dt>Alt + C</dt>
      <dd>Skip to main content</dd>
      
      <dt>Tab</dt>
      <dd>Move to next interactive element</dd>
      
      <dt>Shift + Tab</dt>
      <dd>Move to previous interactive element</dd>
      
      <dt>Enter or Space</dt>
      <dd>Activate buttons and links</dd>
      
      <dt>Escape</dt>
      <dd>Close modals and dropdowns</dd>
    </dl>
  </div>
);
```

## Screen Reader Support

### Screen Reader Only Content
```scss
// Screen reader only utility class
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

// Show on focus for keyboard users
.sr-only-focusable:focus {
  position: static;
  width: auto;
  height: auto;
  padding: inherit;
  margin: inherit;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

### Descriptive Content
```typescript
// Table with proper headers and captions
const AccessibleTable: React.FC<{
  data: Array<Record<string, any>>;
  columns: Array<{ key: string; label: string; sortable?: boolean }>;
  caption: string;
}> = ({ data, columns, caption }) => {
  const [sortColumn, setSortColumn] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc');

  const handleSort = (columnKey: string) => {
    if (sortColumn === columnKey) {
      setSortDirection(sortDirection === 'asc' ? 'desc' : 'asc');
    } else {
      setSortColumn(columnKey);
      setSortDirection('asc');
    }
  };

  return (
    <table className="accessible-table">
      <caption>{caption}</caption>
      <thead>
        <tr>
          {columns.map((column) => (
            <th
              key={column.key}
              scope="col"
              className={column.sortable ? 'sortable' : ''}
              aria-sort={
                sortColumn === column.key
                  ? sortDirection === 'asc' ? 'ascending' : 'descending'
                  : 'none'
              }
            >
              {column.sortable ? (
                <button
                  onClick={() => handleSort(column.key)}
                  className="sort-button"
                >
                  {column.label}
                  <span aria-hidden="true">
                    {sortColumn === column.key
                      ? sortDirection === 'asc' ? ' ↑' : ' ↓'
                      : ' ↕'}
                  </span>
                </button>
              ) : (
                column.label
              )}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row, index) => (
          <tr key={index}>
            {columns.map((column) => (
              <td key={column.key}>
                {row[column.key]}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
};

// Complex content with descriptions
const ChartWithDescription: React.FC<{
  data: any[];
  title: string;
  description: string;
}> = ({ data, title, description }) => (
  <figure role="img" aria-labelledby="chart-title" aria-describedby="chart-desc">
    <h3 id="chart-title">{title}</h3>
    <div className="chart-container">
      {/* Chart visualization */}
      <div className="chart" aria-hidden="true">
        {/* Visual chart content */}
      </div>
    </div>
    <p id="chart-desc" className="chart-description">
      {description}
    </p>
    
    {/* Data table alternative */}
    <details className="chart-data">
      <summary>View chart data as table</summary>
      <table>
        <caption>Chart data in tabular format</caption>
        {/* Table representation of chart data */}
      </table>
    </details>
  </figure>
);
```

## Color and Contrast

### Color Contrast Requirements
```scss
// Color variables with contrast ratios
:root {
  // Primary colors (4.5:1 contrast ratio minimum)
  --color-primary: #0066cc;
  --color-primary-text: #ffffff;
  
  // Secondary colors
  --color-secondary: #6c757d;
  --color-secondary-text: #ffffff;
  
  // Success colors
  --color-success: #28a745;
  --color-success-text: #ffffff;
  
  // Warning colors
  --color-warning: #ffc107;
  --color-warning-text: #212529;
  
  // Error colors
  --color-error: #dc3545;
  --color-error-text: #ffffff;
  
  // Text colors
  --color-text-primary: #212529;
  --color-text-secondary: #6c757d;
  --color-text-muted: #868e96;
  
  // Background colors
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f8f9fa;
  --color-bg-tertiary: #e9ecef;
}

// High contrast mode support
@media (prefers-contrast: high) {
  :root {
    --color-primary: #0000ff;
    --color-text-primary: #000000;
    --color-bg-primary: #ffffff;
  }
  
  .button {
    border: 2px solid currentColor;
  }
  
  .card {
    border: 1px solid #000000;
  }
}

// Reduced motion support
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Color-Independent Information
```typescript
// Status indicator that doesn't rely solely on color
const StatusIndicator: React.FC<{
  status: 'success' | 'warning' | 'error' | 'info';
  message: string;
}> = ({ status, message }) => {
  const icons = {
    success: '✓',
    warning: '⚠',
    error: '✗',
    info: 'ℹ'
  };

  const labels = {
    success: 'Success',
    warning: 'Warning',
    error: 'Error',
    info: 'Information'
  };

  return (
    <div className={`status-indicator status-${status}`} role="alert">
      <span className="status-icon" aria-hidden="true">
        {icons[status]}
      </span>
      <span className="sr-only">{labels[status]}: </span>
      <span className="status-message">{message}</span>
    </div>
  );
};

// Form validation that uses multiple indicators
const ValidationMessage: React.FC<{
  type: 'error' | 'success';
  message: string;
}> = ({ type, message }) => (
  <div className={`validation-message ${type}`} role="alert">
    <span className="validation-icon" aria-hidden="true">
      {type === 'error' ? '⚠' : '✓'}
    </span>
    <span className="validation-text">{message}</span>
  </div>
);
```

## Focus Management

### Focus Indicators
```scss
// Custom focus indicators
.focusable {
  &:focus {
    outline: 3px solid #ffbf47;
    outline-offset: 2px;
  }
  
  // High contrast mode
  @media (prefers-contrast: high) {
    &:focus {
      outline: 3px solid #000000;
    }
  }
}

// Button focus styles
.button {
  &:focus {
    outline: 3px solid #ffbf47;
    outline-offset: 2px;
    box-shadow: 0 0 0 1px #ffffff;
  }
  
  &:focus:not(:focus-visible) {
    outline: none;
    box-shadow: none;
  }
}

// Link focus styles
a {
  &:focus {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
    text-decoration: underline;
  }
}

// Input focus styles
input, textarea, select {
  &:focus {
    outline: 2px solid #0066cc;
    outline-offset: 1px;
    border-color: #0066cc;
  }
}
```

### Focus Management Utilities
```typescript
// Focus management hook
const useFocusManagement = () => {
  const focusElement = useCallback((selector: string) => {
    const element = document.querySelector(selector) as HTMLElement;
    if (element) {
      element.focus();
      element.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
  }, []);

  const focusFirstError = useCallback(() => {
    const firstError = document.querySelector('[aria-invalid="true"]') as HTMLElement;
    if (firstError) {
      firstError.focus();
      firstError.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
  }, []);

  const announceFocus = useCallback((message: string) => {
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', 'assertive');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.className = 'sr-only';
    announcement.textContent = message;
    
    document.body.appendChild(announcement);
    
    setTimeout(() => {
      document.body.removeChild(announcement);
    }, 1000);
  }, []);

  return {
    focusElement,
    focusFirstError,
    announceFocus,
  };
};

// Route change announcements
const RouteAnnouncer: React.FC = () => {
  const location = useLocation();
  const { announceFocus } = useFocusManagement();

  useEffect(() => {
    const pageTitle = document.title;
    announceFocus(`Navigated to ${pageTitle}`);
  }, [location.pathname, announceFocus]);

  return null;
};
```

## Form Accessibility

### Form Structure and Labels
```typescript
// Accessible form field component
const FormField: React.FC<{
  label: string;
  id: string;
  type?: string;
  required?: boolean;
  error?: string;
  help?: string;
  children?: React.ReactNode;
}> = ({ label, id, type = 'text', required = false, error, help, children }) => {
  const helpId = `${id}-help`;
  const errorId = `${id}-error`;

  return (
    <div className="form-field">
      <label htmlFor={id} className="form-label">
        {label}
        {required && (
          <span className="required" aria-label="required">*</span>
        )}
      </label>
      
      {children || (
        <input
          id={id}
          type={type}
          className={`form-input ${error ? 'error' : ''}`}
          aria-describedby={`${help ? helpId : ''} ${error ? errorId : ''}`.trim()}
          aria-invalid={!!error}
          required={required}
        />
      )}
      
      {help && (
        <div id={helpId} className="form-help">
          {help}
        </div>
      )}
      
      {error && (
        <div id={errorId} className="form-error" role="alert">
          {error}
        </div>
      )}
    </div>
  );
};

// Fieldset for grouped form controls
const FormFieldset: React.FC<{
  legend: string;
  children: React.ReactNode;
  required?: boolean;
}> = ({ legend, children, required = false }) => (
  <fieldset className="form-fieldset">
    <legend className="form-legend">
      {legend}
      {required && (
        <span className="required" aria-label="required">*</span>
      )}
    </legend>
    {children}
  </fieldset>
);

// Radio group component
const RadioGroup: React.FC<{
  name: string;
  legend: string;
  options: Array<{ value: string; label: string }>;
  value?: string;
  onChange: (value: string) => void;
  required?: boolean;
}> = ({ name, legend, options, value, onChange, required = false }) => (
  <FormFieldset legend={legend} required={required}>
    {options.map((option) => (
      <div key={option.value} className="radio-option">
        <input
          type="radio"
          id={`${name}-${option.value}`}
          name={name}
          value={option.value}
          checked={value === option.value}
          onChange={(e) => onChange(e.target.value)}
          required={required}
        />
        <label htmlFor={`${name}-${option.value}`}>
          {option.label}
        </label>
      </div>
    ))}
  </FormFieldset>
);
```

## Media Accessibility

### Video Accessibility
```typescript
// Accessible video player
const AccessibleVideoPlayer: React.FC<{
  src: string;
  poster?: string;
  captions?: string;
  audioDescription?: string;
  transcript?: string;
  title: string;
}> = ({ src, poster, captions, audioDescription, transcript, title }) => {
  const [showTranscript, setShowTranscript] = useState(false);
  const videoRef = useRef<HTMLVideoElement>(null);

  return (
    <div className="video-player">
      <video
        ref={videoRef}
        controls
        poster={poster}
        aria-label={title}
        className="video-element"
      >
        <source src={src} type="video/mp4" />
        
        {captions && (
          <track
            kind="captions"
            src={captions}
            srcLang="en"
            label="English Captions"
            default
          />
        )}
        
        {audioDescription && (
          <track
            kind="descriptions"
            src={audioDescription}
            srcLang="en"
            label="Audio Descriptions"
          />
        )}
        
        <p>
          Your browser doesn't support HTML5 video. 
          <a href={src}>Download the video</a> instead.
        </p>
      </video>
      
      <div className="video-controls">
        <button
          onClick={() => setShowTranscript(!showTranscript)}
          aria-expanded={showTranscript}
          aria-controls="video-transcript"
        >
          {showTranscript ? 'Hide' : 'Show'} Transcript
        </button>
      </div>
      
      {transcript && (
        <div
          id="video-transcript"
          className={`transcript ${showTranscript ? 'visible' : 'hidden'}`}
          hidden={!showTranscript}
        >
          <h3>Video Transcript</h3>
          <div dangerouslySetInnerHTML={{ __html: transcript }} />
        </div>
      )}
    </div>
  );
};
```

### Audio Accessibility
```typescript
// Accessible audio player with transcript
const AccessibleAudioPlayer: React.FC<{
  src: string;
  title: string;
  transcript?: string;
}> = ({ src, title, transcript }) => {
  const [showTranscript, setShowTranscript] = useState(false);

  return (
    <div className="audio-player">
      <audio controls aria-label={title}>
        <source src={src} type="audio/mpeg" />
        Your browser does not support the audio element.
        <a href={src}>Download the audio file</a>.
      </audio>
      
      {transcript && (
        <>
          <button
            onClick={() => setShowTranscript(!showTranscript)}
            aria-expanded={showTranscript}
            aria-controls="audio-transcript"
            className="transcript-toggle"
          >
            {showTranscript ? 'Hide' : 'Show'} Transcript
          </button>
          
          <div
            id="audio-transcript"
            className={`transcript ${showTranscript ? 'visible' : 'hidden'}`}
            hidden={!showTranscript}
          >
            <h3>Audio Transcript</h3>
            <div dangerouslySetInnerHTML={{ __html: transcript }} />
          </div>
        </>
      )}
    </div>
  );
};
```

## Testing and Validation

### Automated Testing
```typescript
// Accessibility testing with jest-axe
import { axe, toHaveNoViolations } from 'jest-axe';
import { render } from '@testing-library/react';

expect.extend(toHaveNoViolations);

describe('Accessibility Tests', () => {
  test('should not have accessibility violations', async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  test('should have proper ARIA labels', () => {
    const { getByRole } = render(<Button>Click me</Button>);
    const button = getByRole('button', { name: 'Click me' });
    expect(button).toBeInTheDocument();
  });

  test('should support keyboard navigation', () => {
    const { getByRole } = render(<Modal isOpen={true} />);
    const modal = getByRole('dialog');
    expect(modal).toHaveFocus();
  });
});
```

### Manual Testing Checklist
```typescript
// Accessibility testing checklist component
const AccessibilityChecklist: React.FC = () => (
  <div className="a11y-checklist">
    <h2>Accessibility Testing Checklist</h2>
    
    <section>
      <h3>Keyboard Navigation</h3>
      <ul>
        <li>All interactive elements are keyboard accessible</li>
        <li>Tab order is logical and intuitive</li>
        <li>Focus indicators are visible and clear</li>
        <li>No keyboard traps exist</li>
        <li>Skip links are provided for main content</li>
      </ul>
    </section>
    
    <section>
      <h3>Screen Reader Testing</h3>
      <ul>
        <li>Content is announced in logical order</li>
        <li>Headings create proper document structure</li>
        <li>Form labels are properly associated</li>
        <li>Error messages are announced</li>
        <li>Dynamic content updates are announced</li>
      </ul>
    </section>
    
    <section>
      <h3>Visual Design</h3>
      <ul>
        <li>Color contrast meets WCAG AA standards</li>
        <li>Information is not conveyed by color alone</li>
        <li>Text can be resized up to 200% without loss of functionality</li>
        <li>Content reflows properly at different zoom levels</li>
      </ul>
    </section>
    
    <section>
      <h3>Forms</h3>
      <ul>
        <li>All form controls have labels</li>
        <li>Required fields are clearly marked</li>
        <li>Error messages are descriptive and helpful</li>
        <li>Form validation is accessible</li>
      </ul>
    </section>
  </div>
);
```

### Testing Tools Integration
```typescript
// Accessibility testing utilities
export const a11yTestUtils = {
  // Test color contrast
  testColorContrast: (foreground: string, background: string) => {
    // Implementation would use a color contrast library
    // Return contrast ratio and pass/fail status
  },

  // Test keyboard navigation
  testKeyboardNavigation: async (element: HTMLElement) => {
    // Simulate keyboard navigation
    // Check tab order and focus management
  },

  // Test screen reader announcements
  testScreenReaderAnnouncements: (element: HTMLElement) => {
    // Check for proper ARIA labels and live regions
    // Validate semantic structure
  },

  // Generate accessibility report
  generateA11yReport: async (component: React.ComponentType) => {
    const { container } = render(React.createElement(component));
    const results = await axe(container);
    
    return {
      violations: results.violations,
      passes: results.passes,
      incomplete: results.incomplete,
      summary: {
        violationCount: results.violations.length,
        passCount: results.passes.length,
        incompleteCount: results.incomplete.length,
      },
    };
  },
};
```

---

**Document Status**: Active Implementation Guide  
**Last Review**: 2025-01-27  
**Next Review**: 2025-02-27  
**Maintained By**: DafnckMachine Development Team 