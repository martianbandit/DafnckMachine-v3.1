# Monitoring Integration Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Performance_Optimization_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Guide.md)

## Overview

This guide provides comprehensive monitoring integration strategies for DafnckMachine v3.1, covering performance monitoring, error tracking, user analytics, real user monitoring, and observability implementation to ensure optimal application performance and user experience.

## Table of Contents

1. [Monitoring Strategy](#monitoring-strategy)
2. [Performance Monitoring](#performance-monitoring)
3. [Error Tracking](#error-tracking)
4. [User Analytics](#user-analytics)
5. [Real User Monitoring](#real-user-monitoring)
6. [Observability Implementation](#observability-implementation)

## Monitoring Strategy

### Monitoring Architecture

```typescript
// Monitoring configuration
export const monitoringConfig = {
  // Performance monitoring
  performance: {
    enabled: true,
    provider: 'web-vitals',
    thresholds: {
      lcp: 2500,
      fid: 100,
      cls: 0.1,
      fcp: 1800,
      ttfb: 600
    }
  },
  
  // Error tracking
  errorTracking: {
    enabled: true,
    provider: 'sentry',
    sampleRate: 1.0,
    environment: process.env.NODE_ENV
  },
  
  // User analytics
  analytics: {
    enabled: true,
    providers: ['google-analytics', 'mixpanel'],
    trackingId: process.env.REACT_APP_GA_TRACKING_ID
  },
  
  // Real user monitoring
  rum: {
    enabled: true,
    provider: 'datadog',
    sampleRate: 0.1
  }
} as const;
```

## Performance Monitoring

### Web Vitals Integration

```typescript
// src/utils/performance.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

interface VitalMetric {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
  delta: number;
  id: string;
}

class PerformanceMonitor {
  private metrics: Map<string, VitalMetric> = new Map();
  
  constructor() {
    this.initializeWebVitals();
  }
  
  private initializeWebVitals() {
    getCLS(this.handleMetric.bind(this));
    getFID(this.handleMetric.bind(this));
    getFCP(this.handleMetric.bind(this));
    getLCP(this.handleMetric.bind(this));
    getTTFB(this.handleMetric.bind(this));
  }
  
  private handleMetric(metric: VitalMetric) {
    this.metrics.set(metric.name, metric);
    this.reportMetric(metric);
  }
  
  private reportMetric(metric: VitalMetric) {
    // Send to analytics
    if (window.gtag) {
      window.gtag('event', metric.name, {
        event_category: 'Web Vitals',
        value: Math.round(metric.value),
        metric_rating: metric.rating,
        custom_map: { metric_id: metric.id }
      });
    }
    
    // Send to monitoring service
    this.sendToMonitoringService(metric);
  }
  
  private async sendToMonitoringService(metric: VitalMetric) {
    try {
      await fetch('/api/metrics', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          name: metric.name,
          value: metric.value,
          rating: metric.rating,
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent
        })
      });
    } catch (error) {
      console.error('Failed to send metric:', error);
    }
  }
  
  getMetrics(): VitalMetric[] {
    return Array.from(this.metrics.values());
  }
  
  getMetric(name: string): VitalMetric | undefined {
    return this.metrics.get(name);
  }
}

export const performanceMonitor = new PerformanceMonitor();
```

### Custom Performance Tracking

```typescript
// src/utils/customMetrics.ts
class CustomMetricsTracker {
  private startTimes: Map<string, number> = new Map();
  
  startTiming(label: string): void {
    this.startTimes.set(label, performance.now());
  }
  
  endTiming(label: string): number | null {
    const startTime = this.startTimes.get(label);
    if (!startTime) return null;
    
    const duration = performance.now() - startTime;
    this.startTimes.delete(label);
    
    this.reportCustomMetric(label, duration);
    return duration;
  }
  
  private reportCustomMetric(label: string, duration: number): void {
    // Send to analytics
    if (window.gtag) {
      window.gtag('event', 'timing_complete', {
        name: label,
        value: Math.round(duration)
      });
    }
    
    // Send to monitoring service
    this.sendCustomMetric(label, duration);
  }
  
  private async sendCustomMetric(label: string, duration: number): Promise<void> {
    try {
      await fetch('/api/custom-metrics', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          metric: label,
          duration,
          timestamp: Date.now(),
          url: window.location.href
        })
      });
    } catch (error) {
      console.error('Failed to send custom metric:', error);
    }
  }
  
  measureAsyncOperation<T>(
    label: string,
    operation: () => Promise<T>
  ): Promise<T> {
    this.startTiming(label);
    return operation().finally(() => {
      this.endTiming(label);
    });
  }
}

export const customMetrics = new CustomMetricsTracker();
```

## Error Tracking

### Sentry Integration

```typescript
// src/utils/errorTracking.ts
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

export const initializeErrorTracking = () => {
  Sentry.init({
    dsn: process.env.REACT_APP_SENTRY_DSN,
    environment: process.env.NODE_ENV,
    integrations: [
      new BrowserTracing({
        tracePropagationTargets: ['localhost', /^https:\/\/api\.dafnckmachine\.com/],
      }),
    ],
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
    beforeSend(event) {
      // Filter out development errors
      if (process.env.NODE_ENV === 'development') {
        return null;
      }
      return event;
    },
    beforeBreadcrumb(breadcrumb) {
      // Filter sensitive data from breadcrumbs
      if (breadcrumb.category === 'console' && breadcrumb.level === 'error') {
        return null;
      }
      return breadcrumb;
    }
  });
};

export const captureError = (error: Error, context?: Record<string, any>) => {
  Sentry.withScope((scope) => {
    if (context) {
      scope.setContext('additional_info', context);
    }
    Sentry.captureException(error);
  });
};

export const captureMessage = (message: string, level: Sentry.SeverityLevel = 'info') => {
  Sentry.captureMessage(message, level);
};
```

### Error Boundary Implementation

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import * as Sentry from '@sentry/react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    Sentry.withScope((scope) => {
      scope.setTag('errorBoundary', true);
      scope.setContext('errorInfo', errorInfo);
      Sentry.captureException(error);
    });
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>We've been notified of this error and are working to fix it.</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## User Analytics

### Google Analytics 4 Integration

```typescript
// src/utils/analytics.ts
declare global {
  interface Window {
    gtag: (...args: any[]) => void;
    dataLayer: any[];
  }
}

class AnalyticsService {
  private initialized = false;
  
  initialize(trackingId: string) {
    if (this.initialized || !trackingId) return;
    
    // Load Google Analytics
    const script = document.createElement('script');
    script.async = true;
    script.src = `https://www.googletagmanager.com/gtag/js?id=${trackingId}`;
    document.head.appendChild(script);
    
    window.dataLayer = window.dataLayer || [];
    window.gtag = function() {
      window.dataLayer.push(arguments);
    };
    
    window.gtag('js', new Date());
    window.gtag('config', trackingId, {
      page_title: document.title,
      page_location: window.location.href
    });
    
    this.initialized = true;
  }
  
  trackPageView(path: string, title?: string) {
    if (!this.initialized) return;
    
    window.gtag('config', process.env.REACT_APP_GA_TRACKING_ID, {
      page_path: path,
      page_title: title || document.title
    });
  }
  
  trackEvent(action: string, category: string, label?: string, value?: number) {
    if (!this.initialized) return;
    
    window.gtag('event', action, {
      event_category: category,
      event_label: label,
      value: value
    });
  }
  
  trackUserInteraction(element: string, action: string) {
    this.trackEvent(action, 'User Interaction', element);
  }
  
  trackConversion(conversionId: string, value?: number) {
    if (!this.initialized) return;
    
    window.gtag('event', 'conversion', {
      send_to: conversionId,
      value: value
    });
  }
  
  setUserProperties(properties: Record<string, any>) {
    if (!this.initialized) return;
    
    window.gtag('config', process.env.REACT_APP_GA_TRACKING_ID, {
      custom_map: properties
    });
  }
}

export const analytics = new AnalyticsService();
```

### User Behavior Tracking

```typescript
// src/hooks/useAnalytics.ts
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { analytics } from '../utils/analytics';

export const usePageTracking = () => {
  const location = useLocation();
  
  useEffect(() => {
    analytics.trackPageView(location.pathname + location.search);
  }, [location]);
};

export const useUserInteractionTracking = () => {
  const trackClick = (element: string, additionalData?: Record<string, any>) => {
    analytics.trackEvent('click', 'User Interaction', element);
    
    if (additionalData) {
      analytics.trackEvent('interaction_data', 'User Interaction', JSON.stringify(additionalData));
    }
  };
  
  const trackFormSubmission = (formName: string, success: boolean) => {
    analytics.trackEvent(
      success ? 'form_submit_success' : 'form_submit_error',
      'Form Interaction',
      formName
    );
  };
  
  const trackSearch = (query: string, resultsCount: number) => {
    analytics.trackEvent('search', 'Search', query, resultsCount);
  };
  
  return {
    trackClick,
    trackFormSubmission,
    trackSearch
  };
};
```

## Real User Monitoring

### Performance Observer Implementation

```typescript
// src/utils/realUserMonitoring.ts
class RealUserMonitoring {
  private observer: PerformanceObserver | null = null;
  
  constructor() {
    this.initializeObserver();
  }
  
  private initializeObserver() {
    if (!('PerformanceObserver' in window)) return;
    
    this.observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        this.processEntry(entry);
      });
    });
    
    // Observe different types of performance entries
    try {
      this.observer.observe({ entryTypes: ['navigation', 'resource', 'measure', 'paint'] });
    } catch (error) {
      console.warn('Performance Observer not fully supported:', error);
    }
  }
  
  private processEntry(entry: PerformanceEntry) {
    switch (entry.entryType) {
      case 'navigation':
        this.handleNavigationEntry(entry as PerformanceNavigationTiming);
        break;
      case 'resource':
        this.handleResourceEntry(entry as PerformanceResourceTiming);
        break;
      case 'paint':
        this.handlePaintEntry(entry);
        break;
      case 'measure':
        this.handleMeasureEntry(entry);
        break;
    }
  }
  
  private handleNavigationEntry(entry: PerformanceNavigationTiming) {
    const metrics = {
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      ssl: entry.connectEnd - entry.secureConnectionStart,
      ttfb: entry.responseStart - entry.requestStart,
      download: entry.responseEnd - entry.responseStart,
      domInteractive: entry.domInteractive - entry.navigationStart,
      domComplete: entry.domComplete - entry.navigationStart,
      loadComplete: entry.loadEventEnd - entry.navigationStart
    };
    
    this.sendRUMData('navigation', metrics);
  }
  
  private handleResourceEntry(entry: PerformanceResourceTiming) {
    // Track slow resources
    if (entry.duration > 1000) {
      this.sendRUMData('slow_resource', {
        name: entry.name,
        duration: entry.duration,
        size: entry.transferSize,
        type: this.getResourceType(entry.name)
      });
    }
  }
  
  private handlePaintEntry(entry: PerformanceEntry) {
    this.sendRUMData('paint', {
      name: entry.name,
      startTime: entry.startTime
    });
  }
  
  private handleMeasureEntry(entry: PerformanceEntry) {
    this.sendRUMData('measure', {
      name: entry.name,
      duration: entry.duration,
      startTime: entry.startTime
    });
  }
  
  private getResourceType(url: string): string {
    if (url.includes('.js')) return 'script';
    if (url.includes('.css')) return 'stylesheet';
    if (url.match(/\.(png|jpg|jpeg|gif|svg|webp)$/)) return 'image';
    if (url.includes('/api/')) return 'api';
    return 'other';
  }
  
  private async sendRUMData(type: string, data: any) {
    try {
      await fetch('/api/rum', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          type,
          data,
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection?.effectiveType
        })
      });
    } catch (error) {
      console.error('Failed to send RUM data:', error);
    }
  }
  
  disconnect() {
    if (this.observer) {
      this.observer.disconnect();
    }
  }
}

export const rum = new RealUserMonitoring();
```

## Observability Implementation

### Logging Service

```typescript
// src/utils/logging.ts
enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3
}

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: number;
  context?: Record<string, any>;
  stack?: string;
}

class LoggingService {
  private logs: LogEntry[] = [];
  private maxLogs = 1000;
  private logLevel = LogLevel.INFO;
  
  constructor() {
    this.logLevel = process.env.NODE_ENV === 'development' ? LogLevel.DEBUG : LogLevel.INFO;
  }
  
  debug(message: string, context?: Record<string, any>) {
    this.log(LogLevel.DEBUG, message, context);
  }
  
  info(message: string, context?: Record<string, any>) {
    this.log(LogLevel.INFO, message, context);
  }
  
  warn(message: string, context?: Record<string, any>) {
    this.log(LogLevel.WARN, message, context);
  }
  
  error(message: string, error?: Error, context?: Record<string, any>) {
    this.log(LogLevel.ERROR, message, {
      ...context,
      error: error?.message,
      stack: error?.stack
    });
  }
  
  private log(level: LogLevel, message: string, context?: Record<string, any>) {
    if (level < this.logLevel) return;
    
    const entry: LogEntry = {
      level,
      message,
      timestamp: Date.now(),
      context,
      stack: level === LogLevel.ERROR ? new Error().stack : undefined
    };
    
    this.logs.push(entry);
    
    // Maintain log size limit
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
    
    // Console output
    this.outputToConsole(entry);
    
    // Send to remote logging service
    if (level >= LogLevel.WARN) {
      this.sendToRemoteLogger(entry);
    }
  }
  
  private outputToConsole(entry: LogEntry) {
    const timestamp = new Date(entry.timestamp).toISOString();
    const message = `[${timestamp}] ${LogLevel[entry.level]}: ${entry.message}`;
    
    switch (entry.level) {
      case LogLevel.DEBUG:
        console.debug(message, entry.context);
        break;
      case LogLevel.INFO:
        console.info(message, entry.context);
        break;
      case LogLevel.WARN:
        console.warn(message, entry.context);
        break;
      case LogLevel.ERROR:
        console.error(message, entry.context);
        break;
    }
  }
  
  private async sendToRemoteLogger(entry: LogEntry) {
    try {
      await fetch('/api/logs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(entry)
      });
    } catch (error) {
      console.error('Failed to send log to remote service:', error);
    }
  }
  
  getLogs(level?: LogLevel): LogEntry[] {
    if (level !== undefined) {
      return this.logs.filter(log => log.level >= level);
    }
    return [...this.logs];
  }
  
  clearLogs() {
    this.logs = [];
  }
}

export const logger = new LoggingService();
```

### Health Check Implementation

```typescript
// src/utils/healthCheck.ts
interface HealthStatus {
  status: 'healthy' | 'degraded' | 'unhealthy';
  checks: Record<string, boolean>;
  timestamp: number;
}

class HealthCheckService {
  private checks: Map<string, () => Promise<boolean>> = new Map();
  private lastStatus: HealthStatus | null = null;
  
  registerCheck(name: string, checkFunction: () => Promise<boolean>) {
    this.checks.set(name, checkFunction);
  }
  
  async runHealthChecks(): Promise<HealthStatus> {
    const checks: Record<string, boolean> = {};
    
    for (const [name, checkFn] of this.checks) {
      try {
        checks[name] = await checkFn();
      } catch (error) {
        checks[name] = false;
        logger.error(`Health check failed: ${name}`, error);
      }
    }
    
    const failedChecks = Object.values(checks).filter(result => !result).length;
    const totalChecks = Object.keys(checks).length;
    
    let status: HealthStatus['status'];
    if (failedChecks === 0) {
      status = 'healthy';
    } else if (failedChecks < totalChecks / 2) {
      status = 'degraded';
    } else {
      status = 'unhealthy';
    }
    
    const healthStatus: HealthStatus = {
      status,
      checks,
      timestamp: Date.now()
    };
    
    this.lastStatus = healthStatus;
    this.reportHealthStatus(healthStatus);
    
    return healthStatus;
  }
  
  private async reportHealthStatus(status: HealthStatus) {
    try {
      await fetch('/api/health', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(status)
      });
    } catch (error) {
      logger.error('Failed to report health status', error);
    }
  }
  
  getLastStatus(): HealthStatus | null {
    return this.lastStatus;
  }
}

export const healthCheck = new HealthCheckService();

// Register default health checks
healthCheck.registerCheck('api', async () => {
  try {
    const response = await fetch('/api/health');
    return response.ok;
  } catch {
    return false;
  }
});

healthCheck.registerCheck('localStorage', async () => {
  try {
    localStorage.setItem('health-check', 'test');
    localStorage.removeItem('health-check');
    return true;
  } catch {
    return false;
  }
});
```

---

## Summary

This Monitoring Integration Guide provides comprehensive monitoring strategies for DafnckMachine v3.1, ensuring optimal application performance and user experience through performance monitoring, error tracking, user analytics, real user monitoring, and observability implementation.

Key monitoring areas include:

1. **Performance Monitoring**: Web Vitals and custom metrics tracking
2. **Error Tracking**: Sentry integration and error boundary implementation
3. **User Analytics**: Google Analytics 4 and user behavior tracking
4. **Real User Monitoring**: Performance observer and RUM data collection
5. **Observability**: Logging service and health check implementation

The guide ensures comprehensive monitoring coverage while maintaining performance and providing actionable insights for continuous improvement. 