# Code Documentation Standards - DafnckMachine v3.1

## Overview
Comprehensive code documentation standards for the DafnckMachine v3.1 project, covering inline comments, API documentation, code examples, and documentation maintenance practices.

## Documentation Philosophy

### Core Principles
1. **Clarity Over Cleverness**: Code should be self-documenting, with comments explaining the "why" not the "what"
2. **Consistency**: Follow established patterns and conventions across the codebase
3. **Maintainability**: Documentation should be easy to update and keep in sync with code
4. **Accessibility**: Documentation should be accessible to developers of varying experience levels
5. **Completeness**: All public APIs and complex logic should be documented

### Documentation Hierarchy
1. **Self-Documenting Code**: Clear naming, structure, and patterns
2. **Inline Comments**: Explain complex logic, business rules, and edge cases
3. **Function/Method Documentation**: JSDoc/TSDoc for all public APIs
4. **Module Documentation**: README files and module-level documentation
5. **Architecture Documentation**: High-level system design and decision records

## TypeScript/JavaScript Documentation Standards

### JSDoc/TSDoc Standards

```typescript
/**
 * Represents a user in the DafnckMachine system.
 * 
 * @example
 * ```typescript
 * const user = new User({
 *   email: 'user@example.com',
 *   firstName: 'John',
 *   lastName: 'Doe'
 * });
 * 
 * await user.save();
 * console.log(user.getFullName()); // "John Doe"
 * ```
 * 
 * @since 1.0.0
 * @author development-team
 */
export class User {
  /**
   * The user's unique identifier.
   * @readonly
   */
  public readonly id: string;

  /**
   * The user's email address. Must be unique across the system.
   * @example "user@example.com"
   */
  public email: string;

  /**
   * The user's first name.
   * @minLength 1
   * @maxLength 50
   */
  public firstName: string;

  /**
   * The user's last name.
   * @minLength 1
   * @maxLength 50
   */
  public lastName: string;

  /**
   * Creates a new User instance.
   * 
   * @param userData - The initial user data
   * @param userData.email - The user's email address
   * @param userData.firstName - The user's first name
   * @param userData.lastName - The user's last name
   * 
   * @throws {ValidationError} When email format is invalid
   * @throws {ValidationError} When required fields are missing
   * 
   * @example
   * ```typescript
   * const user = new User({
   *   email: 'john.doe@example.com',
   *   firstName: 'John',
   *   lastName: 'Doe'
   * });
   * ```
   */
  constructor(userData: CreateUserData) {
    this.validateUserData(userData);
    
    this.id = generateUniqueId();
    this.email = userData.email.toLowerCase().trim();
    this.firstName = userData.firstName.trim();
    this.lastName = userData.lastName.trim();
  }

  /**
   * Returns the user's full name by combining first and last names.
   * 
   * @returns The user's full name in "FirstName LastName" format
   * 
   * @example
   * ```typescript
   * const user = new User({ email: 'john@example.com', firstName: 'John', lastName: 'Doe' });
   * console.log(user.getFullName()); // "John Doe"
   * ```
   */
  public getFullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  /**
   * Updates the user's email address with validation.
   * 
   * @param newEmail - The new email address
   * @returns Promise that resolves when the email is successfully updated
   * 
   * @throws {ValidationError} When the new email format is invalid
   * @throws {ConflictError} When the email is already in use by another user
   * @throws {DatabaseError} When the database operation fails
   * 
   * @example
   * ```typescript
   * try {
   *   await user.updateEmail('newemail@example.com');
   *   console.log('Email updated successfully');
   * } catch (error) {
   *   if (error instanceof ValidationError) {
   *     console.error('Invalid email format');
   *   } else if (error instanceof ConflictError) {
   *     console.error('Email already in use');
   *   }
   * }
   * ```
   */
  public async updateEmail(newEmail: string): Promise<void> {
    // Validate email format
    if (!this.isValidEmail(newEmail)) {
      throw new ValidationError('Invalid email format', 'email', newEmail);
    }

    // Check for email uniqueness
    const existingUser = await UserRepository.findByEmail(newEmail);
    if (existingUser && existingUser.id !== this.id) {
      throw new ConflictError('Email already in use', 'email', newEmail);
    }

    // Update email
    this.email = newEmail.toLowerCase().trim();
    await this.save();
  }

  /**
   * Validates email format using RFC 5322 specification.
   * 
   * @private
   * @param email - The email address to validate
   * @returns True if the email format is valid, false otherwise
   * 
   * @internal This method is for internal use only and may change without notice
   */
  private isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}
```

### Interface and Type Documentation

```typescript
/**
 * Configuration options for creating a new user.
 * 
 * @public
 * @interface
 */
export interface CreateUserData {
  /** 
   * The user's email address. Must be unique and valid.
   * @example "user@example.com"
   */
  email: string;

  /** 
   * The user's first name. Cannot be empty.
   * @minLength 1
   * @maxLength 50
   */
  firstName: string;

  /** 
   * The user's last name. Cannot be empty.
   * @minLength 1
   * @maxLength 50
   */
  lastName: string;

  /** 
   * Optional role assignment. Defaults to 'user' if not specified.
   * @defaultValue 'user'
   */
  role?: UserRole;

  /** 
   * Optional metadata for the user account.
   * @example { department: 'Engineering', startDate: '2024-01-15' }
   */
  metadata?: Record<string, any>;
}

/**
 * Available user roles in the system.
 * 
 * @public
 * @enum
 */
export enum UserRole {
  /** Standard user with basic permissions */
  USER = 'user',
  
  /** Administrator with elevated permissions */
  ADMIN = 'admin',
  
  /** Super administrator with full system access */
  SUPER_ADMIN = 'super_admin',
  
  /** Read-only user for reporting and analytics */
  VIEWER = 'viewer'
}

/**
 * Type definition for user search filters.
 * 
 * @public
 * @typedef
 */
export type UserSearchFilters = {
  /** Filter by email pattern (supports wildcards) */
  email?: string;
  
  /** Filter by user role */
  role?: UserRole;
  
  /** Filter by account status */
  status?: 'active' | 'inactive' | 'suspended';
  
  /** Filter by creation date range */
  createdAfter?: Date;
  
  /** Filter by creation date range */
  createdBefore?: Date;
};
```

### Function Documentation

```typescript
/**
 * Calculates the pagination offset based on page number and page size.
 * 
 * This utility function is commonly used in database queries to implement
 * pagination. It handles edge cases like negative page numbers and ensures
 * the offset is never negative.
 * 
 * @param page - The page number (1-based indexing)
 * @param pageSize - The number of items per page
 * @returns The calculated offset for database queries (0-based)
 * 
 * @throws {ValidationError} When page or pageSize is not a positive integer
 * 
 * @example
 * ```typescript
 * // Get offset for page 3 with 10 items per page
 * const offset = calculatePaginationOffset(3, 10); // Returns 20
 * 
 * // Use in database query
 * const users = await db.user.findMany({
 *   skip: offset,
 *   take: pageSize
 * });
 * ```
 * 
 * @example
 * ```typescript
 * // Handle edge cases
 * calculatePaginationOffset(1, 10);  // Returns 0 (first page)
 * calculatePaginationOffset(0, 10);  // Throws ValidationError
 * calculatePaginationOffset(-1, 10); // Throws ValidationError
 * ```
 * 
 * @since 1.0.0
 * @category Utility
 */
export function calculatePaginationOffset(page: number, pageSize: number): number {
  // Validate inputs
  if (!Number.isInteger(page) || page < 1) {
    throw new ValidationError('Page must be a positive integer', 'page', page);
  }
  
  if (!Number.isInteger(pageSize) || pageSize < 1) {
    throw new ValidationError('Page size must be a positive integer', 'pageSize', pageSize);
  }

  // Calculate offset (convert from 1-based to 0-based indexing)
  return (page - 1) * pageSize;
}

/**
 * Formats a date according to the application's standard format.
 * 
 * This function provides consistent date formatting across the application.
 * It handles timezone conversion and provides fallback for invalid dates.
 * 
 * @param date - The date to format (Date object, ISO string, or timestamp)
 * @param options - Formatting options
 * @param options.timezone - Target timezone (defaults to UTC)
 * @param options.includeTime - Whether to include time in the output
 * @param options.format - The output format ('short', 'medium', 'long')
 * 
 * @returns Formatted date string
 * 
 * @throws {ValidationError} When the date parameter is invalid
 * 
 * @example
 * ```typescript
 * const date = new Date('2024-01-15T10:30:00Z');
 * 
 * formatDate(date); // "2024-01-15"
 * formatDate(date, { includeTime: true }); // "2024-01-15 10:30:00"
 * formatDate(date, { format: 'long' }); // "January 15, 2024"
 * formatDate(date, { timezone: 'America/New_York' }); // Converted to EST/EDT
 * ```
 */
export function formatDate(
  date: Date | string | number,
  options: {
    timezone?: string;
    includeTime?: boolean;
    format?: 'short' | 'medium' | 'long';
  } = {}
): string {
  // Implementation details...
}
```

### Complex Logic Documentation

```typescript
/**
 * Implements the user permission resolution algorithm.
 * 
 * This function resolves user permissions by combining:
 * 1. Direct user permissions
 * 2. Role-based permissions
 * 3. Group memberships
 * 4. Temporary permission grants
 * 
 * The algorithm follows a hierarchical approach where more specific
 * permissions override general ones, and explicit denials take precedence
 * over grants.
 * 
 * @param userId - The user ID to resolve permissions for
 * @param context - The permission context (resource, action, etc.)
 * @returns Promise resolving to the effective permissions
 * 
 * @internal
 * @complexity O(n + m + g) where n=direct permissions, m=role permissions, g=group permissions
 */
async function resolveUserPermissions(
  userId: string,
  context: PermissionContext
): Promise<EffectivePermissions> {
  // Step 1: Load all permission sources
  // We need to gather permissions from multiple sources to build the complete picture
  const [directPermissions, rolePermissions, groupPermissions, tempGrants] = await Promise.all([
    loadDirectUserPermissions(userId),
    loadRoleBasedPermissions(userId),
    loadGroupPermissions(userId),
    loadTemporaryGrants(userId, context.timestamp)
  ]);

  // Step 2: Apply permission hierarchy
  // Direct permissions have the highest priority, followed by temporary grants,
  // then role permissions, and finally group permissions
  const effectivePermissions = new PermissionSet();

  // Start with the lowest priority (group permissions)
  effectivePermissions.merge(groupPermissions, { priority: 1 });
  
  // Add role permissions (higher priority)
  effectivePermissions.merge(rolePermissions, { priority: 2 });
  
  // Add temporary grants (even higher priority)
  effectivePermissions.merge(tempGrants, { priority: 3 });
  
  // Finally, add direct permissions (highest priority)
  effectivePermissions.merge(directPermissions, { priority: 4 });

  // Step 3: Apply context-specific filters
  // Some permissions may be context-dependent (time-based, location-based, etc.)
  const contextFiltered = effectivePermissions.applyContextFilters(context);

  // Step 4: Resolve conflicts
  // When there are conflicting permissions (grant vs deny), explicit denials win
  const resolved = contextFiltered.resolveConflicts({
    strategy: 'explicit-deny-wins'
  });

  return resolved;
}
```

## API Documentation Standards

### REST API Documentation

```typescript
/**
 * @fileoverview User management API endpoints
 * @version 1.0.0
 * @author DafnckMachine Team
 */

/**
 * User Controller - Handles all user-related HTTP requests
 * 
 * @route /api/v1/users
 * @tags Users
 */
export class UserController {
  /**
   * Retrieves a paginated list of users with optional filtering.
   * 
   * @route GET /api/v1/users
   * @access Private (requires authentication)
   * @permission users:read
   * 
   * @param req - Express request object
   * @param req.query.page - Page number (default: 1)
   * @param req.query.limit - Items per page (default: 20, max: 100)
   * @param req.query.search - Search term for name/email
   * @param req.query.role - Filter by user role
   * @param req.query.status - Filter by account status
   * @param req.query.sortBy - Sort field (name, email, createdAt)
   * @param req.query.sortOrder - Sort direction (asc, desc)
   * 
   * @param res - Express response object
   * 
   * @returns {Promise<void>} JSON response with user list and pagination info
   * 
   * @example
   * ```http
   * GET /api/v1/users?page=2&limit=10&role=admin&sortBy=name&sortOrder=asc
   * Authorization: Bearer <token>
   * 
   * Response:
   * {
   *   "data": [
   *     {
   *       "id": "user_123",
   *       "email": "admin@example.com",
   *       "firstName": "John",
   *       "lastName": "Doe",
   *       "role": "admin",
   *       "status": "active",
   *       "createdAt": "2024-01-15T10:30:00Z",
   *       "updatedAt": "2024-01-15T10:30:00Z"
   *     }
   *   ],
   *   "pagination": {
   *     "page": 2,
   *     "limit": 10,
   *     "total": 150,
   *     "totalPages": 15,
   *     "hasNext": true,
   *     "hasPrev": true
   *   }
   * }
   * ```
   * 
   * @throws {401} Unauthorized - Invalid or missing authentication token
   * @throws {403} Forbidden - Insufficient permissions
   * @throws {400} Bad Request - Invalid query parameters
   * @throws {500} Internal Server Error - Server-side error
   */
  public async getUsers(req: Request, res: Response): Promise<void> {
    // Implementation...
  }

  /**
   * Creates a new user account.
   * 
   * @route POST /api/v1/users
   * @access Private (requires authentication)
   * @permission users:create
   * 
   * @param req - Express request object
   * @param req.body - User creation data
   * @param req.body.email - User's email address (required, unique)
   * @param req.body.firstName - User's first name (required)
   * @param req.body.lastName - User's last name (required)
   * @param req.body.role - User's role (optional, defaults to 'user')
   * @param req.body.password - User's password (required, min 8 chars)
   * 
   * @param res - Express response object
   * 
   * @returns {Promise<void>} JSON response with created user data
   * 
   * @example
   * ```http
   * POST /api/v1/users
   * Authorization: Bearer <token>
   * Content-Type: application/json
   * 
   * {
   *   "email": "newuser@example.com",
   *   "firstName": "Jane",
   *   "lastName": "Smith",
   *   "role": "user",
   *   "password": "securePassword123"
   * }
   * 
   * Response:
   * {
   *   "data": {
   *     "id": "user_456",
   *     "email": "newuser@example.com",
   *     "firstName": "Jane",
   *     "lastName": "Smith",
   *     "role": "user",
   *     "status": "active",
   *     "createdAt": "2024-01-15T11:00:00Z",
   *     "updatedAt": "2024-01-15T11:00:00Z"
   *   },
   *   "message": "User created successfully"
   * }
   * ```
   * 
   * @throws {400} Bad Request - Invalid input data or validation errors
   * @throws {401} Unauthorized - Invalid or missing authentication token
   * @throws {403} Forbidden - Insufficient permissions
   * @throws {409} Conflict - Email already exists
   * @throws {500} Internal Server Error - Server-side error
   */
  public async createUser(req: Request, res: Response): Promise<void> {
    // Implementation...
  }
}
```

### GraphQL Documentation

```typescript
/**
 * @fileoverview GraphQL schema definitions for User operations
 */

/**
 * User type definition for GraphQL schema.
 * 
 * Represents a user entity in the system with all associated fields
 * and relationships.
 * 
 * @gql
 * @type User
 */
export const UserType = `
  """
  Represents a user in the DafnckMachine system.
  
  Users are the primary entities that interact with the application.
  Each user has a unique email address and can be assigned different roles
  that determine their permissions within the system.
  """
  type User {
    """
    Unique identifier for the user.
    This ID is immutable and generated automatically upon user creation.
    """
    id: ID!
    
    """
    User's email address.
    Must be unique across the system and is used for authentication.
    """
    email: String!
    
    """
    User's first name.
    Used for display purposes and personalization.
    """
    firstName: String!
    
    """
    User's last name.
    Used for display purposes and personalization.
    """
    lastName: String!
    
    """
    User's full name (computed field).
    Combines firstName and lastName for convenience.
    """
    fullName: String!
    
    """
    User's role in the system.
    Determines the permissions and access levels for the user.
    """
    role: UserRole!
    
    """
    Current status of the user account.
    Indicates whether the account is active, suspended, or inactive.
    """
    status: UserStatus!
    
    """
    Timestamp when the user account was created.
    """
    createdAt: DateTime!
    
    """
    Timestamp when the user account was last updated.
    """
    updatedAt: DateTime!
    
    """
    Projects associated with this user.
    Returns projects where the user is either an owner or collaborator.
    """
    projects(
      """Filter projects by status"""
      status: ProjectStatus
      
      """Limit the number of projects returned"""
      limit: Int = 10
      
      """Offset for pagination"""
      offset: Int = 0
    ): [Project!]!
    
    """
    User's profile information.
    Contains additional metadata and preferences.
    """
    profile: UserProfile
  }
`;

/**
 * GraphQL resolver for User queries and mutations.
 * 
 * @class UserResolver
 */
export class UserResolver {
  /**
   * Resolves the fullName field for a User.
   * 
   * This is a computed field that combines the user's first and last names.
   * 
   * @param parent - The parent User object
   * @returns The user's full name
   * 
   * @example
   * ```graphql
   * query {
   *   user(id: "123") {
   *     fullName  # Returns "John Doe"
   *   }
   * }
   * ```
   */
  @FieldResolver(() => String)
  fullName(@Parent() user: User): string {
    return `${user.firstName} ${user.lastName}`;
  }

  /**
   * Resolves the projects field for a User.
   * 
   * Returns all projects associated with the user, with optional filtering
   * and pagination.
   * 
   * @param parent - The parent User object
   * @param args - Query arguments for filtering and pagination
   * @param context - GraphQL context containing request information
   * @returns Array of projects associated with the user
   * 
   * @example
   * ```graphql
   * query {
   *   user(id: "123") {
   *     projects(status: ACTIVE, limit: 5) {
   *       id
   *       name
   *       status
   *     }
   *   }
   * }
   * ```
   */
  @FieldResolver(() => [Project])
  async projects(
    @Parent() user: User,
    @Args() args: UserProjectsArgs,
    @Context() context: GraphQLContext
  ): Promise<Project[]> {
    return this.projectService.findByUser(user.id, args, context);
  }

  /**
   * Query resolver to get a user by ID.
   * 
   * @param id - The user ID to retrieve
   * @param context - GraphQL context containing request information
   * @returns The user object or null if not found
   * 
   * @throws {AuthenticationError} When user is not authenticated
   * @throws {AuthorizationError} When user lacks permission to view the requested user
   * @throws {NotFoundError} When the user doesn't exist
   * 
   * @example
   * ```graphql
   * query GetUser($id: ID!) {
   *   user(id: $id) {
   *     id
   *     email
   *     fullName
   *     role
   *     status
   *   }
   * }
   * ```
   */
  @Query(() => User, { nullable: true })
  async user(
    @Arg('id') id: string,
    @Context() context: GraphQLContext
  ): Promise<User | null> {
    // Check authentication
    if (!context.user) {
      throw new AuthenticationError('Authentication required');
    }

    // Check authorization (users can view their own profile, admins can view any)
    if (context.user.id !== id && !context.user.hasPermission('users:read')) {
      throw new AuthorizationError('Insufficient permissions to view user');
    }

    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundError('User not found', 'user', id);
    }

    return user;
  }
}
```

## Module Documentation Standards

### README Template

```markdown
# Module Name

Brief description of what this module does and its primary purpose.

## Overview

Detailed explanation of the module's functionality, its role in the larger system, and key concepts.

## Installation

```bash
npm install module-name
```

## Quick Start

```typescript
import { ModuleName } from 'module-name';

const instance = new ModuleName({
  option1: 'value1',
  option2: 'value2'
});

const result = await instance.doSomething();
console.log(result);
```

## API Reference

### Classes

#### `ClassName`

Description of the class and its purpose.

**Constructor**

```typescript
new ClassName(options: ClassOptions)
```

**Methods**

- `methodName(param: Type): ReturnType` - Description of what the method does

### Functions

#### `functionName`

```typescript
functionName(param1: Type1, param2: Type2): ReturnType
```

Description of the function and its parameters.

### Types

#### `TypeName`

```typescript
interface TypeName {
  property1: string;
  property2: number;
}
```

Description of the type and its properties.

## Examples

### Basic Usage

```typescript
// Example code with comments
const example = new ModuleName();
const result = await example.process();
```

### Advanced Usage

```typescript
// More complex example
const advanced = new ModuleName({
  complexOption: {
    nested: 'value'
  }
});
```

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| option1 | string | 'default' | Description of option1 |
| option2 | number | 100 | Description of option2 |

## Error Handling

Common errors and how to handle them:

- `ErrorType1`: Description and resolution
- `ErrorType2`: Description and resolution

## Testing

```bash
npm test
```

## Contributing

Guidelines for contributing to this module.

## License

License information.
```

## Inline Comment Standards

### Comment Types and Usage

```typescript
// GOOD: Explain the "why" behind complex business logic
export class OrderProcessor {
  async processOrder(order: Order): Promise<ProcessedOrder> {
    // Apply discount before tax calculation to comply with local tax regulations
    const discountedAmount = this.applyDiscounts(order.amount, order.discounts);
    
    // Tax calculation must happen after discounts per IRS guidelines
    const taxAmount = this.calculateTax(discountedAmount, order.shippingAddress);
    
    // TODO: Implement fraud detection before payment processing (JIRA-123)
    const paymentResult = await this.processPayment(order.paymentMethod, discountedAmount + taxAmount);
    
    return {
      ...order,
      finalAmount: discountedAmount + taxAmount,
      paymentResult
    };
  }

  private calculateTax(amount: number, address: Address): number {
    // HACK: Temporary workaround for missing tax service integration
    // Remove this when TaxService API is available (expected: Q2 2024)
    if (address.state === 'DE') {
      return 0; // Delaware has no sales tax
    }
    
    // FIXME: This is a simplified tax calculation
    // Real implementation should use proper tax service
    return amount * 0.08; // 8% flat rate - not accurate for all states
  }
}

// BAD: Comments that just repeat the code
export class BadExample {
  // Increments the counter by 1
  incrementCounter(): void {
    this.counter++; // Add 1 to counter
  }
  
  // Gets the user name
  getUserName(): string {
    return this.user.name; // Return the name property
  }
}

// GOOD: Comments for complex algorithms
export function calculateComplexMetric(data: DataPoint[]): number {
  // Implementation of the Weighted Moving Average algorithm
  // Formula: WMA = (P1*n + P2*(n-1) + ... + Pn*1) / (n + (n-1) + ... + 1)
  // Where P = price, n = period length
  
  let weightedSum = 0;
  let weightSum = 0;
  
  for (let i = 0; i < data.length; i++) {
    const weight = data.length - i; // More recent data gets higher weight
    weightedSum += data[i].value * weight;
    weightSum += weight;
  }
  
  return weightedSum / weightSum;
}

// GOOD: Comments for edge cases and assumptions
export function parseUserInput(input: string): ParsedInput {
  // Handle empty input - return default values rather than throwing
  if (!input || input.trim().length === 0) {
    return { type: 'empty', value: null };
  }
  
  // Assumption: Input format is "type:value" (e.g., "number:42", "text:hello")
  const parts = input.split(':');
  
  // Handle malformed input - assume it's plain text if no colon found
  if (parts.length !== 2) {
    return { type: 'text', value: input };
  }
  
  const [type, value] = parts;
  
  // Special case: numeric values need type conversion
  if (type === 'number') {
    const numValue = parseFloat(value);
    // Handle NaN case - treat as text instead of failing
    if (isNaN(numValue)) {
      return { type: 'text', value: input };
    }
    return { type: 'number', value: numValue };
  }
  
  return { type, value };
}
```

### Comment Annotations

```typescript
/**
 * Standard comment annotations used throughout the codebase
 */

// TODO: Description of what needs to be done
// TODO(username): Assigned todo with responsible person
// TODO: Implement caching layer (JIRA-456) - with ticket reference

// FIXME: Description of what's broken and needs fixing
// FIXME: Memory leak in event listeners - investigate and fix

// HACK: Description of temporary workaround
// HACK: Using setTimeout instead of proper async handling - refactor needed

// NOTE: Important information for future developers
// NOTE: This function assumes input is already validated

// WARNING: Critical information about potential issues
// WARNING: Modifying this function affects payment processing

// DEPRECATED: Mark code that should no longer be used
// DEPRECATED: Use newFunction() instead - will be removed in v2.0

// OPTIMIZE: Performance improvement opportunities
// OPTIMIZE: Consider using Map instead of Object for better lookup performance

// REVIEW: Code that needs review or verification
// REVIEW: Check if this logic handles all edge cases correctly
```

## Documentation Maintenance

### Automated Documentation Generation

```typescript
/**
 * Configuration for automated documentation generation
 */

// package.json scripts
{
  "scripts": {
    "docs:generate": "typedoc --out docs src",
    "docs:api": "swagger-jsdoc -d swaggerDef.js src/**/*.ts",
    "docs:serve": "docsify serve docs",
    "docs:build": "npm run docs:generate && npm run docs:api",
    "docs:validate": "documentation lint src/**/*.ts"
  }
}

// typedoc.json configuration
{
  "entryPoints": ["src/index.ts"],
  "out": "docs",
  "theme": "default",
  "includeVersion": true,
  "excludeExternals": true,
  "excludePrivate": true,
  "excludeProtected": true,
  "categorizeByGroup": true,
  "defaultCategory": "Other",
  "categoryOrder": [
    "Core",
    "Services",
    "Controllers",
    "Utilities",
    "*",
    "Other"
  ]
}
```

### Documentation Review Process

```typescript
/**
 * Documentation review checklist and process
 */

// Pre-commit hooks for documentation validation
// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Validate JSDoc comments
npm run docs:validate

# Check for missing documentation
npm run lint:docs

# Ensure examples compile
npm run test:examples

// Documentation quality metrics
interface DocumentationMetrics {
  /** Percentage of public APIs with documentation */
  apiCoverage: number;
  
  /** Number of code examples in documentation */
  exampleCount: number;
  
  /** Number of outdated documentation sections */
  outdatedSections: number;
  
  /** Documentation freshness score (0-100) */
  freshnessScore: number;
}

// Automated documentation testing
describe('Documentation Tests', () => {
  it('should have JSDoc for all public methods', () => {
    // Test implementation
  });
  
  it('should have working code examples', () => {
    // Test implementation
  });
  
  it('should have up-to-date API documentation', () => {
    // Test implementation
  });
});
```

## Related Documentation

- [Architecture Decision Records](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Architecture_Decision_Records.md)
- [Developer Onboarding Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Developer_Onboarding_Guide.md)
- [Troubleshooting Documentation](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Troubleshooting_Documentation.md)
- [API Documentation Complete](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: documentation-agent
- **Related Workflows**: 14_Technical_Documentation.md 