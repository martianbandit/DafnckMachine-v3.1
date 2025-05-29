# Backend Testing Strategy - DafnckMachine v3.1

## Overview
Comprehensive testing strategy for DafnckMachine v3.1 backend services, covering unit testing, integration testing, API testing, performance testing, and security testing to ensure robust, reliable, and secure backend operations.

## Testing Pyramid

### Testing Levels
1. **Unit Tests (70%)**: Individual functions, classes, and modules
2. **Integration Tests (20%)**: Component interactions and database operations
3. **End-to-End Tests (10%)**: Complete user workflows and API endpoints
4. **Performance Tests**: Load, stress, and scalability testing
5. **Security Tests**: Vulnerability scanning and penetration testing

## Unit Testing

### Testing Framework Setup
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  testMatch: [
    '<rootDir>/tests/unit/**/*.test.js',
    '<rootDir>/tests/integration/**/*.test.js'
  ],
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
    '!src/config/**',
    '!src/migrations/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testTimeout: 10000
};
```

### Unit Test Examples
```javascript
// tests/unit/services/ProjectService.test.js
const ProjectService = require('../../../src/services/ProjectService');
const { BusinessError } = require('../../../src/domain/errors');

describe('ProjectService', () => {
  let projectService;
  let mockProjectRepository;
  let mockUserRepository;
  let mockEventBus;

  beforeEach(() => {
    mockProjectRepository = {
      create: jest.fn(),
      findByUserIdAndName: jest.fn(),
      countByUserId: jest.fn()
    };
    mockUserRepository = {
      findById: jest.fn()
    };
    mockEventBus = {
      emit: jest.fn()
    };

    projectService = new ProjectService(
      mockProjectRepository,
      mockUserRepository,
      null,
      mockEventBus
    );
  });

  describe('createProject', () => {
    it('should create project successfully', async () => {
      // Arrange
      const userId = 'user123';
      const projectData = {
        name: 'Test Project',
        description: 'Test Description'
      };

      const user = { id: userId, status: 'active', plan: 'pro' };
      const expectedProject = { id: 'proj123', ...projectData, userId };

      mockUserRepository.findById.mockResolvedValue(user);
      mockProjectRepository.countByUserId.mockResolvedValue(5);
      mockProjectRepository.findByUserIdAndName.mockResolvedValue(null);
      mockProjectRepository.create.mockResolvedValue(expectedProject);

      // Act
      const result = await projectService.createProject(userId, projectData);

      // Assert
      expect(mockUserRepository.findById).toHaveBeenCalledWith(userId);
      expect(mockProjectRepository.countByUserId).toHaveBeenCalledWith(userId);
      expect(mockProjectRepository.create).toHaveBeenCalled();
      expect(mockEventBus.emit).toHaveBeenCalledWith('project.created', 
        expect.objectContaining({
          projectId: expectedProject.id,
          userId
        })
      );
      expect(result).toEqual(expectedProject);
    });

    it('should throw error when user not found', async () => {
      // Arrange
      const userId = 'nonexistent';
      const projectData = { name: 'Test Project' };

      mockUserRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(projectService.createProject(userId, projectData))
        .rejects.toThrow(BusinessError);
      
      expect(mockProjectRepository.create).not.toHaveBeenCalled();
    });

    it('should throw error when project limit exceeded', async () => {
      // Arrange
      const userId = 'user123';
      const projectData = { name: 'Test Project' };
      const user = { id: userId, status: 'active', plan: 'free' };

      mockUserRepository.findById.mockResolvedValue(user);
      mockProjectRepository.countByUserId.mockResolvedValue(3); // Free plan limit

      // Act & Assert
      await expect(projectService.createProject(userId, projectData))
        .rejects.toThrow('Maximum project limit (3) reached');
    });

    it('should throw error when project name already exists', async () => {
      // Arrange
      const userId = 'user123';
      const projectData = { name: 'Existing Project' };
      const user = { id: userId, status: 'active', plan: 'pro' };
      const existingProject = { id: 'existing123', name: 'Existing Project' };

      mockUserRepository.findById.mockResolvedValue(user);
      mockProjectRepository.countByUserId.mockResolvedValue(5);
      mockProjectRepository.findByUserIdAndName.mockResolvedValue(existingProject);

      // Act & Assert
      await expect(projectService.createProject(userId, projectData))
        .rejects.toThrow('Project name already exists');
    });
  });

  describe('getMaxProjectsForUser', () => {
    it('should return correct limits for different plans', () => {
      expect(projectService.getMaxProjectsForUser({ plan: 'free' })).toBe(3);
      expect(projectService.getMaxProjectsForUser({ plan: 'pro' })).toBe(25);
      expect(projectService.getMaxProjectsForUser({ plan: 'enterprise' })).toBe(100);
      expect(projectService.getMaxProjectsForUser({ plan: 'unknown' })).toBe(3);
    });
  });
});

// tests/unit/domain/entities/Project.test.js
const Project = require('../../../src/domain/entities/Project');
const { BusinessError } = require('../../../src/domain/errors');

describe('Project Entity', () => {
  describe('constructor', () => {
    it('should create project with default values', () => {
      const projectData = {
        id: 'proj123',
        userId: 'user123',
        name: 'Test Project'
      };

      const project = new Project(projectData);

      expect(project.id).toBe('proj123');
      expect(project.userId).toBe('user123');
      expect(project.name).toBe('Test Project');
      expect(project.status).toBe('active');
      expect(project.settings).toEqual({});
      expect(project.tasks).toEqual([]);
    });
  });

  describe('applyBusinessRules', () => {
    it('should validate name length', () => {
      const project = new Project({ name: 'ab' });
      
      expect(() => project.applyBusinessRules())
        .toThrow('Project name must be at least 3 characters');
    });

    it('should validate maximum name length', () => {
      const longName = 'a'.repeat(101);
      const project = new Project({ name: longName });
      
      expect(() => project.applyBusinessRules())
        .toThrow('Project name cannot exceed 100 characters');
    });

    it('should validate status', () => {
      const project = new Project({ name: 'Valid Name', status: 'invalid' });
      
      expect(() => project.applyBusinessRules())
        .toThrow('Invalid project status: invalid');
    });

    it('should apply default settings', () => {
      const project = new Project({ name: 'Test Project' });
      project.applyBusinessRules();
      
      expect(project.settings).toEqual({
        notifications: true,
        autoArchive: false,
        taskAutoAssignment: false
      });
    });

    it('should preserve existing settings', () => {
      const project = new Project({
        name: 'Test Project',
        settings: { notifications: false, customSetting: true }
      });
      project.applyBusinessRules();
      
      expect(project.settings).toEqual({
        notifications: false,
        autoArchive: false,
        taskAutoAssignment: false,
        customSetting: true
      });
    });
  });

  describe('canBeDeleted', () => {
    it('should return true when project has no tasks', () => {
      const project = new Project({ name: 'Test Project', tasks: [] });
      expect(project.canBeDeleted()).toBe(true);
    });

    it('should return false when project has tasks', () => {
      const project = new Project({
        name: 'Test Project',
        tasks: [{ id: 'task1' }]
      });
      expect(project.canBeDeleted()).toBe(false);
    });

    it('should return false when project is deleted', () => {
      const project = new Project({
        name: 'Test Project',
        status: 'deleted',
        tasks: []
      });
      expect(project.canBeDeleted()).toBe(false);
    });
  });

  describe('archive', () => {
    it('should archive active project', () => {
      const project = new Project({ name: 'Test Project', status: 'active' });
      project.archive();
      
      expect(project.status).toBe('archived');
      expect(project.updatedAt).toBeInstanceOf(Date);
    });

    it('should throw error when archiving deleted project', () => {
      const project = new Project({ name: 'Test Project', status: 'deleted' });
      
      expect(() => project.archive())
        .toThrow('Cannot archive deleted project');
    });
  });
});
```

## Integration Testing

### Database Integration Tests
```javascript
// tests/integration/repositories/ProjectRepository.test.js
const ProjectRepository = require('../../../src/repositories/ProjectRepository');
const { setupTestDatabase, cleanupTestDatabase } = require('../../helpers/database');

describe('ProjectRepository Integration', () => {
  let projectRepository;

  beforeAll(async () => {
    await setupTestDatabase();
    projectRepository = new ProjectRepository();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  beforeEach(async () => {
    await projectRepository.deleteAll(); // Clean slate for each test
  });

  describe('create', () => {
    it('should create project in database', async () => {
      // Arrange
      const projectData = {
        userId: 'user123',
        name: 'Integration Test Project',
        description: 'Test Description',
        status: 'active'
      };

      // Act
      const createdProject = await projectRepository.create(projectData);

      // Assert
      expect(createdProject.id).toBeDefined();
      expect(createdProject.name).toBe(projectData.name);
      expect(createdProject.userId).toBe(projectData.userId);
      expect(createdProject.createdAt).toBeInstanceOf(Date);
      expect(createdProject.updatedAt).toBeInstanceOf(Date);

      // Verify in database
      const foundProject = await projectRepository.findById(createdProject.id);
      expect(foundProject).toEqual(createdProject);
    });

    it('should enforce unique constraint on user_id and name', async () => {
      // Arrange
      const projectData = {
        userId: 'user123',
        name: 'Duplicate Name Project'
      };

      await projectRepository.create(projectData);

      // Act & Assert
      await expect(projectRepository.create(projectData))
        .rejects.toThrow(); // Database constraint violation
    });
  });

  describe('findByUserIdAndName', () => {
    it('should find project by user ID and name', async () => {
      // Arrange
      const projectData = {
        userId: 'user123',
        name: 'Findable Project'
      };
      const createdProject = await projectRepository.create(projectData);

      // Act
      const foundProject = await projectRepository.findByUserIdAndName(
        'user123',
        'Findable Project'
      );

      // Assert
      expect(foundProject).toEqual(createdProject);
    });

    it('should return null when project not found', async () => {
      // Act
      const foundProject = await projectRepository.findByUserIdAndName(
        'nonexistent',
        'Nonexistent Project'
      );

      // Assert
      expect(foundProject).toBeNull();
    });
  });

  describe('countByUserId', () => {
    it('should count projects for user', async () => {
      // Arrange
      const userId = 'user123';
      await projectRepository.create({ userId, name: 'Project 1' });
      await projectRepository.create({ userId, name: 'Project 2' });
      await projectRepository.create({ userId: 'other', name: 'Other Project' });

      // Act
      const count = await projectRepository.countByUserId(userId);

      // Assert
      expect(count).toBe(2);
    });
  });
});
```

### API Integration Tests
```javascript
// tests/integration/api/projects.test.js
const request = require('supertest');
const app = require('../../../src/app');
const { setupTestDatabase, cleanupTestDatabase } = require('../../helpers/database');
const { createTestUser, generateAuthToken } = require('../../helpers/auth');

describe('Projects API Integration', () => {
  let authToken;
  let testUser;

  beforeAll(async () => {
    await setupTestDatabase();
    testUser = await createTestUser();
    authToken = generateAuthToken(testUser);
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  describe('POST /api/projects', () => {
    it('should create project successfully', async () => {
      // Arrange
      const projectData = {
        name: 'API Test Project',
        description: 'Created via API test'
      };

      // Act
      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(projectData)
        .expect(201);

      // Assert
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe(projectData.name);
      expect(response.body.data.description).toBe(projectData.description);
      expect(response.body.data.userId).toBe(testUser.id);
      expect(response.body.data.id).toBeDefined();
    });

    it('should return 400 for invalid project data', async () => {
      // Arrange
      const invalidData = {
        name: 'ab' // Too short
      };

      // Act
      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(invalidData)
        .expect(400);

      // Assert
      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });

    it('should return 401 without authentication', async () => {
      // Arrange
      const projectData = { name: 'Unauthorized Project' };

      // Act
      await request(app)
        .post('/api/projects')
        .send(projectData)
        .expect(401);
    });

    it('should return 409 for duplicate project name', async () => {
      // Arrange
      const projectData = { name: 'Duplicate Project' };

      // Create first project
      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(projectData)
        .expect(201);

      // Act - Try to create duplicate
      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(projectData)
        .expect(409);

      // Assert
      expect(response.body.success).toBe(false);
      expect(response.body.error.message).toContain('already exists');
    });
  });

  describe('GET /api/projects', () => {
    it('should return user projects with pagination', async () => {
      // Arrange - Create test projects
      const projectPromises = Array.from({ length: 5 }, (_, i) =>
        request(app)
          .post('/api/projects')
          .set('Authorization', `Bearer ${authToken}`)
          .send({ name: `Test Project ${i + 1}` })
      );
      await Promise.all(projectPromises);

      // Act
      const response = await request(app)
        .get('/api/projects?page=1&limit=3')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      // Assert
      expect(response.body.success).toBe(true);
      expect(response.body.data.projects).toHaveLength(3);
      expect(response.body.data.pagination.page).toBe(1);
      expect(response.body.data.pagination.limit).toBe(3);
      expect(response.body.data.pagination.total).toBe(5);
      expect(response.body.data.pagination.totalPages).toBe(2);
    });

    it('should filter projects by status', async () => {
      // Arrange
      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Active Project', status: 'active' });

      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Archived Project', status: 'archived' });

      // Act
      const response = await request(app)
        .get('/api/projects?status=active')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      // Assert
      expect(response.body.data.projects).toHaveLength(1);
      expect(response.body.data.projects[0].status).toBe('active');
    });
  });

  describe('GET /api/projects/:id', () => {
    it('should return project details', async () => {
      // Arrange
      const createResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Detail Test Project' });

      const projectId = createResponse.body.data.id;

      // Act
      const response = await request(app)
        .get(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      // Assert
      expect(response.body.success).toBe(true);
      expect(response.body.data.id).toBe(projectId);
      expect(response.body.data.name).toBe('Detail Test Project');
    });

    it('should return 404 for nonexistent project', async () => {
      // Act
      const response = await request(app)
        .get('/api/projects/nonexistent-id')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);

      // Assert
      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('NOT_FOUND');
    });

    it('should return 403 for unauthorized access', async () => {
      // Arrange - Create project with different user
      const otherUser = await createTestUser({ email: 'other@example.com' });
      const otherToken = generateAuthToken(otherUser);

      const createResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${otherToken}`)
        .send({ name: 'Other User Project' });

      const projectId = createResponse.body.data.id;

      // Act - Try to access with different user
      const response = await request(app)
        .get(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(403);

      // Assert
      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('FORBIDDEN');
    });
  });
});
```

## Performance Testing

### Load Testing with Artillery
```yaml
# tests/performance/load-test.yml
config:
  target: 'http://localhost:3000'
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 120
      arrivalRate: 50
      name: "Ramp up load"
    - duration: 300
      arrivalRate: 100
      name: "Sustained load"
  variables:
    authToken: "{{ $randomString() }}"

scenarios:
  - name: "Create and retrieve projects"
    weight: 70
    flow:
      - post:
          url: "/api/auth/login"
          json:
            email: "test@example.com"
            password: "password123"
          capture:
            - json: "$.data.token"
              as: "authToken"
      - post:
          url: "/api/projects"
          headers:
            Authorization: "Bearer {{ authToken }}"
          json:
            name: "Load Test Project {{ $randomString() }}"
            description: "Created during load test"
      - get:
          url: "/api/projects"
          headers:
            Authorization: "Bearer {{ authToken }}"

  - name: "Read-only operations"
    weight: 30
    flow:
      - post:
          url: "/api/auth/login"
          json:
            email: "test@example.com"
            password: "password123"
          capture:
            - json: "$.data.token"
              as: "authToken"
      - get:
          url: "/api/projects"
          headers:
            Authorization: "Bearer {{ authToken }}"
      - get:
          url: "/api/projects/{{ $randomString() }}"
          headers:
            Authorization: "Bearer {{ authToken }}"
```

### Database Performance Tests
```javascript
// tests/performance/database.test.js
const { performance } = require('perf_hooks');
const ProjectRepository = require('../../src/repositories/ProjectRepository');

describe('Database Performance Tests', () => {
  let projectRepository;

  beforeAll(async () => {
    projectRepository = new ProjectRepository();
  });

  describe('Bulk Operations', () => {
    it('should handle bulk project creation efficiently', async () => {
      const projectCount = 1000;
      const projects = Array.from({ length: projectCount }, (_, i) => ({
        userId: `user${i % 10}`, // 10 different users
        name: `Bulk Project ${i}`,
        description: `Description for project ${i}`
      }));

      const startTime = performance.now();
      
      // Use transaction for bulk insert
      await projectRepository.bulkCreate(projects);
      
      const endTime = performance.now();
      const duration = endTime - startTime;

      console.log(`Bulk created ${projectCount} projects in ${duration.toFixed(2)}ms`);
      
      // Should complete within reasonable time (adjust based on your requirements)
      expect(duration).toBeLessThan(5000); // 5 seconds
    });

    it('should handle concurrent reads efficiently', async () => {
      const concurrentReads = 100;
      const userId = 'performance-test-user';

      // Create some test data
      await projectRepository.bulkCreate(
        Array.from({ length: 50 }, (_, i) => ({
          userId,
          name: `Concurrent Test Project ${i}`
        }))
      );

      const startTime = performance.now();
      
      // Execute concurrent reads
      const readPromises = Array.from({ length: concurrentReads }, () =>
        projectRepository.findByUserId(userId)
      );
      
      await Promise.all(readPromises);
      
      const endTime = performance.now();
      const duration = endTime - startTime;

      console.log(`Completed ${concurrentReads} concurrent reads in ${duration.toFixed(2)}ms`);
      
      // Should handle concurrent reads efficiently
      expect(duration).toBeLessThan(2000); // 2 seconds
    });
  });

  describe('Query Performance', () => {
    it('should execute complex queries within acceptable time', async () => {
      const startTime = performance.now();
      
      // Complex query with joins and aggregations
      const result = await projectRepository.getProjectStatistics();
      
      const endTime = performance.now();
      const duration = endTime - startTime;

      console.log(`Complex query executed in ${duration.toFixed(2)}ms`);
      
      expect(duration).toBeLessThan(1000); // 1 second
      expect(result).toBeDefined();
    });
  });
});
```

## Security Testing

### Authentication & Authorization Tests
```javascript
// tests/security/auth.test.js
const request = require('supertest');
const app = require('../../src/app');
const jwt = require('jsonwebtoken');

describe('Security Tests', () => {
  describe('Authentication', () => {
    it('should reject requests without authentication token', async () => {
      await request(app)
        .get('/api/projects')
        .expect(401);
    });

    it('should reject requests with invalid token', async () => {
      await request(app)
        .get('/api/projects')
        .set('Authorization', 'Bearer invalid-token')
        .expect(401);
    });

    it('should reject requests with expired token', async () => {
      const expiredToken = jwt.sign(
        { sub: 'user123', exp: Math.floor(Date.now() / 1000) - 3600 },
        process.env.JWT_ACCESS_SECRET
      );

      await request(app)
        .get('/api/projects')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401);
    });

    it('should reject requests with malformed authorization header', async () => {
      await request(app)
        .get('/api/projects')
        .set('Authorization', 'InvalidFormat token')
        .expect(401);
    });
  });

  describe('Input Validation', () => {
    it('should sanitize SQL injection attempts', async () => {
      const maliciousInput = {
        name: "'; DROP TABLE projects; --",
        description: "Malicious description"
      };

      const authToken = generateValidToken();

      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(maliciousInput)
        .expect(400);

      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });

    it('should reject XSS attempts in input', async () => {
      const xssInput = {
        name: '<script>alert("xss")</script>',
        description: 'Normal description'
      };

      const authToken = generateValidToken();

      const response = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(xssInput)
        .expect(400);

      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });

    it('should enforce input length limits', async () => {
      const oversizedInput = {
        name: 'a'.repeat(1000), // Exceeds limit
        description: 'Normal description'
      };

      const authToken = generateValidToken();

      await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${authToken}`)
        .send(oversizedInput)
        .expect(400);
    });
  });

  describe('Rate Limiting', () => {
    it('should enforce rate limits on authentication endpoints', async () => {
      const loginData = {
        email: 'test@example.com',
        password: 'wrongpassword'
      };

      // Make multiple failed login attempts
      const requests = Array.from({ length: 6 }, () =>
        request(app)
          .post('/api/auth/login')
          .send(loginData)
      );

      const responses = await Promise.all(requests);
      
      // Last request should be rate limited
      expect(responses[5].status).toBe(429);
    });

    it('should enforce general API rate limits', async () => {
      const authToken = generateValidToken();

      // Make many requests quickly
      const requests = Array.from({ length: 101 }, () =>
        request(app)
          .get('/api/projects')
          .set('Authorization', `Bearer ${authToken}`)
      );

      const responses = await Promise.all(requests);
      
      // Some requests should be rate limited
      const rateLimitedResponses = responses.filter(r => r.status === 429);
      expect(rateLimitedResponses.length).toBeGreaterThan(0);
    });
  });

  describe('Authorization', () => {
    it('should prevent access to other users resources', async () => {
      const user1Token = generateValidToken({ userId: 'user1' });
      const user2Token = generateValidToken({ userId: 'user2' });

      // Create project with user1
      const createResponse = await request(app)
        .post('/api/projects')
        .set('Authorization', `Bearer ${user1Token}`)
        .send({ name: 'User1 Project' });

      const projectId = createResponse.body.data.id;

      // Try to access with user2
      await request(app)
        .get(`/api/projects/${projectId}`)
        .set('Authorization', `Bearer ${user2Token}`)
        .expect(403);
    });

    it('should enforce role-based permissions', async () => {
      const userToken = generateValidToken({ role: 'user' });
      const adminToken = generateValidToken({ role: 'admin' });

      // User should not access admin endpoints
      await request(app)
        .get('/api/admin/users')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);

      // Admin should access admin endpoints
      await request(app)
        .get('/api/admin/users')
        .set('Authorization', `Bearer ${adminToken}`)
        .expect(200);
    });
  });
});
```

## Test Data Management

### Test Fixtures and Factories
```javascript
// tests/helpers/factories.js
const { faker } = require('@faker-js/faker');

class UserFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      email: faker.internet.email(),
      firstName: faker.name.firstName(),
      lastName: faker.name.lastName(),
      role: 'user',
      plan: 'free',
      status: 'active',
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }
}

class ProjectFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      userId: faker.datatype.uuid(),
      name: faker.company.name(),
      description: faker.lorem.paragraph(),
      status: 'active',
      settings: {},
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }
}

module.exports = { UserFactory, ProjectFactory };
```

### Database Seeding for Tests
```javascript
// tests/helpers/database.js
const prisma = require('../../src/lib/prisma');
const { UserFactory, ProjectFactory } = require('./factories');

async function setupTestDatabase() {
  // Run migrations
  await prisma.$executeRaw`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`;
  
  // Seed test data
  await seedTestData();
}

async function cleanupTestDatabase() {
  // Clean up in reverse order of dependencies
  await prisma.task.deleteMany();
  await prisma.project.deleteMany();
  await prisma.user.deleteMany();
}

async function seedTestData() {
  // Create test users
  const testUsers = UserFactory.createMany(5);
  await prisma.user.createMany({ data: testUsers });

  // Create test projects
  const testProjects = ProjectFactory.createMany(10, {
    userId: testUsers[0].id
  });
  await prisma.project.createMany({ data: testProjects });
}

async function resetDatabase() {
  await cleanupTestDatabase();
  await seedTestData();
}

module.exports = {
  setupTestDatabase,
  cleanupTestDatabase,
  seedTestData,
  resetDatabase
};
```

## Continuous Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/backend-tests.yml
name: Backend Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: dafnckmachine_test
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
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/dafnckmachine_test

    - name: Run unit tests
      run: npm run test:unit
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/dafnckmachine_test
        REDIS_URL: redis://localhost:6379

    - name: Run integration tests
      run: npm run test:integration
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/dafnckmachine_test
        REDIS_URL: redis://localhost:6379

    - name: Run security tests
      run: npm run test:security
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/dafnckmachine_test

    - name: Generate coverage report
      run: npm run test:coverage
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/dafnckmachine_test

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: backend
        name: backend-coverage

  performance:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Start application
      run: npm start &
      env:
        NODE_ENV: test
        PORT: 3000

    - name: Wait for application
      run: sleep 10

    - name: Run performance tests
      run: npm run test:performance

    - name: Upload performance results
      uses: actions/upload-artifact@v3
      with:
        name: performance-results
        path: tests/performance/results/
```

## Test Reporting and Metrics

### Coverage Reporting
```javascript
// jest.config.js (coverage section)
module.exports = {
  // ... other config
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html', 'json'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/services/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    },
    './src/domain/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95
    }
  }
};
```

This comprehensive backend testing strategy ensures robust, reliable, and secure backend operations for DafnckMachine v3.1 through systematic testing at all levels.

---

**Last Updated**: 2025-01-27  
**Version**: 3.1.0  
**Related Documents**: 
- [API_Documentation_Complete.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)
- [Business_Logic_Implementation.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Business_Logic_Implementation.md)
- [Authentication_Security_Framework.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Authentication_Security_Framework.md) 