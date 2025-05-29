# Database Schema Design
## DafnckMachine v3.1 - Comprehensive Database Architecture & Data Modeling

### Document Overview
This document defines the comprehensive database schema design for DafnckMachine v3.1, establishing database architecture, entity relationship modeling, indexing strategies, normalization optimization, query performance guidelines, and data management standards to ensure efficient, scalable, and maintainable data storage across the AI agent orchestration platform.

### Database Architecture Overview

#### Database Technology Selection
```sql
-- Primary Database: PostgreSQL 15+
-- Rationale: ACID compliance, JSON support, advanced indexing, scalability
-- Secondary: Redis for caching and session management
-- Search: Elasticsearch for full-text search and analytics

-- Database Configuration
CREATE DATABASE dafnckmachine_v3
  WITH 
  ENCODING = 'UTF8'
  LC_COLLATE = 'en_US.UTF-8'
  LC_CTYPE = 'en_US.UTF-8'
  TEMPLATE = template0;

-- Extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "btree_gin";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";
```

#### Database Architecture Principles
1. **Normalization**: Third Normal Form (3NF) with selective denormalization for performance
2. **Scalability**: Horizontal partitioning and read replicas
3. **Performance**: Strategic indexing and query optimization
4. **Security**: Row-level security and data encryption
5. **Consistency**: ACID transactions and referential integrity
6. **Flexibility**: JSON columns for dynamic data structures
7. **Auditability**: Comprehensive audit trails and versioning

### Core Entity Schema Design

#### User Management Schema
```sql
-- Users table - Core user authentication and profile
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    avatar_url TEXT,
    timezone VARCHAR(50) DEFAULT 'UTC',
    locale VARCHAR(10) DEFAULT 'en-US',
    status user_status DEFAULT 'active',
    email_verified BOOLEAN DEFAULT FALSE,
    two_factor_enabled BOOLEAN DEFAULT FALSE,
    two_factor_secret VARCHAR(32),
    last_login_at TIMESTAMP WITH TIME ZONE,
    password_changed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted_at TIMESTAMP WITH TIME ZONE,
    
    -- Constraints
    CONSTRAINT users_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT users_username_format CHECK (username ~* '^[a-zA-Z0-9_-]{3,100}$')
);

-- User status enum
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'suspended', 'pending_verification');

-- User roles and permissions
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    permissions JSONB NOT NULL DEFAULT '[]',
    is_system_role BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE user_roles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    granted_by UUID REFERENCES users(id),
    granted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    
    UNIQUE(user_id, role_id)
);

-- User sessions for authentication tracking
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    refresh_token_hash VARCHAR(255),
    ip_address INET,
    user_agent TEXT,
    device_fingerprint VARCHAR(255),
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    last_activity_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked_at TIMESTAMP WITH TIME ZONE,
    
    CONSTRAINT sessions_not_expired CHECK (expires_at > created_at)
);
```

#### Agent Management Schema
```sql
-- Agent types and capabilities
CREATE TYPE agent_type AS ENUM (
    'orchestrator', 'worker', 'monitor', 'analyzer', 
    'processor', 'communicator', 'validator'
);

CREATE TYPE agent_status AS ENUM (
    'active', 'inactive', 'error', 'maintenance', 
    'initializing', 'terminating'
);

-- Core agents table
CREATE TABLE agents (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    type agent_type NOT NULL,
    status agent_status DEFAULT 'inactive',
    version VARCHAR(50) NOT NULL DEFAULT '1.0.0',
    description TEXT,
    capabilities JSONB NOT NULL DEFAULT '[]',
    configuration JSONB NOT NULL DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    
    -- Resource management
    max_concurrent_tasks INTEGER DEFAULT 10,
    memory_limit_mb INTEGER DEFAULT 1024,
    cpu_limit_percent INTEGER DEFAULT 100,
    
    -- Health and monitoring
    health_check_url TEXT,
    health_status VARCHAR(50) DEFAULT 'unknown',
    last_health_check TIMESTAMP WITH TIME ZONE,
    error_count INTEGER DEFAULT 0,
    last_error_at TIMESTAMP WITH TIME ZONE,
    last_error_message TEXT,
    
    -- Lifecycle management
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    started_at TIMESTAMP WITH TIME ZONE,
    stopped_at TIMESTAMP WITH TIME ZONE,
    deleted_at TIMESTAMP WITH TIME ZONE,
    
    -- Constraints
    CONSTRAINT agents_name_not_empty CHECK (LENGTH(TRIM(name)) > 0),
    CONSTRAINT agents_positive_limits CHECK (
        max_concurrent_tasks > 0 AND 
        memory_limit_mb > 0 AND 
        cpu_limit_percent > 0 AND 
        cpu_limit_percent <= 100
    )
);

-- Agent dependencies and relationships
CREATE TABLE agent_dependencies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    depends_on_agent_id UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    dependency_type VARCHAR(50) NOT NULL DEFAULT 'requires',
    is_critical BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(agent_id, depends_on_agent_id),
    CONSTRAINT no_self_dependency CHECK (agent_id != depends_on_agent_id)
);

-- Agent metrics and performance tracking
CREATE TABLE agent_metrics (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    metric_name VARCHAR(100) NOT NULL,
    metric_value NUMERIC NOT NULL,
    metric_unit VARCHAR(50),
    tags JSONB DEFAULT '{}',
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Partitioning key for time-series data
    CONSTRAINT agent_metrics_partition_key CHECK (recorded_at >= '2025-01-01'::date)
) PARTITION BY RANGE (recorded_at);

-- Create monthly partitions for metrics
CREATE TABLE agent_metrics_2025_01 PARTITION OF agent_metrics
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

#### Workflow Management Schema
```sql
-- Workflow definitions and templates
CREATE TYPE workflow_status AS ENUM (
    'draft', 'active', 'paused', 'archived', 'error'
);

CREATE TABLE workflows (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    version VARCHAR(50) NOT NULL DEFAULT '1.0.0',
    status workflow_status DEFAULT 'draft',
    
    -- Workflow definition
    definition JSONB NOT NULL,
    input_schema JSONB,
    output_schema JSONB,
    
    -- Execution settings
    timeout_seconds INTEGER DEFAULT 3600,
    max_retries INTEGER DEFAULT 3,
    retry_delay_seconds INTEGER DEFAULT 60,
    priority INTEGER DEFAULT 5,
    
    -- Scheduling
    schedule_cron VARCHAR(100),
    schedule_timezone VARCHAR(50) DEFAULT 'UTC',
    next_run_at TIMESTAMP WITH TIME ZONE,
    
    -- Metadata
    tags JSONB DEFAULT '[]',
    metadata JSONB DEFAULT '{}',
    
    -- Lifecycle
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    published_at TIMESTAMP WITH TIME ZONE,
    archived_at TIMESTAMP WITH TIME ZONE,
    
    CONSTRAINT workflows_name_not_empty CHECK (LENGTH(TRIM(name)) > 0),
    CONSTRAINT workflows_positive_timeout CHECK (timeout_seconds > 0),
    CONSTRAINT workflows_valid_priority CHECK (priority BETWEEN 1 AND 10)
);

-- Workflow executions
CREATE TYPE execution_status AS ENUM (
    'queued', 'running', 'completed', 'failed', 
    'cancelled', 'timeout', 'retrying'
);

CREATE TABLE workflow_executions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    status execution_status DEFAULT 'queued',
    
    -- Execution context
    input_data JSONB,
    output_data JSONB,
    error_message TEXT,
    error_details JSONB,
    
    -- Timing
    queued_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    duration_ms INTEGER,
    
    -- Execution metadata
    triggered_by VARCHAR(100), -- 'manual', 'schedule', 'webhook', 'api'
    triggered_by_user_id UUID REFERENCES users(id),
    execution_context JSONB DEFAULT '{}',
    retry_count INTEGER DEFAULT 0,
    parent_execution_id UUID REFERENCES workflow_executions(id),
    
    -- Resource usage
    memory_used_mb INTEGER,
    cpu_time_ms INTEGER,
    
    CONSTRAINT execution_timing_logic CHECK (
        (started_at IS NULL OR started_at >= queued_at) AND
        (completed_at IS NULL OR completed_at >= started_at)
    )
) PARTITION BY RANGE (queued_at);

-- Create monthly partitions for executions
CREATE TABLE workflow_executions_2025_01 PARTITION OF workflow_executions
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

-- Workflow step executions
CREATE TABLE workflow_step_executions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    execution_id UUID NOT NULL REFERENCES workflow_executions(id) ON DELETE CASCADE,
    step_name VARCHAR(255) NOT NULL,
    step_type VARCHAR(100) NOT NULL,
    agent_id UUID REFERENCES agents(id),
    status execution_status DEFAULT 'queued',
    
    -- Step data
    input_data JSONB,
    output_data JSONB,
    error_message TEXT,
    
    -- Timing
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    duration_ms INTEGER,
    
    -- Execution order
    sequence_number INTEGER NOT NULL,
    retry_count INTEGER DEFAULT 0,
    
    CONSTRAINT step_sequence_positive CHECK (sequence_number > 0)
);
```

#### Task and Job Management Schema
```sql
-- Task definitions and management
CREATE TYPE task_status AS ENUM (
    'pending', 'queued', 'running', 'completed', 
    'failed', 'cancelled', 'paused'
);

CREATE TYPE task_priority AS ENUM ('low', 'normal', 'high', 'urgent');

CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(100) NOT NULL,
    status task_status DEFAULT 'pending',
    priority task_priority DEFAULT 'normal',
    
    -- Task definition
    payload JSONB NOT NULL DEFAULT '{}',
    configuration JSONB DEFAULT '{}',
    
    -- Assignment
    assigned_agent_id UUID REFERENCES agents(id),
    workflow_execution_id UUID REFERENCES workflow_executions(id),
    parent_task_id UUID REFERENCES tasks(id),
    
    -- Scheduling and timing
    scheduled_at TIMESTAMP WITH TIME ZONE,
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    timeout_at TIMESTAMP WITH TIME ZONE,
    
    -- Retry logic
    max_retries INTEGER DEFAULT 3,
    retry_count INTEGER DEFAULT 0,
    retry_delay_seconds INTEGER DEFAULT 60,
    
    -- Results
    result JSONB,
    error_message TEXT,
    error_details JSONB,
    
    -- Metadata
    tags JSONB DEFAULT '[]',
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT tasks_name_not_empty CHECK (LENGTH(TRIM(name)) > 0),
    CONSTRAINT tasks_retry_logic CHECK (retry_count <= max_retries)
) PARTITION BY RANGE (created_at);

-- Task dependencies
CREATE TABLE task_dependencies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    depends_on_task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    dependency_type VARCHAR(50) DEFAULT 'completion',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(task_id, depends_on_task_id),
    CONSTRAINT no_task_self_dependency CHECK (task_id != depends_on_task_id)
);
```

#### Audit and Logging Schema
```sql
-- Comprehensive audit trail
CREATE TYPE audit_action AS ENUM (
    'create', 'update', 'delete', 'login', 'logout', 
    'execute', 'start', 'stop', 'pause', 'resume'
);

CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    entity_type VARCHAR(100) NOT NULL,
    entity_id UUID NOT NULL,
    action audit_action NOT NULL,
    
    -- Actor information
    user_id UUID REFERENCES users(id),
    session_id UUID REFERENCES user_sessions(id),
    ip_address INET,
    user_agent TEXT,
    
    -- Change details
    old_values JSONB,
    new_values JSONB,
    changed_fields TEXT[],
    
    -- Context
    context JSONB DEFAULT '{}',
    reason TEXT,
    
    -- Timing
    occurred_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Indexing hints
    CONSTRAINT audit_logs_partition_key CHECK (occurred_at >= '2025-01-01'::date)
) PARTITION BY RANGE (occurred_at);

-- System logs for application events
CREATE TYPE log_level AS ENUM ('debug', 'info', 'warn', 'error', 'fatal');

CREATE TABLE system_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    level log_level NOT NULL,
    message TEXT NOT NULL,
    component VARCHAR(100) NOT NULL,
    
    -- Context
    agent_id UUID REFERENCES agents(id),
    workflow_id UUID REFERENCES workflows(id),
    execution_id UUID REFERENCES workflow_executions(id),
    task_id UUID REFERENCES tasks(id),
    user_id UUID REFERENCES users(id),
    
    -- Additional data
    metadata JSONB DEFAULT '{}',
    stack_trace TEXT,
    
    -- Timing
    logged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT system_logs_partition_key CHECK (logged_at >= '2025-01-01'::date)
) PARTITION BY RANGE (logged_at);
```

### Indexing Strategy

#### Primary Indexes
```sql
-- User management indexes
CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_username ON users(username) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_status ON users(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token_hash ON user_sessions(token_hash);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions(expires_at);

-- Agent management indexes
CREATE INDEX idx_agents_type_status ON agents(type, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_agents_status ON agents(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_agents_capabilities ON agents USING GIN(capabilities);
CREATE INDEX idx_agents_created_at ON agents(created_at);
CREATE INDEX idx_agent_dependencies_agent_id ON agent_dependencies(agent_id);
CREATE INDEX idx_agent_dependencies_depends_on ON agent_dependencies(depends_on_agent_id);

-- Workflow management indexes
CREATE INDEX idx_workflows_status ON workflows(status);
CREATE INDEX idx_workflows_next_run_at ON workflows(next_run_at) WHERE next_run_at IS NOT NULL;
CREATE INDEX idx_workflows_tags ON workflows USING GIN(tags);
CREATE INDEX idx_workflow_executions_workflow_id ON workflow_executions(workflow_id);
CREATE INDEX idx_workflow_executions_status ON workflow_executions(status);
CREATE INDEX idx_workflow_executions_queued_at ON workflow_executions(queued_at);

-- Task management indexes
CREATE INDEX idx_tasks_status_priority ON tasks(status, priority);
CREATE INDEX idx_tasks_assigned_agent_id ON tasks(assigned_agent_id);
CREATE INDEX idx_tasks_workflow_execution_id ON tasks(workflow_execution_id);
CREATE INDEX idx_tasks_parent_task_id ON tasks(parent_task_id);
CREATE INDEX idx_tasks_scheduled_at ON tasks(scheduled_at) WHERE scheduled_at IS NOT NULL;

-- Audit and logging indexes
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_occurred_at ON audit_logs(occurred_at);
CREATE INDEX idx_system_logs_level_component ON system_logs(level, component);
CREATE INDEX idx_system_logs_logged_at ON system_logs(logged_at);
```

#### Composite and Specialized Indexes
```sql
-- Performance optimization indexes
CREATE INDEX idx_agents_active_with_capacity ON agents(id, max_concurrent_tasks) 
    WHERE status = 'active' AND deleted_at IS NULL;

CREATE INDEX idx_tasks_ready_for_execution ON tasks(priority DESC, created_at ASC) 
    WHERE status = 'pending' AND (scheduled_at IS NULL OR scheduled_at <= NOW());

CREATE INDEX idx_workflow_executions_active ON workflow_executions(workflow_id, started_at) 
    WHERE status IN ('queued', 'running');

-- Full-text search indexes
CREATE INDEX idx_agents_search ON agents USING GIN(
    to_tsvector('english', name || ' ' || COALESCE(description, ''))
);

CREATE INDEX idx_workflows_search ON workflows USING GIN(
    to_tsvector('english', name || ' ' || COALESCE(description, ''))
);

-- JSON field indexes
CREATE INDEX idx_agents_capabilities_specific ON agents USING GIN(
    (capabilities -> 'specific_capabilities')
);

CREATE INDEX idx_workflow_definition_triggers ON workflows USING GIN(
    (definition -> 'triggers')
);
```

### Database Optimization Strategies

#### Query Performance Optimization
```sql
-- Materialized views for common aggregations
CREATE MATERIALIZED VIEW agent_performance_summary AS
SELECT 
    a.id,
    a.name,
    a.type,
    a.status,
    COUNT(t.id) as total_tasks,
    COUNT(CASE WHEN t.status = 'completed' THEN 1 END) as completed_tasks,
    COUNT(CASE WHEN t.status = 'failed' THEN 1 END) as failed_tasks,
    AVG(EXTRACT(EPOCH FROM (t.completed_at - t.started_at))) as avg_execution_time,
    MAX(t.completed_at) as last_task_completion
FROM agents a
LEFT JOIN tasks t ON a.id = t.assigned_agent_id
WHERE a.deleted_at IS NULL
GROUP BY a.id, a.name, a.type, a.status;

-- Refresh strategy
CREATE OR REPLACE FUNCTION refresh_agent_performance_summary()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY agent_performance_summary;
END;
$$ LANGUAGE plpgsql;

-- Scheduled refresh (to be called by cron job)
SELECT cron.schedule('refresh-agent-performance', '*/15 * * * *', 'SELECT refresh_agent_performance_summary();');
```

#### Partitioning Strategy
```sql
-- Automatic partition creation function
CREATE OR REPLACE FUNCTION create_monthly_partitions(table_name text, start_date date, end_date date)
RETURNS void AS $$
DECLARE
    partition_date date;
    partition_name text;
    next_partition_date date;
BEGIN
    partition_date := date_trunc('month', start_date);
    
    WHILE partition_date < end_date LOOP
        next_partition_date := partition_date + interval '1 month';
        partition_name := table_name || '_' || to_char(partition_date, 'YYYY_MM');
        
        EXECUTE format('CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
                       FOR VALUES FROM (%L) TO (%L)',
                       partition_name, table_name, partition_date, next_partition_date);
        
        partition_date := next_partition_date;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Create partitions for the next 12 months
SELECT create_monthly_partitions('agent_metrics', '2025-01-01'::date, '2026-01-01'::date);
SELECT create_monthly_partitions('workflow_executions', '2025-01-01'::date, '2026-01-01'::date);
SELECT create_monthly_partitions('audit_logs', '2025-01-01'::date, '2026-01-01'::date);
SELECT create_monthly_partitions('system_logs', '2025-01-01'::date, '2026-01-01'::date);
```

### Data Integrity and Constraints

#### Referential Integrity
```sql
-- Cascade delete policies
ALTER TABLE user_roles DROP CONSTRAINT IF EXISTS user_roles_user_id_fkey;
ALTER TABLE user_roles ADD CONSTRAINT user_roles_user_id_fkey 
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE user_sessions DROP CONSTRAINT IF EXISTS user_sessions_user_id_fkey;
ALTER TABLE user_sessions ADD CONSTRAINT user_sessions_user_id_fkey 
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

-- Soft delete constraints
ALTER TABLE agents ADD CONSTRAINT agents_soft_delete_consistency 
    CHECK ((deleted_at IS NULL AND status != 'deleted') OR (deleted_at IS NOT NULL));

-- Business logic constraints
ALTER TABLE workflow_executions ADD CONSTRAINT execution_duration_consistency
    CHECK (
        (status NOT IN ('completed', 'failed', 'cancelled') AND duration_ms IS NULL) OR
        (status IN ('completed', 'failed', 'cancelled') AND duration_ms IS NOT NULL)
    );
```

#### Data Validation Functions
```sql
-- JSON schema validation function
CREATE OR REPLACE FUNCTION validate_json_schema(data jsonb, schema jsonb)
RETURNS boolean AS $$
BEGIN
    -- Simplified schema validation (in production, use a proper JSON schema validator)
    RETURN true; -- Placeholder for actual validation logic
END;
$$ LANGUAGE plpgsql;

-- Workflow definition validation
ALTER TABLE workflows ADD CONSTRAINT workflows_valid_definition
    CHECK (validate_json_schema(definition, '{"type": "object"}'::jsonb));
```

### Security Implementation

#### Row-Level Security (RLS)
```sql
-- Enable RLS on sensitive tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE agents ENABLE ROW LEVEL SECURITY;
ALTER TABLE workflows ENABLE ROW LEVEL SECURITY;

-- User access policies
CREATE POLICY users_own_data ON users
    FOR ALL TO authenticated_users
    USING (id = current_user_id());

CREATE POLICY users_admin_access ON users
    FOR ALL TO admin_users
    USING (true);

-- Agent access policies
CREATE POLICY agents_creator_access ON agents
    FOR ALL TO authenticated_users
    USING (created_by = current_user_id());

CREATE POLICY agents_operator_access ON agents
    FOR SELECT TO agent_operators
    USING (true);

-- Helper functions for RLS
CREATE OR REPLACE FUNCTION current_user_id()
RETURNS uuid AS $$
BEGIN
    RETURN current_setting('app.current_user_id', true)::uuid;
EXCEPTION
    WHEN OTHERS THEN
        RETURN NULL;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

#### Data Encryption
```sql
-- Sensitive data encryption
CREATE OR REPLACE FUNCTION encrypt_sensitive_data(data text)
RETURNS text AS $$
BEGIN
    RETURN encode(pgp_sym_encrypt(data, current_setting('app.encryption_key')), 'base64');
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE FUNCTION decrypt_sensitive_data(encrypted_data text)
RETURNS text AS $$
BEGIN
    RETURN pgp_sym_decrypt(decode(encrypted_data, 'base64'), current_setting('app.encryption_key'));
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Apply encryption to sensitive columns
ALTER TABLE users ADD COLUMN encrypted_notes text;
UPDATE users SET encrypted_notes = encrypt_sensitive_data(COALESCE(notes, '')) WHERE notes IS NOT NULL;
```

### Database Maintenance and Monitoring

#### Automated Maintenance Tasks
```sql
-- Cleanup old sessions
CREATE OR REPLACE FUNCTION cleanup_expired_sessions()
RETURNS void AS $$
BEGIN
    DELETE FROM user_sessions 
    WHERE expires_at < NOW() - interval '7 days';
    
    GET DIAGNOSTICS rows_affected = ROW_COUNT;
    INSERT INTO system_logs (level, message, component, metadata)
    VALUES ('info', 'Cleaned up expired sessions', 'database_maintenance', 
            jsonb_build_object('rows_deleted', rows_affected));
END;
$$ LANGUAGE plpgsql;

-- Archive old audit logs
CREATE OR REPLACE FUNCTION archive_old_audit_logs()
RETURNS void AS $$
BEGIN
    -- Move logs older than 1 year to archive table
    INSERT INTO audit_logs_archive 
    SELECT * FROM audit_logs 
    WHERE occurred_at < NOW() - interval '1 year';
    
    DELETE FROM audit_logs 
    WHERE occurred_at < NOW() - interval '1 year';
END;
$$ LANGUAGE plpgsql;

-- Schedule maintenance tasks
SELECT cron.schedule('cleanup-sessions', '0 2 * * *', 'SELECT cleanup_expired_sessions();');
SELECT cron.schedule('archive-audit-logs', '0 3 1 * *', 'SELECT archive_old_audit_logs();');
```

#### Performance Monitoring
```sql
-- Query performance monitoring view
CREATE VIEW slow_queries AS
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
WHERE mean_time > 100  -- Queries taking more than 100ms on average
ORDER BY mean_time DESC;

-- Table size monitoring
CREATE VIEW table_sizes AS
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
    pg_total_relation_size(schemaname||'.'||tablename) as size_bytes
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY size_bytes DESC;
```

---

**Document Status**: Active  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-27  
**Owner**: @system-architect-agent 