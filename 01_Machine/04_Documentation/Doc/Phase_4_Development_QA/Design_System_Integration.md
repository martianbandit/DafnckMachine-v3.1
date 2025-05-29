# Design System Integration Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Design System Integration Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Author**: DafnckMachine Development Team
- **Status**: Active Implementation Guide

## Table of Contents
1. [Design System Overview](#design-system-overview)
2. [Design Tokens Implementation](#design-tokens-implementation)
3. [Component Styling Architecture](#component-styling-architecture)
4. [Theme Configuration](#theme-configuration)
5. [Responsive Breakpoints](#responsive-breakpoints)
6. [Typography System](#typography-system)
7. [Color System](#color-system)
8. [Spacing and Layout](#spacing-and-layout)
9. [Component Variants](#component-variants)
10. [Implementation Guidelines](#implementation-guidelines)

## Design System Overview

### Design System Philosophy
The DafnckMachine design system provides a unified visual language and component library that ensures consistency, accessibility, and scalability across all user interfaces.

### Core Principles
- **Consistency**: Unified visual language across all components
- **Accessibility**: WCAG 2.1 AA compliance by default
- **Scalability**: Flexible system that grows with the application
- **Maintainability**: Clear documentation and implementation patterns
- **Performance**: Optimized for fast loading and rendering

### Design System Structure
```
design-system/
├── tokens/
│   ├── colors.json
│   ├── typography.json
│   ├── spacing.json
│   └── breakpoints.json
├── components/
│   ├── base/
│   ├── composite/
│   └── layout/
├── themes/
│   ├── light.json
│   └── dark.json
└── utilities/
    ├── mixins.scss
    └── functions.scss
```

## Design Tokens Implementation

### Token Architecture
Design tokens are the atomic elements of the design system, representing design decisions as data.

### Color Tokens
```json
{
  "color": {
    "primary": {
      "50": "#f0f9ff",
      "100": "#e0f2fe",
      "200": "#bae6fd",
      "300": "#7dd3fc",
      "400": "#38bdf8",
      "500": "#0ea5e9",
      "600": "#0284c7",
      "700": "#0369a1",
      "800": "#075985",
      "900": "#0c4a6e"
    },
    "neutral": {
      "50": "#fafafa",
      "100": "#f5f5f5",
      "200": "#e5e5e5",
      "300": "#d4d4d4",
      "400": "#a3a3a3",
      "500": "#737373",
      "600": "#525252",
      "700": "#404040",
      "800": "#262626",
      "900": "#171717"
    },
    "semantic": {
      "success": "#10b981",
      "warning": "#f59e0b",
      "error": "#ef4444",
      "info": "#3b82f6"
    }
  }
}
```

### Typography Tokens
```json
{
  "typography": {
    "fontFamily": {
      "sans": ["Inter", "system-ui", "sans-serif"],
      "mono": ["JetBrains Mono", "monospace"]
    },
    "fontSize": {
      "xs": "0.75rem",
      "sm": "0.875rem",
      "base": "1rem",
      "lg": "1.125rem",
      "xl": "1.25rem",
      "2xl": "1.5rem",
      "3xl": "1.875rem",
      "4xl": "2.25rem",
      "5xl": "3rem"
    },
    "fontWeight": {
      "light": "300",
      "normal": "400",
      "medium": "500",
      "semibold": "600",
      "bold": "700"
    },
    "lineHeight": {
      "tight": "1.25",
      "normal": "1.5",
      "relaxed": "1.75"
    }
  }
}
```

### Spacing Tokens
```json
{
  "spacing": {
    "0": "0",
    "1": "0.25rem",
    "2": "0.5rem",
    "3": "0.75rem",
    "4": "1rem",
    "5": "1.25rem",
    "6": "1.5rem",
    "8": "2rem",
    "10": "2.5rem",
    "12": "3rem",
    "16": "4rem",
    "20": "5rem",
    "24": "6rem",
    "32": "8rem"
  }
}
```

## Component Styling Architecture

### CSS-in-JS Implementation
Using styled-components for component styling with design token integration.

```typescript
// Theme provider setup
import { ThemeProvider } from 'styled-components';
import { designTokens } from './tokens';

const App = () => (
  <ThemeProvider theme={designTokens}>
    <AppContent />
  </ThemeProvider>
);
```

### Component Styling Pattern
```typescript
import styled from 'styled-components';

const Button = styled.button<ButtonProps>`
  /* Base styles */
  font-family: ${({ theme }) => theme.typography.fontFamily.sans};
  font-size: ${({ theme }) => theme.typography.fontSize.base};
  font-weight: ${({ theme }) => theme.typography.fontWeight.medium};
  line-height: ${({ theme }) => theme.typography.lineHeight.normal};
  
  /* Spacing */
  padding: ${({ theme }) => `${theme.spacing[3]} ${theme.spacing[6]}`};
  margin: 0;
  
  /* Colors */
  background-color: ${({ theme, variant }) => 
    variant === 'primary' 
      ? theme.color.primary[500] 
      : theme.color.neutral[100]
  };
  color: ${({ theme, variant }) => 
    variant === 'primary' 
      ? theme.color.neutral[50] 
      : theme.color.neutral[900]
  };
  
  /* Border and radius */
  border: 1px solid ${({ theme }) => theme.color.neutral[300]};
  border-radius: ${({ theme }) => theme.borderRadius.md};
  
  /* Interactive states */
  &:hover {
    background-color: ${({ theme, variant }) => 
      variant === 'primary' 
        ? theme.color.primary[600] 
        : theme.color.neutral[200]
    };
  }
  
  &:focus {
    outline: 2px solid ${({ theme }) => theme.color.primary[500]};
    outline-offset: 2px;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;
```

### Utility Classes
```scss
// Spacing utilities
@each $key, $value in $spacing-tokens {
  .p-#{$key} { padding: #{$value}; }
  .pt-#{$key} { padding-top: #{$value}; }
  .pr-#{$key} { padding-right: #{$value}; }
  .pb-#{$key} { padding-bottom: #{$value}; }
  .pl-#{$key} { padding-left: #{$value}; }
  
  .m-#{$key} { margin: #{$value}; }
  .mt-#{$key} { margin-top: #{$value}; }
  .mr-#{$key} { margin-right: #{$value}; }
  .mb-#{$key} { margin-bottom: #{$value}; }
  .ml-#{$key} { margin-left: #{$value}; }
}

// Color utilities
@each $color, $shades in $color-tokens {
  @each $shade, $value in $shades {
    .text-#{$color}-#{$shade} { color: #{$value}; }
    .bg-#{$color}-#{$shade} { background-color: #{$value}; }
    .border-#{$color}-#{$shade} { border-color: #{$value}; }
  }
}
```

## Theme Configuration

### Theme Structure
```typescript
interface Theme {
  colors: ColorTokens;
  typography: TypographyTokens;
  spacing: SpacingTokens;
  breakpoints: BreakpointTokens;
  shadows: ShadowTokens;
  borderRadius: BorderRadiusTokens;
  zIndex: ZIndexTokens;
}
```

### Light Theme
```typescript
export const lightTheme: Theme = {
  colors: {
    primary: primaryColors,
    neutral: neutralColors,
    semantic: semanticColors,
    background: {
      primary: '#ffffff',
      secondary: '#f8fafc',
      tertiary: '#f1f5f9'
    },
    text: {
      primary: '#0f172a',
      secondary: '#475569',
      tertiary: '#64748b'
    }
  },
  // ... other tokens
};
```

### Dark Theme
```typescript
export const darkTheme: Theme = {
  colors: {
    primary: primaryColors,
    neutral: neutralColors,
    semantic: semanticColors,
    background: {
      primary: '#0f172a',
      secondary: '#1e293b',
      tertiary: '#334155'
    },
    text: {
      primary: '#f8fafc',
      secondary: '#cbd5e1',
      tertiary: '#94a3b8'
    }
  },
  // ... other tokens
};
```

### Theme Provider Implementation
```typescript
import React, { createContext, useContext, useState } from 'react';
import { ThemeProvider as StyledThemeProvider } from 'styled-components';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const currentTheme = theme === 'light' ? lightTheme : darkTheme;
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <StyledThemeProvider theme={currentTheme}>
        {children}
      </StyledThemeProvider>
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};
```

## Responsive Breakpoints

### Breakpoint System
```json
{
  "breakpoints": {
    "xs": "320px",
    "sm": "640px",
    "md": "768px",
    "lg": "1024px",
    "xl": "1280px",
    "2xl": "1536px"
  }
}
```

### Responsive Utilities
```typescript
// Breakpoint utilities
export const breakpoints = {
  xs: '320px',
  sm: '640px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px'
};

export const mediaQueries = {
  xs: `@media (min-width: ${breakpoints.xs})`,
  sm: `@media (min-width: ${breakpoints.sm})`,
  md: `@media (min-width: ${breakpoints.md})`,
  lg: `@media (min-width: ${breakpoints.lg})`,
  xl: `@media (min-width: ${breakpoints.xl})`,
  '2xl': `@media (min-width: ${breakpoints['2xl']})`
};

// Responsive component example
const ResponsiveGrid = styled.div`
  display: grid;
  gap: ${({ theme }) => theme.spacing[4]};
  grid-template-columns: 1fr;
  
  ${mediaQueries.sm} {
    grid-template-columns: repeat(2, 1fr);
  }
  
  ${mediaQueries.lg} {
    grid-template-columns: repeat(3, 1fr);
  }
`;
```

## Typography System

### Typography Scale
```typescript
export const typographyScale = {
  display: {
    fontSize: '3rem',
    lineHeight: '1.2',
    fontWeight: '700',
    letterSpacing: '-0.02em'
  },
  h1: {
    fontSize: '2.25rem',
    lineHeight: '1.25',
    fontWeight: '700',
    letterSpacing: '-0.01em'
  },
  h2: {
    fontSize: '1.875rem',
    lineHeight: '1.3',
    fontWeight: '600',
    letterSpacing: '-0.01em'
  },
  h3: {
    fontSize: '1.5rem',
    lineHeight: '1.35',
    fontWeight: '600'
  },
  h4: {
    fontSize: '1.25rem',
    lineHeight: '1.4',
    fontWeight: '600'
  },
  body: {
    fontSize: '1rem',
    lineHeight: '1.5',
    fontWeight: '400'
  },
  caption: {
    fontSize: '0.875rem',
    lineHeight: '1.4',
    fontWeight: '400'
  },
  small: {
    fontSize: '0.75rem',
    lineHeight: '1.3',
    fontWeight: '400'
  }
};
```

### Typography Components
```typescript
const Typography = styled.p<TypographyProps>`
  font-family: ${({ theme }) => theme.typography.fontFamily.sans};
  margin: 0;
  
  ${({ variant, theme }) => {
    const styles = typographyScale[variant || 'body'];
    return css`
      font-size: ${styles.fontSize};
      line-height: ${styles.lineHeight};
      font-weight: ${styles.fontWeight};
      letter-spacing: ${styles.letterSpacing || 'normal'};
    `;
  }}
  
  color: ${({ theme, color = 'primary' }) => theme.colors.text[color]};
`;
```

## Color System

### Color Palette
The color system provides semantic meaning and visual hierarchy through carefully selected color palettes.

### Primary Colors
- Used for primary actions, links, and brand elements
- Provides 10 shades from 50 (lightest) to 900 (darkest)
- Ensures sufficient contrast ratios for accessibility

### Neutral Colors
- Used for text, backgrounds, and borders
- Provides foundation for content hierarchy
- Optimized for readability and accessibility

### Semantic Colors
- Success: Green tones for positive actions and states
- Warning: Orange/yellow tones for caution and alerts
- Error: Red tones for errors and destructive actions
- Info: Blue tones for informational content

### Color Usage Guidelines
```typescript
// Color usage examples
const ColorExamples = {
  // Primary actions
  primaryButton: theme.colors.primary[500],
  primaryButtonHover: theme.colors.primary[600],
  
  // Text hierarchy
  headingText: theme.colors.text.primary,
  bodyText: theme.colors.text.secondary,
  captionText: theme.colors.text.tertiary,
  
  // Backgrounds
  pageBackground: theme.colors.background.primary,
  cardBackground: theme.colors.background.secondary,
  sectionBackground: theme.colors.background.tertiary,
  
  // Borders
  defaultBorder: theme.colors.neutral[300],
  focusBorder: theme.colors.primary[500],
  errorBorder: theme.colors.semantic.error
};
```

## Spacing and Layout

### Spacing Scale
The spacing system uses a consistent scale based on 0.25rem (4px) increments.

### Layout Components
```typescript
// Container component
const Container = styled.div<ContainerProps>`
  width: 100%;
  max-width: ${({ maxWidth = '1200px' }) => maxWidth};
  margin: 0 auto;
  padding: 0 ${({ theme }) => theme.spacing[4]};
  
  ${mediaQueries.sm} {
    padding: 0 ${({ theme }) => theme.spacing[6]};
  }
  
  ${mediaQueries.lg} {
    padding: 0 ${({ theme }) => theme.spacing[8]};
  }
`;

// Grid system
const Grid = styled.div<GridProps>`
  display: grid;
  gap: ${({ theme, gap = 4 }) => theme.spacing[gap]};
  grid-template-columns: ${({ columns = 1 }) => `repeat(${columns}, 1fr)`};
  
  ${mediaQueries.sm} {
    grid-template-columns: ${({ smColumns, columns = 1 }) => 
      `repeat(${smColumns || columns}, 1fr)`};
  }
  
  ${mediaQueries.md} {
    grid-template-columns: ${({ mdColumns, smColumns, columns = 1 }) => 
      `repeat(${mdColumns || smColumns || columns}, 1fr)`};
  }
  
  ${mediaQueries.lg} {
    grid-template-columns: ${({ lgColumns, mdColumns, smColumns, columns = 1 }) => 
      `repeat(${lgColumns || mdColumns || smColumns || columns}, 1fr)`};
  }
`;

// Flexbox utilities
const Flex = styled.div<FlexProps>`
  display: flex;
  align-items: ${({ align = 'stretch' }) => align};
  justify-content: ${({ justify = 'flex-start' }) => justify};
  flex-direction: ${({ direction = 'row' }) => direction};
  gap: ${({ theme, gap = 0 }) => theme.spacing[gap]};
  flex-wrap: ${({ wrap = 'nowrap' }) => wrap};
`;
```

## Component Variants

### Button Variants
```typescript
const buttonVariants = {
  primary: {
    backgroundColor: 'primary.500',
    color: 'white',
    '&:hover': {
      backgroundColor: 'primary.600'
    }
  },
  secondary: {
    backgroundColor: 'neutral.100',
    color: 'neutral.900',
    border: '1px solid',
    borderColor: 'neutral.300',
    '&:hover': {
      backgroundColor: 'neutral.200'
    }
  },
  outline: {
    backgroundColor: 'transparent',
    color: 'primary.500',
    border: '1px solid',
    borderColor: 'primary.500',
    '&:hover': {
      backgroundColor: 'primary.50'
    }
  },
  ghost: {
    backgroundColor: 'transparent',
    color: 'primary.500',
    '&:hover': {
      backgroundColor: 'primary.50'
    }
  }
};

const buttonSizes = {
  sm: {
    padding: '0.5rem 0.75rem',
    fontSize: '0.875rem'
  },
  md: {
    padding: '0.75rem 1rem',
    fontSize: '1rem'
  },
  lg: {
    padding: '1rem 1.5rem',
    fontSize: '1.125rem'
  }
};
```

### Input Variants
```typescript
const inputVariants = {
  default: {
    border: '1px solid',
    borderColor: 'neutral.300',
    '&:focus': {
      borderColor: 'primary.500',
      outline: '2px solid',
      outlineColor: 'primary.100'
    }
  },
  error: {
    border: '1px solid',
    borderColor: 'semantic.error',
    '&:focus': {
      borderColor: 'semantic.error',
      outline: '2px solid',
      outlineColor: 'red.100'
    }
  },
  success: {
    border: '1px solid',
    borderColor: 'semantic.success',
    '&:focus': {
      borderColor: 'semantic.success',
      outline: '2px solid',
      outlineColor: 'green.100'
    }
  }
};
```

## Implementation Guidelines

### Getting Started
1. Install design system dependencies
2. Configure theme provider
3. Import design tokens
4. Implement base components
5. Create component variants

### Best Practices
- Always use design tokens instead of hardcoded values
- Implement responsive design with mobile-first approach
- Ensure accessibility compliance in all components
- Maintain consistent spacing and typography
- Use semantic color names for better maintainability

### Component Development Workflow
1. Define component requirements
2. Create base component structure
3. Implement design token integration
4. Add responsive behavior
5. Ensure accessibility compliance
6. Write comprehensive tests
7. Document component usage

### Quality Assurance
- Visual regression testing
- Accessibility auditing
- Cross-browser compatibility testing
- Performance optimization
- Documentation validation

### Maintenance and Updates
- Regular design token updates
- Component library versioning
- Breaking change documentation
- Migration guides for updates
- Community feedback integration

---

**Document Status**: Active Implementation Guide  
**Last Review**: 2025-01-27  
**Next Review**: 2025-02-27  
**Maintained By**: DafnckMachine Development Team 