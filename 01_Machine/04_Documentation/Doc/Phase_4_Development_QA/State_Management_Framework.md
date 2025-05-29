# State Management Framework - DafnckMachine v3.1

## Document Metadata
- **Document Type**: State Management Framework
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active

## Overview

This document defines the state management architecture for DafnckMachine v3.1, providing guidelines for data flow, API integration, and state handling patterns using modern state management solutions.

## State Management Architecture

### 1. State Management Strategy
- **Redux Toolkit**: Primary state management solution for complex application state
- **React Query/TanStack Query**: Server state management and caching
- **Local State**: Component-level state using React hooks
- **Context API**: Shared state for theme, authentication, and UI preferences

### 2. State Structure Design
- **Normalized State**: Flat state structure for better performance
- **Feature-Based Slices**: State organized by application features
- **Separation of Concerns**: Clear distinction between client and server state
- **Immutable Updates**: All state updates follow immutability principles

## Global State Architecture

### Redux Store Configuration
```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import { authSlice } from './slices/authSlice';
import { uiSlice } from './slices/uiSlice';
import { notificationSlice } from './slices/notificationSlice';
import { apiSlice } from './api/apiSlice';

export const store = configureStore({
  reducer: {
    auth: authSlice.reducer,
    ui: uiSlice.reducer,
    notifications: notificationSlice.reducer,
    api: apiSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(apiSlice.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### State Type Definitions
```typescript
// types/state.ts
export interface RootState {
  auth: AuthState;
  ui: UIState;
  notifications: NotificationState;
  api: ApiState;
}

export interface AuthState {
  user: User | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
  permissions: Permission[];
  lastActivity: number;
}

export interface UIState {
  theme: 'light' | 'dark' | 'system';
  sidebar: {
    isOpen: boolean;
    isCollapsed: boolean;
    activeSection: string;
  };
  modals: Record<string, boolean>;
  loading: Record<string, boolean>;
  errors: Record<string, string | null>;
}

export interface NotificationState {
  notifications: Notification[];
  unreadCount: number;
  settings: NotificationSettings;
}

export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  role: string;
  permissions: Permission[];
  preferences: UserPreferences;
  createdAt: string;
  updatedAt: string;
}

export interface Notification {
  id: string;
  type: 'info' | 'success' | 'warning' | 'error';
  title: string;
  message: string;
  timestamp: number;
  read: boolean;
  actions?: NotificationAction[];
}
```

## Feature-Based State Slices

### Authentication Slice
```typescript
// store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { authService } from '../../services/authService';
import { AuthState, User, LoginCredentials } from '../../types/auth';

const initialState: AuthState = {
  user: null,
  token: localStorage.getItem('token'),
  refreshToken: localStorage.getItem('refreshToken'),
  isAuthenticated: false,
  isLoading: false,
  error: null,
  permissions: [],
  lastActivity: Date.now(),
};

// Async Thunks
export const loginUser = createAsyncThunk(
  'auth/login',
  async (credentials: LoginCredentials, { rejectWithValue }) => {
    try {
      const response = await authService.login(credentials);
      localStorage.setItem('token', response.token);
      localStorage.setItem('refreshToken', response.refreshToken);
      return response;
    } catch (error: any) {
      return rejectWithValue(error.message);
    }
  }
);

export const refreshToken = createAsyncThunk(
  'auth/refreshToken',
  async (_, { getState, rejectWithValue }) => {
    try {
      const { auth } = getState() as { auth: AuthState };
      if (!auth.refreshToken) {
        throw new Error('No refresh token available');
      }
      const response = await authService.refreshToken(auth.refreshToken);
      localStorage.setItem('token', response.token);
      return response;
    } catch (error: any) {
      return rejectWithValue(error.message);
    }
  }
);

export const logoutUser = createAsyncThunk(
  'auth/logout',
  async (_, { getState }) => {
    const { auth } = getState() as { auth: AuthState };
    if (auth.token) {
      await authService.logout(auth.token);
    }
    localStorage.removeItem('token');
    localStorage.removeItem('refreshToken');
  }
);

export const fetchUserProfile = createAsyncThunk(
  'auth/fetchProfile',
  async (_, { rejectWithValue }) => {
    try {
      const response = await authService.getProfile();
      return response;
    } catch (error: any) {
      return rejectWithValue(error.message);
    }
  }
);

// Slice Definition
export const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    updateLastActivity: (state) => {
      state.lastActivity = Date.now();
    },
    updateUserPreferences: (state, action: PayloadAction<Partial<UserPreferences>>) => {
      if (state.user) {
        state.user.preferences = { ...state.user.preferences, ...action.payload };
      }
    },
  },
  extraReducers: (builder) => {
    builder
      // Login
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.refreshToken = action.payload.refreshToken;
        state.isAuthenticated = true;
        state.permissions = action.payload.user.permissions;
        state.lastActivity = Date.now();
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload as string;
        state.isAuthenticated = false;
      })
      // Refresh Token
      .addCase(refreshToken.fulfilled, (state, action) => {
        state.token = action.payload.token;
        state.lastActivity = Date.now();
      })
      .addCase(refreshToken.rejected, (state) => {
        state.user = null;
        state.token = null;
        state.refreshToken = null;
        state.isAuthenticated = false;
        state.permissions = [];
      })
      // Logout
      .addCase(logoutUser.fulfilled, (state) => {
        state.user = null;
        state.token = null;
        state.refreshToken = null;
        state.isAuthenticated = false;
        state.permissions = [];
        state.error = null;
      })
      // Fetch Profile
      .addCase(fetchUserProfile.fulfilled, (state, action) => {
        state.user = action.payload;
        state.permissions = action.payload.permissions;
      });
  },
});

export const { clearError, updateLastActivity, updateUserPreferences } = authSlice.actions;
```

### UI State Slice
```typescript
// store/slices/uiSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { UIState } from '../../types/state';

const initialState: UIState = {
  theme: (localStorage.getItem('theme') as 'light' | 'dark' | 'system') || 'system',
  sidebar: {
    isOpen: false,
    isCollapsed: localStorage.getItem('sidebarCollapsed') === 'true',
    activeSection: '',
  },
  modals: {},
  loading: {},
  errors: {},
};

export const uiSlice = createSlice({
  name: 'ui',
  initialState,
  reducers: {
    setTheme: (state, action: PayloadAction<'light' | 'dark' | 'system'>) => {
      state.theme = action.payload;
      localStorage.setItem('theme', action.payload);
    },
    toggleSidebar: (state) => {
      state.sidebar.isOpen = !state.sidebar.isOpen;
    },
    setSidebarOpen: (state, action: PayloadAction<boolean>) => {
      state.sidebar.isOpen = action.payload;
    },
    toggleSidebarCollapsed: (state) => {
      state.sidebar.isCollapsed = !state.sidebar.isCollapsed;
      localStorage.setItem('sidebarCollapsed', state.sidebar.isCollapsed.toString());
    },
    setActiveSection: (state, action: PayloadAction<string>) => {
      state.sidebar.activeSection = action.payload;
    },
    openModal: (state, action: PayloadAction<string>) => {
      state.modals[action.payload] = true;
    },
    closeModal: (state, action: PayloadAction<string>) => {
      state.modals[action.payload] = false;
    },
    setLoading: (state, action: PayloadAction<{ key: string; loading: boolean }>) => {
      state.loading[action.payload.key] = action.payload.loading;
    },
    setError: (state, action: PayloadAction<{ key: string; error: string | null }>) => {
      state.errors[action.payload.key] = action.payload.error;
    },
    clearError: (state, action: PayloadAction<string>) => {
      state.errors[action.payload] = null;
    },
  },
});

export const {
  setTheme,
  toggleSidebar,
  setSidebarOpen,
  toggleSidebarCollapsed,
  setActiveSection,
  openModal,
  closeModal,
  setLoading,
  setError,
  clearError,
} = uiSlice.actions;
```

## API State Management

### RTK Query API Slice
```typescript
// store/api/apiSlice.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { RootState } from '../index';

const baseQuery = fetchBaseQuery({
  baseUrl: process.env.REACT_APP_API_BASE_URL || '/api',
  prepareHeaders: (headers, { getState }) => {
    const token = (getState() as RootState).auth.token;
    if (token) {
      headers.set('authorization', `Bearer ${token}`);
    }
    headers.set('content-type', 'application/json');
    return headers;
  },
});

const baseQueryWithReauth = async (args: any, api: any, extraOptions: any) => {
  let result = await baseQuery(args, api, extraOptions);
  
  if (result.error && result.error.status === 401) {
    // Try to refresh token
    const refreshResult = await baseQuery(
      {
        url: '/auth/refresh',
        method: 'POST',
        body: {
          refreshToken: (api.getState() as RootState).auth.refreshToken,
        },
      },
      api,
      extraOptions
    );
    
    if (refreshResult.data) {
      // Retry original query with new token
      result = await baseQuery(args, api, extraOptions);
    } else {
      // Refresh failed, logout user
      api.dispatch({ type: 'auth/logout/fulfilled' });
    }
  }
  
  return result;
};

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: baseQueryWithReauth,
  tagTypes: ['User', 'Project', 'Task', 'Notification'],
  endpoints: (builder) => ({
    // Users
    getUsers: builder.query<User[], { page?: number; limit?: number; search?: string }>({
      query: ({ page = 1, limit = 10, search = '' }) => 
        `users?page=${page}&limit=${limit}&search=${search}`,
      providesTags: ['User'],
    }),
    getUserById: builder.query<User, string>({
      query: (id) => `users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),
    updateUser: builder.mutation<User, { id: string; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `users/${id}`,
        method: 'PATCH',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),
    deleteUser: builder.mutation<void, string>({
      query: (id) => ({
        url: `users/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['User'],
    }),
    
    // Projects
    getProjects: builder.query<Project[], { status?: string; userId?: string }>({
      query: (params) => ({
        url: 'projects',
        params,
      }),
      providesTags: ['Project'],
    }),
    createProject: builder.mutation<Project, Partial<Project>>({
      query: (data) => ({
        url: 'projects',
        method: 'POST',
        body: data,
      }),
      invalidatesTags: ['Project'],
    }),
    
    // Tasks
    getTasks: builder.query<Task[], { projectId?: string; status?: string }>({
      query: (params) => ({
        url: 'tasks',
        params,
      }),
      providesTags: ['Task'],
    }),
    updateTask: builder.mutation<Task, { id: string; data: Partial<Task> }>({
      query: ({ id, data }) => ({
        url: `tasks/${id}`,
        method: 'PATCH',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Task', id }],
    }),
  }),
});

export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useUpdateUserMutation,
  useDeleteUserMutation,
  useGetProjectsQuery,
  useCreateProjectMutation,
  useGetTasksQuery,
  useUpdateTaskMutation,
} = apiSlice;
```

## Custom Hooks for State Management

### Authentication Hooks
```typescript
// hooks/useAuth.ts
import { useSelector, useDispatch } from 'react-redux';
import { useCallback } from 'react';
import { RootState, AppDispatch } from '../store';
import { loginUser, logoutUser, refreshToken, clearError } from '../store/slices/authSlice';
import { LoginCredentials } from '../types/auth';

export const useAuth = () => {
  const dispatch = useDispatch<AppDispatch>();
  const auth = useSelector((state: RootState) => state.auth);

  const login = useCallback(
    (credentials: LoginCredentials) => {
      return dispatch(loginUser(credentials));
    },
    [dispatch]
  );

  const logout = useCallback(() => {
    return dispatch(logoutUser());
  }, [dispatch]);

  const refresh = useCallback(() => {
    return dispatch(refreshToken());
  }, [dispatch]);

  const clearAuthError = useCallback(() => {
    dispatch(clearError());
  }, [dispatch]);

  const hasPermission = useCallback(
    (permission: string) => {
      return auth.permissions.some(p => p.name === permission);
    },
    [auth.permissions]
  );

  const hasRole = useCallback(
    (role: string) => {
      return auth.user?.role === role;
    },
    [auth.user?.role]
  );

  return {
    ...auth,
    login,
    logout,
    refresh,
    clearError: clearAuthError,
    hasPermission,
    hasRole,
  };
};
```

### UI State Hooks
```typescript
// hooks/useUI.ts
import { useSelector, useDispatch } from 'react-redux';
import { useCallback } from 'react';
import { RootState } from '../store';
import {
  setTheme,
  toggleSidebar,
  setSidebarOpen,
  toggleSidebarCollapsed,
  openModal,
  closeModal,
  setLoading,
  setError,
  clearError,
} from '../store/slices/uiSlice';

export const useUI = () => {
  const dispatch = useDispatch();
  const ui = useSelector((state: RootState) => state.ui);

  const changeTheme = useCallback(
    (theme: 'light' | 'dark' | 'system') => {
      dispatch(setTheme(theme));
    },
    [dispatch]
  );

  const handleToggleSidebar = useCallback(() => {
    dispatch(toggleSidebar());
  }, [dispatch]);

  const handleSetSidebarOpen = useCallback(
    (isOpen: boolean) => {
      dispatch(setSidebarOpen(isOpen));
    },
    [dispatch]
  );

  const handleToggleSidebarCollapsed = useCallback(() => {
    dispatch(toggleSidebarCollapsed());
  }, [dispatch]);

  const handleOpenModal = useCallback(
    (modalId: string) => {
      dispatch(openModal(modalId));
    },
    [dispatch]
  );

  const handleCloseModal = useCallback(
    (modalId: string) => {
      dispatch(closeModal(modalId));
    },
    [dispatch]
  );

  const setLoadingState = useCallback(
    (key: string, loading: boolean) => {
      dispatch(setLoading({ key, loading }));
    },
    [dispatch]
  );

  const setErrorState = useCallback(
    (key: string, error: string | null) => {
      dispatch(setError({ key, error }));
    },
    [dispatch]
  );

  const clearErrorState = useCallback(
    (key: string) => {
      dispatch(clearError(key));
    },
    [dispatch]
  );

  return {
    ...ui,
    changeTheme,
    toggleSidebar: handleToggleSidebar,
    setSidebarOpen: handleSetSidebarOpen,
    toggleSidebarCollapsed: handleToggleSidebarCollapsed,
    openModal: handleOpenModal,
    closeModal: handleCloseModal,
    setLoading: setLoadingState,
    setError: setErrorState,
    clearError: clearErrorState,
  };
};
```

## Local State Management

### Component State Patterns
```typescript
// hooks/useLocalState.ts
import { useState, useCallback, useReducer } from 'react';

// Simple state hook with validation
export const useValidatedState = <T>(
  initialValue: T,
  validator?: (value: T) => string | null
) => {
  const [value, setValue] = useState<T>(initialValue);
  const [error, setError] = useState<string | null>(null);

  const updateValue = useCallback(
    (newValue: T) => {
      const validationError = validator ? validator(newValue) : null;
      setError(validationError);
      setValue(newValue);
      return validationError === null;
    },
    [validator]
  );

  return {
    value,
    setValue: updateValue,
    error,
    isValid: error === null,
  };
};

// Form state management
interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
}

type FormAction<T> =
  | { type: 'SET_VALUE'; field: keyof T; value: any }
  | { type: 'SET_ERROR'; field: keyof T; error: string }
  | { type: 'SET_TOUCHED'; field: keyof T }
  | { type: 'SET_SUBMITTING'; isSubmitting: boolean }
  | { type: 'RESET'; initialValues: T };

function formReducer<T>(state: FormState<T>, action: FormAction<T>): FormState<T> {
  switch (action.type) {
    case 'SET_VALUE':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
      };
    case 'SET_ERROR':
      return {
        ...state,
        errors: { ...state.errors, [action.field]: action.error },
      };
    case 'SET_TOUCHED':
      return {
        ...state,
        touched: { ...state.touched, [action.field]: true },
      };
    case 'SET_SUBMITTING':
      return {
        ...state,
        isSubmitting: action.isSubmitting,
      };
    case 'RESET':
      return {
        values: action.initialValues,
        errors: {},
        touched: {},
        isSubmitting: false,
      };
    default:
      return state;
  }
}

export const useForm = <T extends Record<string, any>>(
  initialValues: T,
  validationSchema?: Partial<Record<keyof T, (value: any) => string | null>>
) => {
  const [state, dispatch] = useReducer(formReducer<T>, {
    values: initialValues,
    errors: {},
    touched: {},
    isSubmitting: false,
  });

  const setValue = useCallback(
    (field: keyof T, value: any) => {
      dispatch({ type: 'SET_VALUE', field, value });
      
      // Validate field if schema provided
      if (validationSchema && validationSchema[field]) {
        const error = validationSchema[field]!(value);
        if (error) {
          dispatch({ type: 'SET_ERROR', field, error });
        }
      }
    },
    [validationSchema]
  );

  const setTouched = useCallback((field: keyof T) => {
    dispatch({ type: 'SET_TOUCHED', field });
  }, []);

  const setSubmitting = useCallback((isSubmitting: boolean) => {
    dispatch({ type: 'SET_SUBMITTING', isSubmitting });
  }, []);

  const reset = useCallback(() => {
    dispatch({ type: 'RESET', initialValues });
  }, [initialValues]);

  const validate = useCallback(() => {
    if (!validationSchema) return true;

    let isValid = true;
    Object.keys(validationSchema).forEach((field) => {
      const validator = validationSchema[field as keyof T];
      if (validator) {
        const error = validator(state.values[field as keyof T]);
        if (error) {
          dispatch({ type: 'SET_ERROR', field: field as keyof T, error });
          isValid = false;
        }
      }
    });

    return isValid;
  }, [state.values, validationSchema]);

  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    setValue,
    setTouched,
    setSubmitting,
    reset,
    validate,
  };
};
```

## Context Providers

### Theme Context
```typescript
// contexts/ThemeContext.tsx
import React, { createContext, useContext, useEffect } from 'react';
import { useUI } from '../hooks/useUI';

interface ThemeContextValue {
  theme: 'light' | 'dark' | 'system';
  resolvedTheme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme, changeTheme } = useUI();
  
  const getResolvedTheme = (): 'light' | 'dark' => {
    if (theme === 'system') {
      return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }
    return theme;
  };

  const resolvedTheme = getResolvedTheme();

  useEffect(() => {
    const root = document.documentElement;
    root.setAttribute('data-theme', resolvedTheme);
    
    // Listen for system theme changes
    if (theme === 'system') {
      const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
      const handleChange = () => {
        root.setAttribute('data-theme', getResolvedTheme());
      };
      
      mediaQuery.addEventListener('change', handleChange);
      return () => mediaQuery.removeEventListener('change', handleChange);
    }
  }, [theme, resolvedTheme]);

  const value: ThemeContextValue = {
    theme,
    resolvedTheme,
    setTheme: changeTheme,
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

## State Persistence

### Redux Persist Configuration
```typescript
// store/persistConfig.ts
import { persistReducer, persistStore } from 'redux-persist';
import storage from 'redux-persist/lib/storage';
import { combineReducers } from '@reduxjs/toolkit';
import { authSlice } from './slices/authSlice';
import { uiSlice } from './slices/uiSlice';

const persistConfig = {
  key: 'root',
  storage,
  whitelist: ['auth', 'ui'], // Only persist auth and ui state
  blacklist: ['api'], // Don't persist API cache
};

const authPersistConfig = {
  key: 'auth',
  storage,
  blacklist: ['isLoading', 'error'], // Don't persist loading and error states
};

const uiPersistConfig = {
  key: 'ui',
  storage,
  blacklist: ['modals', 'loading', 'errors'], // Don't persist temporary UI state
};

const rootReducer = combineReducers({
  auth: persistReducer(authPersistConfig, authSlice.reducer),
  ui: persistReducer(uiPersistConfig, uiSlice.reducer),
});

export const persistedReducer = persistReducer(persistConfig, rootReducer);
```

## Error Handling

### Error Boundary for State
```typescript
// components/StateErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { store } from '../store';
import { setError } from '../store/slices/uiSlice';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class StateErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('State error caught:', error, errorInfo);
    
    // Dispatch error to global state
    store.dispatch(setError({
      key: 'global',
      error: error.message,
    }));
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong with the application state.</h2>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Performance Optimization

### Selector Optimization
```typescript
// selectors/index.ts
import { createSelector } from '@reduxjs/toolkit';
import { RootState } from '../store';

// Memoized selectors for better performance
export const selectAuth = (state: RootState) => state.auth;
export const selectUI = (state: RootState) => state.ui;

export const selectIsAuthenticated = createSelector(
  [selectAuth],
  (auth) => auth.isAuthenticated
);

export const selectCurrentUser = createSelector(
  [selectAuth],
  (auth) => auth.user
);

export const selectUserPermissions = createSelector(
  [selectAuth],
  (auth) => auth.permissions
);

export const selectSidebarState = createSelector(
  [selectUI],
  (ui) => ui.sidebar
);

export const selectActiveModals = createSelector(
  [selectUI],
  (ui) => Object.entries(ui.modals).filter(([_, isOpen]) => isOpen).map(([id]) => id)
);

export const selectLoadingStates = createSelector(
  [selectUI],
  (ui) => ui.loading
);

// Complex selectors with multiple inputs
export const selectUserWithPermissions = createSelector(
  [selectCurrentUser, selectUserPermissions],
  (user, permissions) => ({
    ...user,
    permissions,
    hasAdminAccess: permissions.some(p => p.name === 'admin'),
  })
);
```

## Testing State Management

### Redux Testing Utilities
```typescript
// utils/testUtils.tsx
import React, { PropsWithChildren } from 'react';
import { render } from '@testing-library/react';
import { configureStore } from '@reduxjs/toolkit';
import { Provider } from 'react-redux';
import { authSlice } from '../store/slices/authSlice';
import { uiSlice } from '../store/slices/uiSlice';

interface ExtendedRenderOptions {
  preloadedState?: Partial<RootState>;
  store?: ReturnType<typeof configureStore>;
}

export function renderWithProviders(
  ui: React.ReactElement,
  {
    preloadedState = {},
    store = configureStore({
      reducer: {
        auth: authSlice.reducer,
        ui: uiSlice.reducer,
      },
      preloadedState,
    }),
    ...renderOptions
  }: ExtendedRenderOptions = {}
) {
  function Wrapper({ children }: PropsWithChildren<{}>): JSX.Element {
    return <Provider store={store}>{children}</Provider>;
  }

  return { store, ...render(ui, { wrapper: Wrapper, ...renderOptions }) };
}

// Mock state creators
export const createMockAuthState = (overrides: Partial<AuthState> = {}): AuthState => ({
  user: null,
  token: null,
  refreshToken: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
  permissions: [],
  lastActivity: Date.now(),
  ...overrides,
});

export const createMockUser = (overrides: Partial<User> = {}): User => ({
  id: '1',
  email: 'test@example.com',
  name: 'Test User',
  role: 'user',
  permissions: [],
  preferences: {},
  createdAt: new Date().toISOString(),
  updatedAt: new Date().toISOString(),
  ...overrides,
});
```

### State Testing Examples
```typescript
// __tests__/authSlice.test.ts
import { configureStore } from '@reduxjs/toolkit';
import { authSlice, loginUser } from '../store/slices/authSlice';
import { createMockAuthState, createMockUser } from '../utils/testUtils';

describe('authSlice', () => {
  let store: ReturnType<typeof configureStore>;

  beforeEach(() => {
    store = configureStore({
      reducer: {
        auth: authSlice.reducer,
      },
    });
  });

  it('should handle login success', async () => {
    const mockUser = createMockUser();
    const mockResponse = {
      user: mockUser,
      token: 'mock-token',
      refreshToken: 'mock-refresh-token',
    };

    // Mock the API call
    jest.spyOn(authService, 'login').mockResolvedValue(mockResponse);

    await store.dispatch(loginUser({ email: 'test@example.com', password: 'password' }));

    const state = store.getState().auth;
    expect(state.isAuthenticated).toBe(true);
    expect(state.user).toEqual(mockUser);
    expect(state.token).toBe('mock-token');
    expect(state.isLoading).toBe(false);
  });

  it('should handle login failure', async () => {
    const errorMessage = 'Invalid credentials';
    jest.spyOn(authService, 'login').mockRejectedValue(new Error(errorMessage));

    await store.dispatch(loginUser({ email: 'test@example.com', password: 'wrong' }));

    const state = store.getState().auth;
    expect(state.isAuthenticated).toBe(false);
    expect(state.error).toBe(errorMessage);
    expect(state.isLoading).toBe(false);
  });
});
```

## Best Practices

### State Management Guidelines
1. **Keep State Normalized**: Avoid nested objects and arrays in state
2. **Use Selectors**: Create reusable selectors for computed state
3. **Minimize State**: Only store what's necessary in global state
4. **Separate Concerns**: Distinguish between client state and server state
5. **Handle Loading States**: Always manage loading and error states
6. **Optimize Re-renders**: Use memoization and proper selector patterns

### Performance Considerations
- Use `createSelector` for expensive computations
- Implement proper memoization in components
- Avoid creating objects in render functions
- Use `useCallback` and `useMemo` appropriately
- Consider state normalization for large datasets

---

**Document Status**: Active  
**Next Review**: 2025-04-27  
**Maintained By**: Frontend Development Team 