# Database Integration Guide - DafnckMachine v3.1

## Overview
Comprehensive guide for database integration in DafnckMachine v3.1, covering database setup, connection management, ORM configuration, migrations, and optimization strategies.

## Database Architecture

### Primary Database: PostgreSQL
- **Version**: PostgreSQL 15+
- **Provider**: Supabase (Primary) / Self-hosted (Alternative)
- **Connection**: SSL-enabled with connection pooling
- **Backup Strategy**: Automated daily backups with point-in-time recovery

### Database Schema Design
```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role VARCHAR(50) DEFAULT 'user',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE
);

-- Projects table
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'active',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tasks table
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'pending',
    priority VARCHAR(20) DEFAULT 'medium',
    assigned_to UUID REFERENCES users(id),
    due_date TIMESTAMP WITH TIME ZONE,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Connection Configuration

### Environment Variables
```env
# Primary Database (Supabase)
DATABASE_URL=postgresql://username:password@host:port/database
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Connection Pool Settings
DB_POOL_MIN=2
DB_POOL_MAX=20
DB_POOL_IDLE_TIMEOUT=30000
DB_POOL_ACQUIRE_TIMEOUT=60000

# SSL Configuration
DB_SSL_MODE=require
DB_SSL_REJECT_UNAUTHORIZED=true
```

### Connection Pool Configuration
```javascript
// Database connection pool setup
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false
  } : false,
  min: parseInt(process.env.DB_POOL_MIN) || 2,
  max: parseInt(process.env.DB_POOL_MAX) || 20,
  idleTimeoutMillis: parseInt(process.env.DB_POOL_IDLE_TIMEOUT) || 30000,
  acquireTimeoutMillis: parseInt(process.env.DB_POOL_ACQUIRE_TIMEOUT) || 60000,
});

module.exports = pool;
```

## ORM Integration

### Prisma Configuration
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  email     String   @unique
  password  String
  firstName String?  @map("first_name")
  lastName  String?  @map("last_name")
  role      String   @default("user")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  lastLogin DateTime? @map("last_login")
  
  projects  Project[]
  tasks     Task[]   @relation("AssignedTasks")
  
  @@map("users")
}

model Project {
  id          String   @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  userId      String   @map("user_id") @db.Uuid
  name        String
  description String?
  status      String   @default("active")
  settings    Json     @default("{}")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  tasks       Task[]
  
  @@map("projects")
}

model Task {
  id          String    @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  projectId   String    @map("project_id") @db.Uuid
  title       String
  description String?
  status      String    @default("pending")
  priority    String    @default("medium")
  assignedTo  String?   @map("assigned_to") @db.Uuid
  dueDate     DateTime? @map("due_date")
  metadata    Json      @default("{}")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  
  project     Project   @relation(fields: [projectId], references: [id], onDelete: Cascade)
  assignee    User?     @relation("AssignedTasks", fields: [assignedTo], references: [id])
  
  @@map("tasks")
}
```

### Prisma Client Setup
```javascript
// lib/prisma.js
const { PrismaClient } = require('@prisma/client');

const globalForPrisma = globalThis;

const prisma = globalForPrisma.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  errorFormat: 'pretty',
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

module.exports = prisma;
```

## Migration Management

### Migration Scripts
```bash
# Generate migration
npx prisma migrate dev --name init

# Apply migrations to production
npx prisma migrate deploy

# Reset database (development only)
npx prisma migrate reset

# Generate Prisma client
npx prisma generate
```

### Migration Best Practices
```sql
-- Example migration with proper indexing
-- migrations/001_initial_schema.sql

-- Create indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_projects_user_id ON projects(user_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_tasks_project_id ON tasks(project_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_assigned_to ON tasks(assigned_to);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);

-- Create composite indexes for common queries
CREATE INDEX idx_tasks_project_status ON tasks(project_id, status);
CREATE INDEX idx_projects_user_status ON projects(user_id, status);
```

## Data Access Layer

### Repository Pattern Implementation
```javascript
// repositories/BaseRepository.js
class BaseRepository {
  constructor(model) {
    this.model = model;
  }

  async findById(id) {
    return await this.model.findUnique({
      where: { id }
    });
  }

  async findMany(options = {}) {
    return await this.model.findMany(options);
  }

  async create(data) {
    return await this.model.create({
      data
    });
  }

  async update(id, data) {
    return await this.model.update({
      where: { id },
      data
    });
  }

  async delete(id) {
    return await this.model.delete({
      where: { id }
    });
  }

  async count(where = {}) {
    return await this.model.count({ where });
  }
}

module.exports = BaseRepository;
```

```javascript
// repositories/UserRepository.js
const BaseRepository = require('./BaseRepository');
const prisma = require('../lib/prisma');

class UserRepository extends BaseRepository {
  constructor() {
    super(prisma.user);
  }

  async findByEmail(email) {
    return await this.model.findUnique({
      where: { email }
    });
  }

  async findWithProjects(id) {
    return await this.model.findUnique({
      where: { id },
      include: {
        projects: {
          include: {
            tasks: true
          }
        }
      }
    });
  }

  async updateLastLogin(id) {
    return await this.model.update({
      where: { id },
      data: {
        lastLogin: new Date()
      }
    });
  }
}

module.exports = new UserRepository();
```

## Query Optimization

### Performance Best Practices
```javascript
// Efficient pagination
async function getPaginatedProjects(userId, page = 1, limit = 20) {
  const skip = (page - 1) * limit;
  
  const [projects, total] = await Promise.all([
    prisma.project.findMany({
      where: { userId },
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      include: {
        _count: {
          select: { tasks: true }
        }
      }
    }),
    prisma.project.count({
      where: { userId }
    })
  ]);

  return {
    projects,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  };
}

// Optimized search with full-text search
async function searchProjects(userId, searchTerm) {
  return await prisma.project.findMany({
    where: {
      userId,
      OR: [
        {
          name: {
            contains: searchTerm,
            mode: 'insensitive'
          }
        },
        {
          description: {
            contains: searchTerm,
            mode: 'insensitive'
          }
        }
      ]
    },
    orderBy: {
      _relevance: {
        fields: ['name', 'description'],
        search: searchTerm,
        sort: 'desc'
      }
    }
  });
}
```

### Database Indexes
```sql
-- Performance indexes
CREATE INDEX CONCURRENTLY idx_users_email_hash ON users USING hash(email);
CREATE INDEX CONCURRENTLY idx_projects_name_gin ON projects USING gin(to_tsvector('english', name));
CREATE INDEX CONCURRENTLY idx_tasks_search_gin ON tasks USING gin(to_tsvector('english', title || ' ' || COALESCE(description, '')));

-- Partial indexes for common filters
CREATE INDEX CONCURRENTLY idx_projects_active ON projects(user_id, created_at) WHERE status = 'active';
CREATE INDEX CONCURRENTLY idx_tasks_pending ON tasks(project_id, created_at) WHERE status = 'pending';
```

## Caching Strategy

### Redis Integration
```javascript
// lib/cache.js
const Redis = require('ioredis');

const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD,
  retryDelayOnFailover: 100,
  maxRetriesPerRequest: 3,
});

class CacheService {
  async get(key) {
    try {
      const value = await redis.get(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  async set(key, value, ttl = 3600) {
    try {
      await redis.setex(key, ttl, JSON.stringify(value));
    } catch (error) {
      console.error('Cache set error:', error);
    }
  }

  async del(key) {
    try {
      await redis.del(key);
    } catch (error) {
      console.error('Cache delete error:', error);
    }
  }

  async invalidatePattern(pattern) {
    try {
      const keys = await redis.keys(pattern);
      if (keys.length > 0) {
        await redis.del(...keys);
      }
    } catch (error) {
      console.error('Cache invalidation error:', error);
    }
  }
}

module.exports = new CacheService();
```

### Query Result Caching
```javascript
// services/ProjectService.js
const cache = require('../lib/cache');
const UserRepository = require('../repositories/UserRepository');

class ProjectService {
  async getUserProjects(userId, options = {}) {
    const cacheKey = `user:${userId}:projects:${JSON.stringify(options)}`;
    
    // Try cache first
    let projects = await cache.get(cacheKey);
    
    if (!projects) {
      // Fetch from database
      projects = await UserRepository.findWithProjects(userId);
      
      // Cache for 5 minutes
      await cache.set(cacheKey, projects, 300);
    }
    
    return projects;
  }

  async invalidateUserCache(userId) {
    await cache.invalidatePattern(`user:${userId}:*`);
  }
}

module.exports = new ProjectService();
```

## Backup and Recovery

### Automated Backup Strategy
```bash
#!/bin/bash
# scripts/backup-database.sh

# Environment variables
DB_HOST=${DB_HOST:-localhost}
DB_PORT=${DB_PORT:-5432}
DB_NAME=${DB_NAME:-dafnckmachine}
DB_USER=${DB_USER:-postgres}
BACKUP_DIR=${BACKUP_DIR:-/backups}

# Create backup directory
mkdir -p $BACKUP_DIR

# Generate backup filename with timestamp
BACKUP_FILE="$BACKUP_DIR/dafnckmachine_$(date +%Y%m%d_%H%M%S).sql"

# Create database backup
pg_dump -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME \
  --verbose --clean --no-owner --no-privileges \
  --file=$BACKUP_FILE

# Compress backup
gzip $BACKUP_FILE

# Remove backups older than 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: ${BACKUP_FILE}.gz"
```

### Point-in-Time Recovery
```bash
# Restore from backup
gunzip -c backup_file.sql.gz | psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME

# Point-in-time recovery (if using WAL archiving)
pg_basebackup -h $DB_HOST -D /recovery -U postgres -v -P -W
```

## Monitoring and Health Checks

### Database Health Monitoring
```javascript
// lib/health.js
const prisma = require('./prisma');

class DatabaseHealth {
  async checkConnection() {
    try {
      await prisma.$queryRaw`SELECT 1`;
      return { status: 'healthy', timestamp: new Date() };
    } catch (error) {
      return { 
        status: 'unhealthy', 
        error: error.message, 
        timestamp: new Date() 
      };
    }
  }

  async getConnectionStats() {
    try {
      const result = await prisma.$queryRaw`
        SELECT 
          count(*) as total_connections,
          count(*) FILTER (WHERE state = 'active') as active_connections,
          count(*) FILTER (WHERE state = 'idle') as idle_connections
        FROM pg_stat_activity 
        WHERE datname = current_database()
      `;
      return result[0];
    } catch (error) {
      throw new Error(`Failed to get connection stats: ${error.message}`);
    }
  }

  async getTableSizes() {
    try {
      return await prisma.$queryRaw`
        SELECT 
          schemaname,
          tablename,
          pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
        FROM pg_tables 
        WHERE schemaname = 'public'
        ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
      `;
    } catch (error) {
      throw new Error(`Failed to get table sizes: ${error.message}`);
    }
  }
}

module.exports = new DatabaseHealth();
```

## Security Best Practices

### SQL Injection Prevention
```javascript
// Always use parameterized queries
const getUserProjects = async (userId, status) => {
  // ✅ Safe - using Prisma ORM
  return await prisma.project.findMany({
    where: {
      userId,
      status
    }
  });
  
  // ✅ Safe - using parameterized raw query
  return await prisma.$queryRaw`
    SELECT * FROM projects 
    WHERE user_id = ${userId} AND status = ${status}
  `;
};

// ❌ Dangerous - SQL injection vulnerability
const unsafeQuery = async (userId, status) => {
  return await prisma.$queryRawUnsafe(
    `SELECT * FROM projects WHERE user_id = '${userId}' AND status = '${status}'`
  );
};
```

### Row Level Security (RLS)
```sql
-- Enable RLS on tables
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can only see their own projects" ON projects
  FOR ALL USING (user_id = auth.uid());

CREATE POLICY "Users can only see tasks in their projects" ON tasks
  FOR ALL USING (
    project_id IN (
      SELECT id FROM projects WHERE user_id = auth.uid()
    )
  );
```

## Troubleshooting

### Common Issues and Solutions

#### Connection Pool Exhaustion
```javascript
// Monitor pool status
const pool = require('./lib/database');

setInterval(() => {
  console.log('Pool status:', {
    totalCount: pool.totalCount,
    idleCount: pool.idleCount,
    waitingCount: pool.waitingCount
  });
}, 30000);
```

#### Slow Query Identification
```sql
-- Enable slow query logging
ALTER SYSTEM SET log_min_duration_statement = 1000; -- Log queries > 1 second
SELECT pg_reload_conf();

-- Find slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements 
ORDER BY mean_time DESC 
LIMIT 10;
```

#### Lock Monitoring
```sql
-- Monitor database locks
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
```

## Performance Optimization

### Query Performance Tips
1. **Use appropriate indexes** for frequently queried columns
2. **Limit result sets** with pagination
3. **Use connection pooling** to manage database connections
4. **Implement caching** for frequently accessed data
5. **Monitor query performance** with pg_stat_statements
6. **Use EXPLAIN ANALYZE** to understand query execution plans

### Database Maintenance
```sql
-- Regular maintenance tasks
VACUUM ANALYZE; -- Update statistics and reclaim space
REINDEX DATABASE dafnckmachine; -- Rebuild indexes
```

This guide provides comprehensive coverage of database integration for DafnckMachine v3.1, ensuring robust, scalable, and secure data management. 