# State Management Implementation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: State Management Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive implementation guidelines for state management in DafnckMachine v3.1, covering global state management with Zustand, server state with React Query, local component state patterns, and state persistence strategies.

## State Management Architecture

### State Layer Hierarchy
```
Application State
├── Global State (Zustand)
│   ├── User Authentication State
│   ├── Application Settings
│   ├── UI State (theme, modals, etc.)
│   └── Cross-component Data
├── Server State (React Query)
│   ├── API Data Caching
│   ├── Background Synchronization
│   ├── Optimistic Updates
│   └── Error State Management
├── Local State (React useState/useReducer)
│   ├── Form State
│   ├── Component UI State
│   ├── Temporary Data
│   └── Input Validation
└── Persistent State
    ├── Local Storage
    ├── Session Storage
    └── IndexedDB
```

## Global State Management with Zustand

### Store Configuration

```typescript
// src/stores/types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'moderator';
  avatar?: string;
  preferences: UserPreferences;
}

export interface UserPreferences {
  theme: 'light' | 'dark' | 'system';
  language: string;
  notifications: NotificationSettings;
  dashboard: DashboardSettings;
}

export interface NotificationSettings {
  email: boolean;
  push: boolean;
  inApp: boolean;
  frequency: 'immediate' | 'daily' | 'weekly';
}

export interface AppSettings {
  apiBaseUrl: string;
  environment: 'development' | 'staging' | 'production';
  features: FeatureFlags;
  maintenance: MaintenanceMode;
}

export interface UIState {
  sidebarOpen: boolean;
  activeModal: string | null;
  loading: boolean;
  notifications: Notification[];
  breadcrumbs: BreadcrumbItem[];
}
```

### Authentication Store

```typescript
// src/stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import { User } from './types';
import { authApi } from '@/api/auth';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

interface AuthActions {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  updateUser: (updates: Partial<User>) => void;
  updatePreferences: (preferences: Partial<UserPreferences>) => void;
  clearError: () => void;
  refreshToken: () => Promise<void>;
}

export const useAuthStore = create<AuthState & AuthActions>()(
  persist(
    immer((set, get) => ({
      // Initial state
      user: null,
      token: null,
      isAuthenticated: false,
      isLoading: false,
      error: null,

      // Actions
      login: async (credentials) => {
        set((state) => {
          state.isLoading = true;
          state.error = null;
        });

        try {
          const response = await authApi.login(credentials);
          
          set((state) => {
            state.user = response.user;
            state.token = response.token;
            state.isAuthenticated = true;
            state.isLoading = false;
          });

          // Set token in API client
          authApi.setToken(response.token);
        } catch (error) {
          set((state) => {
            state.error = error.message;
            state.isLoading = false;
          });
          throw error;
        }
      },

      logout: () => {
        set((state) => {
          state.user = null;
          state.token = null;
          state.isAuthenticated = false;
          state.error = null;
        });

        // Clear token from API client
        authApi.clearToken();
      },

      updateUser: (updates) => {
        set((state) => {
          if (state.user) {
            Object.assign(state.user, updates);
          }
        });
      },

      updatePreferences: (preferences) => {
        set((state) => {
          if (state.user) {
            Object.assign(state.user.preferences, preferences);
          }
        });
      },

      clearError: () => {
        set((state) => {
          state.error = null;
        });
      },

      refreshToken: async () => {
        const { token } = get();
        if (!token) return;

        try {
          const response = await authApi.refreshToken(token);
          
          set((state) => {
            state.token = response.token;
            state.user = response.user;
          });

          authApi.setToken(response.token);
        } catch (error) {
          // Token refresh failed, logout user
          get().logout();
          throw error;
        }
      },
    })),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

### Application Settings Store

```typescript
// src/stores/appStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import { AppSettings, UIState } from './types';

interface AppState {
  settings: AppSettings;
  ui: UIState;
}

interface AppActions {
  updateSettings: (settings: Partial<AppSettings>) => void;
  toggleSidebar: () => void;
  openModal: (modalId: string) => void;
  closeModal: () => void;
  setLoading: (loading: boolean) => void;
  addNotification: (notification: Omit<Notification, 'id'>) => void;
  removeNotification: (id: string) => void;
  setBreadcrumbs: (breadcrumbs: BreadcrumbItem[]) => void;
  resetUI: () => void;
}

const initialUIState: UIState = {
  sidebarOpen: true,
  activeModal: null,
  loading: false,
  notifications: [],
  breadcrumbs: [],
};

export const useAppStore = create<AppState & AppActions>()(
  persist(
    immer((set, get) => ({
      // Initial state
      settings: {
        apiBaseUrl: process.env.REACT_APP_API_URL || 'http://localhost:3001',
        environment: process.env.NODE_ENV as any,
        features: {
          darkMode: true,
          notifications: true,
          analytics: true,
          beta: false,
        },
        maintenance: {
          enabled: false,
          message: '',
          estimatedEnd: null,
        },
      },
      ui: initialUIState,

      // Actions
      updateSettings: (settings) => {
        set((state) => {
          Object.assign(state.settings, settings);
        });
      },

      toggleSidebar: () => {
        set((state) => {
          state.ui.sidebarOpen = !state.ui.sidebarOpen;
        });
      },

      openModal: (modalId) => {
        set((state) => {
          state.ui.activeModal = modalId;
        });
      },

      closeModal: () => {
        set((state) => {
          state.ui.activeModal = null;
        });
      },

      setLoading: (loading) => {
        set((state) => {
          state.ui.loading = loading;
        });
      },

      addNotification: (notification) => {
        set((state) => {
          const newNotification = {
            ...notification,
            id: Date.now().toString(),
            timestamp: new Date().toISOString(),
          };
          state.ui.notifications.push(newNotification);
        });
      },

      removeNotification: (id) => {
        set((state) => {
          state.ui.notifications = state.ui.notifications.filter(n => n.id !== id);
        });
      },

      setBreadcrumbs: (breadcrumbs) => {
        set((state) => {
          state.ui.breadcrumbs = breadcrumbs;
        });
      },

      resetUI: () => {
        set((state) => {
          state.ui = { ...initialUIState };
        });
      },
    })),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        settings: state.settings,
        ui: {
          sidebarOpen: state.ui.sidebarOpen,
        },
      }),
    }
  )
);
```

## Server State Management with React Query

### Query Client Configuration

```typescript
// src/lib/queryClient.ts
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';
import { toast } from '@/components/ui/toast';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: (failureCount, error: any) => {
        // Don't retry on 4xx errors
        if (error?.status >= 400 && error?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 1,
    },
  },
  queryCache: new QueryCache({
    onError: (error: any) => {
      // Global error handling for queries
      if (error?.status === 401) {
        // Handle unauthorized errors
        useAuthStore.getState().logout();
        toast.error('Session expired. Please log in again.');
      } else if (error?.status >= 500) {
        toast.error('Server error. Please try again later.');
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error: any) => {
      // Global error handling for mutations
      if (error?.status === 401) {
        useAuthStore.getState().logout();
        toast.error('Session expired. Please log in again.');
      }
    },
  }),
});
```

### API Hooks Implementation

```typescript
// src/hooks/api/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '@/api/users';
import { User, CreateUserData, UpdateUserData } from '@/types/user';
import { toast } from '@/components/ui/toast';

// Query keys factory
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: any) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

// Get users list
export const useUsers = (params?: {
  page?: number;
  limit?: number;
  search?: string;
  role?: string;
  status?: string;
}) => {
  return useQuery({
    queryKey: userKeys.list(params),
    queryFn: () => usersApi.getUsers(params),
    keepPreviousData: true,
  });
};

// Get single user
export const useUser = (id: string) => {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => usersApi.getUser(id),
    enabled: !!id,
  });
};

// Create user mutation
export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserData) => usersApi.createUser(data),
    onSuccess: (newUser) => {
      // Invalidate and refetch users list
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
      
      // Add new user to cache
      queryClient.setQueryData(userKeys.detail(newUser.id), newUser);
      
      toast.success('User created successfully');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to create user');
    },
  });
};

// Update user mutation
export const useUpdateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserData }) =>
      usersApi.updateUser(id, data),
    onSuccess: (updatedUser, { id }) => {
      // Update user in cache
      queryClient.setQueryData(userKeys.detail(id), updatedUser);
      
      // Invalidate users list to reflect changes
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
      
      toast.success('User updated successfully');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to update user');
    },
  });
};

// Delete user mutation
export const useDeleteUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => usersApi.deleteUser(id),
    onSuccess: (_, id) => {
      // Remove user from cache
      queryClient.removeQueries({ queryKey: userKeys.detail(id) });
      
      // Invalidate users list
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
      
      toast.success('User deleted successfully');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to delete user');
    },
  });
};

// Optimistic update example
export const useToggleUserStatus = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, status }: { id: string; status: 'active' | 'inactive' }) =>
      usersApi.updateUserStatus(id, status),
    onMutate: async ({ id, status }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: userKeys.detail(id) });

      // Snapshot previous value
      const previousUser = queryClient.getQueryData<User>(userKeys.detail(id));

      // Optimistically update
      if (previousUser) {
        queryClient.setQueryData(userKeys.detail(id), {
          ...previousUser,
          status,
        });
      }

      return { previousUser };
    },
    onError: (error, { id }, context) => {
      // Rollback on error
      if (context?.previousUser) {
        queryClient.setQueryData(userKeys.detail(id), context.previousUser);
      }
      toast.error('Failed to update user status');
    },
    onSettled: (_, __, { id }) => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: userKeys.detail(id) });
    },
  });
};
```

## Local State Patterns

### Form State Management

```typescript
// src/hooks/useFormState.ts
import { useState, useCallback, useRef } from 'react';

export interface FormField<T = any> {
  value: T;
  error?: string;
  touched: boolean;
  dirty: boolean;
}

export interface FormState<T extends Record<string, any>> {
  fields: { [K in keyof T]: FormField<T[K]> };
  isValid: boolean;
  isDirty: boolean;
  isSubmitting: boolean;
  submitCount: number;
}

export interface FormActions<T extends Record<string, any>> {
  setValue: <K extends keyof T>(field: K, value: T[K]) => void;
  setError: <K extends keyof T>(field: K, error: string) => void;
  clearError: <K extends keyof T>(field: K) => void;
  setTouched: <K extends keyof T>(field: K, touched?: boolean) => void;
  reset: (values?: Partial<T>) => void;
  validate: () => boolean;
  submit: (onSubmit: (values: T) => Promise<void> | void) => Promise<void>;
}

export interface UseFormStateOptions<T extends Record<string, any>> {
  initialValues: T;
  validationSchema?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit?: (values: T) => Promise<void> | void;
}

export const useFormState = <T extends Record<string, any>>({
  initialValues,
  validationSchema,
  onSubmit,
}: UseFormStateOptions<T>): [FormState<T>, FormActions<T>] => {
  const initialState = useRef<FormState<T>>({
    fields: Object.keys(initialValues).reduce((acc, key) => {
      acc[key as keyof T] = {
        value: initialValues[key],
        touched: false,
        dirty: false,
      };
      return acc;
    }, {} as any),
    isValid: true,
    isDirty: false,
    isSubmitting: false,
    submitCount: 0,
  });

  const [state, setState] = useState<FormState<T>>(initialState.current);

  const setValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
    setState(prev => {
      const newFields = { ...prev.fields };
      newFields[field] = {
        ...newFields[field],
        value,
        dirty: value !== initialValues[field],
      };

      const isDirty = Object.values(newFields).some((field: any) => field.dirty);

      return {
        ...prev,
        fields: newFields,
        isDirty,
      };
    });
  }, [initialValues]);

  const setError = useCallback(<K extends keyof T>(field: K, error: string) => {
    setState(prev => ({
      ...prev,
      fields: {
        ...prev.fields,
        [field]: {
          ...prev.fields[field],
          error,
        },
      },
    }));
  }, []);

  const clearError = useCallback(<K extends keyof T>(field: K) => {
    setState(prev => ({
      ...prev,
      fields: {
        ...prev.fields,
        [field]: {
          ...prev.fields[field],
          error: undefined,
        },
      },
    }));
  }, []);

  const setTouched = useCallback(<K extends keyof T>(field: K, touched = true) => {
    setState(prev => ({
      ...prev,
      fields: {
        ...prev.fields,
        [field]: {
          ...prev.fields[field],
          touched,
        },
      },
    }));
  }, []);

  const validate = useCallback(() => {
    if (!validationSchema) return true;

    const values = Object.keys(state.fields).reduce((acc, key) => {
      acc[key as keyof T] = state.fields[key as keyof T].value;
      return acc;
    }, {} as T);

    const errors = validationSchema(values);
    
    setState(prev => {
      const newFields = { ...prev.fields };
      let isValid = true;

      Object.keys(newFields).forEach(key => {
        const fieldKey = key as keyof T;
        const error = errors[fieldKey];
        
        newFields[fieldKey] = {
          ...newFields[fieldKey],
          error,
        };

        if (error) {
          isValid = false;
        }
      });

      return {
        ...prev,
        fields: newFields,
        isValid,
      };
    });

    return !Object.values(errors).some(error => error);
  }, [state.fields, validationSchema]);

  const reset = useCallback((values?: Partial<T>) => {
    const resetValues = { ...initialValues, ...values };
    
    setState({
      fields: Object.keys(resetValues).reduce((acc, key) => {
        acc[key as keyof T] = {
          value: resetValues[key],
          touched: false,
          dirty: false,
        };
        return acc;
      }, {} as any),
      isValid: true,
      isDirty: false,
      isSubmitting: false,
      submitCount: 0,
    });
  }, [initialValues]);

  const submit = useCallback(async (submitFn?: (values: T) => Promise<void> | void) => {
    setState(prev => ({ ...prev, isSubmitting: true }));

    const isValid = validate();
    
    if (!isValid) {
      setState(prev => ({ 
        ...prev, 
        isSubmitting: false,
        submitCount: prev.submitCount + 1,
      }));
      return;
    }

    try {
      const values = Object.keys(state.fields).reduce((acc, key) => {
        acc[key as keyof T] = state.fields[key as keyof T].value;
        return acc;
      }, {} as T);

      await (submitFn || onSubmit)?.(values);
      
      setState(prev => ({ 
        ...prev, 
        isSubmitting: false,
        submitCount: prev.submitCount + 1,
      }));
    } catch (error) {
      setState(prev => ({ 
        ...prev, 
        isSubmitting: false,
        submitCount: prev.submitCount + 1,
      }));
      throw error;
    }
  }, [state.fields, validate, onSubmit]);

  const actions: FormActions<T> = {
    setValue,
    setError,
    clearError,
    setTouched,
    reset,
    validate,
    submit,
  };

  return [state, actions];
};
```

### Component State Patterns

```typescript
// src/hooks/useToggle.ts
import { useState, useCallback } from 'react';

export const useToggle = (initialValue = false) => {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(prev => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return [value, { toggle, setTrue, setFalse, setValue }] as const;
};

// src/hooks/useCounter.ts
import { useState, useCallback } from 'react';

export const useCounter = (initialValue = 0, step = 1) => {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(prev => prev + step), [step]);
  const decrement = useCallback(() => setCount(prev => prev - step), [step]);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  const set = useCallback((value: number) => setCount(value), []);

  return [count, { increment, decrement, reset, set }] as const;
};

// src/hooks/useLocalStorage.ts
import { useState, useEffect, useCallback } from 'react';

export const useLocalStorage = <T>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  const removeValue = useCallback(() => {
    try {
      window.localStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error(`Error removing localStorage key "${key}":`, error);
    }
  }, [key, initialValue]);

  return [storedValue, setValue, removeValue] as const;
};
```

## State Persistence Strategies

### Persistent Store Configuration

```typescript
// src/stores/persistentStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface PersistentState {
  userPreferences: UserPreferences;
  recentSearches: string[];
  favoriteItems: string[];
  viewSettings: ViewSettings;
}

interface PersistentActions {
  updatePreferences: (preferences: Partial<UserPreferences>) => void;
  addRecentSearch: (search: string) => void;
  clearRecentSearches: () => void;
  toggleFavorite: (itemId: string) => void;
  updateViewSettings: (settings: Partial<ViewSettings>) => void;
}

export const usePersistentStore = create<PersistentState & PersistentActions>()(
  persist(
    immer((set, get) => ({
      // Initial state
      userPreferences: {
        theme: 'system',
        language: 'en',
        notifications: {
          email: true,
          push: false,
          inApp: true,
          frequency: 'immediate',
        },
        dashboard: {
          layout: 'grid',
          itemsPerPage: 20,
          showSidebar: true,
        },
      },
      recentSearches: [],
      favoriteItems: [],
      viewSettings: {
        listView: 'grid',
        sortBy: 'name',
        sortOrder: 'asc',
        filters: {},
      },

      // Actions
      updatePreferences: (preferences) => {
        set((state) => {
          Object.assign(state.userPreferences, preferences);
        });
      },

      addRecentSearch: (search) => {
        set((state) => {
          // Remove if already exists
          state.recentSearches = state.recentSearches.filter(s => s !== search);
          // Add to beginning
          state.recentSearches.unshift(search);
          // Keep only last 10
          state.recentSearches = state.recentSearches.slice(0, 10);
        });
      },

      clearRecentSearches: () => {
        set((state) => {
          state.recentSearches = [];
        });
      },

      toggleFavorite: (itemId) => {
        set((state) => {
          const index = state.favoriteItems.indexOf(itemId);
          if (index > -1) {
            state.favoriteItems.splice(index, 1);
          } else {
            state.favoriteItems.push(itemId);
          }
        });
      },

      updateViewSettings: (settings) => {
        set((state) => {
          Object.assign(state.viewSettings, settings);
        });
      },
    })),
    {
      name: 'persistent-storage',
      storage: createJSONStorage(() => localStorage),
      version: 1,
      migrate: (persistedState: any, version: number) => {
        // Handle migration between versions
        if (version === 0) {
          // Migrate from version 0 to 1
          return {
            ...persistedState,
            viewSettings: {
              listView: 'grid',
              sortBy: 'name',
              sortOrder: 'asc',
              filters: {},
            },
          };
        }
        return persistedState;
      },
    }
  )
);
```

## State Synchronization Patterns

### Real-time State Updates

```typescript
// src/hooks/useRealtimeSync.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { useAuthStore } from '@/stores/authStore';
import { websocketService } from '@/services/websocket';

export const useRealtimeSync = () => {
  const queryClient = useQueryClient();
  const { isAuthenticated, user } = useAuthStore();

  useEffect(() => {
    if (!isAuthenticated || !user) return;

    // Connect to WebSocket
    websocketService.connect(user.id);

    // Handle real-time updates
    const handleUserUpdate = (data: any) => {
      queryClient.setQueryData(['users', data.id], data);
      queryClient.invalidateQueries({ queryKey: ['users'] });
    };

    const handleNotification = (notification: any) => {
      queryClient.setQueryData(
        ['notifications'],
        (old: any) => [notification, ...(old || [])]
      );
    };

    const handleDataUpdate = (data: any) => {
      // Invalidate relevant queries based on data type
      queryClient.invalidateQueries({ queryKey: [data.type] });
    };

    // Subscribe to events
    websocketService.on('user:updated', handleUserUpdate);
    websocketService.on('notification:new', handleNotification);
    websocketService.on('data:updated', handleDataUpdate);

    return () => {
      // Cleanup
      websocketService.off('user:updated', handleUserUpdate);
      websocketService.off('notification:new', handleNotification);
      websocketService.off('data:updated', handleDataUpdate);
      websocketService.disconnect();
    };
  }, [isAuthenticated, user, queryClient]);
};
```

## Testing State Management

### Store Testing Utilities

```typescript
// src/utils/testing/storeTestUtils.ts
import { renderHook, act } from '@testing-library/react';
import { useAuthStore } from '@/stores/authStore';
import { useAppStore } from '@/stores/appStore';

export const createMockUser = (overrides = {}) => ({
  id: '1',
  email: 'test@example.com',
  name: 'Test User',
  role: 'user' as const,
  preferences: {
    theme: 'light' as const,
    language: 'en',
    notifications: {
      email: true,
      push: false,
      inApp: true,
      frequency: 'immediate' as const,
    },
    dashboard: {
      layout: 'grid' as const,
      itemsPerPage: 20,
      showSidebar: true,
    },
  },
  ...overrides,
});

export const resetStores = () => {
  // Reset auth store
  act(() => {
    useAuthStore.getState().logout();
  });

  // Reset app store
  act(() => {
    useAppStore.getState().resetUI();
  });
};

export const setupAuthenticatedUser = (user = createMockUser()) => {
  act(() => {
    useAuthStore.setState({
      user,
      token: 'mock-token',
      isAuthenticated: true,
      isLoading: false,
      error: null,
    });
  });
  return user;
};
```

### Store Tests Examples

```typescript
// src/stores/__tests__/authStore.test.ts
import { renderHook, act } from '@testing-library/react';
import { useAuthStore } from '../authStore';
import { authApi } from '@/api/auth';
import { createMockUser, resetStores } from '@/utils/testing/storeTestUtils';

// Mock API
jest.mock('@/api/auth');
const mockAuthApi = authApi as jest.Mocked<typeof authApi>;

describe('authStore', () => {
  beforeEach(() => {
    resetStores();
    jest.clearAllMocks();
  });

  it('should initialize with default state', () => {
    const { result } = renderHook(() => useAuthStore());

    expect(result.current.user).toBeNull();
    expect(result.current.token).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.isLoading).toBe(false);
    expect(result.current.error).toBeNull();
  });

  it('should handle successful login', async () => {
    const mockUser = createMockUser();
    const mockResponse = { user: mockUser, token: 'test-token' };
    
    mockAuthApi.login.mockResolvedValue(mockResponse);

    const { result } = renderHook(() => useAuthStore());

    await act(async () => {
      await result.current.login({ email: 'test@example.com', password: 'password' });
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.token).toBe('test-token');
    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.isLoading).toBe(false);
    expect(mockAuthApi.setToken).toHaveBeenCalledWith('test-token');
  });

  it('should handle login error', async () => {
    const mockError = new Error('Invalid credentials');
    mockAuthApi.login.mockRejectedValue(mockError);

    const { result } = renderHook(() => useAuthStore());

    await act(async () => {
      try {
        await result.current.login({ email: 'test@example.com', password: 'wrong' });
      } catch (error) {
        // Expected to throw
      }
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.error).toBe('Invalid credentials');
    expect(result.current.isLoading).toBe(false);
  });

  it('should handle logout', () => {
    const { result } = renderHook(() => useAuthStore());

    // Set authenticated state
    act(() => {
      useAuthStore.setState({
        user: createMockUser(),
        token: 'test-token',
        isAuthenticated: true,
      });
    });

    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(result.current.token).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(mockAuthApi.clearToken).toHaveBeenCalled();
  });
});
```

## Best Practices

### 1. State Organization
- **Single Responsibility**: Each store handles a specific domain
- **Normalized Data**: Avoid nested objects in global state
- **Immutable Updates**: Use Immer for safe state mutations
- **Type Safety**: Comprehensive TypeScript interfaces

### 2. Performance Optimization
- **Selective Subscriptions**: Subscribe only to needed state slices
- **Memoization**: Use React.memo and useMemo for expensive computations
- **Query Optimization**: Implement proper caching and invalidation strategies
- **Bundle Splitting**: Lazy load stores and state logic

### 3. Error Handling
- **Global Error Boundaries**: Catch and handle state errors
- **Graceful Degradation**: Fallback states for error conditions
- **User Feedback**: Clear error messages and recovery options
- **Logging**: Comprehensive error logging and monitoring

### 4. Testing Strategy
- **Unit Tests**: Test individual store actions and selectors
- **Integration Tests**: Test state interactions and side effects
- **Mock Strategies**: Proper mocking of external dependencies
- **State Snapshots**: Test state transitions and persistence

This comprehensive state management implementation guide provides the foundation for building robust, scalable, and maintainable state management in DafnckMachine v3.1. 