# Troubleshooting Documentation - DafnckMachine v3.1

## Overview
Comprehensive troubleshooting guide for the DafnckMachine v3.1 project, covering common issues, debugging techniques, and resolution strategies for development, testing, and production environments.

## Table of Contents

1. [Quick Reference](#quick-reference)
2. [Development Environment Issues](#development-environment-issues)
3. [Build and Compilation Issues](#build-and-compilation-issues)
4. [Database Issues](#database-issues)
5. [API and Backend Issues](#api-and-backend-issues)
6. [Frontend Issues](#frontend-issues)
7. [Testing Issues](#testing-issues)
8. [Deployment Issues](#deployment-issues)
9. [Performance Issues](#performance-issues)
10. [Security Issues](#security-issues)
11. [Monitoring and Logging](#monitoring-and-logging)
12. [Emergency Procedures](#emergency-procedures)

## Quick Reference

### Emergency Contacts
- **On-Call Engineer**: +1-555-0123 (24/7)
- **Tech Lead**: @tech-lead (Slack: @techleader)
- **DevOps Team**: #devops-alerts (Slack)
- **Security Team**: security@company.com

### Critical Commands
```bash
# Stop all services immediately
docker-compose down --remove-orphans

# Restart all services
npm run restart:all

# Check system health
npm run health:check

# View logs
npm run logs:all

# Emergency database backup
npm run db:backup:emergency
```

### Status Pages
- **Production Status**: https://status.dafnckmachine.com
- **Monitoring Dashboard**: https://monitoring.dafnckmachine.com
- **Error Tracking**: https://sentry.io/dafnckmachine

## Development Environment Issues

### 1. Node.js and npm Issues

#### Problem: Node version mismatch
```bash
# Error: Node version 16.x.x is not supported
# Solution: Update to Node 18+
nvm install 18
nvm use 18
npm install
```

#### Problem: npm install fails
```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and package-lock.json
rm -rf node_modules package-lock.json

# Reinstall dependencies
npm install

# If still failing, try with legacy peer deps
npm install --legacy-peer-deps
```

#### Problem: Permission errors on npm install
```bash
# Fix npm permissions (macOS/Linux)
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib/node_modules

# Or use nvm to avoid permission issues
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

### 2. Environment Configuration Issues

#### Problem: Environment variables not loading
```bash
# Check if .env files exist
ls -la .env*

# Verify environment variables are loaded
node -e "console.log(process.env.NODE_ENV)"

# Common .env file locations
frontend/.env.local
backend/.env
.env
```

#### Problem: Database connection string issues
```bash
# Test database connection
node -e "
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
pool.query('SELECT NOW()', (err, res) => {
  console.log(err ? err : res.rows[0]);
  pool.end();
});
"
```

### 3. Port Conflicts

#### Problem: Port already in use
```bash
# Find process using port 3000
lsof -ti:3000

# Kill process on port 3000
lsof -ti:3000 | xargs kill -9

# Kill multiple ports
for port in 3000 4000 5432 6379; do
  lsof -ti:$port | xargs kill -9 2>/dev/null
done
```

#### Problem: Docker port conflicts
```bash
# Stop all Docker containers
docker stop $(docker ps -aq)

# Remove all containers
docker rm $(docker ps -aq)

# Check port usage
netstat -tulpn | grep :3000
```

## Build and Compilation Issues

### 1. TypeScript Compilation Errors

#### Problem: Type errors in build
```bash
# Run type checking
npm run type-check

# Generate type definitions
npm run build:types

# Clear TypeScript cache
rm -rf node_modules/.cache/typescript
```

#### Problem: Module resolution errors
```typescript
// Check tsconfig.json paths configuration
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"]
    }
  }
}

// Verify import statements
import { Component } from '@/components/Component';
```

### 2. Build Performance Issues

#### Problem: Slow build times
```bash
# Enable build analysis
ANALYZE=true npm run build

# Use build cache
npm run build:cache

# Parallel builds
npm run build:parallel
```

#### Problem: Out of memory during build
```bash
# Increase Node.js memory limit
export NODE_OPTIONS="--max-old-space-size=8192"
npm run build

# Or set in package.json
"build": "NODE_OPTIONS='--max-old-space-size=8192' next build"
```

### 3. Dependency Issues

#### Problem: Conflicting dependencies
```bash
# Check for dependency conflicts
npm ls

# Audit dependencies
npm audit

# Fix vulnerabilities
npm audit fix

# Force resolution (package.json)
"overrides": {
  "package-name": "version"
}
```

## Database Issues

### 1. Connection Issues

#### Problem: Cannot connect to PostgreSQL
```bash
# Check if PostgreSQL is running
docker ps | grep postgres

# Start PostgreSQL container
docker-compose up -d postgres

# Test connection
psql $DATABASE_URL -c "SELECT version();"

# Check connection pool
node -e "
const { Pool } = require('pg');
const pool = new Pool();
pool.query('SELECT NOW()', console.log);
"
```

#### Problem: Redis connection issues
```bash
# Check Redis status
docker ps | grep redis

# Test Redis connection
redis-cli -u $REDIS_URL ping

# Check Redis memory usage
redis-cli info memory
```

### 2. Migration Issues

#### Problem: Migration fails
```bash
# Check migration status
npm run db:migrate:status

# Rollback last migration
npm run db:migrate:down

# Reset database (development only)
npm run db:reset

# Manual migration
npx prisma migrate deploy
```

#### Problem: Schema drift
```bash
# Generate new migration
npx prisma migrate dev --name fix-schema-drift

# Reset and regenerate
npx prisma migrate reset
npx prisma db push
```

### 3. Performance Issues

#### Problem: Slow database queries
```sql
-- Enable query logging (PostgreSQL)
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();

-- Check slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

#### Problem: Database locks
```sql
-- Check for locks
SELECT 
  blocked_locks.pid AS blocked_pid,
  blocked_activity.usename AS blocked_user,
  blocking_locks.pid AS blocking_pid,
  blocking_activity.usename AS blocking_user,
  blocked_activity.query AS blocked_statement,
  blocking_activity.query AS current_statement_in_blocking_process
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Kill blocking query (use with caution)
SELECT pg_terminate_backend(pid);
```

## API and Backend Issues

### 1. Server Startup Issues

#### Problem: Server won't start
```bash
# Check for syntax errors
npm run lint:backend

# Check for missing dependencies
npm install

# Start with debug logging
DEBUG=* npm run dev:backend

# Check port availability
netstat -tulpn | grep :4000
```

#### Problem: Module not found errors
```bash
# Clear require cache
rm -rf node_modules/.cache

# Rebuild native modules
npm rebuild

# Check import paths
node -e "console.log(require.resolve('module-name'))"
```

### 2. Authentication Issues

#### Problem: JWT token validation fails
```javascript
// Debug JWT token
const jwt = require('jsonwebtoken');
const token = 'your-jwt-token';

try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  console.log('Token valid:', decoded);
} catch (error) {
  console.log('Token invalid:', error.message);
}

// Check token expiration
const decoded = jwt.decode(token);
console.log('Expires at:', new Date(decoded.exp * 1000));
```

#### Problem: Session issues
```bash
# Clear Redis sessions
redis-cli FLUSHDB

# Check session storage
redis-cli KEYS "sess:*"

# Debug session middleware
DEBUG=express-session npm run dev:backend
```

### 3. API Response Issues

#### Problem: CORS errors
```javascript
// Check CORS configuration
const cors = require('cors');

app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

#### Problem: Rate limiting issues
```bash
# Check rate limit status
redis-cli GET "rate_limit:user_id"

# Reset rate limits
redis-cli DEL "rate_limit:*"

# Adjust rate limit configuration
```

## Frontend Issues

### 1. React Issues

#### Problem: Component not re-rendering
```javascript
// Check React DevTools
// Verify state updates
console.log('State before:', state);
setState(newState);
console.log('State after:', newState);

// Check dependencies in useEffect
useEffect(() => {
  // Effect logic
}, [dependency1, dependency2]); // Ensure all dependencies are listed
```

#### Problem: Memory leaks
```javascript
// Clean up subscriptions
useEffect(() => {
  const subscription = api.subscribe();
  
  return () => {
    subscription.unsubscribe();
  };
}, []);

// Cancel async operations
useEffect(() => {
  let cancelled = false;
  
  async function fetchData() {
    const data = await api.getData();
    if (!cancelled) {
      setData(data);
    }
  }
  
  fetchData();
  
  return () => {
    cancelled = true;
  };
}, []);
```

### 2. State Management Issues

#### Problem: Zustand store not updating
```javascript
// Check store subscription
import { useStore } from './store';

const Component = () => {
  const state = useStore();
  console.log('Current state:', state);
  
  // Verify store actions
  const updateState = useStore(state => state.updateState);
  
  return <div>{/* Component JSX */}</div>;
};
```

#### Problem: React Query cache issues
```javascript
// Invalidate queries
queryClient.invalidateQueries(['users']);

// Clear cache
queryClient.clear();

// Debug query state
const queryState = queryClient.getQueryState(['users']);
console.log('Query state:', queryState);
```

### 3. Routing Issues

#### Problem: Next.js routing not working
```bash
# Check file structure
ls -la pages/

# Verify dynamic routes
pages/users/[id].js

# Check router configuration
```

#### Problem: Client-side navigation issues
```javascript
// Debug router state
import { useRouter } from 'next/router';

const Component = () => {
  const router = useRouter();
  console.log('Router state:', router);
  
  // Check route parameters
  console.log('Query params:', router.query);
  
  return <div>{/* Component JSX */}</div>;
};
```

## Testing Issues

### 1. Unit Test Failures

#### Problem: Tests timing out
```javascript
// Increase timeout
jest.setTimeout(30000);

// Use async/await properly
test('async test', async () => {
  const result = await asyncFunction();
  expect(result).toBe(expected);
});

// Mock async functions
jest.mock('./api', () => ({
  getData: jest.fn().mockResolvedValue(mockData)
}));
```

#### Problem: Mock issues
```javascript
// Clear mocks between tests
beforeEach(() => {
  jest.clearAllMocks();
});

// Reset modules
beforeEach(() => {
  jest.resetModules();
});

// Mock implementation
const mockFunction = jest.fn().mockImplementation((arg) => {
  return `mocked-${arg}`;
});
```

### 2. Integration Test Issues

#### Problem: Database test isolation
```javascript
// Use test database
process.env.DATABASE_URL = 'postgresql://test:test@localhost:5432/test_db';

// Clean database between tests
beforeEach(async () => {
  await db.query('TRUNCATE TABLE users CASCADE');
});

// Use transactions for isolation
beforeEach(async () => {
  await db.query('BEGIN');
});

afterEach(async () => {
  await db.query('ROLLBACK');
});
```

### 3. E2E Test Issues

#### Problem: Playwright tests failing
```bash
# Update browsers
npx playwright install

# Run tests in headed mode
npx playwright test --headed

# Debug specific test
npx playwright test --debug test-file.spec.ts

# Generate test report
npx playwright show-report
```

#### Problem: Test flakiness
```javascript
// Add explicit waits
await page.waitForSelector('[data-testid="element"]');

// Wait for network idle
await page.waitForLoadState('networkidle');

// Retry assertions
await expect(page.locator('[data-testid="element"]')).toBeVisible({
  timeout: 10000
});
```

## Deployment Issues

### 1. Docker Issues

#### Problem: Container build failures
```bash
# Build with verbose output
docker build --progress=plain .

# Check Dockerfile syntax
docker build --dry-run .

# Clear Docker cache
docker system prune -a

# Build without cache
docker build --no-cache .
```

#### Problem: Container runtime issues
```bash
# Check container logs
docker logs container-name

# Execute into container
docker exec -it container-name /bin/bash

# Check container resources
docker stats container-name
```

### 2. Kubernetes Issues

#### Problem: Pod not starting
```bash
# Check pod status
kubectl get pods

# Describe pod
kubectl describe pod pod-name

# Check pod logs
kubectl logs pod-name

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

#### Problem: Service connectivity
```bash
# Test service connectivity
kubectl exec -it pod-name -- curl service-name:port

# Check service endpoints
kubectl get endpoints service-name

# Port forward for debugging
kubectl port-forward service/service-name 8080:80
```

### 3. CI/CD Pipeline Issues

#### Problem: GitHub Actions failing
```yaml
# Add debug steps
- name: Debug Environment
  run: |
    echo "Node version: $(node --version)"
    echo "NPM version: $(npm --version)"
    echo "Environment: $NODE_ENV"
    ls -la

# Check secrets
- name: Check Secrets
  run: |
    echo "Database URL exists: ${{ secrets.DATABASE_URL != '' }}"
```

#### Problem: Deployment rollback
```bash
# Kubernetes rollback
kubectl rollout undo deployment/app-deployment

# Check rollout status
kubectl rollout status deployment/app-deployment

# Manual rollback to specific revision
kubectl rollout undo deployment/app-deployment --to-revision=2
```

## Performance Issues

### 1. Frontend Performance

#### Problem: Slow page loads
```javascript
// Use React DevTools Profiler
// Check bundle size
npm run analyze

// Implement code splitting
const LazyComponent = React.lazy(() => import('./LazyComponent'));

// Optimize images
import Image from 'next/image';

<Image
  src="/image.jpg"
  alt="Description"
  width={500}
  height={300}
  priority
/>
```

#### Problem: Memory leaks
```javascript
// Use Chrome DevTools Memory tab
// Check for detached DOM nodes
// Monitor heap snapshots

// Fix common memory leaks
useEffect(() => {
  const timer = setInterval(() => {
    // Timer logic
  }, 1000);
  
  return () => clearInterval(timer); // Cleanup
}, []);
```

### 2. Backend Performance

#### Problem: Slow API responses
```javascript
// Add performance monitoring
const startTime = Date.now();
// API logic
const endTime = Date.now();
console.log(`API took ${endTime - startTime}ms`);

// Use APM tools
const apm = require('elastic-apm-node');
const span = apm.startSpan('database-query');
// Database operation
span.end();
```

#### Problem: High CPU usage
```bash
# Profile Node.js application
node --prof app.js

# Generate profile report
node --prof-process isolate-*.log > profile.txt

# Use clinic.js for profiling
npx clinic doctor -- node app.js
```

### 3. Database Performance

#### Problem: Slow queries
```sql
-- Enable slow query log
SET log_min_duration_statement = 1000;

-- Analyze query plan
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE email = 'test@example.com';

-- Check index usage
SELECT schemaname, tablename, attname, n_distinct, correlation
FROM pg_stats
WHERE tablename = 'users';
```

## Security Issues

### 1. Authentication Vulnerabilities

#### Problem: JWT token security
```javascript
// Check token security
const jwt = require('jsonwebtoken');

// Verify token signature
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET, {
    algorithms: ['HS256'],
    issuer: 'your-app',
    audience: 'your-audience'
  });
} catch (error) {
  console.log('Token verification failed:', error.message);
}

// Implement token rotation
const refreshToken = jwt.sign(
  { userId: user.id, type: 'refresh' },
  process.env.REFRESH_TOKEN_SECRET,
  { expiresIn: '7d' }
);
```

### 2. Input Validation Issues

#### Problem: SQL injection prevention
```javascript
// Use parameterized queries
const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [email]);

// Validate input
const Joi = require('joi');
const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

const { error, value } = schema.validate(input);
```

### 3. HTTPS and SSL Issues

#### Problem: SSL certificate issues
```bash
# Check certificate validity
openssl x509 -in certificate.crt -text -noout

# Test SSL connection
openssl s_client -connect domain.com:443

# Check certificate chain
curl -I https://domain.com
```

## Monitoring and Logging

### 1. Application Logging

#### Problem: Missing logs
```javascript
// Configure Winston logger
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Add request logging
app.use((req, res, next) => {
  logger.info(`${req.method} ${req.url}`, {
    ip: req.ip,
    userAgent: req.get('User-Agent')
  });
  next();
});
```

### 2. Error Tracking

#### Problem: Unhandled errors
```javascript
// Global error handlers
process.on('uncaughtException', (error) => {
  logger.error('Uncaught Exception:', error);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  logger.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

// Express error handler
app.use((error, req, res, next) => {
  logger.error('Express Error:', error);
  res.status(500).json({ error: 'Internal Server Error' });
});
```

### 3. Performance Monitoring

#### Problem: APM setup
```javascript
// Elastic APM
const apm = require('elastic-apm-node').start({
  serviceName: 'dafnckmachine-api',
  serverUrl: process.env.ELASTIC_APM_SERVER_URL,
  secretToken: process.env.ELASTIC_APM_SECRET_TOKEN
});

// New Relic
require('newrelic');

// DataDog
const tracer = require('dd-trace').init({
  service: 'dafnckmachine-api',
  env: process.env.NODE_ENV
});
```

## Emergency Procedures

### 1. Production Outage

#### Immediate Response (0-5 minutes)
1. **Acknowledge the incident** in #incidents Slack channel
2. **Check status page** and update if necessary
3. **Identify scope** of the outage
4. **Implement immediate mitigation** if possible

```bash
# Quick health checks
curl -f https://api.dafnckmachine.com/health
kubectl get pods -n production
docker ps | grep -v "Up"

# Emergency rollback
kubectl rollout undo deployment/api-deployment
```

#### Investigation (5-30 minutes)
1. **Gather logs** from all relevant services
2. **Check monitoring dashboards**
3. **Identify root cause**
4. **Implement fix** or workaround

```bash
# Gather logs
kubectl logs -f deployment/api-deployment --tail=100
docker logs container-name --tail=100

# Check system resources
kubectl top nodes
kubectl top pods
```

### 2. Database Emergency

#### Data Loss Prevention
```bash
# Immediate backup
pg_dump $DATABASE_URL > emergency_backup_$(date +%Y%m%d_%H%M%S).sql

# Stop write operations
kubectl scale deployment api-deployment --replicas=0

# Enable read-only mode
psql $DATABASE_URL -c "ALTER DATABASE dbname SET default_transaction_read_only = on;"
```

#### Recovery Procedures
```bash
# Restore from backup
psql $DATABASE_URL < backup_file.sql

# Point-in-time recovery
pg_basebackup -D /var/lib/postgresql/recovery -Ft -z -P

# Re-enable write operations
psql $DATABASE_URL -c "ALTER DATABASE dbname SET default_transaction_read_only = off;"
kubectl scale deployment api-deployment --replicas=3
```

### 3. Security Incident

#### Immediate Response
1. **Isolate affected systems**
2. **Revoke compromised credentials**
3. **Enable additional logging**
4. **Notify security team**

```bash
# Revoke JWT tokens
redis-cli FLUSHDB

# Disable user accounts
psql $DATABASE_URL -c "UPDATE users SET status = 'suspended' WHERE id = 'user_id';"

# Enable audit logging
psql $DATABASE_URL -c "ALTER SYSTEM SET log_statement = 'all';"
```

## Escalation Procedures

### Level 1: Development Team
- **Response Time**: 15 minutes during business hours
- **Contact**: #dev-team Slack channel
- **Scope**: Code issues, minor bugs, development environment

### Level 2: Senior Engineers
- **Response Time**: 30 minutes during business hours, 1 hour off-hours
- **Contact**: @senior-engineers Slack group
- **Scope**: Production issues, performance problems, complex bugs

### Level 3: On-Call Engineer
- **Response Time**: 15 minutes 24/7
- **Contact**: +1-555-0123 (PagerDuty)
- **Scope**: Critical production outages, security incidents

### Level 4: Management
- **Response Time**: 1 hour
- **Contact**: CTO, VP Engineering
- **Scope**: Major incidents affecting business operations

## Related Documentation

- [Code Documentation Standards](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Code_Documentation_Standards.md)
- [Architecture Decision Records](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Architecture_Decision_Records.md)
- [Developer Onboarding Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Developer_Onboarding_Guide.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: devops-team, development-team
- **Related Workflows**: 14_Technical_Documentation.md 