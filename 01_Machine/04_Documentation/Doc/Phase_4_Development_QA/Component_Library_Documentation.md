# Component Library Documentation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Component Library Reference
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active

## Overview

This document provides comprehensive documentation for the DafnckMachine v3.1 component library, including specifications, usage guidelines, and implementation examples for all reusable UI components.

## Component Library Structure

### Atomic Design Hierarchy
```
components/
├── atoms/              # Basic building blocks
│   ├── Button/
│   ├── Input/
│   ├── Typography/
│   ├── Icon/
│   └── Avatar/
├── molecules/          # Simple combinations
│   ├── SearchInput/
│   ├── FormField/
│   ├── Card/
│   ├── Dropdown/
│   └── Pagination/
├── organisms/          # Complex assemblies
│   ├── Navigation/
│   ├── Header/
│   ├── Sidebar/
│   ├── DataTable/
│   └── Modal/
└── templates/          # Page layouts
    ├── DashboardLayout/
    ├── AuthLayout/
    └── ContentLayout/
```

## Atoms (Basic Components)

### Button Component

#### Props Interface
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger' | 'ghost' | 'link';
  size: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
  children: React.ReactNode;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  type?: 'button' | 'submit' | 'reset';
  'aria-label'?: string;
}
```

#### Usage Examples
```typescript
// Primary button
<Button variant="primary" size="medium">
  Save Changes
</Button>

// Button with icon
<Button variant="secondary" icon={<SaveIcon />} iconPosition="left">
  Save Draft
</Button>

// Loading state
<Button variant="primary" loading disabled>
  Processing...
</Button>

// Full width button
<Button variant="primary" fullWidth>
  Continue
</Button>
```

#### Styling Variants
- **Primary**: Main call-to-action buttons
- **Secondary**: Secondary actions and alternatives
- **Danger**: Destructive actions (delete, remove)
- **Ghost**: Subtle actions with transparent background
- **Link**: Text-only buttons that look like links

### Input Component

#### Props Interface
```typescript
interface InputProps {
  type: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url';
  value: string;
  onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
  placeholder?: string;
  disabled?: boolean;
  error?: boolean;
  helperText?: string;
  label?: string;
  required?: boolean;
  autoComplete?: string;
  maxLength?: number;
  pattern?: string;
  'aria-describedby'?: string;
}
```

#### Usage Examples
```typescript
// Basic input
<Input
  type="text"
  value={value}
  onChange={handleChange}
  placeholder="Enter your name"
  label="Full Name"
/>

// Input with error state
<Input
  type="email"
  value={email}
  onChange={handleEmailChange}
  label="Email Address"
  error={!!emailError}
  helperText={emailError || "We'll never share your email"}
  required
/>

// Password input
<Input
  type="password"
  value={password}
  onChange={handlePasswordChange}
  label="Password"
  autoComplete="current-password"
/>
```

### Typography Component

#### Props Interface
```typescript
interface TypographyProps {
  variant: 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6' | 'body1' | 'body2' | 'caption' | 'overline';
  component?: keyof JSX.IntrinsicElements;
  color?: 'primary' | 'secondary' | 'error' | 'warning' | 'info' | 'success' | 'textPrimary' | 'textSecondary';
  align?: 'left' | 'center' | 'right' | 'justify';
  gutterBottom?: boolean;
  noWrap?: boolean;
  children: React.ReactNode;
}
```

#### Usage Examples
```typescript
// Headings
<Typography variant="h1" component="h1" gutterBottom>
  Page Title
</Typography>

<Typography variant="h2" component="h2" color="primary">
  Section Header
</Typography>

// Body text
<Typography variant="body1" color="textPrimary">
  This is the main body text with standard styling.
</Typography>

<Typography variant="caption" color="textSecondary" align="center">
  Helper text or captions
</Typography>
```

### Icon Component

#### Props Interface
```typescript
interface IconProps {
  name: string;
  size?: 'small' | 'medium' | 'large' | number;
  color?: string;
  className?: string;
  'aria-label'?: string;
  'aria-hidden'?: boolean;
}
```

#### Usage Examples
```typescript
// Standard icons
<Icon name="search" size="medium" aria-label="Search" />
<Icon name="user" size="large" color="primary" />
<Icon name="close" size={24} aria-label="Close dialog" />

// Decorative icons
<Icon name="star" aria-hidden="true" />
```

### Avatar Component

#### Props Interface
```typescript
interface AvatarProps {
  src?: string;
  alt?: string;
  size?: 'small' | 'medium' | 'large' | number;
  variant?: 'circular' | 'rounded' | 'square';
  fallback?: string;
  color?: 'primary' | 'secondary' | 'random';
  onClick?: () => void;
}
```

#### Usage Examples
```typescript
// Image avatar
<Avatar
  src="/path/to/image.jpg"
  alt="John Doe"
  size="large"
  variant="circular"
/>

// Fallback avatar with initials
<Avatar
  fallback="JD"
  size="medium"
  color="primary"
  alt="John Doe"
/>

// Clickable avatar
<Avatar
  src="/path/to/image.jpg"
  alt="User profile"
  onClick={handleProfileClick}
/>
```

## Molecules (Composite Components)

### SearchInput Component

#### Props Interface
```typescript
interface SearchInputProps {
  value: string;
  onChange: (value: string) => void;
  onSearch: () => void;
  onClear?: () => void;
  placeholder?: string;
  loading?: boolean;
  disabled?: boolean;
  suggestions?: string[];
  onSuggestionSelect?: (suggestion: string) => void;
}
```

#### Usage Examples
```typescript
// Basic search
<SearchInput
  value={searchTerm}
  onChange={setSearchTerm}
  onSearch={handleSearch}
  placeholder="Search products..."
/>

// Search with suggestions
<SearchInput
  value={searchTerm}
  onChange={setSearchTerm}
  onSearch={handleSearch}
  suggestions={searchSuggestions}
  onSuggestionSelect={handleSuggestionSelect}
  loading={isSearching}
/>
```

### FormField Component

#### Props Interface
```typescript
interface FormFieldProps {
  label: string;
  required?: boolean;
  error?: string;
  helperText?: string;
  children: React.ReactElement;
  htmlFor?: string;
}
```

#### Usage Examples
```typescript
// Form field wrapper
<FormField
  label="Email Address"
  required
  error={emailError}
  helperText="We'll use this to send you updates"
  htmlFor="email-input"
>
  <Input
    id="email-input"
    type="email"
    value={email}
    onChange={handleEmailChange}
  />
</FormField>
```

### Card Component

#### Props Interface
```typescript
interface CardProps {
  variant?: 'elevated' | 'outlined' | 'filled';
  padding?: 'none' | 'small' | 'medium' | 'large';
  clickable?: boolean;
  onClick?: () => void;
  header?: React.ReactNode;
  footer?: React.ReactNode;
  children: React.ReactNode;
}
```

#### Usage Examples
```typescript
// Basic card
<Card variant="elevated" padding="medium">
  <Typography variant="h3">Card Title</Typography>
  <Typography variant="body1">Card content goes here.</Typography>
</Card>

// Card with header and footer
<Card
  variant="outlined"
  header={
    <Typography variant="h4" color="primary">
      Product Details
    </Typography>
  }
  footer={
    <Button variant="primary" fullWidth>
      Add to Cart
    </Button>
  }
>
  <Typography variant="body1">Product description...</Typography>
</Card>

// Clickable card
<Card variant="elevated" clickable onClick={handleCardClick}>
  <Typography variant="h4">Clickable Card</Typography>
  <Typography variant="body2">Click anywhere to navigate</Typography>
</Card>
```

### Dropdown Component

#### Props Interface
```typescript
interface DropdownProps {
  trigger: React.ReactElement;
  items: DropdownItem[];
  placement?: 'bottom-start' | 'bottom-end' | 'top-start' | 'top-end';
  disabled?: boolean;
  onSelect?: (item: DropdownItem) => void;
}

interface DropdownItem {
  id: string;
  label: string;
  icon?: React.ReactNode;
  disabled?: boolean;
  divider?: boolean;
}
```

#### Usage Examples
```typescript
// Basic dropdown
<Dropdown
  trigger={<Button variant="secondary">Options</Button>}
  items={[
    { id: 'edit', label: 'Edit', icon: <EditIcon /> },
    { id: 'delete', label: 'Delete', icon: <DeleteIcon /> },
    { id: 'divider', divider: true },
    { id: 'archive', label: 'Archive', icon: <ArchiveIcon /> }
  ]}
  onSelect={handleDropdownSelect}
/>

// User menu dropdown
<Dropdown
  trigger={<Avatar src={user.avatar} alt={user.name} />}
  placement="bottom-end"
  items={[
    { id: 'profile', label: 'Profile', icon: <UserIcon /> },
    { id: 'settings', label: 'Settings', icon: <SettingsIcon /> },
    { id: 'divider', divider: true },
    { id: 'logout', label: 'Logout', icon: <LogoutIcon /> }
  ]}
  onSelect={handleUserMenuSelect}
/>
```

## Organisms (Complex Components)

### Navigation Component

#### Props Interface
```typescript
interface NavigationProps {
  items: NavigationItem[];
  activeItem?: string;
  onItemClick?: (item: NavigationItem) => void;
  variant?: 'horizontal' | 'vertical';
  collapsed?: boolean;
  logo?: React.ReactNode;
  userMenu?: React.ReactNode;
}

interface NavigationItem {
  id: string;
  label: string;
  icon?: React.ReactNode;
  href?: string;
  children?: NavigationItem[];
  badge?: string | number;
}
```

#### Usage Examples
```typescript
// Main navigation
<Navigation
  variant="horizontal"
  logo={<Logo />}
  items={[
    { id: 'dashboard', label: 'Dashboard', icon: <DashboardIcon />, href: '/dashboard' },
    { id: 'projects', label: 'Projects', icon: <ProjectsIcon />, href: '/projects', badge: 3 },
    { id: 'settings', label: 'Settings', icon: <SettingsIcon />, href: '/settings' }
  ]}
  activeItem="dashboard"
  userMenu={<UserDropdown />}
  onItemClick={handleNavigationClick}
/>

// Sidebar navigation
<Navigation
  variant="vertical"
  collapsed={sidebarCollapsed}
  items={navigationItems}
  activeItem={activeNavItem}
  onItemClick={handleSidebarClick}
/>
```

### DataTable Component

#### Props Interface
```typescript
interface DataTableProps<T> {
  data: T[];
  columns: TableColumn<T>[];
  loading?: boolean;
  pagination?: PaginationConfig;
  sorting?: SortingConfig;
  selection?: SelectionConfig;
  onRowClick?: (row: T) => void;
  emptyState?: React.ReactNode;
}

interface TableColumn<T> {
  key: keyof T;
  title: string;
  sortable?: boolean;
  width?: string | number;
  render?: (value: any, row: T) => React.ReactNode;
}
```

#### Usage Examples
```typescript
// Basic data table
<DataTable
  data={users}
  columns={[
    { key: 'name', title: 'Name', sortable: true },
    { key: 'email', title: 'Email', sortable: true },
    { key: 'role', title: 'Role' },
    {
      key: 'actions',
      title: 'Actions',
      render: (_, user) => (
        <Dropdown
          trigger={<Button variant="ghost">Actions</Button>}
          items={getActionItems(user)}
        />
      )
    }
  ]}
  pagination={{
    page: currentPage,
    pageSize: 10,
    total: totalUsers,
    onPageChange: handlePageChange
  }}
  loading={isLoading}
  onRowClick={handleUserClick}
/>
```

### Modal Component

#### Props Interface
```typescript
interface ModalProps {
  open: boolean;
  onClose: () => void;
  title?: string;
  size?: 'small' | 'medium' | 'large' | 'fullscreen';
  closeOnBackdrop?: boolean;
  closeOnEscape?: boolean;
  header?: React.ReactNode;
  footer?: React.ReactNode;
  children: React.ReactNode;
}
```

#### Usage Examples
```typescript
// Confirmation modal
<Modal
  open={showDeleteModal}
  onClose={() => setShowDeleteModal(false)}
  title="Confirm Deletion"
  size="small"
  footer={
    <>
      <Button variant="ghost" onClick={() => setShowDeleteModal(false)}>
        Cancel
      </Button>
      <Button variant="danger" onClick={handleDelete}>
        Delete
      </Button>
    </>
  }
>
  <Typography variant="body1">
    Are you sure you want to delete this item? This action cannot be undone.
  </Typography>
</Modal>

// Form modal
<Modal
  open={showEditModal}
  onClose={() => setShowEditModal(false)}
  title="Edit User"
  size="medium"
>
  <UserForm
    user={selectedUser}
    onSubmit={handleUserUpdate}
    onCancel={() => setShowEditModal(false)}
  />
</Modal>
```

## Templates (Layout Components)

### DashboardLayout Component

#### Props Interface
```typescript
interface DashboardLayoutProps {
  sidebar?: React.ReactNode;
  header?: React.ReactNode;
  children: React.ReactNode;
  sidebarCollapsed?: boolean;
  onSidebarToggle?: () => void;
}
```

#### Usage Examples
```typescript
// Dashboard layout
<DashboardLayout
  header={<DashboardHeader />}
  sidebar={<DashboardSidebar />}
  sidebarCollapsed={sidebarCollapsed}
  onSidebarToggle={toggleSidebar}
>
  <DashboardContent />
</DashboardLayout>
```

## Component Styling

### CSS-in-JS with Styled Components
```typescript
// Styled component example
const StyledButton = styled.button<{ variant: string; size: string }>`
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: ${({ theme }) => theme.borderRadius.medium};
  font-family: ${({ theme }) => theme.typography.fontFamily};
  font-weight: ${({ theme }) => theme.typography.fontWeight.medium};
  cursor: pointer;
  transition: all 0.2s ease-in-out;

  ${({ variant, theme }) => getVariantStyles(variant, theme)}
  ${({ size, theme }) => getSizeStyles(size, theme)}

  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  &:focus-visible {
    outline: 2px solid ${({ theme }) => theme.colors.primary.main};
    outline-offset: 2px;
  }
`;
```

### CSS Modules Approach
```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: var(--border-radius-medium);
  font-family: var(--font-family);
  font-weight: var(--font-weight-medium);
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}

.button--primary {
  background-color: var(--color-primary);
  color: var(--color-primary-contrast);
}

.button--secondary {
  background-color: var(--color-secondary);
  color: var(--color-secondary-contrast);
}

.button--small {
  padding: var(--spacing-xs) var(--spacing-sm);
  font-size: var(--font-size-sm);
}

.button--medium {
  padding: var(--spacing-sm) var(--spacing-md);
  font-size: var(--font-size-base);
}
```

## Accessibility Guidelines

### ARIA Implementation
```typescript
// Accessible button with proper ARIA attributes
<Button
  variant="primary"
  onClick={handleSubmit}
  disabled={isSubmitting}
  aria-label={isSubmitting ? "Submitting form" : "Submit form"}
  aria-busy={isSubmitting}
>
  {isSubmitting ? "Submitting..." : "Submit"}
</Button>

// Accessible modal with focus management
<Modal
  open={isOpen}
  onClose={handleClose}
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
>
  <Typography id="modal-title" variant="h2">
    Modal Title
  </Typography>
  <Typography id="modal-description" variant="body1">
    Modal description content
  </Typography>
</Modal>
```

### Keyboard Navigation
- All interactive components support keyboard navigation
- Tab order follows logical flow
- Enter and Space keys activate buttons
- Escape key closes modals and dropdowns
- Arrow keys navigate through lists and menus

### Screen Reader Support
- Semantic HTML elements used where appropriate
- ARIA labels and descriptions provided
- Live regions for dynamic content updates
- Proper heading hierarchy maintained

## Testing Guidelines

### Component Testing
```typescript
// Button component test
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button variant="primary">Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button variant="primary" onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button variant="primary" loading>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Accessibility Testing
```typescript
// Accessibility test example
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

describe('Button Accessibility', () => {
  it('should not have accessibility violations', async () => {
    const { container } = render(
      <Button variant="primary">Accessible Button</Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

## Performance Considerations

### Component Optimization
- Use React.memo for expensive components
- Implement useMemo for expensive calculations
- Use useCallback for event handlers
- Lazy load heavy components

### Bundle Optimization
- Tree shaking for unused components
- Code splitting at component level
- Optimized imports to reduce bundle size

## Documentation Standards

### Component Documentation Template
```typescript
/**
 * Button component for user interactions
 * 
 * @example
 * ```tsx
 * <Button variant="primary" onClick={handleClick}>
 *   Click me
 * </Button>
 * ```
 */
export interface ButtonProps {
  /** Visual style variant */
  variant: 'primary' | 'secondary' | 'danger';
  /** Size of the button */
  size: 'small' | 'medium' | 'large';
  /** Whether the button is disabled */
  disabled?: boolean;
  /** Content of the button */
  children: React.ReactNode;
  /** Click event handler */
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
}
```

## Version History

### v3.1.0 (Current)
- Initial component library implementation
- Atomic design structure
- Comprehensive accessibility support
- TypeScript interfaces for all components

### Future Enhancements
- Animation and transition components
- Advanced data visualization components
- Mobile-specific component variants
- Dark mode theme support

---

**Document Status**: Active  
**Next Review**: 2025-04-27  
**Maintained By**: Frontend Development Team 