# UI Design System
## DafnckMachine v3.1 - Comprehensive Visual Design Standards & Component Library

### Document Overview
This document establishes the complete UI design system for DafnckMachine v3.1, providing comprehensive visual standards, component specifications, and implementation guidelines that ensure consistent, accessible, and scalable user interface design across all platforms and touchpoints.

### Design System Philosophy

#### Core Principles
1. **Consistency**: Unified visual language across all interfaces
2. **Accessibility**: WCAG 2.1 AA compliance and inclusive design
3. **Scalability**: Flexible system that grows with product needs
4. **Efficiency**: Streamlined design and development workflows
5. **Quality**: High standards for visual and functional excellence

#### Design Foundation
- **Visual Hierarchy**: Clear information architecture and content prioritization
- **Grid Systems**: Flexible layout foundation for responsive design
- **Spacing Standards**: Consistent spatial relationships and rhythm
- **Layout Principles**: Balanced composition and visual flow
- **Design Consistency**: Unified approach across all components and patterns

### Color System

#### Primary Color Palette
```css
:root {
  /* Primary Colors */
  --color-primary-50: #EFF6FF;
  --color-primary-100: #DBEAFE;
  --color-primary-200: #BFDBFE;
  --color-primary-300: #93C5FD;
  --color-primary-400: #60A5FA;
  --color-primary-500: #3B82F6;  /* Primary */
  --color-primary-600: #2563EB;  /* Primary Dark */
  --color-primary-700: #1D4ED8;
  --color-primary-800: #1E40AF;
  --color-primary-900: #1E3A8A;
  
  /* Secondary Colors */
  --color-secondary-50: #F5F3FF;
  --color-secondary-100: #EDE9FE;
  --color-secondary-200: #DDD6FE;
  --color-secondary-300: #C4B5FD;
  --color-secondary-400: #A78BFA;
  --color-secondary-500: #8B5CF6;
  --color-secondary-600: #7C3AED;  /* Secondary */
  --color-secondary-700: #6D28D9;
  --color-secondary-800: #5B21B6;
  --color-secondary-900: #4C1D95;
  
  /* Accent Colors */
  --color-accent-50: #ECFDF5;
  --color-accent-100: #D1FAE5;
  --color-accent-200: #A7F3D0;
  --color-accent-300: #6EE7B7;
  --color-accent-400: #34D399;
  --color-accent-500: #10B981;
  --color-accent-600: #059669;   /* Accent */
  --color-accent-700: #047857;
  --color-accent-800: #065F46;
  --color-accent-900: #064E3B;
}
```

#### Semantic Colors
```css
:root {
  /* Success */
  --color-success-50: #F0FDF4;
  --color-success-500: #22C55E;
  --color-success-600: #16A34A;
  --color-success-700: #15803D;
  
  /* Warning */
  --color-warning-50: #FFFBEB;
  --color-warning-500: #F59E0B;
  --color-warning-600: #D97706;
  --color-warning-700: #B45309;
  
  /* Error */
  --color-error-50: #FEF2F2;
  --color-error-500: #EF4444;
  --color-error-600: #DC2626;
  --color-error-700: #B91C1C;
  
  /* Info */
  --color-info-50: #EFF6FF;
  --color-info-500: #3B82F6;
  --color-info-600: #2563EB;
  --color-info-700: #1D4ED8;
}
```

#### Neutral Colors
```css
:root {
  /* Grays */
  --color-gray-50: #F9FAFB;
  --color-gray-100: #F3F4F6;
  --color-gray-200: #E5E7EB;
  --color-gray-300: #D1D5DB;
  --color-gray-400: #9CA3AF;
  --color-gray-500: #6B7280;
  --color-gray-600: #4B5563;
  --color-gray-700: #374151;
  --color-gray-800: #1F2937;
  --color-gray-900: #111827;
  
  /* Special */
  --color-white: #FFFFFF;
  --color-black: #000000;
}
```

### Typography System

#### Font Families
```css
:root {
  --font-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', 'Monaco', 'Consolas', monospace;
  --font-display: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
```

#### Type Scale
```css
:root {
  /* Font Sizes */
  --text-xs: 0.75rem;     /* 12px */
  --text-sm: 0.875rem;    /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-lg: 1.125rem;    /* 18px */
  --text-xl: 1.25rem;     /* 20px */
  --text-2xl: 1.5rem;     /* 24px */
  --text-3xl: 1.875rem;   /* 30px */
  --text-4xl: 2.25rem;    /* 36px */
  --text-5xl: 3rem;       /* 48px */
  --text-6xl: 3.75rem;    /* 60px */
  
  /* Line Heights */
  --leading-none: 1;
  --leading-tight: 1.25;
  --leading-snug: 1.375;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
  --leading-loose: 2;
  
  /* Font Weights */
  --font-thin: 100;
  --font-light: 300;
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
  --font-extrabold: 800;
  --font-black: 900;
}
```

#### Typography Classes
```css
/* Headings */
.dm-heading-1 {
  font-family: var(--font-display);
  font-size: var(--text-4xl);
  font-weight: var(--font-bold);
  line-height: var(--leading-tight);
  color: var(--color-gray-900);
}

.dm-heading-2 {
  font-family: var(--font-display);
  font-size: var(--text-3xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-tight);
  color: var(--color-gray-900);
}

.dm-heading-3 {
  font-family: var(--font-display);
  font-size: var(--text-2xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-snug);
  color: var(--color-gray-900);
}

/* Body Text */
.dm-body-large {
  font-family: var(--font-primary);
  font-size: var(--text-lg);
  font-weight: var(--font-normal);
  line-height: var(--leading-relaxed);
  color: var(--color-gray-700);
}

.dm-body {
  font-family: var(--font-primary);
  font-size: var(--text-base);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
  color: var(--color-gray-700);
}

.dm-body-small {
  font-family: var(--font-primary);
  font-size: var(--text-sm);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
  color: var(--color-gray-600);
}

/* Code */
.dm-code {
  font-family: var(--font-mono);
  font-size: var(--text-sm);
  font-weight: var(--font-normal);
  background-color: var(--color-gray-100);
  padding: 0.125rem 0.25rem;
  border-radius: 0.25rem;
}
```

### Spacing System

#### Spacing Scale
```css
:root {
  --space-0: 0;
  --space-px: 1px;
  --space-0-5: 0.125rem;  /* 2px */
  --space-1: 0.25rem;     /* 4px */
  --space-1-5: 0.375rem;  /* 6px */
  --space-2: 0.5rem;      /* 8px */
  --space-2-5: 0.625rem;  /* 10px */
  --space-3: 0.75rem;     /* 12px */
  --space-3-5: 0.875rem;  /* 14px */
  --space-4: 1rem;        /* 16px */
  --space-5: 1.25rem;     /* 20px */
  --space-6: 1.5rem;      /* 24px */
  --space-7: 1.75rem;     /* 28px */
  --space-8: 2rem;        /* 32px */
  --space-9: 2.25rem;     /* 36px */
  --space-10: 2.5rem;     /* 40px */
  --space-11: 2.75rem;    /* 44px */
  --space-12: 3rem;       /* 48px */
  --space-14: 3.5rem;     /* 56px */
  --space-16: 4rem;       /* 64px */
  --space-20: 5rem;       /* 80px */
  --space-24: 6rem;       /* 96px */
  --space-28: 7rem;       /* 112px */
  --space-32: 8rem;       /* 128px */
}
```

#### Component Spacing
- **Button Padding**: 12px vertical, 24px horizontal (standard)
- **Card Padding**: 24px all sides
- **Form Field Spacing**: 16px between fields
- **Section Margins**: 32px between sections
- **Page Margins**: 16px mobile, 24px tablet, 32px desktop

### Component Library

#### Button Components
```css
/* Base Button */
.dm-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-3) var(--space-6);
  font-family: var(--font-primary);
  font-size: var(--text-base);
  font-weight: var(--font-medium);
  line-height: var(--leading-none);
  border-radius: 0.375rem;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 150ms ease-in-out;
  text-decoration: none;
  min-height: 44px; /* Accessibility */
}

/* Primary Button */
.dm-button--primary {
  background-color: var(--color-primary-600);
  color: var(--color-white);
  border-color: var(--color-primary-600);
}

.dm-button--primary:hover {
  background-color: var(--color-primary-700);
  border-color: var(--color-primary-700);
}

.dm-button--primary:focus {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

/* Secondary Button */
.dm-button--secondary {
  background-color: transparent;
  color: var(--color-primary-600);
  border-color: var(--color-primary-600);
}

.dm-button--secondary:hover {
  background-color: var(--color-primary-50);
}

/* Tertiary Button */
.dm-button--tertiary {
  background-color: transparent;
  color: var(--color-gray-700);
  border-color: transparent;
}

.dm-button--tertiary:hover {
  background-color: var(--color-gray-100);
}

/* Button Sizes */
.dm-button--small {
  padding: var(--space-2) var(--space-4);
  font-size: var(--text-sm);
  min-height: 36px;
}

.dm-button--large {
  padding: var(--space-4) var(--space-8);
  font-size: var(--text-lg);
  min-height: 52px;
}

/* Button States */
.dm-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.dm-button--loading {
  position: relative;
  color: transparent;
}

.dm-button--loading::after {
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  border: 2px solid currentColor;
  border-radius: 50%;
  border-top-color: transparent;
  animation: spin 1s linear infinite;
}
```

#### Form Components
```css
/* Input Field */
.dm-input {
  width: 100%;
  padding: var(--space-3) var(--space-4);
  font-family: var(--font-primary);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--color-gray-900);
  background-color: var(--color-white);
  border: 1px solid var(--color-gray-300);
  border-radius: 0.375rem;
  transition: border-color 150ms ease-in-out, box-shadow 150ms ease-in-out;
  min-height: 44px; /* Accessibility */
}

.dm-input:focus {
  outline: none;
  border-color: var(--color-primary-500);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.dm-input--error {
  border-color: var(--color-error-500);
}

.dm-input--error:focus {
  border-color: var(--color-error-500);
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
}

/* Label */
.dm-label {
  display: block;
  font-family: var(--font-primary);
  font-size: var(--text-sm);
  font-weight: var(--font-medium);
  color: var(--color-gray-700);
  margin-bottom: var(--space-2);
}

/* Form Group */
.dm-form-group {
  margin-bottom: var(--space-4);
}

/* Error Message */
.dm-error-message {
  display: block;
  font-family: var(--font-primary);
  font-size: var(--text-sm);
  color: var(--color-error-600);
  margin-top: var(--space-1);
}
```

#### Card Components
```css
/* Base Card */
.dm-card {
  background-color: var(--color-white);
  border: 1px solid var(--color-gray-200);
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  overflow: hidden;
  transition: box-shadow 150ms ease-in-out;
}

.dm-card:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

/* Card Header */
.dm-card__header {
  padding: var(--space-6);
  border-bottom: 1px solid var(--color-gray-200);
}

/* Card Body */
.dm-card__body {
  padding: var(--space-6);
}

/* Card Footer */
.dm-card__footer {
  padding: var(--space-6);
  border-top: 1px solid var(--color-gray-200);
  background-color: var(--color-gray-50);
}
```

### Layout System

#### Container
```css
.dm-container {
  width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-left: var(--space-4);
  padding-right: var(--space-4);
}

@media (min-width: 640px) {
  .dm-container {
    max-width: 640px;
    padding-left: var(--space-6);
    padding-right: var(--space-6);
  }
}

@media (min-width: 768px) {
  .dm-container {
    max-width: 768px;
  }
}

@media (min-width: 1024px) {
  .dm-container {
    max-width: 1024px;
  }
}

@media (min-width: 1280px) {
  .dm-container {
    max-width: 1280px;
  }
}
```

#### Grid System
```css
.dm-grid {
  display: grid;
  gap: var(--space-4);
}

.dm-grid--cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
.dm-grid--cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
.dm-grid--cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.dm-grid--cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
.dm-grid--cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
.dm-grid--cols-12 { grid-template-columns: repeat(12, minmax(0, 1fr)); }

/* Responsive Grid */
@media (min-width: 640px) {
  .dm-grid--sm-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .dm-grid--sm-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
}

@media (min-width: 768px) {
  .dm-grid--md-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
  .dm-grid--md-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
}

@media (min-width: 1024px) {
  .dm-grid--lg-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
  .dm-grid--lg-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
}
```

### Animation System

#### Transitions
```css
:root {
  --transition-fast: 150ms ease-in-out;
  --transition-normal: 250ms ease-in-out;
  --transition-slow: 350ms ease-in-out;
  
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
}
```

#### Keyframes
```css
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(100%);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### Accessibility Standards

#### Focus Management
```css
.dm-focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

.dm-sr-only {
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
```

#### Color Contrast
- **Normal Text**: Minimum 4.5:1 contrast ratio
- **Large Text**: Minimum 3:1 contrast ratio
- **Interactive Elements**: Minimum 3:1 contrast ratio
- **Focus Indicators**: Minimum 3:1 contrast ratio

### Implementation Guidelines

#### CSS Architecture
```scss
// 1. Design tokens and variables
@import 'tokens/colors';
@import 'tokens/typography';
@import 'tokens/spacing';

// 2. Base styles
@import 'base/reset';
@import 'base/typography';

// 3. Layout components
@import 'layout/container';
@import 'layout/grid';

// 4. UI components
@import 'components/button';
@import 'components/form';
@import 'components/card';

// 5. Utilities
@import 'utilities/spacing';
@import 'utilities/display';
```

#### Component Naming Convention
- **Prefix**: All classes use `dm-` prefix
- **Block**: Component name (e.g., `dm-button`)
- **Element**: Component part (e.g., `dm-card__header`)
- **Modifier**: Variation (e.g., `dm-button--primary`)

#### Usage Guidelines
1. **Always use design tokens** instead of hardcoded values
2. **Follow accessibility guidelines** for all interactive elements
3. **Test across devices** and screen sizes
4. **Validate color contrast** for all text and interactive elements
5. **Use semantic HTML** with proper ARIA attributes

### Quality Assurance

#### Testing Checklist
- [ ] Visual consistency across components
- [ ] Accessibility compliance (WCAG 2.1 AA)
- [ ] Cross-browser compatibility
- [ ] Responsive behavior
- [ ] Color contrast validation
- [ ] Focus management
- [ ] Keyboard navigation
- [ ] Screen reader compatibility

#### Maintenance
- **Monthly**: Review component usage and feedback
- **Quarterly**: Update design tokens and patterns
- **Annually**: Comprehensive design system audit
- **As Needed**: Add new components and patterns

---

**Document Status**: Active  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-27  
**Owner**: @ui-designer-agent, @design-system-agent 