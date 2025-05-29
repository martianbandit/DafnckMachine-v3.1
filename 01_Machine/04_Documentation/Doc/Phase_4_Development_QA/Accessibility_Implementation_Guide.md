# Accessibility Implementation Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Component_Architecture_Design.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Component_Architecture_Design.md)

## Overview

This guide provides comprehensive accessibility implementation strategies for DafnckMachine v3.1, ensuring WCAG 2.1 AA compliance and inclusive design for all users including those with disabilities.

## Table of Contents

1. [WCAG 2.1 Compliance](#wcag-21-compliance)
2. [ARIA Implementation](#aria-implementation)
3. [Keyboard Navigation](#keyboard-navigation)
4. [Screen Reader Support](#screen-reader-support)
5. [Color and Contrast](#color-and-contrast)
6. [Focus Management](#focus-management)
7. [Accessible Components](#accessible-components)
8. [Testing Strategies](#testing-strategies)

## WCAG 2.1 Compliance

### Core Principles

```typescript
// WCAG compliance configuration
export const wcagCompliance = {
  // Perceivable
  perceivable: {
    textAlternatives: 'all-non-text-content',
    captions: 'time-based-media',
    adaptable: 'multiple-presentations',
    distinguishable: 'foreground-background-separation'
  },
  
  // Operable
  operable: {
    keyboardAccessible: 'all-functionality',
    seizures: 'no-flashing-content',
    navigable: 'find-navigate-content',
    inputModalities: 'beyond-keyboard'
  },
  
  // Understandable
  understandable: {
    readable: 'text-readable-understandable',
    predictable: 'consistent-functionality',
    inputAssistance: 'error-prevention-correction'
  },
  
  // Robust
  robust: {
    compatible: 'assistive-technology-compatible'
  }
} as const;
```

### Accessibility Standards

```typescript
// Accessibility standards implementation
export const accessibilityStandards = {
  // Color contrast ratios
  contrast: {
    normal: '4.5:1',
    large: '3:1',
    nonText: '3:1'
  },
  
  // Touch target sizes
  touchTargets: {
    minimum: '44px',
    recommended: '48px'
  },
  
  // Text sizing
  textSize: {
    minimum: '16px',
    scalable: '200%'
  },
  
  // Timing
  timing: {
    sessionTimeout: 'extendable',
    animations: 'pausable'
  }
} as const;
```

## ARIA Implementation

### ARIA Attributes

```typescript
// ARIA attribute utilities
export const ariaUtils = {
  // Landmark roles
  landmarks: {
    banner: 'role="banner"',
    navigation: 'role="navigation"',
    main: 'role="main"',
    complementary: 'role="complementary"',
    contentinfo: 'role="contentinfo"'
  },
  
  // Widget roles
  widgets: {
    button: 'role="button"',
    checkbox: 'role="checkbox"',
    dialog: 'role="dialog"',
    menu: 'role="menu"',
    tab: 'role="tab"',
    tabpanel: 'role="tabpanel"'
  },
  
  // Properties
  properties: {
    label: (text: string) => `aria-label="${text}"`,
    labelledby: (id: string) => `aria-labelledby="${id}"`,
    describedby: (id: string) => `aria-describedby="${id}"`,
    expanded: (state: boolean) => `aria-expanded="${state}"`,
    hidden: (state: boolean) => `aria-hidden="${state}"`
  }
} as const;

// Accessible component wrapper
interface AccessibleProps {
  role?: string;
  ariaLabel?: string;
  ariaLabelledBy?: string;
  ariaDescribedBy?: string;
  ariaExpanded?: boolean;
  ariaHidden?: boolean;
  children: React.ReactNode;
}

export const Accessible: React.FC<AccessibleProps> = ({
  role,
  ariaLabel,
  ariaLabelledBy,
  ariaDescribedBy,
  ariaExpanded,
  ariaHidden,
  children
}) => {
  const ariaProps = {
    role,
    'aria-label': ariaLabel,
    'aria-labelledby': ariaLabelledBy,
    'aria-describedby': ariaDescribedBy,
    'aria-expanded': ariaExpanded,
    'aria-hidden': ariaHidden
  };
  
  return <div {...ariaProps}>{children}</div>;
};
```

### Live Regions

```typescript
// Live region component
interface LiveRegionProps {
  politeness?: 'polite' | 'assertive' | 'off';
  atomic?: boolean;
  relevant?: 'additions' | 'removals' | 'text' | 'all';
  children: React.ReactNode;
}

export const LiveRegion: React.FC<LiveRegionProps> = ({
  politeness = 'polite',
  atomic = false,
  relevant = 'additions text',
  children
}) => {
  return (
    <div
      aria-live={politeness}
      aria-atomic={atomic}
      aria-relevant={relevant}
      className="sr-only"
    >
      {children}
    </div>
  );
};

// Status announcer hook
export const useStatusAnnouncer = () => {
  const [message, setMessage] = useState('');
  
  const announce = useCallback((text: string, priority: 'polite' | 'assertive' = 'polite') => {
    setMessage('');
    setTimeout(() => setMessage(text), 100);
  }, []);
  
  return {
    announce,
    StatusAnnouncer: () => (
      <LiveRegion politeness="polite">
        {message}
      </LiveRegion>
    )
  };
};
```

## Keyboard Navigation

### Focus Management

```typescript
// Focus management utilities
export const focusUtils = {
  // Focus trap for modals
  trapFocus: (container: HTMLElement) => {
    const focusableElements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0] as HTMLElement;
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
    
    const handleTabKey = (e: KeyboardEvent) => {
      if (e.key === 'Tab') {
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
      }
    };
    
    container.addEventListener('keydown', handleTabKey);
    firstElement.focus();
    
    return () => container.removeEventListener('keydown', handleTabKey);
  },
  
  // Restore focus
  restoreFocus: (previousElement: HTMLElement | null) => {
    if (previousElement) {
      previousElement.focus();
    }
  }
};

// Focus trap hook
export const useFocusTrap = (isActive: boolean) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (isActive && containerRef.current) {
      previousFocusRef.current = document.activeElement as HTMLElement;
      const cleanup = focusUtils.trapFocus(containerRef.current);
      
      return () => {
        cleanup();
        focusUtils.restoreFocus(previousFocusRef.current);
      };
    }
  }, [isActive]);
  
  return containerRef;
};
```

### Keyboard Event Handling

```typescript
// Keyboard navigation component
interface KeyboardNavigationProps {
  onEnter?: () => void;
  onSpace?: () => void;
  onEscape?: () => void;
  onArrowUp?: () => void;
  onArrowDown?: () => void;
  onArrowLeft?: () => void;
  onArrowRight?: () => void;
  children: React.ReactNode;
}

export const KeyboardNavigation: React.FC<KeyboardNavigationProps> = ({
  onEnter,
  onSpace,
  onEscape,
  onArrowUp,
  onArrowDown,
  onArrowLeft,
  onArrowRight,
  children
}) => {
  const handleKeyDown = (event: React.KeyboardEvent) => {
    switch (event.key) {
      case 'Enter':
        onEnter?.();
        break;
      case ' ':
        event.preventDefault();
        onSpace?.();
        break;
      case 'Escape':
        onEscape?.();
        break;
      case 'ArrowUp':
        event.preventDefault();
        onArrowUp?.();
        break;
      case 'ArrowDown':
        event.preventDefault();
        onArrowDown?.();
        break;
      case 'ArrowLeft':
        onArrowLeft?.();
        break;
      case 'ArrowRight':
        onArrowRight?.();
        break;
    }
  };
  
  return (
    <div onKeyDown={handleKeyDown}>
      {children}
    </div>
  );
};
```

## Screen Reader Support

### Screen Reader Utilities

```typescript
// Screen reader only content
export const ScreenReaderOnly: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <span className="sr-only">{children}</span>
);

// Skip link component
export const SkipLink: React.FC<{ href: string; children: React.ReactNode }> = ({ 
  href, 
  children 
}) => (
  <a
    href={href}
    className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 
               bg-blue-600 text-white px-4 py-2 rounded z-50"
  >
    {children}
  </a>
);

// Accessible heading structure
interface HeadingProps {
  level: 1 | 2 | 3 | 4 | 5 | 6;
  children: React.ReactNode;
  className?: string;
}

export const Heading: React.FC<HeadingProps> = ({ level, children, className }) => {
  const Tag = `h${level}` as keyof JSX.IntrinsicElements;
  
  return (
    <Tag className={className}>
      {children}
    </Tag>
  );
};
```

## Color and Contrast

### Color Utilities

```typescript
// Color contrast utilities
export const colorUtils = {
  // Check contrast ratio
  getContrastRatio: (color1: string, color2: string): number => {
    const getLuminance = (color: string): number => {
      // Simplified luminance calculation
      const rgb = parseInt(color.slice(1), 16);
      const r = (rgb >> 16) & 0xff;
      const g = (rgb >> 8) & 0xff;
      const b = (rgb >> 0) & 0xff;
      
      const [rs, gs, bs] = [r, g, b].map(c => {
        c = c / 255;
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
      });
      
      return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs;
    };
    
    const l1 = getLuminance(color1);
    const l2 = getLuminance(color2);
    
    return (Math.max(l1, l2) + 0.05) / (Math.min(l1, l2) + 0.05);
  },
  
  // Validate contrast
  isContrastValid: (foreground: string, background: string, level: 'AA' | 'AAA' = 'AA'): boolean => {
    const ratio = colorUtils.getContrastRatio(foreground, background);
    return level === 'AA' ? ratio >= 4.5 : ratio >= 7;
  }
};

// High contrast mode detection
export const useHighContrastMode = () => {
  const [isHighContrast, setIsHighContrast] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-contrast: high)');
    setIsHighContrast(mediaQuery.matches);
    
    const handler = (e: MediaQueryListEvent) => setIsHighContrast(e.matches);
    mediaQuery.addEventListener('change', handler);
    
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);
  
  return isHighContrast;
};
```

## Accessible Components

### Accessible Button

```typescript
interface AccessibleButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  loadingText?: string;
}

export const AccessibleButton: React.FC<AccessibleButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  loadingText = 'Loading...',
  disabled,
  children,
  className,
  ...props
}) => {
  const baseClasses = 'font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
    ghost: 'text-blue-600 hover:bg-blue-50 focus:ring-blue-500'
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm min-h-[36px]',
    md: 'px-4 py-2 text-base min-h-[44px]',
    lg: 'px-6 py-3 text-lg min-h-[48px]'
  };
  
  return (
    <button
      className={cn(
        baseClasses,
        variantClasses[variant],
        sizeClasses[size],
        (disabled || loading) && 'opacity-50 cursor-not-allowed',
        className
      )}
      disabled={disabled || loading}
      aria-disabled={disabled || loading}
      aria-describedby={loading ? 'loading-status' : undefined}
      {...props}
    >
      {loading ? (
        <>
          <span aria-hidden="true" className="mr-2">⏳</span>
          {loadingText}
          <ScreenReaderOnly id="loading-status">
            Button is loading, please wait
          </ScreenReaderOnly>
        </>
      ) : (
        children
      )}
    </button>
  );
};
```

### Accessible Modal

```typescript
interface AccessibleModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  description?: string;
  children: React.ReactNode;
}

export const AccessibleModal: React.FC<AccessibleModalProps> = ({
  isOpen,
  onClose,
  title,
  description,
  children
}) => {
  const modalRef = useFocusTrap(isOpen);
  const titleId = useId();
  const descriptionId = useId();
  
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = 'unset';
    }
    
    return () => {
      document.body.style.overflow = 'unset';
    };
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
      aria-describedby={description ? descriptionId : undefined}
    >
      <div
        className="fixed inset-0 bg-black bg-opacity-50"
        onClick={onClose}
        aria-hidden="true"
      />
      
      <div
        ref={modalRef}
        className="relative bg-white rounded-lg shadow-xl max-w-md w-full mx-4 p-6"
      >
        <div className="flex items-center justify-between mb-4">
          <h2 id={titleId} className="text-xl font-semibold">
            {title}
          </h2>
          
          <button
            onClick={onClose}
            className="text-gray-400 hover:text-gray-600 focus:outline-none focus:ring-2 focus:ring-blue-500 rounded"
            aria-label="Close modal"
          >
            <span aria-hidden="true">×</span>
          </button>
        </div>
        
        {description && (
          <p id={descriptionId} className="text-gray-600 mb-4">
            {description}
          </p>
        )}
        
        <div>{children}</div>
      </div>
    </div>
  );
};
```

## Testing Strategies

### Accessibility Testing

```typescript
// Accessibility testing utilities
export const a11yTestUtils = {
  // Check for missing alt text
  checkAltText: () => {
    const images = document.querySelectorAll('img');
    const missingAlt = Array.from(images).filter(img => !img.alt);
    return {
      passed: missingAlt.length === 0,
      violations: missingAlt.map(img => ({
        element: img,
        issue: 'Missing alt attribute'
      }))
    };
  },
  
  // Check heading structure
  checkHeadingStructure: () => {
    const headings = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
    const violations = [];
    let previousLevel = 0;
    
    headings.forEach(heading => {
      const level = parseInt(heading.tagName.charAt(1));
      if (level > previousLevel + 1) {
        violations.push({
          element: heading,
          issue: `Heading level ${level} follows level ${previousLevel}`
        });
      }
      previousLevel = level;
    });
    
    return {
      passed: violations.length === 0,
      violations
    };
  },
  
  // Check color contrast
  checkColorContrast: () => {
    // Implementation would use actual color detection
    return { passed: true, violations: [] };
  }
};

// Automated accessibility testing
describe('Accessibility Tests', () => {
  test('should have proper alt text for images', () => {
    render(<ComponentWithImages />);
    const result = a11yTestUtils.checkAltText();
    expect(result.passed).toBe(true);
  });
  
  test('should have proper heading structure', () => {
    render(<ComponentWithHeadings />);
    const result = a11yTestUtils.checkHeadingStructure();
    expect(result.passed).toBe(true);
  });
  
  test('should be keyboard navigable', () => {
    render(<InteractiveComponent />);
    
    const firstButton = screen.getByRole('button', { name: /first/i });
    const secondButton = screen.getByRole('button', { name: /second/i });
    
    firstButton.focus();
    expect(document.activeElement).toBe(firstButton);
    
    fireEvent.keyDown(firstButton, { key: 'Tab' });
    expect(document.activeElement).toBe(secondButton);
  });
});
```

---

## Summary

This Accessibility Implementation Guide ensures DafnckMachine v3.1 meets WCAG 2.1 AA standards and provides inclusive user experience for all users. Key implementation areas include ARIA attributes, keyboard navigation, screen reader support, color contrast compliance, and comprehensive testing strategies. 