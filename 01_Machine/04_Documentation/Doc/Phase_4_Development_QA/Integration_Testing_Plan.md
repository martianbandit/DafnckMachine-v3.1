# Integration Testing Plan - DafnckMachine v3.1

## Overview
Comprehensive integration testing plan for the DafnckMachine v3.1 project, covering API testing, database integration, third-party service integration, and inter-service communication testing.

## Integration Testing Strategy

### Testing Scope

#### 1. API Integration Testing
- **REST API Endpoints**: All CRUD operations and business logic endpoints
- **Authentication & Authorization**: JWT token validation, role-based access
- **Request/Response Validation**: Schema validation, error handling
- **Rate Limiting**: API throttling and quota management
- **CORS Configuration**: Cross-origin request handling

#### 2. Database Integration Testing
- **Data Persistence**: Create, read, update, delete operations
- **Transaction Management**: ACID compliance and rollback scenarios
- **Constraint Validation**: Foreign keys, unique constraints, data types
- **Migration Testing**: Schema changes and data migration scripts
- **Performance Testing**: Query optimization and indexing

#### 3. Third-Party Service Integration
- **Payment Gateways**: Stripe, PayPal integration testing
- **Email Services**: SendGrid, AWS SES integration
- **Cloud Storage**: AWS S3, Google Cloud Storage
- **Authentication Providers**: OAuth, SAML, social login
- **Analytics Services**: Google Analytics, Mixpanel

#### 4. Inter-Service Communication
- **Microservice APIs**: Service-to-service communication
- **Message Queues**: Redis, RabbitMQ message handling
- **Event Streaming**: Real-time data synchronization
- **Service Discovery**: Load balancing and failover

## Test Environment Setup

### 1. Test Database Configuration

#### PostgreSQL Test Setup
```sql
-- Create test database
CREATE DATABASE dafnckmachine_integration_test;

-- Create test user with limited permissions
CREATE USER test_user WITH PASSWORD 'test_password';
GRANT CONNECT ON DATABASE dafnckmachine_integration_test TO test_user;
GRANT USAGE ON SCHEMA public TO test_user;
GRANT CREATE ON SCHEMA public TO test_user;

-- Grant table permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO test_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO test_user;
```

#### Database Test Utilities
```typescript
// src/test/database/testDb.ts
import { Pool } from 'pg';
import { migrate } from 'postgres-migrations';

export class TestDatabase {
  private pool: Pool;

  constructor() {
    this.pool = new Pool({
      host: process.env.TEST_DB_HOST || 'localhost',
      port: parseInt(process.env.TEST_DB_PORT || '5432'),
      database: process.env.TEST_DB_NAME || 'dafnckmachine_integration_test',
      user: process.env.TEST_DB_USER || 'test_user',
      password: process.env.TEST_DB_PASSWORD || 'test_password'
    });
  }

  async setup() {
    // Run migrations
    await migrate({
      client: this.pool,
      migrationsDirectory: './migrations',
      migrationTableName: 'migrations'
    });
  }

  async cleanup() {
    // Truncate all tables
    const tables = await this.getTables();
    for (const table of tables) {
      await this.pool.query(`TRUNCATE TABLE ${table} CASCADE`);
    }
  }

  async teardown() {
    await this.pool.end();
  }

  private async getTables(): Promise<string[]> {
    const result = await this.pool.query(`
      SELECT tablename 
      FROM pg_tables 
      WHERE schemaname = 'public' 
      AND tablename != 'migrations'
    `);
    return result.rows.map(row => row.tablename);
  }

  getPool() {
    return this.pool;
  }
}

// Global test database instance
export const testDb = new TestDatabase();
```

### 2. API Test Server Setup

#### Express Test Server
```typescript
// src/test/server/testServer.ts
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { setupRoutes } from '../../routes';
import { setupMiddleware } from '../../middleware';
import { testDb } from '../database/testDb';

export class TestServer {
  private app: express.Application;
  private server: any;
  private io: Server;
  private port: number;

  constructor(port = 0) {
    this.app = express();
    this.port = port;
    this.setupServer();
  }

  private setupServer() {
    // Setup middleware
    setupMiddleware(this.app);
    
    // Setup routes
    setupRoutes(this.app);
    
    // Create HTTP server
    this.server = createServer(this.app);
    
    // Setup Socket.IO
    this.io = new Server(this.server, {
      cors: {
        origin: "*",
        methods: ["GET", "POST"]
      }
    });
  }

  async start(): Promise<number> {
    await testDb.setup();
    
    return new Promise((resolve) => {
      this.server.listen(this.port, () => {
        const actualPort = this.server.address()?.port;
        console.log(`Test server running on port ${actualPort}`);
        resolve(actualPort);
      });
    });
  }

  async stop() {
    await testDb.cleanup();
    
    return new Promise<void>((resolve) => {
      this.server.close(() => {
        console.log('Test server stopped');
        resolve();
      });
    });
  }

  getApp() {
    return this.app;
  }

  getServer() {
    return this.server;
  }

  getIO() {
    return this.io;
  }
}
```

## API Integration Tests

### 1. Authentication API Tests

```typescript
// src/test/integration/auth.test.ts
import request from 'supertest';
import { TestServer } from '../server/testServer';
import { testDb } from '../database/testDb';

describe('Authentication API Integration', () => {
  let testServer: TestServer;
  let app: any;

  beforeAll(async () => {
    testServer = new TestServer();
    await testServer.start();
    app = testServer.getApp();
  });

  afterAll(async () => {
    await testServer.stop();
    await testDb.teardown();
  });

  beforeEach(async () => {
    await testDb.cleanup();
  });

  describe('POST /api/auth/register', () => {
    it('should register a new user successfully', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body).toMatchObject({
        user: {
          email: userData.email,
          firstName: userData.firstName,
          lastName: userData.lastName
        },
        token: expect.any(String)
      });

      expect(response.body.user.password).toBeUndefined();
    });

    it('should reject duplicate email registration', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };

      // First registration
      await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      // Duplicate registration
      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(409);

      expect(response.body.error).toContain('email already exists');
    });

    it('should validate password strength', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'weak',
        firstName: 'John',
        lastName: 'Doe'
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(400);

      expect(response.body.errors).toContain('Password must be at least 8 characters');
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      // Create test user
      await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'SecurePass123!',
          firstName: 'John',
          lastName: 'Doe'
        });
    });

    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'SecurePass123!'
        })
        .expect(200);

      expect(response.body).toMatchObject({
        user: {
          email: 'test@example.com',
          firstName: 'John',
          lastName: 'Doe'
        },
        token: expect.any(String)
      });
    });

    it('should reject invalid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
        .expect(401);

      expect(response.body.error).toContain('Invalid credentials');
    });

    it('should implement rate limiting', async () => {
      const loginData = {
        email: 'test@example.com',
        password: 'wrongpassword'
      };

      // Make multiple failed login attempts
      for (let i = 0; i < 5; i++) {
        await request(app)
          .post('/api/auth/login')
          .send(loginData)
          .expect(401);
      }

      // Next attempt should be rate limited
      const response = await request(app)
        .post('/api/auth/login')
        .send(loginData)
        .expect(429);

      expect(response.body.error).toContain('Too many login attempts');
    });
  });

  describe('Protected Routes', () => {
    let authToken: string;

    beforeEach(async () => {
      // Register and login to get token
      await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'SecurePass123!',
          firstName: 'John',
          lastName: 'Doe'
        });

      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'SecurePass123!'
        });

      authToken = loginResponse.body.token;
    });

    it('should access protected route with valid token', async () => {
      const response = await request(app)
        .get('/api/user/profile')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.user.email).toBe('test@example.com');
    });

    it('should reject access without token', async () => {
      const response = await request(app)
        .get('/api/user/profile')
        .expect(401);

      expect(response.body.error).toContain('No token provided');
    });

    it('should reject access with invalid token', async () => {
      const response = await request(app)
        .get('/api/user/profile')
        .set('Authorization', 'Bearer invalid-token')
        .expect(401);

      expect(response.body.error).toContain('Invalid token');
    });
  });
});
```

### 2. CRUD Operations Tests

```typescript
// src/test/integration/projects.test.ts
import request from 'supertest';
import { TestServer } from '../server/testServer';
import { testDb } from '../database/testDb';

describe('Projects API Integration', () => {
  let testServer: TestServer;
  let app: any;
  let authToken: string;
  let userId: string;

  beforeAll(async () => {
    testServer = new TestServer();
    await testServer.start();
    app = testServer.getApp();
  });

  afterAll(async () => {
    await testServer.stop();
    await testDb.teardown();
  });

  beforeEach(async () => {
    await testDb.cleanup();
    
    // Create and authenticate user
    const registerResponse = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      });

    authToken = registerResponse.body.token;
    userId = registerResponse.body.user.id;
  });

  describe('POST /api/projects', () => {
    it('should create a new project', async () => {
      const projectData = {
        name: 'Test Project',
        description: 'A test project for integration testing',
        status: 'active'
      };

      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(projectData)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        name: projectData.name,
        description: projectData.description,
        status: projectData.status,
        ownerId: userId,
        createdAt: expect.any(String),
        updatedAt: expect.any(String)
      });
    });

    it('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({})
        .expect(400);

      expect(response.body.errors).toContain('Name is required');
    });
  });

  describe('GET /api/projects', () => {
    beforeEach(async () => {
      // Create test projects
      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Project 1',
          description: 'First project',
          status: 'active'
        });

      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Project 2',
          description: 'Second project',
          status: 'completed'
        });
    });

    it('should return user projects', async () => {
      const response = await request(app)
        .get('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.projects).toHaveLength(2);
      expect(response.body.projects[0]).toMatchObject({
        name: 'Project 1',
        ownerId: userId
      });
    });

    it('should support pagination', async () => {
      const response = await request(app)
        .get('/api/projects?page=1&limit=1')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.projects).toHaveLength(1);
      expect(response.body.pagination).toMatchObject({
        page: 1,
        limit: 1,
        total: 2,
        pages: 2
      });
    });

    it('should support filtering by status', async () => {
      const response = await request(app)
        .get('/api/projects?status=active')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.projects).toHaveLength(1);
      expect(response.body.projects[0].status).toBe('active');
    });
  });

  describe('PUT /api/projects/:id', () => {
    let projectId: string;

    beforeEach(async () => {
      const createResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Test Project',
          description: 'Original description',
          status: 'active'
        });

      projectId = createResponse.body.id;
    });

    it('should update project successfully', async () => {
      const updateData = {
        name: 'Updated Project',
        description: 'Updated description',
        status: 'completed'
      };

      const response = await request(app)
        .put(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(updateData)
        .expect(200);

      expect(response.body).toMatchObject(updateData);
    });

    it('should prevent unauthorized updates', async () => {
      // Create another user
      const otherUserResponse = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'other@example.com',
          password: 'SecurePass123!',
          firstName: 'Jane',
          lastName: 'Smith'
        });

      const response = await request(app)
        .put(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${otherUserResponse.body.token}`)
        .send({ name: 'Hacked Project' })
        .expect(403);

      expect(response.body.error).toContain('Access denied');
    });
  });

  describe('DELETE /api/projects/:id', () => {
    let projectId: string;

    beforeEach(async () => {
      const createResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Test Project',
          description: 'To be deleted',
          status: 'active'
        });

      projectId = createResponse.body.id;
    });

    it('should delete project successfully', async () => {
      await request(app)
        .delete(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(204);

      // Verify project is deleted
      await request(app)
        .get(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });

    it('should handle cascade deletion of related data', async () => {
      // Create tasks for the project
      await request(app)
        .post(`/api/projects/${projectId}/tasks`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Test Task',
          description: 'Task to be deleted with project'
        });

      // Delete project
      await request(app)
        .delete(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(204);

      // Verify tasks are also deleted
      const tasksResponse = await request(app)
        .get(`/api/projects/${projectId}/tasks`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
  });
});
```

## Database Integration Tests

### 1. Transaction Testing

```typescript
// src/test/integration/transactions.test.ts
import { testDb } from '../database/testDb';
import { UserService } from '../../services/UserService';
import { ProjectService } from '../../services/ProjectService';

describe('Database Transaction Integration', () => {
  let userService: UserService;
  let projectService: ProjectService;

  beforeAll(async () => {
    await testDb.setup();
    userService = new UserService(testDb.getPool());
    projectService = new ProjectService(testDb.getPool());
  });

  afterAll(async () => {
    await testDb.teardown();
  });

  beforeEach(async () => {
    await testDb.cleanup();
  });

  describe('Transaction Rollback', () => {
    it('should rollback transaction on error', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };

      // Mock a service that creates user and project in transaction
      const createUserWithProject = async () => {
        const client = await testDb.getPool().connect();
        
        try {
          await client.query('BEGIN');
          
          // Create user
          const user = await userService.create(userData, client);
          
          // Create project (this will fail due to invalid data)
          await projectService.create({
            name: '', // Invalid: empty name
            ownerId: user.id
          }, client);
          
          await client.query('COMMIT');
          return user;
        } catch (error) {
          await client.query('ROLLBACK');
          throw error;
        } finally {
          client.release();
        }
      };

      // Expect the transaction to fail
      await expect(createUserWithProject()).rejects.toThrow();

      // Verify no user was created (transaction rolled back)
      const users = await userService.findByEmail(userData.email);
      expect(users).toBeNull();
    });

    it('should commit transaction on success', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };

      const createUserWithProject = async () => {
        const client = await testDb.getPool().connect();
        
        try {
          await client.query('BEGIN');
          
          const user = await userService.create(userData, client);
          const project = await projectService.create({
            name: 'Test Project',
            description: 'Valid project',
            ownerId: user.id
          }, client);
          
          await client.query('COMMIT');
          return { user, project };
        } catch (error) {
          await client.query('ROLLBACK');
          throw error;
        } finally {
          client.release();
        }
      };

      const result = await createUserWithProject();

      // Verify both user and project were created
      expect(result.user.email).toBe(userData.email);
      expect(result.project.name).toBe('Test Project');

      // Verify data persisted in database
      const persistedUser = await userService.findByEmail(userData.email);
      expect(persistedUser).not.toBeNull();
    });
  });

  describe('Concurrent Access', () => {
    it('should handle concurrent updates correctly', async () => {
      // Create initial user
      const user = await userService.create({
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      });

      // Simulate concurrent updates
      const update1 = userService.update(user.id, { firstName: 'Jane' });
      const update2 = userService.update(user.id, { lastName: 'Smith' });

      await Promise.all([update1, update2]);

      // Verify final state
      const updatedUser = await userService.findById(user.id);
      expect(updatedUser.firstName).toBe('Jane');
      expect(updatedUser.lastName).toBe('Smith');
    });
  });
});
```

## Third-Party Service Integration Tests

### 1. Email Service Integration

```typescript
// src/test/integration/email.test.ts
import { EmailService } from '../../services/EmailService';
import { TestServer } from '../server/testServer';

describe('Email Service Integration', () => {
  let emailService: EmailService;
  let testServer: TestServer;

  beforeAll(async () => {
    testServer = new TestServer();
    await testServer.start();
    
    // Use test email service configuration
    emailService = new EmailService({
      provider: 'sendgrid',
      apiKey: process.env.SENDGRID_TEST_API_KEY,
      testMode: true
    });
  });

  afterAll(async () => {
    await testServer.stop();
  });

  describe('Send Email', () => {
    it('should send welcome email successfully', async () => {
      const emailData = {
        to: 'test@example.com',
        subject: 'Welcome to DafnckMachine',
        template: 'welcome',
        data: {
          firstName: 'John',
          activationLink: 'https://app.dafnckmachine.com/activate/123'
        }
      };

      const result = await emailService.send(emailData);

      expect(result).toMatchObject({
        messageId: expect.any(String),
        status: 'sent',
        provider: 'sendgrid'
      });
    });

    it('should handle email sending failures gracefully', async () => {
      const emailData = {
        to: 'invalid-email', // Invalid email format
        subject: 'Test Email',
        template: 'welcome',
        data: {}
      };

      await expect(emailService.send(emailData)).rejects.toThrow('Invalid email address');
    });

    it('should retry failed email sends', async () => {
      // Mock temporary service failure
      const originalSend = emailService.send;
      let attempts = 0;
      
      emailService.send = jest.fn().mockImplementation(async (data) => {
        attempts++;
        if (attempts < 3) {
          throw new Error('Service temporarily unavailable');
        }
        return originalSend.call(emailService, data);
      });

      const emailData = {
        to: 'test@example.com',
        subject: 'Retry Test',
        template: 'welcome',
        data: { firstName: 'John' }
      };

      const result = await emailService.send(emailData);

      expect(attempts).toBe(3);
      expect(result.status).toBe('sent');
    });
  });

  describe('Email Templates', () => {
    it('should render email templates correctly', async () => {
      const templateData = {
        firstName: 'John',
        projectName: 'Test Project',
        inviteLink: 'https://app.dafnckmachine.com/invite/abc123'
      };

      const rendered = await emailService.renderTemplate('project-invite', templateData);

      expect(rendered.html).toContain('John');
      expect(rendered.html).toContain('Test Project');
      expect(rendered.html).toContain('https://app.dafnckmachine.com/invite/abc123');
      expect(rendered.text).toContain('John');
    });

    it('should handle missing template data gracefully', async () => {
      const templateData = {
        firstName: 'John'
        // Missing required projectName and inviteLink
      };

      await expect(
        emailService.renderTemplate('project-invite', templateData)
      ).rejects.toThrow('Missing required template data');
    });
  });
});
```

### 2. Payment Integration Tests

```typescript
// src/test/integration/payments.test.ts
import { PaymentService } from '../../services/PaymentService';
import { TestServer } from '../server/testServer';
import request from 'supertest';

describe('Payment Service Integration', () => {
  let paymentService: PaymentService;
  let testServer: TestServer;
  let app: any;
  let authToken: string;

  beforeAll(async () => {
    testServer = new TestServer();
    await testServer.start();
    app = testServer.getApp();
    
    paymentService = new PaymentService({
      provider: 'stripe',
      apiKey: process.env.STRIPE_TEST_SECRET_KEY,
      testMode: true
    });

    // Create and authenticate test user
    const registerResponse = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'test@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      });

    authToken = registerResponse.body.token;
  });

  afterAll(async () => {
    await testServer.stop();
  });

  describe('Payment Processing', () => {
    it('should process successful payment', async () => {
      const paymentData = {
        amount: 2999, // $29.99
        currency: 'usd',
        paymentMethodId: 'pm_card_visa', // Stripe test payment method
        description: 'DafnckMachine Pro Subscription'
      };

      const response = await request(app)
        .post('/api/payments/process')
        .set('Authorization', `Bearer ${authToken}`)
        .send(paymentData)
        .expect(200);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        amount: paymentData.amount,
        currency: paymentData.currency,
        status: 'succeeded',
        paymentMethod: expect.any(Object)
      });
    });

    it('should handle declined payments', async () => {
      const paymentData = {
        amount: 2999,
        currency: 'usd',
        paymentMethodId: 'pm_card_chargeDeclined', // Stripe test declined card
        description: 'DafnckMachine Pro Subscription'
      };

      const response = await request(app)
        .post('/api/payments/process')
        .set('Authorization', `Bearer ${authToken}`)
        .send(paymentData)
        .expect(402);

      expect(response.body.error).toContain('payment_declined');
    });

    it('should handle insufficient funds', async () => {
      const paymentData = {
        amount: 2999,
        currency: 'usd',
        paymentMethodId: 'pm_card_chargeDeclinedInsufficientFunds',
        description: 'DafnckMachine Pro Subscription'
      };

      const response = await request(app)
        .post('/api/payments/process')
        .set('Authorization', `Bearer ${authToken}`)
        .send(paymentData)
        .expect(402);

      expect(response.body.error).toContain('insufficient_funds');
    });
  });

  describe('Subscription Management', () => {
    it('should create subscription successfully', async () => {
      const subscriptionData = {
        priceId: 'price_test_pro_monthly',
        paymentMethodId: 'pm_card_visa'
      };

      const response = await request(app)
        .post('/api/subscriptions')
        .set('Authorization', `Bearer ${authToken}`)
        .send(subscriptionData)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        status: 'active',
        currentPeriodStart: expect.any(String),
        currentPeriodEnd: expect.any(String),
        plan: expect.objectContaining({
          id: subscriptionData.priceId
        })
      });
    });

    it('should cancel subscription', async () => {
      // First create a subscription
      const createResponse = await request(app)
        .post('/api/subscriptions')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          priceId: 'price_test_pro_monthly',
          paymentMethodId: 'pm_card_visa'
        });

      const subscriptionId = createResponse.body.id;

      // Then cancel it
      const response = await request(app)
        .delete(`/api/subscriptions/${subscriptionId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.status).toBe('canceled');
    });
  });

  describe('Webhook Handling', () => {
    it('should handle payment success webhook', async () => {
      const webhookPayload = {
        id: 'evt_test_webhook',
        object: 'event',
        type: 'payment_intent.succeeded',
        data: {
          object: {
            id: 'pi_test_payment',
            amount: 2999,
            currency: 'usd',
            status: 'succeeded'
          }
        }
      };

      const response = await request(app)
        .post('/api/webhooks/stripe')
        .send(webhookPayload)
        .expect(200);

      expect(response.body.received).toBe(true);
    });

    it('should handle subscription update webhook', async () => {
      const webhookPayload = {
        id: 'evt_test_webhook',
        object: 'event',
        type: 'customer.subscription.updated',
        data: {
          object: {
            id: 'sub_test_subscription',
            status: 'active',
            current_period_start: Math.floor(Date.now() / 1000),
            current_period_end: Math.floor(Date.now() / 1000) + 2592000 // 30 days
          }
        }
      };

      const response = await request(app)
        .post('/api/webhooks/stripe')
        .send(webhookPayload)
        .expect(200);

      expect(response.body.received).toBe(true);
    });
  });
});
```

## Real-time Communication Tests

### 1. WebSocket Integration Tests

```typescript
// src/test/integration/websocket.test.ts
import { io as Client, Socket } from 'socket.io-client';
import { TestServer } from '../server/testServer';

describe('WebSocket Integration', () => {
  let testServer: TestServer;
  let serverPort: number;
  let clientSocket: Socket;

  beforeAll(async () => {
    testServer = new TestServer();
    serverPort = await testServer.start();
  });

  afterAll(async () => {
    await testServer.stop();
  });

  beforeEach((done) => {
    clientSocket = Client(`http://localhost:${serverPort}`);
    clientSocket.on('connect', done);
  });

  afterEach(() => {
    if (clientSocket.connected) {
      clientSocket.disconnect();
    }
  });

  describe('Connection Management', () => {
    it('should establish connection successfully', () => {
      expect(clientSocket.connected).toBe(true);
    });

    it('should authenticate user on connection', (done) => {
      const authToken = 'valid-jwt-token';
      
      clientSocket.emit('authenticate', { token: authToken });
      
      clientSocket.on('authenticated', (data) => {
        expect(data.success).toBe(true);
        expect(data.user).toBeDefined();
        done();
      });
    });

    it('should reject invalid authentication', (done) => {
      const invalidToken = 'invalid-token';
      
      clientSocket.emit('authenticate', { token: invalidToken });
      
      clientSocket.on('authentication_error', (error) => {
        expect(error.message).toContain('Invalid token');
        done();
      });
    });
  });

  describe('Real-time Updates', () => {
    it('should broadcast project updates to team members', (done) => {
      const projectId = 'project-123';
      const updateData = {
        id: projectId,
        name: 'Updated Project Name',
        updatedAt: new Date().toISOString()
      };

      // Join project room
      clientSocket.emit('join_project', { projectId });
      
      // Listen for project updates
      clientSocket.on('project_updated', (data) => {
        expect(data).toMatchObject(updateData);
        done();
      });

      // Simulate project update from server
      setTimeout(() => {
        testServer.getIO().to(`project_${projectId}`).emit('project_updated', updateData);
      }, 100);
    });

    it('should handle task status changes in real-time', (done) => {
      const taskId = 'task-456';
      const statusUpdate = {
        id: taskId,
        status: 'completed',
        completedAt: new Date().toISOString()
      };

      clientSocket.on('task_status_changed', (data) => {
        expect(data).toMatchObject(statusUpdate);
        done();
      });

      // Simulate task status change
      setTimeout(() => {
        testServer.getIO().emit('task_status_changed', statusUpdate);
      }, 100);
    });
  });

  describe('Collaboration Features', () => {
    it('should handle multiple users in same project', (done) => {
      const projectId = 'project-789';
      let connectedUsers = 0;

      // Create second client
      const secondClient = Client(`http://localhost:${serverPort}`);
      
      const checkCompletion = () => {
        connectedUsers++;
        if (connectedUsers === 2) {
          secondClient.disconnect();
          done();
        }
      };

      clientSocket.emit('join_project', { projectId });
      clientSocket.on('user_joined_project', checkCompletion);

      secondClient.on('connect', () => {
        secondClient.emit('join_project', { projectId });
        secondClient.on('user_joined_project', checkCompletion);
      });
    });

    it('should handle cursor tracking for collaborative editing', (done) => {
      const documentId = 'doc-123';
      const cursorData = {
        userId: 'user-456',
        position: { line: 10, column: 25 },
        selection: { start: { line: 10, column: 20 }, end: { line: 10, column: 30 } }
      };

      clientSocket.emit('join_document', { documentId });
      
      clientSocket.on('cursor_moved', (data) => {
        expect(data).toMatchObject(cursorData);
        done();
      });

      // Simulate cursor movement
      setTimeout(() => {
        clientSocket.emit('cursor_move', cursorData);
      }, 100);
    });
  });
});
```

## Performance Integration Tests

### 1. Load Testing

```typescript
// src/test/integration/performance.test.ts
import request from 'supertest';
import { TestServer } from '../server/testServer';

describe('Performance Integration Tests', () => {
  let testServer: TestServer;
  let app: any;

  beforeAll(async () => {
    testServer = new TestServer();
    await testServer.start();
    app = testServer.getApp();
  });

  afterAll(async () => {
    await testServer.stop();
  });

  describe('API Performance', () => {
    it('should handle concurrent requests efficiently', async () => {
      const concurrentRequests = 50;
      const startTime = Date.now();

      const requests = Array.from({ length: concurrentRequests }, () =>
        request(app)
          .get('/api/health')
          .expect(200)
      );

      await Promise.all(requests);

      const endTime = Date.now();
      const totalTime = endTime - startTime;
      const averageTime = totalTime / concurrentRequests;

      expect(averageTime).toBeLessThan(100); // Average response time under 100ms
    });

    it('should maintain performance under load', async () => {
      const requestCount = 100;
      const batchSize = 10;
      const results: number[] = [];

      for (let i = 0; i < requestCount; i += batchSize) {
        const batch = Array.from({ length: batchSize }, async () => {
          const startTime = Date.now();
          await request(app).get('/api/health').expect(200);
          return Date.now() - startTime;
        });

        const batchResults = await Promise.all(batch);
        results.push(...batchResults);
      }

      const averageResponseTime = results.reduce((a, b) => a + b, 0) / results.length;
      const p95ResponseTime = results.sort((a, b) => a - b)[Math.floor(results.length * 0.95)];

      expect(averageResponseTime).toBeLessThan(50);
      expect(p95ResponseTime).toBeLessThan(200);
    });
  });

  describe('Database Performance', () => {
    it('should handle bulk operations efficiently', async () => {
      // Create auth token
      const authResponse = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'SecurePass123!',
          firstName: 'John',
          lastName: 'Doe'
        });

      const authToken = authResponse.body.token;

      // Create project
      const projectResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Bulk Test Project',
          description: 'Testing bulk operations'
        });

      const projectId = projectResponse.body.id;

      // Bulk create tasks
      const taskCount = 100;
      const startTime = Date.now();

      const taskPromises = Array.from({ length: taskCount }, (_, i) =>
        request(app)
          .post(`/api/projects/${projectId}/tasks`)
          .set('Authorization', `Bearer ${authToken}`)
          .send({
            title: `Task ${i + 1}`,
            description: `Description for task ${i + 1}`
          })
      );

      await Promise.all(taskPromises);

      const endTime = Date.now();
      const totalTime = endTime - startTime;
      const averageTime = totalTime / taskCount;

      expect(averageTime).toBeLessThan(50); // Average creation time under 50ms
    });
  });
});
```

## Test Execution and Reporting

### 1. Test Configuration

```javascript
// jest.integration.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/test/integration/**/*.test.ts'],
  setupFilesAfterEnv: ['<rootDir>/src/test/integration/setup.ts'],
  testTimeout: 30000,
  maxWorkers: 1, // Run integration tests sequentially
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/test/**/*',
    '!src/**/*.d.ts'
  ],
  coverageDirectory: 'coverage/integration',
  coverageReporters: ['text', 'lcov', 'html']
};
```

### 2. CI/CD Integration

```yaml
# .github/workflows/integration-tests.yml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: dafnckmachine_integration_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run database migrations
        run: npm run migrate:test
        env:
          TEST_DB_HOST: localhost
          TEST_DB_PORT: 5432
          TEST_DB_NAME: dafnckmachine_integration_test
          TEST_DB_USER: postgres
          TEST_DB_PASSWORD: postgres
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          TEST_DB_HOST: localhost
          TEST_DB_PORT: 5432
          TEST_DB_NAME: dafnckmachine_integration_test
          TEST_DB_USER: postgres
          TEST_DB_PASSWORD: postgres
          REDIS_URL: redis://localhost:6379
          SENDGRID_TEST_API_KEY: ${{ secrets.SENDGRID_TEST_API_KEY }}
          STRIPE_TEST_SECRET_KEY: ${{ secrets.STRIPE_TEST_SECRET_KEY }}
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: integration-test-results
          path: |
            coverage/integration/
            test-results/
```

## Related Documentation

- [Test Strategy Framework](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Test_Strategy_Framework.md)
- [Unit Testing Guidelines](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Unit_Testing_Guidelines.md)
- [E2E Testing Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/E2E_Testing_Configuration.md)
- [API Documentation Complete](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: qa-team, development-team
- **Related Workflows**: 15_Automated_Testing.md 