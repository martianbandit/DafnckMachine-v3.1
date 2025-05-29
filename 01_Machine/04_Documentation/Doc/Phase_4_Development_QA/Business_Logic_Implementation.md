# Business Logic Implementation - DafnckMachine v3.1

## Overview
Comprehensive guide for implementing business logic in DafnckMachine v3.1, covering domain modeling, service layer architecture, business rules implementation, and workflow orchestration to ensure robust and maintainable business operations.

## Business Logic Architecture

### Domain-Driven Design (DDD) Approach
- **Domain Layer**: Core business entities and rules
- **Application Layer**: Use cases and application services
- **Infrastructure Layer**: External dependencies and data persistence
- **Interface Layer**: API controllers and external interfaces

### Service Layer Pattern
```javascript
// services/ProjectService.js
class ProjectService {
  constructor(projectRepository, userRepository, taskService, eventBus) {
    this.projectRepository = projectRepository;
    this.userRepository = userRepository;
    this.taskService = taskService;
    this.eventBus = eventBus;
  }

  async createProject(userId, projectData) {
    // Business rule validation
    await this.validateProjectCreation(userId, projectData);
    
    // Create project entity
    const project = new Project({
      ...projectData,
      userId,
      status: 'active',
      createdAt: new Date()
    });

    // Apply business rules
    project.applyBusinessRules();
    
    // Persist to database
    const savedProject = await this.projectRepository.create(project);
    
    // Emit domain event
    await this.eventBus.emit('project.created', {
      projectId: savedProject.id,
      userId,
      timestamp: new Date()
    });

    return savedProject;
  }

  async validateProjectCreation(userId, projectData) {
    // Check user exists and is active
    const user = await this.userRepository.findById(userId);
    if (!user || user.status !== 'active') {
      throw new BusinessError('User not found or inactive');
    }

    // Check project limits
    const userProjectCount = await this.projectRepository.countByUserId(userId);
    const maxProjects = this.getMaxProjectsForUser(user);
    
    if (userProjectCount >= maxProjects) {
      throw new BusinessError(`Maximum project limit (${maxProjects}) reached`);
    }

    // Validate project name uniqueness for user
    const existingProject = await this.projectRepository.findByUserIdAndName(
      userId, 
      projectData.name
    );
    
    if (existingProject) {
      throw new BusinessError('Project name already exists');
    }
  }

  getMaxProjectsForUser(user) {
    const limits = {
      'free': 3,
      'pro': 25,
      'enterprise': 100
    };
    return limits[user.plan] || limits.free;
  }
}
```

## Domain Entities

### Core Business Entities
```javascript
// domain/entities/Project.js
class Project {
  constructor(data) {
    this.id = data.id;
    this.userId = data.userId;
    this.name = data.name;
    this.description = data.description;
    this.status = data.status || 'active';
    this.settings = data.settings || {};
    this.createdAt = data.createdAt;
    this.updatedAt = data.updatedAt;
    this.tasks = data.tasks || [];
  }

  applyBusinessRules() {
    this.validateName();
    this.validateStatus();
    this.applyDefaultSettings();
  }

  validateName() {
    if (!this.name || this.name.trim().length < 3) {
      throw new BusinessError('Project name must be at least 3 characters');
    }
    
    if (this.name.length > 100) {
      throw new BusinessError('Project name cannot exceed 100 characters');
    }
  }

  validateStatus() {
    const validStatuses = ['active', 'archived', 'deleted'];
    if (!validStatuses.includes(this.status)) {
      throw new BusinessError(`Invalid project status: ${this.status}`);
    }
  }

  applyDefaultSettings() {
    this.settings = {
      notifications: true,
      autoArchive: false,
      taskAutoAssignment: false,
      ...this.settings
    };
  }

  canBeDeleted() {
    return this.status !== 'deleted' && this.tasks.length === 0;
  }

  archive() {
    if (this.status === 'deleted') {
      throw new BusinessError('Cannot archive deleted project');
    }
    this.status = 'archived';
    this.updatedAt = new Date();
  }

  activate() {
    if (this.status === 'deleted') {
      throw new BusinessError('Cannot activate deleted project');
    }
    this.status = 'active';
    this.updatedAt = new Date();
  }
}

// domain/entities/Task.js
class Task {
  constructor(data) {
    this.id = data.id;
    this.projectId = data.projectId;
    this.title = data.title;
    this.description = data.description;
    this.status = data.status || 'pending';
    this.priority = data.priority || 'medium';
    this.assignedTo = data.assignedTo;
    this.dueDate = data.dueDate;
    this.estimatedHours = data.estimatedHours;
    this.actualHours = data.actualHours || 0;
    this.tags = data.tags || [];
    this.createdAt = data.createdAt;
    this.updatedAt = data.updatedAt;
  }

  applyBusinessRules() {
    this.validateTitle();
    this.validateStatus();
    this.validatePriority();
    this.validateDueDate();
  }

  validateTitle() {
    if (!this.title || this.title.trim().length < 3) {
      throw new BusinessError('Task title must be at least 3 characters');
    }
  }

  validateStatus() {
    const validStatuses = ['pending', 'in-progress', 'completed', 'cancelled'];
    if (!validStatuses.includes(this.status)) {
      throw new BusinessError(`Invalid task status: ${this.status}`);
    }
  }

  validatePriority() {
    const validPriorities = ['low', 'medium', 'high', 'urgent'];
    if (!validPriorities.includes(this.priority)) {
      throw new BusinessError(`Invalid task priority: ${this.priority}`);
    }
  }

  validateDueDate() {
    if (this.dueDate && new Date(this.dueDate) < new Date()) {
      throw new BusinessError('Due date cannot be in the past');
    }
  }

  canBeCompleted() {
    return this.status === 'in-progress';
  }

  complete() {
    if (!this.canBeCompleted()) {
      throw new BusinessError('Task must be in progress to be completed');
    }
    this.status = 'completed';
    this.updatedAt = new Date();
  }

  calculateProgress() {
    if (this.estimatedHours && this.actualHours) {
      return Math.min((this.actualHours / this.estimatedHours) * 100, 100);
    }
    return this.status === 'completed' ? 100 : 0;
  }
}
```

## Business Rules Engine

### Rule Definition and Execution
```javascript
// domain/rules/BusinessRuleEngine.js
class BusinessRuleEngine {
  constructor() {
    this.rules = new Map();
    this.registerDefaultRules();
  }

  registerRule(name, rule) {
    this.rules.set(name, rule);
  }

  async executeRules(context, entityType, operation) {
    const applicableRules = this.getApplicableRules(entityType, operation);
    
    for (const rule of applicableRules) {
      const result = await rule.execute(context);
      if (!result.isValid) {
        throw new BusinessRuleViolationError(result.message, result.code);
      }
    }
  }

  getApplicableRules(entityType, operation) {
    return Array.from(this.rules.values()).filter(rule => 
      rule.appliesTo(entityType, operation)
    );
  }

  registerDefaultRules() {
    // Project creation rules
    this.registerRule('project-name-uniqueness', new ProjectNameUniquenessRule());
    this.registerRule('project-limit-check', new ProjectLimitRule());
    this.registerRule('user-active-check', new UserActiveRule());
    
    // Task assignment rules
    this.registerRule('task-assignment-validation', new TaskAssignmentRule());
    this.registerRule('task-due-date-validation', new TaskDueDateRule());
    
    // User permission rules
    this.registerRule('user-permission-check', new UserPermissionRule());
  }
}

// domain/rules/ProjectRules.js
class ProjectNameUniquenessRule {
  appliesTo(entityType, operation) {
    return entityType === 'Project' && ['create', 'update'].includes(operation);
  }

  async execute(context) {
    const { entity, repository } = context;
    const existingProject = await repository.findByUserIdAndName(
      entity.userId, 
      entity.name
    );

    if (existingProject && existingProject.id !== entity.id) {
      return {
        isValid: false,
        message: 'Project name must be unique for user',
        code: 'PROJECT_NAME_NOT_UNIQUE'
      };
    }

    return { isValid: true };
  }
}

class ProjectLimitRule {
  appliesTo(entityType, operation) {
    return entityType === 'Project' && operation === 'create';
  }

  async execute(context) {
    const { entity, userRepository, projectRepository } = context;
    
    const user = await userRepository.findById(entity.userId);
    const projectCount = await projectRepository.countByUserId(entity.userId);
    const maxProjects = this.getMaxProjectsForPlan(user.plan);

    if (projectCount >= maxProjects) {
      return {
        isValid: false,
        message: `Maximum project limit (${maxProjects}) reached for ${user.plan} plan`,
        code: 'PROJECT_LIMIT_EXCEEDED'
      };
    }

    return { isValid: true };
  }

  getMaxProjectsForPlan(plan) {
    const limits = { 'free': 3, 'pro': 25, 'enterprise': 100 };
    return limits[plan] || limits.free;
  }
}
```

## Use Cases Implementation

### Application Services
```javascript
// application/usecases/CreateProjectUseCase.js
class CreateProjectUseCase {
  constructor(
    projectRepository,
    userRepository,
    businessRuleEngine,
    eventBus,
    logger
  ) {
    this.projectRepository = projectRepository;
    this.userRepository = userRepository;
    this.businessRuleEngine = businessRuleEngine;
    this.eventBus = eventBus;
    this.logger = logger;
  }

  async execute(command) {
    try {
      // Validate command
      this.validateCommand(command);

      // Create project entity
      const project = new Project({
        userId: command.userId,
        name: command.name,
        description: command.description,
        settings: command.settings
      });

      // Apply business rules
      const context = {
        entity: project,
        repository: this.projectRepository,
        userRepository: this.userRepository
      };

      await this.businessRuleEngine.executeRules(context, 'Project', 'create');

      // Apply domain rules
      project.applyBusinessRules();

      // Persist project
      const savedProject = await this.projectRepository.create(project);

      // Emit domain event
      await this.eventBus.emit('project.created', {
        projectId: savedProject.id,
        userId: command.userId,
        name: savedProject.name,
        timestamp: new Date()
      });

      // Log success
      this.logger.info('Project created successfully', {
        projectId: savedProject.id,
        userId: command.userId
      });

      return savedProject;

    } catch (error) {
      this.logger.error('Failed to create project', {
        userId: command.userId,
        error: error.message
      });
      throw error;
    }
  }

  validateCommand(command) {
    if (!command.userId) {
      throw new ValidationError('User ID is required');
    }
    if (!command.name) {
      throw new ValidationError('Project name is required');
    }
  }
}

// application/usecases/AssignTaskUseCase.js
class AssignTaskUseCase {
  constructor(
    taskRepository,
    userRepository,
    projectRepository,
    notificationService,
    eventBus
  ) {
    this.taskRepository = taskRepository;
    this.userRepository = userRepository;
    this.projectRepository = projectRepository;
    this.notificationService = notificationService;
    this.eventBus = eventBus;
  }

  async execute(command) {
    // Validate assignment
    await this.validateAssignment(command);

    // Get task and update assignment
    const task = await this.taskRepository.findById(command.taskId);
    if (!task) {
      throw new NotFoundError('Task not found');
    }

    const previousAssignee = task.assignedTo;
    task.assignedTo = command.assigneeId;
    task.updatedAt = new Date();

    // Save updated task
    const updatedTask = await this.taskRepository.update(task);

    // Send notification to new assignee
    if (command.assigneeId) {
      await this.notificationService.sendTaskAssignmentNotification(
        command.assigneeId,
        updatedTask
      );
    }

    // Emit domain event
    await this.eventBus.emit('task.assigned', {
      taskId: updatedTask.id,
      projectId: updatedTask.projectId,
      assigneeId: command.assigneeId,
      previousAssigneeId: previousAssignee,
      timestamp: new Date()
    });

    return updatedTask;
  }

  async validateAssignment(command) {
    // Check if assignee exists and has access to project
    if (command.assigneeId) {
      const assignee = await this.userRepository.findById(command.assigneeId);
      if (!assignee) {
        throw new ValidationError('Assignee not found');
      }

      const task = await this.taskRepository.findById(command.taskId);
      const project = await this.projectRepository.findById(task.projectId);
      
      // Check if assignee has access to project
      const hasAccess = await this.checkProjectAccess(command.assigneeId, project.id);
      if (!hasAccess) {
        throw new ValidationError('Assignee does not have access to this project');
      }
    }
  }

  async checkProjectAccess(userId, projectId) {
    // Implementation depends on your access control model
    const project = await this.projectRepository.findById(projectId);
    return project.userId === userId || 
           await this.projectRepository.hasCollaboratorAccess(userId, projectId);
  }
}
```

## Event-Driven Architecture

### Domain Events
```javascript
// domain/events/DomainEvent.js
class DomainEvent {
  constructor(eventType, aggregateId, data, timestamp = new Date()) {
    this.eventType = eventType;
    this.aggregateId = aggregateId;
    this.data = data;
    this.timestamp = timestamp;
    this.id = this.generateEventId();
  }

  generateEventId() {
    return `${this.eventType}_${this.aggregateId}_${Date.now()}`;
  }
}

// infrastructure/events/EventBus.js
class EventBus {
  constructor() {
    this.handlers = new Map();
    this.eventStore = new EventStore();
  }

  subscribe(eventType, handler) {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType).push(handler);
  }

  async emit(eventType, data) {
    const event = new DomainEvent(eventType, data.aggregateId || data.id, data);
    
    // Store event
    await this.eventStore.store(event);

    // Notify handlers
    const handlers = this.handlers.get(eventType) || [];
    await Promise.all(handlers.map(handler => handler.handle(event)));
  }
}

// Event handlers
class ProjectCreatedHandler {
  constructor(analyticsService, emailService) {
    this.analyticsService = analyticsService;
    this.emailService = emailService;
  }

  async handle(event) {
    // Track analytics
    await this.analyticsService.track('project_created', {
      userId: event.data.userId,
      projectId: event.data.projectId,
      timestamp: event.timestamp
    });

    // Send welcome email
    await this.emailService.sendProjectCreatedEmail(
      event.data.userId,
      event.data.name
    );
  }
}
```

## Workflow Orchestration

### Saga Pattern Implementation
```javascript
// application/sagas/ProjectSetupSaga.js
class ProjectSetupSaga {
  constructor(
    projectService,
    taskService,
    notificationService,
    templateService
  ) {
    this.projectService = projectService;
    this.taskService = taskService;
    this.notificationService = notificationService;
    this.templateService = templateService;
  }

  async execute(command) {
    const sagaId = this.generateSagaId();
    
    try {
      // Step 1: Create project
      const project = await this.projectService.createProject(
        command.userId,
        command.projectData
      );

      // Step 2: Apply project template if specified
      if (command.templateId) {
        await this.templateService.applyTemplate(
          project.id,
          command.templateId
        );
      }

      // Step 3: Create initial tasks
      if (command.initialTasks) {
        await this.createInitialTasks(project.id, command.initialTasks);
      }

      // Step 4: Setup project notifications
      await this.notificationService.setupProjectNotifications(
        project.id,
        command.userId
      );

      // Step 5: Send completion notification
      await this.notificationService.sendProjectSetupCompleteNotification(
        command.userId,
        project
      );

      return { success: true, projectId: project.id };

    } catch (error) {
      // Compensating actions
      await this.compensate(sagaId, error);
      throw error;
    }
  }

  async createInitialTasks(projectId, tasks) {
    for (const taskData of tasks) {
      await this.taskService.createTask(projectId, taskData);
    }
  }

  async compensate(sagaId, error) {
    // Implement compensating actions
    // This could involve rolling back created resources
    console.error(`Saga ${sagaId} failed:`, error);
  }

  generateSagaId() {
    return `saga_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

## Error Handling

### Business Exception Hierarchy
```javascript
// domain/errors/BusinessError.js
class BusinessError extends Error {
  constructor(message, code = 'BUSINESS_ERROR', details = {}) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
    this.details = details;
    this.timestamp = new Date();
  }
}

class BusinessRuleViolationError extends BusinessError {
  constructor(message, code = 'BUSINESS_RULE_VIOLATION') {
    super(message, code);
    this.name = 'BusinessRuleViolationError';
  }
}

class ValidationError extends BusinessError {
  constructor(message, field = null) {
    super(message, 'VALIDATION_ERROR', { field });
    this.name = 'ValidationError';
  }
}

class NotFoundError extends BusinessError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 'NOT_FOUND');
    this.name = 'NotFoundError';
  }
}

class UnauthorizedError extends BusinessError {
  constructor(message = 'Unauthorized access') {
    super(message, 'UNAUTHORIZED');
    this.name = 'UnauthorizedError';
  }
}

module.exports = {
  BusinessError,
  BusinessRuleViolationError,
  ValidationError,
  NotFoundError,
  UnauthorizedError
};
```

## Testing Strategy

### Unit Testing Business Logic
```javascript
// tests/domain/entities/Project.test.js
describe('Project Entity', () => {
  describe('applyBusinessRules', () => {
    it('should validate project name length', () => {
      const project = new Project({ name: 'ab' });
      
      expect(() => project.applyBusinessRules())
        .toThrow('Project name must be at least 3 characters');
    });

    it('should apply default settings', () => {
      const project = new Project({ name: 'Test Project' });
      project.applyBusinessRules();
      
      expect(project.settings.notifications).toBe(true);
      expect(project.settings.autoArchive).toBe(false);
    });
  });

  describe('canBeDeleted', () => {
    it('should return false if project has tasks', () => {
      const project = new Project({
        name: 'Test Project',
        tasks: [{ id: 1 }]
      });
      
      expect(project.canBeDeleted()).toBe(false);
    });
  });
});

// tests/application/usecases/CreateProjectUseCase.test.js
describe('CreateProjectUseCase', () => {
  let useCase;
  let mockProjectRepository;
  let mockUserRepository;
  let mockBusinessRuleEngine;
  let mockEventBus;

  beforeEach(() => {
    mockProjectRepository = {
      create: jest.fn(),
      findByUserIdAndName: jest.fn()
    };
    mockUserRepository = {
      findById: jest.fn()
    };
    mockBusinessRuleEngine = {
      executeRules: jest.fn()
    };
    mockEventBus = {
      emit: jest.fn()
    };

    useCase = new CreateProjectUseCase(
      mockProjectRepository,
      mockUserRepository,
      mockBusinessRuleEngine,
      mockEventBus,
      console
    );
  });

  it('should create project successfully', async () => {
    const command = {
      userId: 'user123',
      name: 'Test Project',
      description: 'Test Description'
    };

    const expectedProject = { id: 'proj123', ...command };
    mockProjectRepository.create.mockResolvedValue(expectedProject);

    const result = await useCase.execute(command);

    expect(mockBusinessRuleEngine.executeRules).toHaveBeenCalled();
    expect(mockProjectRepository.create).toHaveBeenCalled();
    expect(mockEventBus.emit).toHaveBeenCalledWith('project.created', 
      expect.objectContaining({
        projectId: expectedProject.id,
        userId: command.userId
      })
    );
    expect(result).toEqual(expectedProject);
  });
});
```

## Performance Considerations

### Optimization Strategies
- **Lazy Loading**: Load related entities only when needed
- **Caching**: Cache frequently accessed business rules and configurations
- **Batch Processing**: Process multiple operations in batches
- **Async Processing**: Use event-driven architecture for non-critical operations
- **Database Optimization**: Optimize queries and use appropriate indexes

### Monitoring and Metrics
```javascript
// infrastructure/monitoring/BusinessMetrics.js
class BusinessMetrics {
  constructor(metricsCollector) {
    this.metrics = metricsCollector;
  }

  trackProjectCreation(userId, projectId, duration) {
    this.metrics.increment('projects.created');
    this.metrics.histogram('projects.creation_duration', duration);
    this.metrics.increment(`projects.created.by_plan.${this.getUserPlan(userId)}`);
  }

  trackTaskCompletion(projectId, taskId, duration) {
    this.metrics.increment('tasks.completed');
    this.metrics.histogram('tasks.completion_duration', duration);
  }

  trackBusinessRuleViolation(ruleType, entityType) {
    this.metrics.increment('business_rules.violations', {
      rule_type: ruleType,
      entity_type: entityType
    });
  }
}
```

This comprehensive business logic implementation provides a solid foundation for DafnckMachine v3.1's backend operations, ensuring maintainable, testable, and scalable business processes.

---

**Last Updated**: 2025-01-27  
**Version**: 3.1.0  
**Related Documents**: 
- [API_Documentation_Complete.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)
- [Database_Integration_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Database_Integration_Guide.md)
- [Authentication_Security_Framework.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Authentication_Security_Framework.md) 