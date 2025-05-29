# Developer Onboarding Guide - DafnckMachine v3.1

## Welcome to DafnckMachine v3.1! 🚀

This guide will help you get up and running as a developer on the DafnckMachine v3.1 project. Follow these steps to set up your development environment, understand our codebase, and start contributing effectively.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Project Structure](#project-structure)
4. [Development Workflow](#development-workflow)
5. [Coding Standards](#coding-standards)
6. [Testing Guidelines](#testing-guidelines)
7. [Deployment Process](#deployment-process)
8. [Tools and Resources](#tools-and-resources)
9. [Getting Help](#getting-help)
10. [First Tasks](#first-tasks)

## Prerequisites

### Required Knowledge
- **JavaScript/TypeScript**: Intermediate to advanced level
- **React**: Experience with hooks, context, and modern patterns
- **Node.js**: Backend development experience
- **Git**: Version control and collaboration workflows
- **SQL**: Database design and query optimization
- **REST/GraphQL**: API design and consumption

### Recommended Knowledge
- **Docker**: Containerization and orchestration
- **AWS**: Cloud services and deployment
- **Testing**: Unit, integration, and E2E testing
- **CI/CD**: Automated deployment pipelines

### Required Tools
- **Node.js**: Version 18.0 or higher
- **npm**: Version 8.0 or higher (or yarn 3.0+)
- **Git**: Version 2.30 or higher
- **Docker**: Version 20.0 or higher
- **VS Code**: Recommended IDE with extensions

## Environment Setup

### 1. System Requirements

```bash
# Check Node.js version
node --version  # Should be 18.0+

# Check npm version
npm --version   # Should be 8.0+

# Check Git version
git --version   # Should be 2.30+

# Check Docker version
docker --version # Should be 20.0+
```

### 2. Clone the Repository

```bash
# Clone the main repository
git clone https://github.com/your-org/dafnckmachine-v3.git
cd dafnckmachine-v3

# Set up upstream remote (for contributors)
git remote add upstream https://github.com/your-org/dafnckmachine-v3.git
```

### 3. Install Dependencies

```bash
# Install root dependencies
npm install

# Install frontend dependencies
cd frontend
npm install

# Install backend dependencies
cd ../backend
npm install

# Return to root
cd ..
```

### 4. Environment Configuration

```bash
# Copy environment templates
cp .env.example .env
cp frontend/.env.example frontend/.env.local
cp backend/.env.example backend/.env

# Edit environment files with your local settings
# See Environment Variables section below for details
```

### 5. Database Setup

```bash
# Start local PostgreSQL and Redis with Docker
docker-compose up -d postgres redis

# Run database migrations
cd backend
npm run db:migrate

# Seed development data
npm run db:seed

# Return to root
cd ..
```

### 6. Start Development Servers

```bash
# Start all services (recommended)
npm run dev

# Or start services individually:
# Frontend (React/Next.js)
npm run dev:frontend

# Backend (Node.js/Express)
npm run dev:backend

# Database services
npm run dev:db
```

### 7. Verify Installation

```bash
# Check if services are running
curl http://localhost:3000      # Frontend
curl http://localhost:4000/api  # Backend API
curl http://localhost:4000/graphql # GraphQL endpoint

# Run tests to ensure everything works
npm test
```

## Environment Variables

### Frontend (.env.local)
```bash
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:4000/api
NEXT_PUBLIC_GRAPHQL_URL=http://localhost:4000/graphql
NEXT_PUBLIC_WS_URL=ws://localhost:4000/graphql

# Authentication
NEXT_PUBLIC_AUTH_DOMAIN=your-auth-domain
NEXT_PUBLIC_AUTH_CLIENT_ID=your-client-id

# Feature Flags
NEXT_PUBLIC_ENABLE_ANALYTICS=false
NEXT_PUBLIC_ENABLE_DEBUG=true

# External Services
NEXT_PUBLIC_SENTRY_DSN=your-sentry-dsn
```

### Backend (.env)
```bash
# Server Configuration
PORT=4000
NODE_ENV=development
API_VERSION=v1

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dafnckmachine_dev
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=24h
REFRESH_TOKEN_SECRET=your-refresh-token-secret

# External Services
AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
AWS_REGION=us-east-1
S3_BUCKET=your-s3-bucket

# Email Service
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Monitoring
SENTRY_DSN=your-backend-sentry-dsn
LOG_LEVEL=debug
```

## Project Structure

```
dafnckmachine-v3/
├── frontend/                 # React/Next.js application
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   ├── pages/          # Next.js pages
│   │   ├── hooks/          # Custom React hooks
│   │   ├── services/       # API service layer
│   │   ├── store/          # State management
│   │   ├── styles/         # CSS and styling
│   │   └── utils/          # Utility functions
│   ├── public/             # Static assets
│   └── tests/              # Frontend tests
├── backend/                 # Node.js/Express API
│   ├── src/
│   │   ├── controllers/    # Route handlers
│   │   ├── services/       # Business logic
│   │   ├── models/         # Database models
│   │   ├── middleware/     # Express middleware
│   │   ├── routes/         # API routes
│   │   ├── graphql/        # GraphQL schema and resolvers
│   │   └── utils/          # Utility functions
│   ├── migrations/         # Database migrations
│   ├── seeds/              # Database seed data
│   └── tests/              # Backend tests
├── shared/                  # Shared types and utilities
│   ├── types/              # TypeScript type definitions
│   └── constants/          # Shared constants
├── docs/                    # Documentation
├── scripts/                 # Build and deployment scripts
├── docker/                  # Docker configurations
└── .github/                 # GitHub workflows and templates
```

### Key Directories Explained

#### Frontend Structure
- **`components/`**: Reusable UI components organized by feature
- **`pages/`**: Next.js file-based routing
- **`hooks/`**: Custom React hooks for shared logic
- **`services/`**: API client and external service integrations
- **`store/`**: Zustand stores and React Query configurations

#### Backend Structure
- **`controllers/`**: HTTP request handlers and response formatting
- **`services/`**: Business logic and data processing
- **`models/`**: Database models and schema definitions
- **`middleware/`**: Authentication, validation, and error handling
- **`graphql/`**: GraphQL schema, resolvers, and subscriptions

## Development Workflow

### 1. Branch Strategy

We use **Git Flow** with the following branches:

```bash
# Main branches
main        # Production-ready code
develop     # Integration branch for features

# Supporting branches
feature/*   # New features (branch from develop)
hotfix/*    # Critical fixes (branch from main)
release/*   # Release preparation (branch from develop)
```

### 2. Feature Development Process

```bash
# 1. Create feature branch
git checkout develop
git pull upstream develop
git checkout -b feature/user-authentication

# 2. Make changes and commit
git add .
git commit -m "feat: implement user authentication"

# 3. Push and create pull request
git push origin feature/user-authentication
# Create PR on GitHub targeting develop branch

# 4. After review and approval, merge via GitHub
# 5. Clean up local branch
git checkout develop
git pull upstream develop
git branch -d feature/user-authentication
```

### 3. Commit Message Convention

We follow **Conventional Commits** specification:

```bash
# Format: type(scope): description
feat(auth): add JWT token validation
fix(api): resolve user creation bug
docs(readme): update installation instructions
style(ui): improve button component styling
refactor(db): optimize user query performance
test(auth): add unit tests for login flow
chore(deps): update dependencies to latest versions

# Breaking changes
feat(api)!: change user endpoint response format
```

### 4. Code Review Process

1. **Self-Review**: Review your own code before creating PR
2. **Automated Checks**: Ensure all CI checks pass
3. **Peer Review**: At least one team member must approve
4. **Testing**: Verify changes work in development environment
5. **Documentation**: Update relevant documentation

### 5. Pull Request Template

```markdown
## Description
Brief description of changes made.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] E2E tests pass (if applicable)

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Code is commented where necessary
- [ ] Documentation updated
- [ ] No console.log statements left in code
```

## Coding Standards

### 1. TypeScript Configuration

```typescript
// tsconfig.json highlights
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### 2. ESLint and Prettier

```bash
# Run linting
npm run lint

# Fix auto-fixable issues
npm run lint:fix

# Format code
npm run format

# Check formatting
npm run format:check
```

### 3. Naming Conventions

```typescript
// Files and directories
kebab-case: user-profile.component.tsx
camelCase: userProfile.service.ts

// Variables and functions
const userName = 'john';
const getUserById = (id: string) => { };

// Classes and interfaces
class UserService { }
interface UserProfile { }

// Constants
const API_ENDPOINTS = {
  USERS: '/api/users',
  PROJECTS: '/api/projects'
};

// Components
const UserProfileCard = () => { };
```

### 4. Import Organization

```typescript
// 1. Node modules
import React from 'react';
import { NextPage } from 'next';
import axios from 'axios';

// 2. Internal modules (absolute imports)
import { UserService } from '@/services/user.service';
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/use-auth';

// 3. Relative imports
import './user-profile.styles.css';
import { validateUserData } from '../utils/validation';
```

### 5. Component Structure

```typescript
// React component template
import React from 'react';
import { ComponentProps } from './component.types';
import styles from './component.module.css';

interface Props extends ComponentProps {
  // Component-specific props
}

export const Component: React.FC<Props> = ({
  prop1,
  prop2,
  ...rest
}) => {
  // Hooks
  const [state, setState] = useState();
  
  // Effects
  useEffect(() => {
    // Effect logic
  }, []);
  
  // Event handlers
  const handleClick = () => {
    // Handler logic
  };
  
  // Render helpers
  const renderContent = () => {
    // Render logic
  };
  
  return (
    <div className={styles.container} {...rest}>
      {renderContent()}
    </div>
  );
};

Component.displayName = 'Component';
```

## Testing Guidelines

### 1. Testing Strategy

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test user.service.test.ts

# Run E2E tests
npm run test:e2e
```

### 2. Unit Testing

```typescript
// Example unit test
import { UserService } from '../user.service';
import { mockUserRepository } from '../__mocks__/user.repository';

describe('UserService', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService(mockUserRepository);
  });
  
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        firstName: 'John',
        lastName: 'Doe'
      };
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result.id).toBeDefined();
      expect(result.email).toBe(userData.email);
      expect(mockUserRepository.save).toHaveBeenCalledWith(
        expect.objectContaining(userData)
      );
    });
    
    it('should throw error for invalid email', async () => {
      // Arrange
      const userData = {
        email: 'invalid-email',
        firstName: 'John',
        lastName: 'Doe'
      };
      
      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects
        .toThrow('Invalid email format');
    });
  });
});
```

### 3. Component Testing

```typescript
// Example component test
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from '../user-card.component';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  };
  
  it('should display user information', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });
  
  it('should call onEdit when edit button is clicked', () => {
    const mockOnEdit = jest.fn();
    
    render(<UserCard user={mockUser} onEdit={mockOnEdit} />);
    
    fireEvent.click(screen.getByRole('button', { name: /edit/i }));
    
    expect(mockOnEdit).toHaveBeenCalledWith(mockUser.id);
  });
});
```

### 4. E2E Testing

```typescript
// Example E2E test
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  test('should create and edit user', async ({ page }) => {
    // Navigate to users page
    await page.goto('/users');
    
    // Create new user
    await page.click('[data-testid="create-user-button"]');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="first-name-input"]', 'John');
    await page.fill('[data-testid="last-name-input"]', 'Doe');
    await page.click('[data-testid="save-button"]');
    
    // Verify user was created
    await expect(page.locator('[data-testid="user-list"]'))
      .toContainText('John Doe');
    
    // Edit user
    await page.click('[data-testid="edit-user-button"]');
    await page.fill('[data-testid="first-name-input"]', 'Jane');
    await page.click('[data-testid="save-button"]');
    
    // Verify user was updated
    await expect(page.locator('[data-testid="user-list"]'))
      .toContainText('Jane Doe');
  });
});
```

## Deployment Process

### 1. Development Environment

```bash
# Start development environment
npm run dev

# Access applications
# Frontend: http://localhost:3000
# Backend API: http://localhost:4000/api
# GraphQL Playground: http://localhost:4000/graphql
```

### 2. Staging Environment

```bash
# Deploy to staging
npm run deploy:staging

# Run staging tests
npm run test:staging

# Access staging environment
# Frontend: https://staging.dafnckmachine.com
# Backend API: https://api-staging.dafnckmachine.com
```

### 3. Production Deployment

```bash
# Production deployment is automated via GitHub Actions
# Triggered by merging to main branch

# Manual deployment (emergency only)
npm run deploy:production
```

## Tools and Resources

### 1. Development Tools

#### VS Code Extensions
```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-eslint",
    "ms-playwright.playwright",
    "GraphQL.vscode-graphql",
    "ms-vscode.vscode-json"
  ]
}
```

#### VS Code Settings
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "emmet.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  }
}
```

### 2. Useful Commands

```bash
# Development
npm run dev              # Start all services
npm run dev:frontend     # Start frontend only
npm run dev:backend      # Start backend only
npm run dev:db          # Start database services

# Testing
npm test                # Run all tests
npm run test:unit       # Run unit tests
npm run test:integration # Run integration tests
npm run test:e2e        # Run E2E tests
npm run test:coverage   # Run tests with coverage

# Code Quality
npm run lint            # Run ESLint
npm run lint:fix        # Fix ESLint issues
npm run format          # Format code with Prettier
npm run type-check      # TypeScript type checking

# Database
npm run db:migrate      # Run migrations
npm run db:seed         # Seed database
npm run db:reset        # Reset database
npm run db:studio       # Open Prisma Studio

# Build
npm run build           # Build all applications
npm run build:frontend  # Build frontend
npm run build:backend   # Build backend

# Deployment
npm run deploy:staging  # Deploy to staging
npm run deploy:prod     # Deploy to production
```

### 3. Documentation Links

- **API Documentation**: http://localhost:4000/api-docs
- **GraphQL Playground**: http://localhost:4000/graphql
- **Storybook**: http://localhost:6006
- **Architecture Decisions**: [ADR Documents](./Architecture_Decision_Records.md)
- **Code Standards**: [Coding Standards](./Code_Documentation_Standards.md)

## Getting Help

### 1. Internal Resources

- **Team Chat**: #dafnckmachine-dev Slack channel
- **Documentation**: `/docs` directory in repository
- **Issue Tracker**: GitHub Issues
- **Code Reviews**: GitHub Pull Requests

### 2. Team Contacts

- **Tech Lead**: @tech-lead (Slack: @techleader)
- **Frontend Lead**: @frontend-lead (Slack: @frontendguru)
- **Backend Lead**: @backend-lead (Slack: @backendmaster)
- **DevOps Lead**: @devops-lead (Slack: @devopsexpert)

### 3. External Resources

- **React Documentation**: https://react.dev
- **Next.js Documentation**: https://nextjs.org/docs
- **Node.js Documentation**: https://nodejs.org/docs
- **TypeScript Documentation**: https://www.typescriptlang.org/docs
- **PostgreSQL Documentation**: https://www.postgresql.org/docs

## First Tasks

### Week 1: Environment and Codebase Familiarization

1. **Day 1-2**: Complete environment setup
   - [ ] Set up development environment
   - [ ] Run all tests successfully
   - [ ] Explore project structure
   - [ ] Read architecture documentation

2. **Day 3-4**: Code exploration
   - [ ] Review key components and services
   - [ ] Understand data flow and state management
   - [ ] Explore API endpoints and GraphQL schema
   - [ ] Review testing patterns

3. **Day 5**: First contribution
   - [ ] Fix a "good first issue" bug
   - [ ] Add a small feature or improvement
   - [ ] Create your first pull request
   - [ ] Participate in code review

### Week 2: Feature Development

1. **Small Feature**: Implement a minor feature end-to-end
   - [ ] Design and plan the feature
   - [ ] Implement frontend components
   - [ ] Create backend API endpoints
   - [ ] Write comprehensive tests
   - [ ] Update documentation

2. **Code Review**: Participate in team code reviews
   - [ ] Review other team members' PRs
   - [ ] Provide constructive feedback
   - [ ] Learn from feedback on your code

### Week 3-4: Integration and Ownership

1. **Medium Feature**: Take ownership of a larger feature
   - [ ] Work with product team on requirements
   - [ ] Design technical solution
   - [ ] Implement across full stack
   - [ ] Coordinate with other team members

2. **Process Improvement**: Contribute to team processes
   - [ ] Suggest improvements to development workflow
   - [ ] Update documentation based on your experience
   - [ ] Help onboard the next new team member

## Troubleshooting Common Issues

### 1. Environment Issues

```bash
# Node version issues
nvm use 18
npm install

# Port conflicts
lsof -ti:3000 | xargs kill -9  # Kill process on port 3000
lsof -ti:4000 | xargs kill -9  # Kill process on port 4000

# Database connection issues
docker-compose down
docker-compose up -d postgres redis
npm run db:migrate
```

### 2. Build Issues

```bash
# Clear caches
npm run clean
rm -rf node_modules package-lock.json
npm install

# TypeScript issues
npm run type-check
npx tsc --noEmit

# Dependency conflicts
npm ls
npm audit fix
```

### 3. Test Issues

```bash
# Update snapshots
npm test -- --updateSnapshot

# Clear test cache
npm test -- --clearCache

# Debug specific test
npm test -- --verbose user.test.ts
```

## Welcome Message

Congratulations on joining the DafnckMachine v3.1 team! 🎉

We're excited to have you on board and look forward to your contributions. This project represents the cutting edge of our technology stack, and your fresh perspective will be invaluable.

Remember:
- **Ask questions** - No question is too small
- **Take your time** - Learning a new codebase takes time
- **Contribute early** - Small contributions help you learn faster
- **Share feedback** - Help us improve our onboarding process

Welcome to the team! 🚀

## Related Documentation

- [Code Documentation Standards](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Code_Documentation_Standards.md)
- [Architecture Decision Records](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Architecture_Decision_Records.md)
- [Troubleshooting Documentation](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Troubleshooting_Documentation.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: development-team
- **Related Workflows**: 14_Technical_Documentation.md 