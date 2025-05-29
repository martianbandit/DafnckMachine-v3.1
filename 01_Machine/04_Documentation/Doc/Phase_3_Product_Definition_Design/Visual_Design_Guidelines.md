# Visual Design Guidelines
## DafnckMachine v3.1 - Visual Design System & Branding Integration

### Document Overview
This document establishes comprehensive visual design guidelines for DafnckMachine v3.1, ensuring consistent brand integration, visual identity, and component library specifications across all user interfaces and touchpoints.

### Visual Identity System

#### Brand Integration Framework
- **Primary Brand Colors**
  - Primary: #2563EB (Blue 600)
  - Secondary: #7C3AED (Violet 600)
  - Accent: #059669 (Emerald 600)
  - Neutral: #374151 (Gray 700)

- **Typography System**
  - Primary Font: Inter (Sans-serif)
  - Secondary Font: JetBrains Mono (Monospace)
  - Heading Scale: 2.25rem, 1.875rem, 1.5rem, 1.25rem, 1.125rem, 1rem
  - Body Text: 1rem (16px) base size
  - Line Height: 1.5 for body text, 1.2 for headings

- **Spacing System**
  - Base Unit: 4px
  - Scale: 4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px, 96px, 128px
  - Component Padding: 16px standard, 24px large, 8px compact
  - Section Margins: 32px between sections, 64px between major blocks

#### Component Visual Specifications

##### Buttons
- **Primary Button**
  - Background: Primary brand color
  - Text: White
  - Border Radius: 6px
  - Padding: 12px 24px
  - Font Weight: 500
  - Hover State: Darken by 10%

- **Secondary Button**
  - Background: Transparent
  - Text: Primary brand color
  - Border: 1px solid primary color
  - Same dimensions as primary

- **Tertiary Button**
  - Background: Transparent
  - Text: Neutral color
  - No border
  - Underline on hover

##### Form Elements
- **Input Fields**
  - Border: 1px solid #D1D5DB
  - Border Radius: 6px
  - Padding: 12px 16px
  - Focus State: Primary color border, box-shadow
  - Error State: Red border and text

- **Labels**
  - Font Weight: 500
  - Color: #374151
  - Margin Bottom: 8px

##### Cards & Containers
- **Card Component**
  - Background: White
  - Border: 1px solid #E5E7EB
  - Border Radius: 8px
  - Box Shadow: 0 1px 3px rgba(0, 0, 0, 0.1)
  - Padding: 24px

- **Panel Component**
  - Background: #F9FAFB
  - Border Radius: 8px
  - Padding: 16px

#### Iconography Standards
- **Icon Library**: Heroicons or Lucide React
- **Icon Sizes**: 16px, 20px, 24px, 32px
- **Icon Colors**: Match text color or use brand colors for emphasis
- **Icon Spacing**: 8px margin from adjacent text

#### Layout Grid System
- **Container Max Width**: 1200px
- **Grid Columns**: 12-column system
- **Gutters**: 24px between columns
- **Breakpoints**:
  - Mobile: 320px - 767px
  - Tablet: 768px - 1023px
  - Desktop: 1024px+

#### Animation & Transitions
- **Duration**: 150ms for micro-interactions, 300ms for page transitions
- **Easing**: ease-in-out for most transitions
- **Hover Effects**: Scale 1.05 for buttons, opacity 0.8 for images
- **Loading States**: Skeleton screens with subtle pulse animation

#### Accessibility Considerations
- **Color Contrast**: Minimum 4.5:1 for normal text, 3:1 for large text
- **Focus Indicators**: Visible focus rings with 2px outline
- **Touch Targets**: Minimum 44px for interactive elements
- **Text Scaling**: Support up to 200% zoom without horizontal scrolling

### Implementation Guidelines

#### CSS Custom Properties
```css
:root {
  --color-primary: #2563EB;
  --color-secondary: #7C3AED;
  --color-accent: #059669;
  --color-neutral: #374151;
  --font-primary: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --border-radius: 6px;
  --border-radius-lg: 8px;
}
```

#### Component Classes
- Use BEM methodology for CSS class naming
- Prefix all component classes with `dm-` (DafnckMachine)
- Example: `.dm-button`, `.dm-button--primary`, `.dm-button--large`

#### Responsive Design Patterns
- Mobile-first approach
- Progressive enhancement for larger screens
- Flexible layouts using CSS Grid and Flexbox
- Responsive typography using clamp() function

### Quality Assurance
- Visual regression testing for all components
- Cross-browser compatibility testing
- Accessibility audit using automated tools
- Design system documentation updates with each component change

### Maintenance & Updates
- Monthly design system review
- Component usage analytics tracking
- Feedback collection from development team
- Version control for design system changes

---

**Document Status**: Active  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-27  
**Owner**: @branding-agent, @ui-designer-agent 