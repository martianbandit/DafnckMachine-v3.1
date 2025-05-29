# Security Implementation Guide - DafnckMachine v3.1

## Overview
Comprehensive security implementation guidelines for the DafnckMachine v3.1 project, covering authentication, authorization, data protection, and security best practices.

## Security Architecture

### Defense in Depth Strategy
- **Perimeter Security**: Firewall, DDoS protection, rate limiting
- **Application Security**: Input validation, output encoding, secure coding
- **Data Security**: Encryption, access controls, data classification
- **Infrastructure Security**: Secure configurations, monitoring, logging

### Security Principles
1. **Least Privilege**: Grant minimum necessary permissions
2. **Zero Trust**: Verify every request and user
3. **Fail Secure**: Default to secure state on failures
4. **Security by Design**: Build security into every component

## Authentication Implementation

### JWT-Based Authentication

```typescript
// JWT Configuration
interface JWTConfig {
  accessTokenSecret: string;
  refreshTokenSecret: string;
  accessTokenExpiry: string; // '15m'
  refreshTokenExpiry: string; // '7d'
  issuer: string;
  audience: string;
}

// Token Generation
export class AuthService {
  async generateTokens(user: User): Promise<TokenPair> {
    const payload = {
      sub: user.id,
      email: user.email,
      role: user.role,
      permissions: user.permissions
    };

    const accessToken = jwt.sign(payload, config.accessTokenSecret, {
      expiresIn: config.accessTokenExpiry,
      issuer: config.issuer,
      audience: config.audience
    });

    const refreshToken = jwt.sign(
      { sub: user.id, type: 'refresh' },
      config.refreshTokenSecret,
      { expiresIn: config.refreshTokenExpiry }
    );

    return { accessToken, refreshToken };
  }

  async verifyToken(token: string): Promise<JWTPayload> {
    try {
      return jwt.verify(token, config.accessTokenSecret) as JWTPayload;
    } catch (error) {
      throw new UnauthorizedError('Invalid token');
    }
  }
}
```

### Multi-Factor Authentication (MFA)

```typescript
// TOTP Implementation
export class MFAService {
  async generateSecret(userId: string): Promise<MFASecret> {
    const secret = speakeasy.generateSecret({
      name: `DafnckMachine (${userId})`,
      issuer: 'DafnckMachine v3.1'
    });

    await this.storeMFASecret(userId, secret.base32);
    
    return {
      secret: secret.base32,
      qrCode: await QRCode.toDataURL(secret.otpauth_url),
      backupCodes: await this.generateBackupCodes(userId)
    };
  }

  async verifyTOTP(userId: string, token: string): Promise<boolean> {
    const secret = await this.getMFASecret(userId);
    
    return speakeasy.totp.verify({
      secret,
      encoding: 'base32',
      token,
      window: 2 // Allow 2 time steps tolerance
    });
  }

  async generateBackupCodes(userId: string): Promise<string[]> {
    const codes = Array.from({ length: 10 }, () => 
      crypto.randomBytes(4).toString('hex').toUpperCase()
    );
    
    await this.storeBackupCodes(userId, codes);
    return codes;
  }
}
```

### Session Management

```typescript
// Secure Session Configuration
export class SessionManager {
  private readonly sessionConfig = {
    name: 'dafnck_session',
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === 'production',
      httpOnly: true,
      maxAge: 24 * 60 * 60 * 1000, // 24 hours
      sameSite: 'strict' as const
    },
    store: new RedisStore({
      client: redisClient,
      prefix: 'sess:',
      ttl: 86400 // 24 hours
    })
  };

  async createSession(user: User, req: Request): Promise<void> {
    req.session.userId = user.id;
    req.session.role = user.role;
    req.session.loginTime = new Date();
    req.session.ipAddress = req.ip;
    req.session.userAgent = req.get('User-Agent');
  }

  async invalidateSession(sessionId: string): Promise<void> {
    await redisClient.del(`sess:${sessionId}`);
  }

  async invalidateAllUserSessions(userId: string): Promise<void> {
    const sessions = await redisClient.keys('sess:*');
    for (const sessionKey of sessions) {
      const session = await redisClient.get(sessionKey);
      if (session && JSON.parse(session).userId === userId) {
        await redisClient.del(sessionKey);
      }
    }
  }
}
```

## Authorization Implementation

### Role-Based Access Control (RBAC)

```typescript
// Role and Permission Definitions
export enum Role {
  SUPER_ADMIN = 'super_admin',
  ADMIN = 'admin',
  PROJECT_MANAGER = 'project_manager',
  DEVELOPER = 'developer',
  VIEWER = 'viewer'
}

export enum Permission {
  // User management
  USER_CREATE = 'user:create',
  USER_READ = 'user:read',
  USER_UPDATE = 'user:update',
  USER_DELETE = 'user:delete',
  
  // Project management
  PROJECT_CREATE = 'project:create',
  PROJECT_READ = 'project:read',
  PROJECT_UPDATE = 'project:update',
  PROJECT_DELETE = 'project:delete',
  
  // Task management
  TASK_CREATE = 'task:create',
  TASK_READ = 'task:read',
  TASK_UPDATE = 'task:update',
  TASK_DELETE = 'task:delete'
}

// Permission Matrix
const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  [Role.SUPER_ADMIN]: Object.values(Permission),
  [Role.ADMIN]: [
    Permission.USER_READ, Permission.USER_UPDATE,
    Permission.PROJECT_CREATE, Permission.PROJECT_READ, 
    Permission.PROJECT_UPDATE, Permission.PROJECT_DELETE,
    Permission.TASK_CREATE, Permission.TASK_READ, 
    Permission.TASK_UPDATE, Permission.TASK_DELETE
  ],
  [Role.PROJECT_MANAGER]: [
    Permission.PROJECT_READ, Permission.PROJECT_UPDATE,
    Permission.TASK_CREATE, Permission.TASK_READ, 
    Permission.TASK_UPDATE, Permission.TASK_DELETE
  ],
  [Role.DEVELOPER]: [
    Permission.PROJECT_READ,
    Permission.TASK_READ, Permission.TASK_UPDATE
  ],
  [Role.VIEWER]: [
    Permission.PROJECT_READ,
    Permission.TASK_READ
  ]
};
```

### Authorization Middleware

```typescript
// Permission Check Middleware
export function requirePermission(permission: Permission) {
  return async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    try {
      const user = req.user;
      if (!user) {
        return res.status(401).json({ error: 'Authentication required' });
      }

      const hasPermission = await checkUserPermission(user.id, permission);
      if (!hasPermission) {
        return res.status(403).json({ 
          error: 'Insufficient permissions',
          required: permission
        });
      }

      next();
    } catch (error) {
      next(error);
    }
  };
}

// Resource-based Authorization
export function requireResourceAccess(resourceType: string) {
  return async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    try {
      const resourceId = req.params.id;
      const userId = req.user.id;

      const hasAccess = await checkResourceAccess(userId, resourceType, resourceId);
      if (!hasAccess) {
        return res.status(403).json({ 
          error: 'Access denied to resource',
          resource: { type: resourceType, id: resourceId }
        });
      }

      next();
    } catch (error) {
      next(error);
    }
  };
}
```

## Data Protection

### Encryption Implementation

```typescript
// Data Encryption Service
export class EncryptionService {
  private readonly algorithm = 'aes-256-gcm';
  private readonly keyDerivationIterations = 100000;

  async encryptSensitiveData(data: string, userKey?: string): Promise<EncryptedData> {
    const key = userKey || process.env.ENCRYPTION_KEY;
    const iv = crypto.randomBytes(16);
    const salt = crypto.randomBytes(32);
    
    const derivedKey = crypto.pbkdf2Sync(key, salt, this.keyDerivationIterations, 32, 'sha256');
    const cipher = crypto.createCipher(this.algorithm, derivedKey);
    
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      salt: salt.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  async decryptSensitiveData(encryptedData: EncryptedData, userKey?: string): Promise<string> {
    const key = userKey || process.env.ENCRYPTION_KEY;
    const derivedKey = crypto.pbkdf2Sync(
      key, 
      Buffer.from(encryptedData.salt, 'hex'), 
      this.keyDerivationIterations, 
      32, 
      'sha256'
    );
    
    const decipher = crypto.createDecipher(this.algorithm, derivedKey);
    decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

### Password Security

```typescript
// Password Hashing and Validation
export class PasswordService {
  private readonly saltRounds = 12;
  private readonly minLength = 8;
  private readonly maxLength = 128;

  async hashPassword(password: string): Promise<string> {
    this.validatePasswordStrength(password);
    return bcrypt.hash(password, this.saltRounds);
  }

  async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  validatePasswordStrength(password: string): void {
    if (password.length < this.minLength || password.length > this.maxLength) {
      throw new ValidationError(`Password must be ${this.minLength}-${this.maxLength} characters`);
    }

    const requirements = [
      { regex: /[a-z]/, message: 'lowercase letter' },
      { regex: /[A-Z]/, message: 'uppercase letter' },
      { regex: /\d/, message: 'number' },
      { regex: /[!@#$%^&*(),.?":{}|<>]/, message: 'special character' }
    ];

    const missing = requirements.filter(req => !req.regex.test(password));
    if (missing.length > 0) {
      throw new ValidationError(
        `Password must contain: ${missing.map(m => m.message).join(', ')}`
      );
    }
  }

  async checkPasswordHistory(userId: string, newPassword: string): Promise<boolean> {
    const passwordHistory = await this.getPasswordHistory(userId, 5); // Last 5 passwords
    
    for (const oldHash of passwordHistory) {
      if (await this.verifyPassword(newPassword, oldHash)) {
        return false; // Password was used before
      }
    }
    
    return true; // Password is new
  }
}
```

## Input Validation and Sanitization

### Request Validation

```typescript
// Input Validation Schemas
export const userValidationSchema = {
  create: Joi.object({
    email: Joi.string().email().required().max(255),
    username: Joi.string().alphanum().min(3).max(50).required(),
    password: Joi.string().min(8).max(128).required(),
    firstName: Joi.string().max(100).optional(),
    lastName: Joi.string().max(100).optional()
  }),
  
  update: Joi.object({
    email: Joi.string().email().max(255).optional(),
    firstName: Joi.string().max(100).optional(),
    lastName: Joi.string().max(100).optional(),
    bio: Joi.string().max(1000).optional()
  })
};

// Validation Middleware
export function validateRequest(schema: Joi.ObjectSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });

    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));
      
      return res.status(400).json({
        error: 'Validation failed',
        details: errors
      });
    }

    req.body = value;
    next();
  };
}
```

### SQL Injection Prevention

```typescript
// Safe Database Queries with Prisma
export class UserRepository {
  async findUserByEmail(email: string): Promise<User | null> {
    // Prisma automatically prevents SQL injection
    return prisma.user.findUnique({
      where: { email },
      select: {
        id: true,
        email: true,
        username: true,
        firstName: true,
        lastName: true,
        role: true,
        status: true
      }
    });
  }

  async searchUsers(query: string, limit: number = 10): Promise<User[]> {
    // Safe parameterized query
    return prisma.user.findMany({
      where: {
        OR: [
          { username: { contains: query, mode: 'insensitive' } },
          { firstName: { contains: query, mode: 'insensitive' } },
          { lastName: { contains: query, mode: 'insensitive' } }
        ]
      },
      take: Math.min(limit, 100), // Prevent large result sets
      select: {
        id: true,
        username: true,
        firstName: true,
        lastName: true
      }
    });
  }
}
```

## Security Headers and CORS

### Security Headers Middleware

```typescript
// Security Headers Configuration
export function securityHeaders() {
  return helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        imgSrc: ["'self'", "data:", "https:"],
        scriptSrc: ["'self'"],
        connectSrc: ["'self'", "https://api.dafnckmachine.com"],
        frameSrc: ["'none'"],
        objectSrc: ["'none'"],
        baseUri: ["'self'"],
        formAction: ["'self'"]
      }
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true
    },
    noSniff: true,
    xssFilter: true,
    referrerPolicy: { policy: "strict-origin-when-cross-origin" }
  });
}

// CORS Configuration
export const corsOptions: CorsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  maxAge: 86400 // 24 hours
};
```

## Rate Limiting and DDoS Protection

### Rate Limiting Implementation

```typescript
// Rate Limiting Configuration
export class RateLimitService {
  private readonly redisClient: Redis;

  constructor() {
    this.redisClient = new Redis(process.env.REDIS_URL);
  }

  createRateLimit(options: RateLimitOptions) {
    return rateLimit({
      store: new RedisStore({
        client: this.redisClient,
        prefix: 'rl:'
      }),
      windowMs: options.windowMs,
      max: options.max,
      message: {
        error: 'Too many requests',
        retryAfter: Math.ceil(options.windowMs / 1000)
      },
      standardHeaders: true,
      legacyHeaders: false,
      keyGenerator: (req) => {
        return req.user?.id || req.ip;
      }
    });
  }

  // Different rate limits for different endpoints
  authRateLimit = this.createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5 // 5 attempts per window
  });

  apiRateLimit = this.createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // 100 requests per window
  });

  uploadRateLimit = this.createRateLimit({
    windowMs: 60 * 60 * 1000, // 1 hour
    max: 10 // 10 uploads per hour
  });
}
```

## Security Monitoring and Logging

### Security Event Logging

```typescript
// Security Event Logger
export class SecurityLogger {
  private readonly logger: Logger;

  constructor() {
    this.logger = winston.createLogger({
      level: 'info',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
      ),
      transports: [
        new winston.transports.File({ 
          filename: 'logs/security.log',
          level: 'warn'
        }),
        new winston.transports.Console({
          format: winston.format.simple()
        })
      ]
    });
  }

  logAuthenticationAttempt(event: AuthEvent): void {
    this.logger.info('Authentication attempt', {
      type: 'auth_attempt',
      userId: event.userId,
      email: event.email,
      success: event.success,
      ip: event.ip,
      userAgent: event.userAgent,
      timestamp: new Date().toISOString()
    });
  }

  logSecurityViolation(event: SecurityViolation): void {
    this.logger.warn('Security violation detected', {
      type: 'security_violation',
      violation: event.type,
      userId: event.userId,
      ip: event.ip,
      details: event.details,
      severity: event.severity,
      timestamp: new Date().toISOString()
    });
  }

  logPrivilegeEscalation(event: PrivilegeEvent): void {
    this.logger.error('Privilege escalation attempt', {
      type: 'privilege_escalation',
      userId: event.userId,
      attemptedAction: event.action,
      requiredPermission: event.permission,
      userRole: event.userRole,
      ip: event.ip,
      timestamp: new Date().toISOString()
    });
  }
}
```

### Intrusion Detection

```typescript
// Anomaly Detection Service
export class AnomalyDetectionService {
  async detectSuspiciousActivity(userId: string, activity: UserActivity): Promise<boolean> {
    const recentActivities = await this.getRecentActivities(userId, 24); // Last 24 hours
    
    // Check for unusual patterns
    const suspiciousPatterns = [
      this.checkRapidLocationChanges(recentActivities),
      this.checkUnusualAccessPatterns(recentActivities),
      this.checkBruteForceAttempts(recentActivities),
      this.checkPrivilegeEscalationAttempts(recentActivities)
    ];

    return suspiciousPatterns.some(pattern => pattern);
  }

  private checkRapidLocationChanges(activities: UserActivity[]): boolean {
    const locations = activities
      .map(a => a.location)
      .filter(Boolean)
      .slice(-5); // Last 5 activities

    if (locations.length < 2) return false;

    // Check if locations are geographically impossible
    for (let i = 1; i < locations.length; i++) {
      const distance = this.calculateDistance(locations[i-1], locations[i]);
      const timeDiff = activities[i].timestamp - activities[i-1].timestamp;
      const speed = distance / (timeDiff / 3600000); // km/h

      if (speed > 1000) { // Impossible travel speed
        return true;
      }
    }

    return false;
  }
}
```

## Security Testing

### Automated Security Tests

```typescript
// Security Test Suite
describe('Security Tests', () => {
  describe('Authentication', () => {
    it('should reject invalid JWT tokens', async () => {
      const response = await request(app)
        .get('/api/protected')
        .set('Authorization', 'Bearer invalid-token');
      
      expect(response.status).toBe(401);
    });

    it('should enforce rate limiting on login attempts', async () => {
      const loginData = { email: 'test@example.com', password: 'wrong' };
      
      // Make 6 failed attempts (limit is 5)
      for (let i = 0; i < 6; i++) {
        await request(app).post('/api/auth/login').send(loginData);
      }
      
      const response = await request(app)
        .post('/api/auth/login')
        .send(loginData);
      
      expect(response.status).toBe(429); // Too Many Requests
    });
  });

  describe('Input Validation', () => {
    it('should sanitize SQL injection attempts', async () => {
      const maliciousInput = "'; DROP TABLE users; --";
      
      const response = await request(app)
        .post('/api/users/search')
        .send({ query: maliciousInput })
        .set('Authorization', `Bearer ${validToken}`);
      
      expect(response.status).toBe(400);
    });

    it('should prevent XSS attacks', async () => {
      const xssPayload = '<script>alert("xss")</script>';
      
      const response = await request(app)
        .post('/api/projects')
        .send({ name: xssPayload })
        .set('Authorization', `Bearer ${validToken}`);
      
      expect(response.status).toBe(400);
    });
  });
});
```

## Security Configuration Checklist

### Production Security Checklist

- [ ] **Environment Variables**
  - [ ] All secrets stored in environment variables
  - [ ] No hardcoded credentials in code
  - [ ] Separate configs for different environments

- [ ] **HTTPS/TLS**
  - [ ] SSL/TLS certificates properly configured
  - [ ] HTTP redirects to HTTPS
  - [ ] HSTS headers enabled

- [ ] **Database Security**
  - [ ] Database connections encrypted
  - [ ] Principle of least privilege for DB users
  - [ ] Regular security updates applied

- [ ] **API Security**
  - [ ] Rate limiting implemented
  - [ ] Input validation on all endpoints
  - [ ] Proper error handling (no sensitive info leakage)

- [ ] **Monitoring**
  - [ ] Security event logging enabled
  - [ ] Anomaly detection configured
  - [ ] Alert systems in place

## Related Documentation

- [Database Schema Design](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Database_Schema_Design.md)
- [API Documentation Complete](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)
- [Performance Optimization Strategies](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Strategies.md)
- [Error Handling Framework](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Error_Handling_Framework.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: security-auditor-agent
- **Related Workflows**: 13_Backend_Development.md 