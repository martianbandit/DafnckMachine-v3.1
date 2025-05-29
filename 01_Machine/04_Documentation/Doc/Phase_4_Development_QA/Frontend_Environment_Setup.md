# Frontend Environment Setup - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Frontend Environment Setup Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive guidance for setting up the frontend development environment for DafnckMachine v3.1, including development tools, build configuration, and deployment preparation.

## Prerequisites

### System Requirements

#### Operating System Support
- **macOS**: 10.15+ (Catalina or later)
- **Windows**: 10/11 with WSL2 recommended
- **Linux**: Ubuntu 18.04+, CentOS 7+, or equivalent

#### Hardware Requirements
- **RAM**: Minimum 8GB, Recommended 16GB+
- **Storage**: 10GB+ free space for development tools
- **CPU**: Multi-core processor recommended
- **Network**: Stable internet connection for package downloads

### Required Software

#### Node.js and Package Manager
```bash
# Install Node.js (LTS version recommended)
# Using Node Version Manager (nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
nvm use --lts

# Verify installation
node --version  # Should be v18.x.x or later
npm --version   # Should be v9.x.x or later

# Install Yarn (optional but recommended)
npm install -g yarn
yarn --version
```

#### Git Version Control
```bash
# Install Git
# macOS (using Homebrew)
brew install git

# Ubuntu/Debian
sudo apt-get install git

# Windows (download from git-scm.com)
# Verify installation
git --version

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## Development Environment Setup

### Project Initialization

#### Clone Repository
```bash
# Clone the DafnckMachine repository
git clone https://github.com/your-org/dafnckmachine-v3.1.git
cd dafnckmachine-v3.1

# Switch to frontend directory
cd frontend

# Install dependencies
npm install
# or
yarn install
```

#### Environment Configuration
```bash
# Copy environment template
cp .env.example .env.local

# Edit environment variables
# .env.local
REACT_APP_API_URL=http://localhost:3001/api
REACT_APP_ENVIRONMENT=development
REACT_APP_VERSION=3.1.0
REACT_APP_SENTRY_DSN=your_sentry_dsn_here
REACT_APP_GOOGLE_ANALYTICS_ID=your_ga_id_here
```

### Development Tools Installation

#### Code Editor Setup (VS Code)
```bash
# Install VS Code extensions
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-eslint
code --install-extension formulahendry.auto-rename-tag
code --install-extension christian-kohler.path-intellisense
code --install-extension ms-vscode.vscode-json
```

#### VS Code Configuration
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "emmet.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  },
  "files.associations": {
    "*.css": "tailwindcss"
  },
  "tailwindCSS.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  }
}
```

#### Browser Development Tools
```bash
# Install browser extensions for development
# Chrome/Edge:
# - React Developer Tools
# - Redux DevTools
# - Lighthouse
# - Web Vitals

# Firefox:
# - React Developer Tools
# - Redux DevTools
```

## Build Configuration

### Package.json Scripts
```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build:staging": "REACT_APP_ENVIRONMENT=staging npm run build",
    "build:production": "REACT_APP_ENVIRONMENT=production npm run build",
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false",
    "test:ci": "CI=true react-scripts test --coverage --watchAll=false",
    "eject": "react-scripts eject",
    "lint": "eslint src --ext .ts,.tsx,.js,.jsx",
    "lint:fix": "eslint src --ext .ts,.tsx,.js,.jsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,js,jsx,css,md}\"",
    "type-check": "tsc --noEmit",
    "analyze": "npm run build && npx bundle-analyzer build/static/js/*.js",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook"
  }
}
```

### TypeScript Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "es6"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@/components/*": ["components/*"],
      "@/hooks/*": ["hooks/*"],
      "@/utils/*": ["utils/*"],
      "@/types/*": ["types/*"],
      "@/services/*": ["services/*"],
      "@/assets/*": ["assets/*"]
    }
  },
  "include": [
    "src"
  ],
  "exclude": [
    "node_modules",
    "build",
    "dist"
  ]
}
```

### ESLint Configuration
```json
// .eslintrc.json
{
  "extends": [
    "react-app",
    "react-app/jest",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2020,
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "plugins": [
    "@typescript-eslint",
    "react-hooks",
    "import"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-explicit-any": "warn",
    "import/order": [
      "error",
      {
        "groups": [
          "builtin",
          "external",
          "internal",
          "parent",
          "sibling",
          "index"
        ],
        "newlines-between": "always"
      }
    ]
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

### Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

## Development Server Configuration

### Webpack Dev Server Setup
```javascript
// craco.config.js (if using CRACO)
const path = require('path');

module.exports = {
  webpack: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  devServer: {
    port: 3000,
    hot: true,
    open: true,
    historyApiFallback: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        secure: false,
      },
    },
  },
  jest: {
    configure: {
      moduleNameMapping: {
        '^@/(.*)$': '<rootDir>/src/$1',
      },
    },
  },
};
```

### Environment Variables Setup
```bash
# .env.development
REACT_APP_API_URL=http://localhost:3001/api
REACT_APP_ENVIRONMENT=development
REACT_APP_LOG_LEVEL=debug
REACT_APP_ENABLE_MOCK_API=true

# .env.staging
REACT_APP_API_URL=https://staging-api.dafnckmachine.com/api
REACT_APP_ENVIRONMENT=staging
REACT_APP_LOG_LEVEL=info
REACT_APP_ENABLE_MOCK_API=false

# .env.production
REACT_APP_API_URL=https://api.dafnckmachine.com/api
REACT_APP_ENVIRONMENT=production
REACT_APP_LOG_LEVEL=error
REACT_APP_ENABLE_MOCK_API=false
```

## Testing Environment

### Jest Configuration
```json
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/reportWebVitals.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  transform: {
    '^.+\\.(ts|tsx)$': 'ts-jest',
  },
  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx', 'json'],
};
```

### Testing Setup File
```typescript
// src/setupTests.ts
import '@testing-library/jest-dom';
import { configure } from '@testing-library/react';

// Configure testing library
configure({ testIdAttribute: 'data-testid' });

// Mock environment variables
process.env.REACT_APP_API_URL = 'http://localhost:3001/api';
process.env.REACT_APP_ENVIRONMENT = 'test';

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

## Styling and UI Framework

### Tailwind CSS Setup
```bash
# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
        secondary: {
          50: '#f8fafc',
          500: '#64748b',
          900: '#0f172a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: Inter, system-ui, sans-serif;
  }
}

@layer components {
  .btn {
    @apply px-4 py-2 rounded-md font-medium transition-colors;
  }
  
  .btn-primary {
    @apply bg-primary-500 text-white hover:bg-primary-600;
  }
  
  .btn-secondary {
    @apply bg-secondary-500 text-white hover:bg-secondary-600;
  }
}
```

## State Management Setup

### Redux Toolkit Configuration
```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import authSlice from './slices/authSlice';
import uiSlice from './slices/uiSlice';
import { api } from './api';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    ui: uiSlice,
    api: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(api.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### React Query Setup
```typescript
// src/providers/QueryProvider.tsx
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: (failureCount, error: any) => {
        if (error?.status === 404) return false;
        return failureCount < 3;
      },
    },
  },
});

export const QueryProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      {process.env.NODE_ENV === 'development' && (
        <ReactQueryDevtools initialIsOpen={false} />
      )}
    </QueryClientProvider>
  );
};
```

## API Integration Setup

### Axios Configuration
```typescript
// src/services/api.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';

const createApiClient = (): AxiosInstance => {
  const client = axios.create({
    baseURL: process.env.REACT_APP_API_URL,
    timeout: 10000,
    headers: {
      'Content-Type': 'application/json',
    },
  });

  // Request interceptor
  client.interceptors.request.use(
    (config) => {
      const token = localStorage.getItem('access_token');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    },
    (error) => Promise.reject(error)
  );

  // Response interceptor
  client.interceptors.response.use(
    (response) => response,
    async (error) => {
      if (error.response?.status === 401) {
        // Handle token refresh or redirect to login
        localStorage.removeItem('access_token');
        window.location.href = '/login';
      }
      return Promise.reject(error);
    }
  );

  return client;
};

export const apiClient = createApiClient();
```

## Development Workflow

### Git Hooks Setup
```bash
# Install husky for git hooks
npm install -D husky lint-staged

# Initialize husky
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"

# Add commit-msg hook
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```

```json
// package.json
{
  "lint-staged": {
    "src/**/*.{ts,tsx,js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "src/**/*.{css,md}": [
      "prettier --write"
    ]
  }
}
```

### Commit Convention Setup
```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'chore',
        'ci',
        'build',
      ],
    ],
  },
};
```

## Performance Monitoring

### Bundle Analyzer Setup
```bash
# Install bundle analyzer
npm install -D webpack-bundle-analyzer

# Add script to package.json
"analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js"
```

### Web Vitals Monitoring
```typescript
// src/utils/webVitals.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

const sendToAnalytics = (metric: any) => {
  // Send to your analytics service
  console.log(metric);
};

export const reportWebVitals = () => {
  getCLS(sendToAnalytics);
  getFID(sendToAnalytics);
  getFCP(sendToAnalytics);
  getLCP(sendToAnalytics);
  getTTFB(sendToAnalytics);
};
```

## Debugging Tools

### React Developer Tools
```bash
# Browser extension installation
# Chrome: https://chrome.google.com/webstore/detail/react-developer-tools
# Firefox: https://addons.mozilla.org/en-US/firefox/addon/react-devtools/
```

### Redux DevTools
```typescript
// src/store/index.ts
export const store = configureStore({
  // ... other config
  devTools: process.env.NODE_ENV !== 'production' && {
    name: 'DafnckMachine v3.1',
    trace: true,
    traceLimit: 25,
  },
});
```

### Error Boundary Setup
```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Send to error reporting service
    if (process.env.NODE_ENV === 'production') {
      // Sentry.captureException(error);
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="error-boundary">
            <h2>Something went wrong.</h2>
            <details style={{ whiteSpace: 'pre-wrap' }}>
              {this.state.error && this.state.error.toString()}
            </details>
          </div>
        )
      );
    }

    return this.props.children;
  }
}
```

## Deployment Preparation

### Build Optimization
```json
// package.json
{
  "scripts": {
    "build:analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js",
    "build:profile": "npm run build -- --profile",
    "build:stats": "npm run build -- --stats"
  }
}
```

### Environment-Specific Builds
```bash
# Development build
npm run start

# Staging build
npm run build:staging

# Production build
npm run build:production
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Troubleshooting

### Common Issues and Solutions

#### Node Modules Issues
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

#### TypeScript Compilation Errors
```bash
# Check TypeScript configuration
npx tsc --noEmit

# Clear TypeScript cache
rm -rf node_modules/.cache
```

#### Build Performance Issues
```bash
# Analyze bundle size
npm run analyze

# Check for duplicate dependencies
npx npm-check-duplicates
```

### Development Server Issues
```bash
# Clear development cache
rm -rf node_modules/.cache

# Reset development server
npm start -- --reset-cache
```

## Best Practices

### 1. Code Organization
- **Feature-based structure**: Organize by features, not file types
- **Absolute imports**: Use path mapping for cleaner imports
- **Component co-location**: Keep related files together
- **Barrel exports**: Use index files for clean exports

### 2. Performance
- **Code splitting**: Implement route-based code splitting
- **Bundle optimization**: Analyze and optimize bundle size
- **Lazy loading**: Load components and routes on demand
- **Caching**: Implement proper caching strategies

### 3. Development Experience
- **Hot reloading**: Fast development feedback
- **Type safety**: Comprehensive TypeScript usage
- **Linting**: Consistent code quality
- **Testing**: Comprehensive test coverage

### 4. Deployment
- **Environment configuration**: Proper environment variable management
- **Build optimization**: Optimized production builds
- **Error monitoring**: Comprehensive error tracking
- **Performance monitoring**: Real-time performance metrics

This comprehensive environment setup ensures a robust, scalable, and maintainable frontend development environment for DafnckMachine v3.1. 