# Database Schema Design - DafnckMachine v3.1

## Overview
Comprehensive database schema design and management guidelines for the DafnckMachine v3.1 project, covering data modeling, relationships, constraints, and optimization strategies.

## Database Architecture

### Primary Database: PostgreSQL
- **Version**: 16.0+
- **Purpose**: Primary relational database for application data
- **Connection Pool**: pgBouncer for connection management
- **Backup Strategy**: Automated daily backups with point-in-time recovery

### Secondary Storage: Redis
- **Version**: 7.2+
- **Purpose**: Caching, session storage, and real-time data
- **Configuration**: Cluster mode for high availability
- **Persistence**: RDB + AOF for data durability

## Core Schema Design

### User Management Schema

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    avatar_url TEXT,
    email_verified BOOLEAN DEFAULT FALSE,
    status user_status DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login_at TIMESTAMP WITH TIME ZONE,
    metadata JSONB DEFAULT '{}'::jsonb
);

-- User profiles
CREATE TABLE user_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    location VARCHAR(255),
    website_url TEXT,
    social_links JSONB DEFAULT '{}'::jsonb,
    preferences JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- User sessions
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) UNIQUE NOT NULL,
    refresh_token VARCHAR(255) UNIQUE,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    ip_address INET,
    user_agent TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Project Management Schema

```sql
-- Projects
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id UUID NOT NULL REFERENCES users(id),
    status project_status DEFAULT 'active',
    visibility project_visibility DEFAULT 'private',
    settings JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    archived_at TIMESTAMP WITH TIME ZONE
);

-- Project members
CREATE TABLE project_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role project_role DEFAULT 'member',
    permissions JSONB DEFAULT '{}'::jsonb,
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(project_id, user_id)
);

-- Tasks
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    parent_task_id UUID REFERENCES tasks(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status task_status DEFAULT 'pending',
    priority task_priority DEFAULT 'medium',
    assignee_id UUID REFERENCES users(id),
    creator_id UUID NOT NULL REFERENCES users(id),
    due_date TIMESTAMP WITH TIME ZONE,
    estimated_hours INTEGER,
    actual_hours INTEGER,
    tags TEXT[],
    metadata JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE
);

-- Task dependencies
CREATE TABLE task_dependencies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    depends_on_task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    dependency_type dependency_type DEFAULT 'blocks',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(task_id, depends_on_task_id)
);
```

### Content Management Schema

```sql
-- Documents
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    content_type VARCHAR(50) DEFAULT 'markdown',
    author_id UUID NOT NULL REFERENCES users(id),
    version INTEGER DEFAULT 1,
    status document_status DEFAULT 'draft',
    tags TEXT[],
    metadata JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    published_at TIMESTAMP WITH TIME ZONE
);

-- Document versions
CREATE TABLE document_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    version_number INTEGER NOT NULL,
    content TEXT NOT NULL,
    change_summary TEXT,
    author_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(document_id, version_number)
);

-- File attachments
CREATE TABLE file_attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(50) NOT NULL, -- 'task', 'document', 'project'
    entity_id UUID NOT NULL,
    filename VARCHAR(255) NOT NULL,
    original_filename VARCHAR(255) NOT NULL,
    file_size BIGINT NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    storage_path TEXT NOT NULL,
    uploader_id UUID NOT NULL REFERENCES users(id),
    metadata JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Custom Types and Enums

```sql
-- User status enum
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'suspended', 'deleted');

-- Project status enum
CREATE TYPE project_status AS ENUM ('active', 'completed', 'archived', 'cancelled');

-- Project visibility enum
CREATE TYPE project_visibility AS ENUM ('private', 'internal', 'public');

-- Project role enum
CREATE TYPE project_role AS ENUM ('owner', 'admin', 'member', 'viewer');

-- Task status enum
CREATE TYPE task_status AS ENUM ('pending', 'in_progress', 'review', 'done', 'cancelled', 'blocked');

-- Task priority enum
CREATE TYPE task_priority AS ENUM ('low', 'medium', 'high', 'urgent');

-- Dependency type enum
CREATE TYPE dependency_type AS ENUM ('blocks', 'relates_to', 'duplicates', 'subtask_of');

-- Document status enum
CREATE TYPE document_status AS ENUM ('draft', 'review', 'published', 'archived');
```

## Indexes and Performance Optimization

### Primary Indexes

```sql
-- User-related indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Session indexes
CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token ON user_sessions(session_token);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions(expires_at);

-- Project indexes
CREATE INDEX idx_projects_owner_id ON projects(owner_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_created_at ON projects(created_at);

-- Task indexes
CREATE INDEX idx_tasks_project_id ON tasks(project_id);
CREATE INDEX idx_tasks_assignee_id ON tasks(assignee_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_priority ON tasks(priority);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_parent_task_id ON tasks(parent_task_id);

-- Document indexes
CREATE INDEX idx_documents_project_id ON documents(project_id);
CREATE INDEX idx_documents_author_id ON documents(author_id);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_documents_created_at ON documents(created_at);
```

### Composite Indexes

```sql
-- User session composite index
CREATE INDEX idx_user_sessions_user_active ON user_sessions(user_id, is_active);

-- Project member composite index
CREATE INDEX idx_project_members_project_role ON project_members(project_id, role);

-- Task composite indexes
CREATE INDEX idx_tasks_project_status ON tasks(project_id, status);
CREATE INDEX idx_tasks_assignee_status ON tasks(assignee_id, status);
CREATE INDEX idx_tasks_project_priority ON tasks(project_id, priority);

-- Document composite index
CREATE INDEX idx_documents_project_status ON documents(project_id, status);
```

### JSONB Indexes

```sql
-- JSONB GIN indexes for metadata searches
CREATE INDEX idx_users_metadata_gin ON users USING GIN (metadata);
CREATE INDEX idx_projects_settings_gin ON projects USING GIN (settings);
CREATE INDEX idx_tasks_metadata_gin ON tasks USING GIN (metadata);
CREATE INDEX idx_documents_metadata_gin ON documents USING GIN (metadata);

-- Specific JSONB path indexes
CREATE INDEX idx_user_preferences ON user_profiles USING GIN ((preferences));
CREATE INDEX idx_project_settings_theme ON projects USING GIN ((settings->'theme'));
```

## Data Constraints and Validation

### Check Constraints

```sql
-- User constraints
ALTER TABLE users ADD CONSTRAINT chk_email_format 
    CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

ALTER TABLE users ADD CONSTRAINT chk_username_length 
    CHECK (char_length(username) >= 3 AND char_length(username) <= 50);

-- Task constraints
ALTER TABLE tasks ADD CONSTRAINT chk_estimated_hours_positive 
    CHECK (estimated_hours IS NULL OR estimated_hours > 0);

ALTER TABLE tasks ADD CONSTRAINT chk_actual_hours_positive 
    CHECK (actual_hours IS NULL OR actual_hours > 0);

ALTER TABLE tasks ADD CONSTRAINT chk_due_date_future 
    CHECK (due_date IS NULL OR due_date > created_at);

-- File attachment constraints
ALTER TABLE file_attachments ADD CONSTRAINT chk_file_size_positive 
    CHECK (file_size > 0);

ALTER TABLE file_attachments ADD CONSTRAINT chk_filename_not_empty 
    CHECK (char_length(filename) > 0);
```

### Foreign Key Constraints

```sql
-- Ensure referential integrity
ALTER TABLE user_profiles ADD CONSTRAINT fk_user_profiles_user_id 
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE project_members ADD CONSTRAINT fk_project_members_project_id 
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE;

ALTER TABLE project_members ADD CONSTRAINT fk_project_members_user_id 
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

-- Prevent circular task dependencies
ALTER TABLE task_dependencies ADD CONSTRAINT chk_no_self_dependency 
    CHECK (task_id != depends_on_task_id);
```

## Database Functions and Triggers

### Audit Trail Function

```sql
-- Generic audit trail function
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' THEN
        NEW.updated_at = NOW();
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Apply to relevant tables
CREATE TRIGGER users_updated_at_trigger
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER projects_updated_at_trigger
    BEFORE UPDATE ON projects
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER tasks_updated_at_trigger
    BEFORE UPDATE ON tasks
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();
```

### Task Completion Function

```sql
-- Function to handle task completion
CREATE OR REPLACE FUNCTION complete_task(task_uuid UUID)
RETURNS VOID AS $$
BEGIN
    UPDATE tasks 
    SET status = 'done', 
        completed_at = NOW(),
        updated_at = NOW()
    WHERE id = task_uuid;
    
    -- Update project progress if needed
    -- Additional business logic here
END;
$$ LANGUAGE plpgsql;
```

## Data Migration Strategy

### Version Control
- Use database migration files with sequential numbering
- Each migration includes both up and down scripts
- Track migrations in dedicated `schema_migrations` table

### Migration Best Practices
1. **Backward Compatibility**: Ensure migrations don't break existing functionality
2. **Data Preservation**: Always backup data before destructive operations
3. **Rollback Strategy**: Test rollback procedures before deployment
4. **Performance Impact**: Consider migration performance on large datasets

### Sample Migration Structure

```sql
-- Migration: 001_create_users_table.sql
-- Up
CREATE TABLE users (
    -- table definition
);

-- Down
DROP TABLE IF EXISTS users;
```

## Security Considerations

### Data Encryption
- **At Rest**: Use PostgreSQL's built-in encryption features
- **In Transit**: Enforce SSL/TLS connections
- **Application Level**: Encrypt sensitive fields (passwords, PII)

### Access Control
- **Row Level Security**: Implement RLS policies for multi-tenant data
- **Column Level**: Restrict access to sensitive columns
- **Connection Security**: Use connection pooling with authentication

### Example RLS Policy

```sql
-- Enable RLS on projects table
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Policy for project access
CREATE POLICY project_access_policy ON projects
    FOR ALL TO application_role
    USING (
        owner_id = current_setting('app.current_user_id')::UUID
        OR id IN (
            SELECT project_id FROM project_members 
            WHERE user_id = current_setting('app.current_user_id')::UUID
        )
    );
```

## Backup and Recovery

### Backup Strategy
- **Full Backups**: Daily automated backups
- **Incremental Backups**: Hourly transaction log backups
- **Point-in-Time Recovery**: 30-day retention period
- **Cross-Region Replication**: For disaster recovery

### Recovery Procedures
1. **Automated Recovery**: Scripts for common recovery scenarios
2. **Manual Recovery**: Step-by-step procedures for complex situations
3. **Testing**: Regular recovery testing and validation

## Monitoring and Maintenance

### Performance Monitoring
- **Query Performance**: Track slow queries and optimization opportunities
- **Index Usage**: Monitor index effectiveness and unused indexes
- **Connection Pooling**: Monitor connection pool utilization
- **Storage Growth**: Track database size and growth patterns

### Maintenance Tasks
- **VACUUM and ANALYZE**: Automated maintenance schedules
- **Index Maintenance**: Regular index rebuilding and optimization
- **Statistics Updates**: Keep query planner statistics current
- **Log Rotation**: Manage database log files

## Related Documentation

- [Backend Architecture Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Backend_Architecture_Implementation.md)
- [API Documentation Complete](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)
- [Performance Optimization Strategies](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Strategies.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: system-architect-agent
- **Related Workflows**: 13_Backend_Development.md 