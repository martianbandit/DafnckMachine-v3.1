# Monitoring Analytics Implementation - DafnckMachine v3.1

## Overview
Comprehensive monitoring and analytics implementation for the DafnckMachine v3.1 project, covering application performance monitoring, logging, alerting, user analytics, and business intelligence.

## Monitoring Strategy

### Monitoring Pillars
- **Metrics**: Quantitative measurements of system behavior
- **Logs**: Detailed records of system events and transactions
- **Traces**: Request flow through distributed systems
- **Alerts**: Proactive notifications of system issues
- **Analytics**: User behavior and business metrics

### Observability Stack
- **Application Monitoring**: New Relic, Sentry
- **Infrastructure Monitoring**: AWS CloudWatch, Prometheus
- **Log Management**: AWS CloudWatch Logs, ELK Stack
- **User Analytics**: Google Analytics, Mixpanel
- **Business Intelligence**: Custom dashboards, Grafana

## Application Performance Monitoring

### New Relic Integration

```typescript
// src/monitoring/newrelic.ts
import newrelic from 'newrelic';
import { config } from '../config/environment';

export class NewRelicMonitoring {
  private static instance: NewRelicMonitoring;

  private constructor() {
    if (config.NEW_RELIC_LICENSE_KEY) {
      this.initializeNewRelic();
    }
  }

  static getInstance(): NewRelicMonitoring {
    if (!NewRelicMonitoring.instance) {
      NewRelicMonitoring.instance = new NewRelicMonitoring();
    }
    return NewRelicMonitoring.instance;
  }

  private initializeNewRelic(): void {
    // Custom attributes for all transactions
    newrelic.addCustomAttributes({
      environment: config.NODE_ENV,
      version: config.APP_VERSION,
      region: config.AWS_REGION
    });
  }

  // Custom transaction tracking
  recordCustomEvent(eventType: string, attributes: Record<string, any>): void {
    newrelic.recordCustomEvent(eventType, attributes);
  }

  // Custom metric recording
  recordMetric(name: string, value: number): void {
    newrelic.recordMetric(name, value);
  }

  // Error tracking
  noticeError(error: Error, customAttributes?: Record<string, any>): void {
    newrelic.noticeError(error, customAttributes);
  }

  // Transaction naming
  setTransactionName(category: string, name: string): void {
    newrelic.setTransactionName(category, name);
  }

  // Custom timing
  startWebTransaction(url: string, handle: () => Promise<any>): Promise<any> {
    return newrelic.startWebTransaction(url, handle);
  }

  // Database query tracking
  startSegment(name: string, record: boolean = true, handler: () => Promise<any>): Promise<any> {
    return newrelic.startSegment(name, record, handler);
  }
}

export const monitoring = NewRelicMonitoring.getInstance();
```

### Sentry Error Tracking

```typescript
// src/monitoring/sentry.ts
import * as Sentry from '@sentry/node';
import { config } from '../config/environment';

export class SentryMonitoring {
  private static instance: SentryMonitoring;

  private constructor() {
    if (config.SENTRY_DSN) {
      this.initializeSentry();
    }
  }

  static getInstance(): SentryMonitoring {
    if (!SentryMonitoring.instance) {
      SentryMonitoring.instance = new SentryMonitoring();
    }
    return SentryMonitoring.instance;
  }

  private initializeSentry(): void {
    Sentry.init({
      dsn: config.SENTRY_DSN,
      environment: config.NODE_ENV,
      release: config.APP_VERSION,
      tracesSampleRate: config.NODE_ENV === 'production' ? 0.1 : 1.0,
      
      beforeSend(event) {
        // Filter out sensitive information
        if (event.request?.headers) {
          delete event.request.headers.authorization;
          delete event.request.headers.cookie;
        }
        return event;
      },

      integrations: [
        new Sentry.Integrations.Http({ tracing: true }),
        new Sentry.Integrations.Express({ app: undefined }),
      ],
    });
  }

  // Capture exceptions
  captureException(error: Error, context?: Sentry.Scope): void {
    Sentry.captureException(error, context);
  }

  // Capture messages
  captureMessage(message: string, level: Sentry.SeverityLevel = 'info'): void {
    Sentry.captureMessage(message, level);
  }

  // Add breadcrumb
  addBreadcrumb(breadcrumb: Sentry.Breadcrumb): void {
    Sentry.addBreadcrumb(breadcrumb);
  }

  // Set user context
  setUser(user: { id: string; email?: string; username?: string }): void {
    Sentry.setUser(user);
  }

  // Set tags
  setTag(key: string, value: string): void {
    Sentry.setTag(key, value);
  }

  // Set extra context
  setExtra(key: string, extra: any): void {
    Sentry.setExtra(key, extra);
  }

  // Performance monitoring
  startTransaction(name: string, op: string): Sentry.Transaction {
    return Sentry.startTransaction({ name, op });
  }
}

export const errorTracking = SentryMonitoring.getInstance();
```

### Custom Metrics Collection

```typescript
// src/monitoring/metrics.ts
import { monitoring } from './newrelic';
import { logger } from '../utils/logger';

export class MetricsCollector {
  private static instance: MetricsCollector;
  private metrics: Map<string, number> = new Map();

  static getInstance(): MetricsCollector {
    if (!MetricsCollector.instance) {
      MetricsCollector.instance = new MetricsCollector();
    }
    return MetricsCollector.instance;
  }

  // Business metrics
  recordUserRegistration(userId: string, source: string): void {
    monitoring.recordCustomEvent('UserRegistration', {
      userId,
      source,
      timestamp: Date.now()
    });
    
    this.incrementCounter('user.registrations.total');
    this.incrementCounter(`user.registrations.source.${source}`);
  }

  recordUserLogin(userId: string, method: string): void {
    monitoring.recordCustomEvent('UserLogin', {
      userId,
      method,
      timestamp: Date.now()
    });
    
    this.incrementCounter('user.logins.total');
    this.incrementCounter(`user.logins.method.${method}`);
  }

  recordProjectCreation(userId: string, projectType: string): void {
    monitoring.recordCustomEvent('ProjectCreation', {
      userId,
      projectType,
      timestamp: Date.now()
    });
    
    this.incrementCounter('projects.created.total');
    this.incrementCounter(`projects.created.type.${projectType}`);
  }

  recordTaskCompletion(userId: string, taskId: string, duration: number): void {
    monitoring.recordCustomEvent('TaskCompletion', {
      userId,
      taskId,
      duration,
      timestamp: Date.now()
    });
    
    this.recordTiming('tasks.completion.duration', duration);
    this.incrementCounter('tasks.completed.total');
  }

  // Performance metrics
  recordApiResponse(endpoint: string, method: string, statusCode: number, duration: number): void {
    monitoring.recordCustomEvent('ApiResponse', {
      endpoint,
      method,
      statusCode,
      duration,
      timestamp: Date.now()
    });
    
    this.recordTiming(`api.response.${endpoint}.${method}`, duration);
    this.incrementCounter(`api.requests.${statusCode}`);
  }

  recordDatabaseQuery(operation: string, table: string, duration: number): void {
    monitoring.recordCustomEvent('DatabaseQuery', {
      operation,
      table,
      duration,
      timestamp: Date.now()
    });
    
    this.recordTiming(`database.query.${operation}.${table}`, duration);
  }

  recordCacheOperation(operation: string, key: string, hit: boolean, duration: number): void {
    monitoring.recordCustomEvent('CacheOperation', {
      operation,
      key,
      hit,
      duration,
      timestamp: Date.now()
    });
    
    this.recordTiming(`cache.${operation}`, duration);
    this.incrementCounter(`cache.${hit ? 'hits' : 'misses'}`);
  }

  // Helper methods
  private incrementCounter(name: string, value: number = 1): void {
    const current = this.metrics.get(name) || 0;
    this.metrics.set(name, current + value);
    monitoring.recordMetric(name, current + value);
  }

  private recordTiming(name: string, duration: number): void {
    monitoring.recordMetric(name, duration);
  }

  // Get current metrics
  getMetrics(): Record<string, number> {
    return Object.fromEntries(this.metrics);
  }

  // Reset metrics
  resetMetrics(): void {
    this.metrics.clear();
  }
}

export const metrics = MetricsCollector.getInstance();
```

## Logging Implementation

### Structured Logging

```typescript
// src/utils/logger.ts
import winston from 'winston';
import { config } from '../config/environment';

// Custom log format
const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json(),
  winston.format.printf(({ timestamp, level, message, ...meta }) => {
    return JSON.stringify({
      timestamp,
      level,
      message,
      environment: config.NODE_ENV,
      version: config.APP_VERSION,
      ...meta
    });
  })
);

// Create logger instance
export const logger = winston.createLogger({
  level: config.LOG_LEVEL,
  format: logFormat,
  defaultMeta: {
    service: 'dafnckmachine-api',
    environment: config.NODE_ENV,
    version: config.APP_VERSION
  },
  transports: [
    // Console transport for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // File transport for all logs
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

// Add CloudWatch transport for production
if (config.NODE_ENV === 'production') {
  const CloudWatchTransport = require('winston-cloudwatch');
  
  logger.add(new CloudWatchTransport({
    logGroupName: `/aws/ecs/dafnckmachine-${config.NODE_ENV}`,
    logStreamName: `api-${new Date().toISOString().split('T')[0]}`,
    awsRegion: config.AWS_REGION,
    messageFormatter: ({ level, message, ...meta }) => {
      return JSON.stringify({ level, message, ...meta });
    }
  }));
}

// Request logging middleware
export const requestLogger = (req: any, res: any, next: any) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration,
      userAgent: req.get('User-Agent'),
      ip: req.ip,
      userId: req.user?.id
    });
  });
  
  next();
};

// Error logging
export const logError = (error: Error, context?: Record<string, any>) => {
  logger.error('Application Error', {
    error: {
      name: error.name,
      message: error.message,
      stack: error.stack
    },
    ...context
  });
};

// Business event logging
export const logBusinessEvent = (event: string, data: Record<string, any>) => {
  logger.info('Business Event', {
    event,
    ...data
  });
};
```

### Log Analysis and Alerting

```typescript
// src/monitoring/log-analysis.ts
import { logger } from '../utils/logger';
import { errorTracking } from './sentry';

export class LogAnalyzer {
  private static instance: LogAnalyzer;
  private errorCounts: Map<string, number> = new Map();
  private alertThresholds = {
    errorRate: 10, // errors per minute
    responseTime: 5000, // milliseconds
    failureRate: 0.05 // 5%
  };

  static getInstance(): LogAnalyzer {
    if (!LogAnalyzer.instance) {
      LogAnalyzer.instance = new LogAnalyzer();
    }
    return LogAnalyzer.instance;
  }

  // Analyze error patterns
  analyzeError(error: Error, context?: Record<string, any>): void {
    const errorKey = `${error.name}:${error.message}`;
    const count = this.errorCounts.get(errorKey) || 0;
    this.errorCounts.set(errorKey, count + 1);

    // Check if error rate exceeds threshold
    if (count + 1 >= this.alertThresholds.errorRate) {
      this.triggerAlert('high_error_rate', {
        error: errorKey,
        count: count + 1,
        context
      });
    }

    // Log to Sentry for detailed analysis
    errorTracking.captureException(error, (scope) => {
      if (context) {
        Object.keys(context).forEach(key => {
          scope.setExtra(key, context[key]);
        });
      }
      return scope;
    });
  }

  // Analyze response times
  analyzeResponseTime(endpoint: string, duration: number): void {
    if (duration > this.alertThresholds.responseTime) {
      this.triggerAlert('slow_response', {
        endpoint,
        duration,
        threshold: this.alertThresholds.responseTime
      });
    }
  }

  // Analyze failure rates
  analyzeFailureRate(endpoint: string, successCount: number, failureCount: number): void {
    const totalRequests = successCount + failureCount;
    const failureRate = failureCount / totalRequests;

    if (failureRate > this.alertThresholds.failureRate) {
      this.triggerAlert('high_failure_rate', {
        endpoint,
        failureRate,
        threshold: this.alertThresholds.failureRate,
        totalRequests
      });
    }
  }

  private triggerAlert(alertType: string, data: Record<string, any>): void {
    logger.error('Alert Triggered', {
      alertType,
      ...data,
      timestamp: new Date().toISOString()
    });

    // Send to monitoring systems
    errorTracking.captureMessage(`Alert: ${alertType}`, 'warning');
  }

  // Reset counters (called periodically)
  resetCounters(): void {
    this.errorCounts.clear();
  }
}

export const logAnalyzer = LogAnalyzer.getInstance();
```

## User Analytics Implementation

### Google Analytics Integration

```typescript
// src/analytics/google-analytics.ts
import { config } from '../config/environment';

export class GoogleAnalytics {
  private static instance: GoogleAnalytics;
  private trackingId: string;

  private constructor() {
    this.trackingId = config.GOOGLE_ANALYTICS_ID || '';
  }

  static getInstance(): GoogleAnalytics {
    if (!GoogleAnalytics.instance) {
      GoogleAnalytics.instance = new GoogleAnalytics();
    }
    return GoogleAnalytics.instance;
  }

  // Track page views
  trackPageView(page: string, title: string, userId?: string): void {
    if (!this.trackingId) return;

    const data = {
      page_title: title,
      page_location: page,
      user_id: userId
    };

    this.sendEvent('page_view', data);
  }

  // Track user events
  trackEvent(action: string, category: string, label?: string, value?: number): void {
    if (!this.trackingId) return;

    const data = {
      event_category: category,
      event_label: label,
      value: value
    };

    this.sendEvent(action, data);
  }

  // Track user registration
  trackUserRegistration(userId: string, method: string): void {
    this.trackEvent('sign_up', 'user', method);
    this.setUserId(userId);
  }

  // Track user login
  trackUserLogin(userId: string, method: string): void {
    this.trackEvent('login', 'user', method);
    this.setUserId(userId);
  }

  // Track project creation
  trackProjectCreation(projectType: string): void {
    this.trackEvent('create_project', 'project', projectType);
  }

  // Track task completion
  trackTaskCompletion(taskType: string, duration: number): void {
    this.trackEvent('complete_task', 'task', taskType, duration);
  }

  // Set user ID
  private setUserId(userId: string): void {
    // Implementation depends on GA4 setup
  }

  // Send event to Google Analytics
  private sendEvent(eventName: string, parameters: Record<string, any>): void {
    // Implementation for server-side GA4 tracking
    // This would typically use the Measurement Protocol
  }
}

export const googleAnalytics = GoogleAnalytics.getInstance();
```

### Mixpanel Integration

```typescript
// src/analytics/mixpanel.ts
import Mixpanel from 'mixpanel';
import { config } from '../config/environment';

export class MixpanelAnalytics {
  private static instance: MixpanelAnalytics;
  private mixpanel: Mixpanel.Mixpanel;

  private constructor() {
    if (config.MIXPANEL_TOKEN) {
      this.mixpanel = Mixpanel.init(config.MIXPANEL_TOKEN);
    }
  }

  static getInstance(): MixpanelAnalytics {
    if (!MixpanelAnalytics.instance) {
      MixpanelAnalytics.instance = new MixpanelAnalytics();
    }
    return MixpanelAnalytics.instance;
  }

  // Track events
  track(event: string, properties: Record<string, any>, userId?: string): void {
    if (!this.mixpanel) return;

    const eventProperties = {
      ...properties,
      environment: config.NODE_ENV,
      timestamp: new Date().toISOString()
    };

    if (userId) {
      eventProperties.distinct_id = userId;
    }

    this.mixpanel.track(event, eventProperties);
  }

  // Set user properties
  setUserProperties(userId: string, properties: Record<string, any>): void {
    if (!this.mixpanel) return;

    this.mixpanel.people.set(userId, {
      ...properties,
      $last_seen: new Date().toISOString()
    });
  }

  // Track user journey
  trackUserJourney(userId: string, step: string, metadata?: Record<string, any>): void {
    this.track('User Journey Step', {
      step,
      user_id: userId,
      ...metadata
    }, userId);
  }

  // Track feature usage
  trackFeatureUsage(feature: string, userId: string, metadata?: Record<string, any>): void {
    this.track('Feature Used', {
      feature,
      user_id: userId,
      ...metadata
    }, userId);
  }

  // Track conversion events
  trackConversion(event: string, userId: string, value?: number): void {
    this.track('Conversion', {
      conversion_event: event,
      user_id: userId,
      value
    }, userId);
  }
}

export const mixpanelAnalytics = MixpanelAnalytics.getInstance();
```

## Infrastructure Monitoring

### CloudWatch Metrics and Alarms

```typescript
// src/monitoring/cloudwatch.ts
import { CloudWatchClient, PutMetricDataCommand, MetricDatum } from '@aws-sdk/client-cloudwatch';
import { config } from '../config/environment';

export class CloudWatchMonitoring {
  private static instance: CloudWatchMonitoring;
  private cloudWatch: CloudWatchClient;
  private namespace = 'DafnckMachine/Application';

  private constructor() {
    this.cloudWatch = new CloudWatchClient({ region: config.AWS_REGION });
  }

  static getInstance(): CloudWatchMonitoring {
    if (!CloudWatchMonitoring.instance) {
      CloudWatchMonitoring.instance = new CloudWatchMonitoring();
    }
    return CloudWatchMonitoring.instance;
  }

  // Send custom metrics to CloudWatch
  async putMetric(metricName: string, value: number, unit: string = 'Count', dimensions?: Record<string, string>): Promise<void> {
    const metricData: MetricDatum = {
      MetricName: metricName,
      Value: value,
      Unit: unit,
      Timestamp: new Date()
    };

    if (dimensions) {
      metricData.Dimensions = Object.entries(dimensions).map(([Name, Value]) => ({ Name, Value }));
    }

    try {
      await this.cloudWatch.send(new PutMetricDataCommand({
        Namespace: this.namespace,
        MetricData: [metricData]
      }));
    } catch (error) {
      console.error('Failed to send metric to CloudWatch:', error);
    }
  }

  // Business metrics
  async recordUserActivity(activityType: string, count: number = 1): Promise<void> {
    await this.putMetric('UserActivity', count, 'Count', {
      ActivityType: activityType,
      Environment: config.NODE_ENV
    });
  }

  async recordApiLatency(endpoint: string, latency: number): Promise<void> {
    await this.putMetric('ApiLatency', latency, 'Milliseconds', {
      Endpoint: endpoint,
      Environment: config.NODE_ENV
    });
  }

  async recordErrorRate(errorType: string, count: number = 1): Promise<void> {
    await this.putMetric('ErrorRate', count, 'Count', {
      ErrorType: errorType,
      Environment: config.NODE_ENV
    });
  }

  async recordDatabaseConnections(activeConnections: number): Promise<void> {
    await this.putMetric('DatabaseConnections', activeConnections, 'Count', {
      Environment: config.NODE_ENV
    });
  }

  async recordCacheHitRate(hitRate: number): Promise<void> {
    await this.putMetric('CacheHitRate', hitRate, 'Percent', {
      Environment: config.NODE_ENV
    });
  }
}

export const cloudWatchMonitoring = CloudWatchMonitoring.getInstance();
```

### Prometheus Metrics

```typescript
// src/monitoring/prometheus.ts
import client from 'prom-client';
import { config } from '../config/environment';

export class PrometheusMonitoring {
  private static instance: PrometheusMonitoring;
  private register: client.Registry;
  
  // Metrics
  private httpRequestDuration: client.Histogram<string>;
  private httpRequestTotal: client.Counter<string>;
  private activeUsers: client.Gauge<string>;
  private databaseQueryDuration: client.Histogram<string>;
  private cacheOperations: client.Counter<string>;

  private constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
    
    // Collect default metrics
    client.collectDefaultMetrics({ register: this.register });
  }

  static getInstance(): PrometheusMonitoring {
    if (!PrometheusMonitoring.instance) {
      PrometheusMonitoring.instance = new PrometheusMonitoring();
    }
    return PrometheusMonitoring.instance;
  }

  private initializeMetrics(): void {
    // HTTP request duration
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.1, 0.5, 1, 2, 5, 10]
    });

    // HTTP request total
    this.httpRequestTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code']
    });

    // Active users
    this.activeUsers = new client.Gauge({
      name: 'active_users_total',
      help: 'Number of active users'
    });

    // Database query duration
    this.databaseQueryDuration = new client.Histogram({
      name: 'database_query_duration_seconds',
      help: 'Duration of database queries in seconds',
      labelNames: ['operation', 'table'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 2]
    });

    // Cache operations
    this.cacheOperations = new client.Counter({
      name: 'cache_operations_total',
      help: 'Total number of cache operations',
      labelNames: ['operation', 'result']
    });

    // Register all metrics
    this.register.registerMetric(this.httpRequestDuration);
    this.register.registerMetric(this.httpRequestTotal);
    this.register.registerMetric(this.activeUsers);
    this.register.registerMetric(this.databaseQueryDuration);
    this.register.registerMetric(this.cacheOperations);
  }

  // Record HTTP request
  recordHttpRequest(method: string, route: string, statusCode: number, duration: number): void {
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode.toString() },
      duration / 1000
    );
    
    this.httpRequestTotal.inc({
      method,
      route,
      status_code: statusCode.toString()
    });
  }

  // Update active users
  setActiveUsers(count: number): void {
    this.activeUsers.set(count);
  }

  // Record database query
  recordDatabaseQuery(operation: string, table: string, duration: number): void {
    this.databaseQueryDuration.observe(
      { operation, table },
      duration / 1000
    );
  }

  // Record cache operation
  recordCacheOperation(operation: string, result: 'hit' | 'miss'): void {
    this.cacheOperations.inc({ operation, result });
  }

  // Get metrics for scraping
  getMetrics(): Promise<string> {
    return this.register.metrics();
  }

  // Get register for custom metrics
  getRegister(): client.Registry {
    return this.register;
  }
}

export const prometheusMonitoring = PrometheusMonitoring.getInstance();
```

## Alerting System

### Alert Manager

```typescript
// src/monitoring/alerts.ts
import { logger } from '../utils/logger';
import { errorTracking } from './sentry';

interface AlertRule {
  name: string;
  condition: (value: number, threshold: number) => boolean;
  threshold: number;
  severity: 'low' | 'medium' | 'high' | 'critical';
  cooldown: number; // minutes
}

interface AlertChannel {
  type: 'email' | 'slack' | 'webhook';
  config: Record<string, any>;
}

export class AlertManager {
  private static instance: AlertManager;
  private alertRules: Map<string, AlertRule> = new Map();
  private alertChannels: AlertChannel[] = [];
  private alertHistory: Map<string, number> = new Map();

  private constructor() {
    this.initializeAlertRules();
    this.initializeAlertChannels();
  }

  static getInstance(): AlertManager {
    if (!AlertManager.instance) {
      AlertManager.instance = new AlertManager();
    }
    return AlertManager.instance;
  }

  private initializeAlertRules(): void {
    // System alerts
    this.alertRules.set('high_cpu_usage', {
      name: 'High CPU Usage',
      condition: (value, threshold) => value > threshold,
      threshold: 80,
      severity: 'high',
      cooldown: 15
    });

    this.alertRules.set('high_memory_usage', {
      name: 'High Memory Usage',
      condition: (value, threshold) => value > threshold,
      threshold: 85,
      severity: 'high',
      cooldown: 15
    });

    this.alertRules.set('high_error_rate', {
      name: 'High Error Rate',
      condition: (value, threshold) => value > threshold,
      threshold: 5, // 5% error rate
      severity: 'critical',
      cooldown: 5
    });

    this.alertRules.set('slow_response_time', {
      name: 'Slow Response Time',
      condition: (value, threshold) => value > threshold,
      threshold: 2000, // 2 seconds
      severity: 'medium',
      cooldown: 10
    });

    // Business alerts
    this.alertRules.set('low_user_activity', {
      name: 'Low User Activity',
      condition: (value, threshold) => value < threshold,
      threshold: 10, // 10 active users
      severity: 'low',
      cooldown: 60
    });

    this.alertRules.set('payment_failures', {
      name: 'Payment Failures',
      condition: (value, threshold) => value > threshold,
      threshold: 3, // 3 failed payments
      severity: 'high',
      cooldown: 5
    });
  }

  private initializeAlertChannels(): void {
    // Email channel
    this.alertChannels.push({
      type: 'email',
      config: {
        recipients: ['alerts@dafnckmachine.com', 'devops@dafnckmachine.com']
      }
    });

    // Slack channel
    this.alertChannels.push({
      type: 'slack',
      config: {
        webhookUrl: process.env.SLACK_WEBHOOK_URL,
        channel: '#alerts'
      }
    });
  }

  // Check alert condition
  checkAlert(ruleName: string, value: number): void {
    const rule = this.alertRules.get(ruleName);
    if (!rule) return;

    const lastAlert = this.alertHistory.get(ruleName) || 0;
    const now = Date.now();
    const cooldownMs = rule.cooldown * 60 * 1000;

    // Check if we're still in cooldown period
    if (now - lastAlert < cooldownMs) return;

    // Check if alert condition is met
    if (rule.condition(value, rule.threshold)) {
      this.triggerAlert(rule, value);
      this.alertHistory.set(ruleName, now);
    }
  }

  private async triggerAlert(rule: AlertRule, value: number): Promise<void> {
    const alert = {
      name: rule.name,
      severity: rule.severity,
      value,
      threshold: rule.threshold,
      timestamp: new Date().toISOString(),
      environment: process.env.NODE_ENV
    };

    logger.error('Alert Triggered', alert);
    errorTracking.captureMessage(`Alert: ${rule.name}`, 'warning');

    // Send to all alert channels
    for (const channel of this.alertChannels) {
      await this.sendAlert(channel, alert);
    }
  }

  private async sendAlert(channel: AlertChannel, alert: any): Promise<void> {
    try {
      switch (channel.type) {
        case 'email':
          await this.sendEmailAlert(channel.config, alert);
          break;
        case 'slack':
          await this.sendSlackAlert(channel.config, alert);
          break;
        case 'webhook':
          await this.sendWebhookAlert(channel.config, alert);
          break;
      }
    } catch (error) {
      logger.error('Failed to send alert', { channel: channel.type, error });
    }
  }

  private async sendEmailAlert(config: any, alert: any): Promise<void> {
    // Implementation for email alerts
    // This would integrate with your email service
  }

  private async sendSlackAlert(config: any, alert: any): Promise<void> {
    if (!config.webhookUrl) return;

    const message = {
      text: `🚨 Alert: ${alert.name}`,
      attachments: [
        {
          color: this.getSeverityColor(alert.severity),
          fields: [
            { title: 'Severity', value: alert.severity, short: true },
            { title: 'Value', value: alert.value.toString(), short: true },
            { title: 'Threshold', value: alert.threshold.toString(), short: true },
            { title: 'Environment', value: alert.environment, short: true },
            { title: 'Time', value: alert.timestamp, short: false }
          ]
        }
      ]
    };

    // Send to Slack webhook
    // Implementation would use fetch or axios to send to webhook
  }

  private async sendWebhookAlert(config: any, alert: any): Promise<void> {
    // Implementation for webhook alerts
  }

  private getSeverityColor(severity: string): string {
    switch (severity) {
      case 'critical': return 'danger';
      case 'high': return 'warning';
      case 'medium': return '#ffaa00';
      case 'low': return 'good';
      default: return '#cccccc';
    }
  }
}

export const alertManager = AlertManager.getInstance();
```

## Dashboard Implementation

### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "id": null,
    "title": "DafnckMachine v3.1 - Application Monitoring",
    "tags": ["dafnckmachine", "application"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{route}}"
          }
        ],
        "yAxes": [
          {
            "label": "Requests/sec"
          }
        ]
      },
      {
        "id": 2,
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          },
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile"
          }
        ],
        "yAxes": [
          {
            "label": "Seconds"
          }
        ]
      },
      {
        "id": 3,
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m])",
            "legendFormat": "5xx errors"
          },
          {
            "expr": "rate(http_requests_total{status_code=~\"4..\"}[5m])",
            "legendFormat": "4xx errors"
          }
        ],
        "yAxes": [
          {
            "label": "Errors/sec"
          }
        ]
      },
      {
        "id": 4,
        "title": "Active Users",
        "type": "singlestat",
        "targets": [
          {
            "expr": "active_users_total",
            "legendFormat": "Active Users"
          }
        ]
      },
      {
        "id": 5,
        "title": "Database Query Performance",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(database_query_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ],
        "yAxes": [
          {
            "label": "Seconds"
          }
        ]
      },
      {
        "id": 6,
        "title": "Cache Hit Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(cache_operations_total{result=\"hit\"}[5m]) / rate(cache_operations_total[5m]) * 100",
            "legendFormat": "Hit Rate %"
          }
        ],
        "yAxes": [
          {
            "label": "Percentage"
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

### Custom Analytics Dashboard

```typescript
// src/analytics/dashboard.ts
import { metrics } from '../monitoring/metrics';
import { logger } from '../utils/logger';

export class AnalyticsDashboard {
  private static instance: AnalyticsDashboard;

  static getInstance(): AnalyticsDashboard {
    if (!AnalyticsDashboard.instance) {
      AnalyticsDashboard.instance = new AnalyticsDashboard();
    }
    return AnalyticsDashboard.instance;
  }

  // Get real-time metrics
  async getRealTimeMetrics(): Promise<any> {
    return {
      activeUsers: await this.getActiveUsers(),
      requestsPerMinute: await this.getRequestsPerMinute(),
      errorRate: await this.getErrorRate(),
      averageResponseTime: await this.getAverageResponseTime(),
      topPages: await this.getTopPages(),
      userActivity: await this.getUserActivity()
    };
  }

  // Get business metrics
  async getBusinessMetrics(timeRange: string = '24h'): Promise<any> {
    return {
      newUsers: await this.getNewUsers(timeRange),
      projectsCreated: await this.getProjectsCreated(timeRange),
      tasksCompleted: await this.getTasksCompleted(timeRange),
      revenue: await this.getRevenue(timeRange),
      conversionRate: await this.getConversionRate(timeRange)
    };
  }

  // Get system health
  async getSystemHealth(): Promise<any> {
    return {
      cpuUsage: await this.getCpuUsage(),
      memoryUsage: await this.getMemoryUsage(),
      diskUsage: await this.getDiskUsage(),
      databaseConnections: await this.getDatabaseConnections(),
      cacheHitRate: await this.getCacheHitRate()
    };
  }

  private async getActiveUsers(): Promise<number> {
    // Implementation to get active users from analytics
    return 0;
  }

  private async getRequestsPerMinute(): Promise<number> {
    // Implementation to get requests per minute
    return 0;
  }

  private async getErrorRate(): Promise<number> {
    // Implementation to get error rate
    return 0;
  }

  private async getAverageResponseTime(): Promise<number> {
    // Implementation to get average response time
    return 0;
  }

  private async getTopPages(): Promise<any[]> {
    // Implementation to get top pages
    return [];
  }

  private async getUserActivity(): Promise<any[]> {
    // Implementation to get user activity
    return [];
  }

  private async getNewUsers(timeRange: string): Promise<number> {
    // Implementation to get new users
    return 0;
  }

  private async getProjectsCreated(timeRange: string): Promise<number> {
    // Implementation to get projects created
    return 0;
  }

  private async getTasksCompleted(timeRange: string): Promise<number> {
    // Implementation to get tasks completed
    return 0;
  }

  private async getRevenue(timeRange: string): Promise<number> {
    // Implementation to get revenue
    return 0;
  }

  private async getConversionRate(timeRange: string): Promise<number> {
    // Implementation to get conversion rate
    return 0;
  }

  private async getCpuUsage(): Promise<number> {
    // Implementation to get CPU usage
    return 0;
  }

  private async getMemoryUsage(): Promise<number> {
    // Implementation to get memory usage
    return 0;
  }

  private async getDiskUsage(): Promise<number> {
    // Implementation to get disk usage
    return 0;
  }

  private async getDatabaseConnections(): Promise<number> {
    // Implementation to get database connections
    return 0;
  }

  private async getCacheHitRate(): Promise<number> {
    // Implementation to get cache hit rate
    return 0;
  }
}

export const analyticsDashboard = AnalyticsDashboard.getInstance();
```

## Related Documentation

- [CI/CD Pipeline Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/CICD_Pipeline_Configuration.md)
- [Infrastructure Architecture Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Infrastructure_Architecture_Implementation.md)
- [Environment Configuration Management](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Environment_Configuration_Management.md)
- [Performance Optimization Strategies](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Strategies.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: monitoring-analytics-agent
- **Related Workflows**: 17_Monitoring_Analytics.md 