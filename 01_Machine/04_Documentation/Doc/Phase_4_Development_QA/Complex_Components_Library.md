# Complex Components Library - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Complex Components Library Documentation
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive documentation for the complex components library in DafnckMachine v3.1, building upon the base components to create sophisticated, feature-rich UI elements that handle complex user interactions, data management, and advanced functionality.

## Library Architecture

### Complex Component Principles
- **Composition-Based**: Built from base components using composition patterns
- **Feature-Rich**: Advanced functionality with multiple interaction modes
- **Data-Driven**: Integrated with state management and API layers
- **Accessibility-First**: Enhanced accessibility features for complex interactions
- **Performance-Optimized**: Efficient rendering and memory management

### Component Categories
1. **Data Display**: Tables, lists, grids, charts, dashboards
2. **Navigation**: Menus, tabs, breadcrumbs, pagination, steppers
3. **Input & Forms**: Multi-step forms, file uploads, rich editors
4. **Feedback & Overlays**: Modals, drawers, notifications, tooltips
5. **Layout & Structure**: Sidebars, headers, panels, resizable containers

## Data Display Components

### Advanced Data Table

```typescript
// src/components/complex/DataTable/DataTable.tsx
import React, { useState, useMemo, useCallback } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';
import { flexRender, getCoreRowModel, useReactTable, getSortedRowModel, getFilteredRowModel, getPaginationRowModel } from '@tanstack/react-table';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/atoms/Table';
import { Button } from '@/components/atoms/Button';
import { Input } from '@/components/atoms/Input';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/atoms/Select';
import { Checkbox } from '@/components/atoms/Checkbox';
import { Badge } from '@/components/atoms/Badge';
import { cn } from '@/utils/cn';

export interface DataTableColumn<T> {
  id: string;
  header: string;
  accessorKey?: keyof T;
  cell?: (props: { row: T; value: any }) => React.ReactNode;
  sortable?: boolean;
  filterable?: boolean;
  width?: number;
  minWidth?: number;
  maxWidth?: number;
}

export interface DataTableProps<T> {
  data: T[];
  columns: DataTableColumn<T>[];
  loading?: boolean;
  error?: string;
  pagination?: {
    pageSize?: number;
    showSizeSelector?: boolean;
    showInfo?: boolean;
  };
  selection?: {
    enabled?: boolean;
    onSelectionChange?: (selectedRows: T[]) => void;
  };
  sorting?: {
    enabled?: boolean;
    defaultSort?: { id: string; desc: boolean }[];
  };
  filtering?: {
    enabled?: boolean;
    globalFilter?: boolean;
    columnFilters?: boolean;
  };
  virtualization?: {
    enabled?: boolean;
    estimateSize?: number;
  };
  actions?: {
    onRowClick?: (row: T) => void;
    onRowDoubleClick?: (row: T) => void;
    contextMenu?: (row: T) => React.ReactNode;
  };
  className?: string;
}

export function DataTable<T extends Record<string, any>>({
  data,
  columns,
  loading = false,
  error,
  pagination = { pageSize: 10, showSizeSelector: true, showInfo: true },
  selection = { enabled: false },
  sorting = { enabled: true },
  filtering = { enabled: true, globalFilter: true, columnFilters: true },
  virtualization = { enabled: false, estimateSize: 50 },
  actions,
  className,
}: DataTableProps<T>) {
  const [globalFilter, setGlobalFilter] = useState('');
  const [columnFilters, setColumnFilters] = useState<Record<string, string>>({});
  const [rowSelection, setRowSelection] = useState({});

  // Enhanced columns with selection
  const enhancedColumns = useMemo(() => {
    const cols = [...columns];
    
    if (selection.enabled) {
      cols.unshift({
        id: 'select',
        header: ({ table }) => (
          <Checkbox
            checked={table.getIsAllPageRowsSelected()}
            onCheckedChange={(value) => table.toggleAllPageRowsSelected(!!value)}
            aria-label="Select all"
          />
        ),
        cell: ({ row }) => (
          <Checkbox
            checked={row.getIsSelected()}
            onCheckedChange={(value) => row.toggleSelected(!!value)}
            aria-label="Select row"
          />
        ),
        enableSorting: false,
        enableHiding: false,
        width: 50,
      });
    }

    return cols;
  }, [columns, selection.enabled]);

  const table = useReactTable({
    data,
    columns: enhancedColumns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    onRowSelectionChange: setRowSelection,
    onGlobalFilterChange: setGlobalFilter,
    globalFilterFn: 'includesString',
    state: {
      rowSelection,
      globalFilter,
    },
    initialState: {
      pagination: {
        pageSize: pagination.pageSize || 10,
      },
      sorting: sorting.defaultSort || [],
    },
  });

  // Virtualization setup
  const { rows } = table.getRowModel();
  const parentRef = React.useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => virtualization.estimateSize || 50,
    enabled: virtualization.enabled,
  });

  // Handle selection changes
  React.useEffect(() => {
    if (selection.onSelectionChange) {
      const selectedRows = table.getFilteredSelectedRowModel().rows.map(row => row.original);
      selection.onSelectionChange(selectedRows);
    }
  }, [rowSelection, selection.onSelectionChange, table]);

  if (error) {
    return (
      <div className="flex items-center justify-center p-8 text-destructive">
        <p>Error loading data: {error}</p>
      </div>
    );
  }

  return (
    <div className={cn('space-y-4', className)}>
      {/* Toolbar */}
      {(filtering.globalFilter || selection.enabled) && (
        <div className="flex items-center justify-between">
          <div className="flex items-center space-x-2">
            {filtering.globalFilter && (
              <Input
                placeholder="Search all columns..."
                value={globalFilter}
                onChange={(e) => setGlobalFilter(e.target.value)}
                className="max-w-sm"
              />
            )}
          </div>
          
          {selection.enabled && (
            <div className="flex items-center space-x-2">
              <span className="text-sm text-muted-foreground">
                {table.getFilteredSelectedRowModel().rows.length} of{' '}
                {table.getFilteredRowModel().rows.length} row(s) selected
              </span>
            </div>
          )}
        </div>
      )}

      {/* Table */}
      <div className="rounded-md border">
        <div ref={parentRef} className={virtualization.enabled ? 'h-[400px] overflow-auto' : ''}>
          <Table>
            <TableHeader>
              {table.getHeaderGroups().map((headerGroup) => (
                <TableRow key={headerGroup.id}>
                  {headerGroup.headers.map((header) => (
                    <TableHead
                      key={header.id}
                      style={{ width: header.getSize() }}
                      className={cn(
                        header.column.getCanSort() && 'cursor-pointer select-none hover:bg-muted',
                      )}
                      onClick={header.column.getToggleSortingHandler()}
                    >
                      <div className="flex items-center space-x-2">
                        {flexRender(header.column.columnDef.header, header.getContext())}
                        {header.column.getIsSorted() && (
                          <span className="text-xs">
                            {header.column.getIsSorted() === 'desc' ? '↓' : '↑'}
                          </span>
                        )}
                      </div>
                    </TableHead>
                  ))}
                </TableRow>
              ))}
            </TableHeader>
            <TableBody>
              {loading ? (
                Array.from({ length: pagination.pageSize || 10 }).map((_, index) => (
                  <TableRow key={index}>
                    {enhancedColumns.map((_, colIndex) => (
                      <TableCell key={colIndex}>
                        <div className="h-4 bg-muted animate-pulse rounded" />
                      </TableCell>
                    ))}
                  </TableRow>
                ))
              ) : virtualization.enabled ? (
                virtualizer.getVirtualItems().map((virtualRow) => {
                  const row = rows[virtualRow.index];
                  return (
                    <TableRow
                      key={row.id}
                      data-state={row.getIsSelected() && 'selected'}
                      style={{
                        height: `${virtualRow.size}px`,
                        transform: `translateY(${virtualRow.start}px)`,
                      }}
                      onClick={() => actions?.onRowClick?.(row.original)}
                      onDoubleClick={() => actions?.onRowDoubleClick?.(row.original)}
                      className={cn(
                        actions?.onRowClick && 'cursor-pointer hover:bg-muted/50',
                      )}
                    >
                      {row.getVisibleCells().map((cell) => (
                        <TableCell key={cell.id}>
                          {flexRender(cell.column.columnDef.cell, cell.getContext())}
                        </TableCell>
                      ))}
                    </TableRow>
                  );
                })
              ) : (
                table.getRowModel().rows.map((row) => (
                  <TableRow
                    key={row.id}
                    data-state={row.getIsSelected() && 'selected'}
                    onClick={() => actions?.onRowClick?.(row.original)}
                    onDoubleClick={() => actions?.onRowDoubleClick?.(row.original)}
                    className={cn(
                      actions?.onRowClick && 'cursor-pointer hover:bg-muted/50',
                    )}
                  >
                    {row.getVisibleCells().map((cell) => (
                      <TableCell key={cell.id}>
                        {flexRender(cell.column.columnDef.cell, cell.getContext())}
                      </TableCell>
                    ))}
                  </TableRow>
                ))
              )}
            </TableBody>
          </Table>
        </div>
      </div>

      {/* Pagination */}
      {pagination && (
        <div className="flex items-center justify-between px-2">
          <div className="flex items-center space-x-6 lg:space-x-8">
            {pagination.showSizeSelector && (
              <div className="flex items-center space-x-2">
                <p className="text-sm font-medium">Rows per page</p>
                <Select
                  value={`${table.getState().pagination.pageSize}`}
                  onValueChange={(value) => table.setPageSize(Number(value))}
                >
                  <SelectTrigger className="h-8 w-[70px]">
                    <SelectValue placeholder={table.getState().pagination.pageSize} />
                  </SelectTrigger>
                  <SelectContent side="top">
                    {[10, 20, 30, 40, 50].map((pageSize) => (
                      <SelectItem key={pageSize} value={`${pageSize}`}>
                        {pageSize}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
              </div>
            )}
            
            {pagination.showInfo && (
              <div className="flex w-[100px] items-center justify-center text-sm font-medium">
                Page {table.getState().pagination.pageIndex + 1} of{' '}
                {table.getPageCount()}
              </div>
            )}
          </div>
          
          <div className="flex items-center space-x-2">
            <Button
              variant="outline"
              className="hidden h-8 w-8 p-0 lg:flex"
              onClick={() => table.setPageIndex(0)}
              disabled={!table.getCanPreviousPage()}
            >
              <span className="sr-only">Go to first page</span>
              ⟪
            </Button>
            <Button
              variant="outline"
              className="h-8 w-8 p-0"
              onClick={() => table.previousPage()}
              disabled={!table.getCanPreviousPage()}
            >
              <span className="sr-only">Go to previous page</span>
              ⟨
            </Button>
            <Button
              variant="outline"
              className="h-8 w-8 p-0"
              onClick={() => table.nextPage()}
              disabled={!table.getCanNextPage()}
            >
              <span className="sr-only">Go to next page</span>
              ⟩
            </Button>
            <Button
              variant="outline"
              className="hidden h-8 w-8 p-0 lg:flex"
              onClick={() => table.setPageIndex(table.getPageCount() - 1)}
              disabled={!table.getCanNextPage()}
            >
              <span className="sr-only">Go to last page</span>
              ⟫
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### Dashboard Component

```typescript
// src/components/complex/Dashboard/Dashboard.tsx
import React from 'react';
import { Card } from '@/components/molecules/Card';
import { Grid } from '@/components/atoms/Grid';
import { Heading } from '@/components/atoms/Heading';
import { Text } from '@/components/atoms/Text';
import { Badge } from '@/components/atoms/Badge';
import { Progress } from '@/components/atoms/Progress';
import { cn } from '@/utils/cn';

export interface DashboardWidget {
  id: string;
  title: string;
  type: 'metric' | 'chart' | 'list' | 'progress' | 'custom';
  size: 'sm' | 'md' | 'lg' | 'xl';
  data: any;
  component?: React.ComponentType<any>;
}

export interface DashboardProps {
  title?: string;
  widgets: DashboardWidget[];
  layout?: 'grid' | 'masonry' | 'custom';
  className?: string;
}

const MetricWidget: React.FC<{ widget: DashboardWidget }> = ({ widget }) => (
  <Card className="p-6">
    <div className="flex items-center justify-between">
      <div>
        <Text size="sm" color="muted" className="mb-1">
          {widget.title}
        </Text>
        <Heading level={2} className="text-2xl font-bold">
          {widget.data.value}
        </Heading>
        {widget.data.change && (
          <div className="flex items-center mt-2">
            <Badge variant={widget.data.change > 0 ? 'success' : 'destructive'}>
              {widget.data.change > 0 ? '+' : ''}{widget.data.change}%
            </Badge>
            <Text size="sm" color="muted" className="ml-2">
              vs last period
            </Text>
          </div>
        )}
      </div>
      {widget.data.icon && (
        <div className="text-muted-foreground">
          {widget.data.icon}
        </div>
      )}
    </div>
  </Card>
);

const ProgressWidget: React.FC<{ widget: DashboardWidget }> = ({ widget }) => (
  <Card className="p-6">
    <Text size="sm" color="muted" className="mb-2">
      {widget.title}
    </Text>
    <div className="space-y-2">
      <div className="flex justify-between">
        <Text size="sm">{widget.data.label}</Text>
        <Text size="sm" color="muted">
          {widget.data.current}/{widget.data.total}
        </Text>
      </div>
      <Progress value={(widget.data.current / widget.data.total) * 100} />
    </div>
  </Card>
);

const ListWidget: React.FC<{ widget: DashboardWidget }> = ({ widget }) => (
  <Card className="p-6">
    <Heading level={3} className="mb-4">
      {widget.title}
    </Heading>
    <div className="space-y-3">
      {widget.data.items.map((item: any, index: number) => (
        <div key={index} className="flex items-center justify-between">
          <div>
            <Text size="sm" className="font-medium">
              {item.title}
            </Text>
            <Text size="xs" color="muted">
              {item.subtitle}
            </Text>
          </div>
          {item.badge && (
            <Badge variant={item.badge.variant}>
              {item.badge.text}
            </Badge>
          )}
        </div>
      ))}
    </div>
  </Card>
);

export const Dashboard: React.FC<DashboardProps> = ({
  title,
  widgets,
  layout = 'grid',
  className,
}) => {
  const getWidgetSize = (size: string) => {
    switch (size) {
      case 'sm': return 'col-span-1';
      case 'md': return 'col-span-2';
      case 'lg': return 'col-span-3';
      case 'xl': return 'col-span-4';
      default: return 'col-span-1';
    }
  };

  const renderWidget = (widget: DashboardWidget) => {
    if (widget.component) {
      const Component = widget.component;
      return <Component widget={widget} />;
    }

    switch (widget.type) {
      case 'metric':
        return <MetricWidget widget={widget} />;
      case 'progress':
        return <ProgressWidget widget={widget} />;
      case 'list':
        return <ListWidget widget={widget} />;
      default:
        return (
          <Card className="p-6">
            <Text>Unsupported widget type: {widget.type}</Text>
          </Card>
        );
    }
  };

  return (
    <div className={cn('space-y-6', className)}>
      {title && (
        <Heading level={1} className="text-3xl font-bold">
          {title}
        </Heading>
      )}
      
      <div className="grid grid-cols-4 gap-6">
        {widgets.map((widget) => (
          <div key={widget.id} className={getWidgetSize(widget.size)}>
            {renderWidget(widget)}
          </div>
        ))}
      </div>
    </div>
  );
};
```

## Navigation Components

### Advanced Navigation Menu

```typescript
// src/components/complex/NavigationMenu/NavigationMenu.tsx
import React, { useState } from 'react';
import { Link } from 'react-router-dom';
import { ChevronDown, ChevronRight } from 'lucide-react';
import { cn } from '@/utils/cn';

export interface NavigationItem {
  id: string;
  label: string;
  href?: string;
  icon?: React.ReactNode;
  badge?: {
    text: string;
    variant?: 'default' | 'destructive' | 'success' | 'warning';
  };
  children?: NavigationItem[];
  disabled?: boolean;
}

export interface NavigationMenuProps {
  items: NavigationItem[];
  orientation?: 'horizontal' | 'vertical';
  variant?: 'default' | 'sidebar' | 'header';
  collapsible?: boolean;
  defaultExpanded?: string[];
  onItemClick?: (item: NavigationItem) => void;
  className?: string;
}

export const NavigationMenu: React.FC<NavigationMenuProps> = ({
  items,
  orientation = 'horizontal',
  variant = 'default',
  collapsible = true,
  defaultExpanded = [],
  onItemClick,
  className,
}) => {
  const [expandedItems, setExpandedItems] = useState<Set<string>>(
    new Set(defaultExpanded)
  );

  const toggleExpanded = (itemId: string) => {
    if (!collapsible) return;
    
    setExpandedItems(prev => {
      const newSet = new Set(prev);
      if (newSet.has(itemId)) {
        newSet.delete(itemId);
      } else {
        newSet.add(itemId);
      }
      return newSet;
    });
  };

  const renderNavigationItem = (item: NavigationItem, level = 0) => {
    const hasChildren = item.children && item.children.length > 0;
    const isExpanded = expandedItems.has(item.id);
    const isDisabled = item.disabled;

    const itemContent = (
      <div
        className={cn(
          'flex items-center justify-between w-full px-3 py-2 text-sm font-medium rounded-md transition-colors',
          {
            'text-muted-foreground cursor-not-allowed': isDisabled,
            'hover:bg-accent hover:text-accent-foreground cursor-pointer': !isDisabled,
            'bg-accent text-accent-foreground': isExpanded && hasChildren,
            'pl-6': level > 0 && variant === 'sidebar',
            'pl-9': level > 1 && variant === 'sidebar',
          }
        )}
        onClick={() => {
          if (isDisabled) return;
          
          if (hasChildren) {
            toggleExpanded(item.id);
          } else {
            onItemClick?.(item);
          }
        }}
      >
        <div className="flex items-center space-x-2">
          {item.icon && (
            <span className="flex-shrink-0">
              {item.icon}
            </span>
          )}
          <span>{item.label}</span>
          {item.badge && (
            <span className={cn(
              'inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium',
              {
                'bg-primary text-primary-foreground': item.badge.variant === 'default',
                'bg-destructive text-destructive-foreground': item.badge.variant === 'destructive',
                'bg-green-500 text-white': item.badge.variant === 'success',
                'bg-yellow-500 text-white': item.badge.variant === 'warning',
              }
            )}>
              {item.badge.text}
            </span>
          )}
        </div>
        
        {hasChildren && collapsible && (
          <span className="flex-shrink-0">
            {isExpanded ? (
              <ChevronDown className="h-4 w-4" />
            ) : (
              <ChevronRight className="h-4 w-4" />
            )}
          </span>
        )}
      </div>
    );

    return (
      <div key={item.id}>
        {item.href && !hasChildren ? (
          <Link to={item.href} className="block">
            {itemContent}
          </Link>
        ) : (
          itemContent
        )}
        
        {hasChildren && (isExpanded || !collapsible) && (
          <div className={cn(
            'mt-1 space-y-1',
            variant === 'sidebar' && 'ml-4'
          )}>
            {item.children!.map(child => renderNavigationItem(child, level + 1))}
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

### Tab Navigation Component

```typescript
// src/components/complex/TabNavigation/TabNavigation.tsx
import React, { useState } from 'react';
import { cn } from '@/utils/cn';

export interface TabItem {
  id: string;
  label: string;
  content: React.ReactNode;
  icon?: React.ReactNode;
  disabled?: boolean;
  badge?: {
    text: string;
    variant?: 'default' | 'destructive' | 'success' | 'warning';
  };
}

export interface TabNavigationProps {
  tabs: TabItem[];
  defaultTab?: string;
  variant?: 'default' | 'pills' | 'underline';
  size?: 'sm' | 'md' | 'lg';
  orientation?: 'horizontal' | 'vertical';
  onTabChange?: (tabId: string) => void;
  className?: string;
}

export const TabNavigation: React.FC<TabNavigationProps> = ({
  tabs,
  defaultTab,
  variant = 'default',
  size = 'md',
  orientation = 'horizontal',
  onTabChange,
  className,
}) => {
  const [activeTab, setActiveTab] = useState(defaultTab || tabs[0]?.id);

  const handleTabChange = (tabId: string) => {
    setActiveTab(tabId);
    onTabChange?.(tabId);
  };

  const getTabClasses = (tab: TabItem, isActive: boolean) => {
    const baseClasses = cn(
      'inline-flex items-center justify-center whitespace-nowrap font-medium ring-offset-background transition-all focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2',
      {
        // Size variants
        'px-3 py-1.5 text-sm': size === 'sm',
        'px-4 py-2 text-sm': size === 'md',
        'px-6 py-3 text-base': size === 'lg',
        
        // Disabled state
        'pointer-events-none opacity-50': tab.disabled,
        
        // Variant styles
        'rounded-md': variant === 'pills',
        'border-b-2 border-transparent': variant === 'underline',
        'rounded-t-lg border border-b-0': variant === 'default',
      }
    );

    const activeClasses = cn({
      // Default variant
      'bg-background text-foreground border-border': variant === 'default' && isActive,
      'bg-muted text-muted-foreground hover:bg-background hover:text-foreground': variant === 'default' && !isActive,
      
      // Pills variant
      'bg-primary text-primary-foreground': variant === 'pills' && isActive,
      'hover:bg-accent hover:text-accent-foreground': variant === 'pills' && !isActive,
      
      // Underline variant
      'border-primary text-foreground': variant === 'underline' && isActive,
      'hover:text-foreground': variant === 'underline' && !isActive,
    });

    return cn(baseClasses, activeClasses);
  };

  const activeTabContent = tabs.find(tab => tab.id === activeTab)?.content;

  return (
    <div className={cn('w-full', className)}>
      {/* Tab List */}
      <div className={cn(
        'flex',
        {
          'border-b border-border': variant === 'default' || variant === 'underline',
          'bg-muted p-1 rounded-lg': variant === 'pills',
          'flex-col space-y-1': orientation === 'vertical',
          'space-x-1': orientation === 'horizontal' && variant === 'pills',
        }
      )}>
        {tabs.map((tab) => {
          const isActive = tab.id === activeTab;
          
          return (
            <button
              key={tab.id}
              className={getTabClasses(tab, isActive)}
              onClick={() => !tab.disabled && handleTabChange(tab.id)}
              disabled={tab.disabled}
              role="tab"
              aria-selected={isActive}
              aria-controls={`tabpanel-${tab.id}`}
              id={`tab-${tab.id}`}
            >
              {tab.icon && (
                <span className="mr-2 h-4 w-4">
                  {tab.icon}
                </span>
              )}
              <span>{tab.label}</span>
              {tab.badge && (
                <span className={cn(
                  'ml-2 inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium',
                  {
                    'bg-primary/20 text-primary': tab.badge.variant === 'default',
                    'bg-destructive/20 text-destructive': tab.badge.variant === 'destructive',
                    'bg-green-500/20 text-green-700': tab.badge.variant === 'success',
                    'bg-yellow-500/20 text-yellow-700': tab.badge.variant === 'warning',
                  }
                )}>
                  {tab.badge.text}
                </span>
              )}
            </button>
          );
        })}
      </div>

      {/* Tab Content */}
      <div className="mt-4">
        {tabs.map((tab) => (
          <div
            key={tab.id}
            role="tabpanel"
            id={`tabpanel-${tab.id}`}
            aria-labelledby={`tab-${tab.id}`}
            hidden={tab.id !== activeTab}
          >
            {tab.id === activeTab && tab.content}
          </div>
        ))}
      </div>
    </div>
  );
};
```

## Form Components

### Multi-Step Form

```typescript
// src/components/complex/MultiStepForm/MultiStepForm.tsx
import React, { useState, useCallback } from 'react';
import { Button } from '@/components/atoms/Button';
import { Progress } from '@/components/atoms/Progress';
import { Heading } from '@/components/atoms/Heading';
import { Text } from '@/components/atoms/Text';
import { cn } from '@/utils/cn';

export interface FormStep {
  id: string;
  title: string;
  description?: string;
  component: React.ComponentType<{
    data: any;
    onDataChange: (data: any) => void;
    onNext: () => void;
    onPrevious: () => void;
    isValid: boolean;
  }>;
  validation?: (data: any) => boolean | Promise<boolean>;
  optional?: boolean;
}

export interface MultiStepFormProps {
  steps: FormStep[];
  initialData?: Record<string, any>;
  onComplete: (data: Record<string, any>) => void | Promise<void>;
  onCancel?: () => void;
  showProgress?: boolean;
  showStepNumbers?: boolean;
  allowSkipOptional?: boolean;
  className?: string;
}

export const MultiStepForm: React.FC<MultiStepFormProps> = ({
  steps,
  initialData = {},
  onComplete,
  onCancel,
  showProgress = true,
  showStepNumbers = true,
  allowSkipOptional = true,
  className,
}) => {
  const [currentStepIndex, setCurrentStepIndex] = useState(0);
  const [formData, setFormData] = useState(initialData);
  const [stepValidation, setStepValidation] = useState<Record<string, boolean>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const currentStep = steps[currentStepIndex];
  const isFirstStep = currentStepIndex === 0;
  const isLastStep = currentStepIndex === steps.length - 1;
  const progress = ((currentStepIndex + 1) / steps.length) * 100;

  const validateStep = useCallback(async (stepIndex: number, data: any) => {
    const step = steps[stepIndex];
    if (!step.validation) return true;

    try {
      const isValid = await step.validation(data);
      setStepValidation(prev => ({ ...prev, [step.id]: isValid }));
      return isValid;
    } catch (error) {
      setStepValidation(prev => ({ ...prev, [step.id]: false }));
      return false;
    }
  }, [steps]);

  const handleDataChange = useCallback((stepData: any) => {
    setFormData(prev => ({
      ...prev,
      [currentStep.id]: stepData,
    }));
  }, [currentStep.id]);

  const handleNext = useCallback(async () => {
    const isValid = await validateStep(currentStepIndex, formData[currentStep.id]);
    
    if (isValid || (currentStep.optional && allowSkipOptional)) {
      if (isLastStep) {
        setIsSubmitting(true);
        try {
          await onComplete(formData);
        } finally {
          setIsSubmitting(false);
        }
      } else {
        setCurrentStepIndex(prev => prev + 1);
      }
    }
  }, [currentStepIndex, currentStep, formData, isLastStep, onComplete, validateStep, allowSkipOptional]);

  const handlePrevious = useCallback(() => {
    if (!isFirstStep) {
      setCurrentStepIndex(prev => prev - 1);
    }
  }, [isFirstStep]);

  const handleStepClick = useCallback(async (stepIndex: number) => {
    // Validate all previous steps
    for (let i = 0; i < stepIndex; i++) {
      const step = steps[i];
      const isValid = await validateStep(i, formData[step.id]);
      if (!isValid && !step.optional) {
        return; // Cannot skip to this step
      }
    }
    
    setCurrentStepIndex(stepIndex);
  }, [steps, formData, validateStep]);

  const StepComponent = currentStep.component;
  const isCurrentStepValid = stepValidation[currentStep.id] !== false;

  return (
    <div className={cn('max-w-2xl mx-auto space-y-6', className)}>
      {/* Progress Indicator */}
      {showProgress && (
        <div className="space-y-2">
          <div className="flex justify-between text-sm text-muted-foreground">
            <span>Step {currentStepIndex + 1} of {steps.length}</span>
            <span>{Math.round(progress)}% complete</span>
          </div>
          <Progress value={progress} />
        </div>
      )}

      {/* Step Navigation */}
      {showStepNumbers && (
        <div className="flex items-center justify-center space-x-4">
          {steps.map((step, index) => {
            const isCompleted = index < currentStepIndex;
            const isCurrent = index === currentStepIndex;
            const isAccessible = index <= currentStepIndex;

            return (
              <button
                key={step.id}
                onClick={() => isAccessible && handleStepClick(index)}
                disabled={!isAccessible}
                className={cn(
                  'flex items-center justify-center w-8 h-8 rounded-full text-sm font-medium transition-colors',
                  {
                    'bg-primary text-primary-foreground': isCompleted,
                    'bg-primary/20 text-primary border-2 border-primary': isCurrent,
                    'bg-muted text-muted-foreground': !isCompleted && !isCurrent && isAccessible,
                    'bg-muted/50 text-muted-foreground/50 cursor-not-allowed': !isAccessible,
                    'hover:bg-primary/10': isAccessible && !isCurrent,
                  }
                )}
              >
                {isCompleted ? '✓' : index + 1}
              </button>
            );
          })}
        </div>
      )}

      {/* Step Content */}
      <div className="bg-card rounded-lg border p-6">
        <div className="mb-6">
          <Heading level={2} className="text-xl font-semibold">
            {currentStep.title}
          </Heading>
          {currentStep.description && (
            <Text color="muted" className="mt-2">
              {currentStep.description}
            </Text>
          )}
        </div>

        <StepComponent
          data={formData[currentStep.id] || {}}
          onDataChange={handleDataChange}
          onNext={handleNext}
          onPrevious={handlePrevious}
          isValid={isCurrentStepValid}
        />
      </div>

      {/* Navigation Buttons */}
      <div className="flex items-center justify-between">
        <div>
          {!isFirstStep && (
            <Button variant="outline" onClick={handlePrevious}>
              Previous
            </Button>
          )}
        </div>

        <div className="flex items-center space-x-2">
          {onCancel && (
            <Button variant="ghost" onClick={onCancel}>
              Cancel
            </Button>
          )}
          
          {currentStep.optional && allowSkipOptional && !isLastStep && (
            <Button variant="outline" onClick={handleNext}>
              Skip
            </Button>
          )}
          
          <Button
            onClick={handleNext}
            disabled={!isCurrentStepValid && !currentStep.optional}
            loading={isSubmitting}
          >
            {isLastStep ? 'Complete' : 'Next'}
          </Button>
        </div>
      </div>
    </div>
  );
};
```

## Feedback Components

### Advanced Modal

```typescript
// src/components/complex/Modal/Modal.tsx
import React, { useEffect } from 'react';
import { createPortal } from 'react-dom';
import { X } from 'lucide-react';
import { Button } from '@/components/atoms/Button';
import { Heading } from '@/components/atoms/Heading';
import { cn } from '@/utils/cn';

export interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  description?: string;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  variant?: 'default' | 'destructive' | 'success' | 'warning';
  showCloseButton?: boolean;
  closeOnOverlayClick?: boolean;
  closeOnEscape?: boolean;
  actions?: {
    primary?: {
      label: string;
      onClick: () => void;
      loading?: boolean;
      disabled?: boolean;
    };
    secondary?: {
      label: string;
      onClick: () => void;
    };
  };
  className?: string;
}

export const Modal: React.FC<ModalProps> = ({
  isOpen,
  onClose,
  title,
  description,
  children,
  size = 'md',
  variant = 'default',
  showCloseButton = true,
  closeOnOverlayClick = true,
  closeOnEscape = true,
  actions,
  className,
}) => {
  // Handle escape key
  useEffect(() => {
    if (!closeOnEscape || !isOpen) return;

    const handleEscape = (event: KeyboardEvent) => {
      if (event.key === 'Escape') {
        onClose();
      }
    };

    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen, onClose, closeOnEscape]);

  // Prevent body scroll when modal is open
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

  const getSizeClasses = () => {
    switch (size) {
      case 'sm': return 'max-w-md';
      case 'md': return 'max-w-lg';
      case 'lg': return 'max-w-2xl';
      case 'xl': return 'max-w-4xl';
      case 'full': return 'max-w-[95vw] max-h-[95vh]';
      default: return 'max-w-lg';
    }
  };

  const getVariantClasses = () => {
    switch (variant) {
      case 'destructive': return 'border-destructive/20';
      case 'success': return 'border-green-500/20';
      case 'warning': return 'border-yellow-500/20';
      default: return '';
    }
  };

  const modalContent = (
    <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
      {/* Overlay */}
      <div
        className="fixed inset-0 bg-black/50 backdrop-blur-sm"
        onClick={closeOnOverlayClick ? onClose : undefined}
      />

      {/* Modal */}
      <div
        className={cn(
          'relative w-full bg-background rounded-lg border shadow-lg',
          getSizeClasses(),
          getVariantClasses(),
          className
        )}
        onClick={(e) => e.stopPropagation()}
      >
        {/* Header */}
        {(title || showCloseButton) && (
          <div className="flex items-center justify-between p-6 border-b">
            <div>
              {title && (
                <Heading level={3} className="text-lg font-semibold">
                  {title}
                </Heading>
              )}
              {description && (
                <p className="text-sm text-muted-foreground mt-1">
                  {description}
                </p>
              )}
            </div>
            
            {showCloseButton && (
              <Button
                variant="ghost"
                size="icon"
                onClick={onClose}
                className="h-6 w-6"
              >
                <X className="h-4 w-4" />
                <span className="sr-only">Close</span>
              </Button>
            )}
          </div>
        )}

        {/* Content */}
        <div className={cn(
          'p-6',
          size === 'full' && 'overflow-auto max-h-[calc(95vh-8rem)]'
        )}>
          {children}
        </div>

        {/* Actions */}
        {actions && (
          <div className="flex items-center justify-end space-x-2 p-6 border-t">
            {actions.secondary && (
              <Button variant="outline" onClick={actions.secondary.onClick}>
                {actions.secondary.label}
              </Button>
            )}
            {actions.primary && (
              <Button
                variant={variant === 'destructive' ? 'destructive' : 'default'}
                onClick={actions.primary.onClick}
                loading={actions.primary.loading}
                disabled={actions.primary.disabled}
              >
                {actions.primary.label}
              </Button>
            )}
          </div>
        )}
      </div>
    </div>
  );

  return createPortal(modalContent, document.body);
};
```

## Best Practices

### 1. Component Composition
- **Flexible APIs**: Support multiple configuration options
- **Extensibility**: Allow custom components and renderers
- **Performance**: Optimize for large datasets and complex interactions
- **Accessibility**: Enhanced ARIA support for complex interactions

### 2. State Management
- **Local State**: Efficient component-level state management
- **External State**: Integration with global state management
- **Persistence**: State persistence across sessions when needed
- **Synchronization**: Real-time updates and data synchronization

### 3. User Experience
- **Loading States**: Comprehensive loading and skeleton states
- **Error Handling**: Graceful error handling and recovery
- **Responsive Design**: Mobile-first responsive behavior
- **Keyboard Navigation**: Full keyboard accessibility support

### 4. Performance Optimization
- **Virtualization**: Virtual scrolling for large datasets
- **Memoization**: Optimized re-rendering with React.memo
- **Code Splitting**: Lazy loading for complex components
- **Bundle Optimization**: Efficient component bundling

This comprehensive complex components library provides sophisticated UI elements that handle advanced user interactions and data management scenarios for DafnckMachine v3.1. 