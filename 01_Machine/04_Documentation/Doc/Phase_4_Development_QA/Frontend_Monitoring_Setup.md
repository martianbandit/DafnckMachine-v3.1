# Frontend Monitoring Setup - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Frontend Monitoring Setup Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: 12_Frontend_Development.md
- **Monitoring Framework**: Comprehensive Frontend Observability

## Executive Summary

This document provides comprehensive frontend monitoring setup guidelines for the DafnckMachine v3.1 application, covering performance monitoring, error tracking, user analytics, real user monitoring (RUM), and observability implementation strategies.

## Monitoring Architecture Overview

### Monitoring Layers
1. **Performance Monitoring**: Core Web Vitals, loading metrics, runtime performance
2. **Error Tracking**: JavaScript errors, unhandled exceptions, API failures
3. **User Analytics**: User behavior, feature usage, conversion tracking
4. **Real User Monitoring (RUM)**: Actual user experience metrics
5. **Synthetic Monitoring**: Automated testing and monitoring
6. **Infrastructure Monitoring**: CDN, hosting, and network performance

### Key Metrics to Track
- **Core Web Vitals**: LCP, FID, CLS
- **Performance**: TTFB, FCP, TTI, Speed Index
- **Reliability**: Error rates, crash rates, availability
- **User Experience**: Session duration, bounce rate, conversion rates
- **Business Metrics**: Feature adoption, user engagement

## Performance Monitoring Implementation

### Core Web Vitals Tracking

```javascript
// web-vitals.js
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  // Send to your analytics service
  gtag('event', metric.name, {
    event_category: 'Web Vitals',
    value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
    event_label: metric.id,
    non_interaction: true,
  });
}

// Measure and send Core Web Vitals
getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

### Performance Observer Setup

```javascript
// performance-observer.js
class PerformanceMonitor {
  constructor() {
    this.setupObservers();
  }

  setupObservers() {
    // Navigation timing
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.handlePerformanceEntry(entry);
        }
      });

      observer.observe({ entryTypes: ['navigation', 'resource', 'paint', 'largest-contentful-paint'] });
    }
  }

  handlePerformanceEntry(entry) {
    switch (entry.entryType) {
      case 'navigation':
        this.trackNavigationTiming(entry);
        break;
      case 'resource':
        this.trackResourceTiming(entry);
        break;
      case 'paint':
        this.trackPaintTiming(entry);
        break;
      case 'largest-contentful-paint':
        this.trackLCP(entry);
        break;
    }
  }

  trackNavigationTiming(entry) {
    const metrics = {
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      ssl: entry.secureConnectionStart > 0 ? entry.connectEnd - entry.secureConnectionStart : 0,
      ttfb: entry.responseStart - entry.requestStart,
      download: entry.responseEnd - entry.responseStart,
      domParse: entry.domContentLoadedEventStart - entry.responseEnd,
      domReady: entry.domContentLoadedEventEnd - entry.domContentLoadedEventStart,
      loadComplete: entry.loadEventEnd - entry.loadEventStart
    };

    this.sendMetrics('navigation', metrics);
  }

  trackResourceTiming(entry) {
    if (entry.duration > 100) { // Only track slow resources
      const resourceMetrics = {
        name: entry.name,
        duration: entry.duration,
        size: entry.transferSize,
        type: this.getResourceType(entry.name)
      };

      this.sendMetrics('resource', resourceMetrics);
    }
  }

  getResourceType(url) {
    if (url.includes('.js')) return 'script';
    if (url.includes('.css')) return 'stylesheet';
    if (url.match(/\.(png|jpg|jpeg|gif|svg|webp)$/)) return 'image';
    if (url.includes('/api/')) return 'api';
    return 'other';
  }

  sendMetrics(type, data) {
    // Send to monitoring service
    fetch('/api/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ type, data, timestamp: Date.now() })
    });
  }
}

// Initialize performance monitoring
const performanceMonitor = new PerformanceMonitor();
```

## Error Tracking Implementation

### Global Error Handler

```javascript
// error-tracking.js
class ErrorTracker {
  constructor(config) {
    this.config = config;
    this.setupGlobalHandlers();
  }

  setupGlobalHandlers() {
    // Catch JavaScript errors
    window.addEventListener('error', (event) => {
      this.trackError({
        type: 'javascript',
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
        timestamp: Date.now()
      });
    });

    // Catch unhandled promise rejections
    window.addEventListener('unhandledrejection', (event) => {
      this.trackError({
        type: 'promise',
        message: event.reason?.message || 'Unhandled promise rejection',
        stack: event.reason?.stack,
        timestamp: Date.now()
      });
    });

    // Catch React errors (if using React)
    this.setupReactErrorBoundary();
  }

  setupReactErrorBoundary() {
    // This would be implemented in your React error boundary
    window.reportReactError = (error, errorInfo) => {
      this.trackError({
        type: 'react',
        message: error.message,
        stack: error.stack,
        componentStack: errorInfo.componentStack,
        timestamp: Date.now()
      });
    };
  }

  trackError(errorData) {
    // Add user context
    const enrichedError = {
      ...errorData,
      userAgent: navigator.userAgent,
      url: window.location.href,
      userId: this.getCurrentUserId(),
      sessionId: this.getSessionId(),
      buildVersion: process.env.REACT_APP_VERSION
    };

    // Send to error tracking service
    this.sendToErrorService(enrichedError);
  }

  sendToErrorService(error) {
    // Send to Sentry, LogRocket, or custom service
    if (window.Sentry) {
      window.Sentry.captureException(error);
    } else {
      fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(error)
      });
    }
  }

  getCurrentUserId() {
    // Get current user ID from your auth system
    return localStorage.getItem('userId') || 'anonymous';
  }

  getSessionId() {
    // Get or create session ID
    let sessionId = sessionStorage.getItem('sessionId');
    if (!sessionId) {
      sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      sessionStorage.setItem('sessionId', sessionId);
    }
    return sessionId;
  }
}

// Initialize error tracking
const errorTracker = new ErrorTracker({
  apiEndpoint: '/api/errors',
  enableConsoleLogging: process.env.NODE_ENV === 'development'
});
```

### React Error Boundary

```jsx
// ErrorBoundary.jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo
    });

    // Report to error tracking service
    if (window.reportReactError) {
      window.reportReactError(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

## User Analytics Implementation

### Custom Analytics Tracker

```javascript
// analytics.js
class AnalyticsTracker {
  constructor(config) {
    this.config = config;
    this.queue = [];
    this.isInitialized = false;
    this.init();
  }

  async init() {
    // Initialize analytics services
    await this.loadGoogleAnalytics();
    await this.loadCustomAnalytics();
    this.isInitialized = true;
    this.flushQueue();
  }

  loadGoogleAnalytics() {
    return new Promise((resolve) => {
      if (this.config.googleAnalyticsId) {
        const script = document.createElement('script');
        script.src = `https://www.googletagmanager.com/gtag/js?id=${this.config.googleAnalyticsId}`;
        script.onload = () => {
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', this.config.googleAnalyticsId);
          resolve();
        };
        document.head.appendChild(script);
      } else {
        resolve();
      }
    });
  }

  loadCustomAnalytics() {
    // Load your custom analytics service
    return Promise.resolve();
  }

  track(event, properties = {}) {
    const eventData = {
      event,
      properties: {
        ...properties,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
        sessionId: this.getSessionId()
      }
    };

    if (this.isInitialized) {
      this.sendEvent(eventData);
    } else {
      this.queue.push(eventData);
    }
  }

  sendEvent(eventData) {
    // Send to Google Analytics
    if (window.gtag) {
      gtag('event', eventData.event, eventData.properties);
    }

    // Send to custom analytics
    fetch('/api/analytics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(eventData)
    });
  }

  flushQueue() {
    while (this.queue.length > 0) {
      const event = this.queue.shift();
      this.sendEvent(event);
    }
  }

  // Convenience methods
  trackPageView(page) {
    this.track('page_view', { page });
  }

  trackUserAction(action, element) {
    this.track('user_action', { action, element });
  }

  trackFeatureUsage(feature, context) {
    this.track('feature_usage', { feature, context });
  }

  trackConversion(type, value) {
    this.track('conversion', { type, value });
  }

  getSessionId() {
    let sessionId = sessionStorage.getItem('sessionId');
    if (!sessionId) {
      sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      sessionStorage.setItem('sessionId', sessionId);
    }
    return sessionId;
  }
}

// Initialize analytics
const analytics = new AnalyticsTracker({
  googleAnalyticsId: process.env.REACT_APP_GA_ID,
  customEndpoint: '/api/analytics'
});

export default analytics;
```

### React Hook for Analytics

```jsx
// useAnalytics.js
import { useEffect, useCallback } from 'react';
import analytics from './analytics';

export const useAnalytics = () => {
  const trackEvent = useCallback((event, properties) => {
    analytics.track(event, properties);
  }, []);

  const trackPageView = useCallback((page) => {
    analytics.trackPageView(page);
  }, []);

  const trackUserAction = useCallback((action, element) => {
    analytics.trackUserAction(action, element);
  }, []);

  const trackFeatureUsage = useCallback((feature, context) => {
    analytics.trackFeatureUsage(feature, context);
  }, []);

  return {
    trackEvent,
    trackPageView,
    trackUserAction,
    trackFeatureUsage
  };
};

// Hook for automatic page view tracking
export const usePageTracking = () => {
  const { trackPageView } = useAnalytics();

  useEffect(() => {
    trackPageView(window.location.pathname);
  }, [trackPageView]);
};
```

## Real User Monitoring (RUM)

### RUM Data Collection

```javascript
// rum-collector.js
class RUMCollector {
  constructor() {
    this.data = {
      session: this.generateSessionId(),
      user: this.getUserInfo(),
      device: this.getDeviceInfo(),
      performance: {},
      interactions: [],
      errors: []
    };

    this.startCollection();
  }

  startCollection() {
    this.collectPerformanceData();
    this.trackUserInteractions();
    this.monitorNetworkConditions();
    this.trackVisibilityChanges();
  }

  collectPerformanceData() {
    // Collect navigation timing
    if (performance.navigation) {
      this.data.performance.navigation = {
        type: performance.navigation.type,
        redirectCount: performance.navigation.redirectCount
      };
    }

    // Collect resource timing
    if (performance.getEntriesByType) {
      this.data.performance.resources = performance.getEntriesByType('resource')
        .map(entry => ({
          name: entry.name,
          duration: entry.duration,
          size: entry.transferSize,
          type: this.getResourceType(entry.name)
        }));
    }

    // Collect memory info (Chrome only)
    if (performance.memory) {
      this.data.performance.memory = {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit
      };
    }
  }

  trackUserInteractions() {
    ['click', 'scroll', 'keydown', 'touchstart'].forEach(eventType => {
      document.addEventListener(eventType, (event) => {
        this.recordInteraction(eventType, event);
      }, { passive: true });
    });
  }

  recordInteraction(type, event) {
    const interaction = {
      type,
      timestamp: Date.now(),
      target: event.target.tagName,
      x: event.clientX,
      y: event.clientY
    };

    this.data.interactions.push(interaction);

    // Keep only last 100 interactions
    if (this.data.interactions.length > 100) {
      this.data.interactions.shift();
    }
  }

  monitorNetworkConditions() {
    if ('connection' in navigator) {
      this.data.network = {
        effectiveType: navigator.connection.effectiveType,
        downlink: navigator.connection.downlink,
        rtt: navigator.connection.rtt,
        saveData: navigator.connection.saveData
      };

      navigator.connection.addEventListener('change', () => {
        this.data.network = {
          effectiveType: navigator.connection.effectiveType,
          downlink: navigator.connection.downlink,
          rtt: navigator.connection.rtt,
          saveData: navigator.connection.saveData
        };
      });
    }
  }

  trackVisibilityChanges() {
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.sendRUMData();
      }
    });

    window.addEventListener('beforeunload', () => {
      this.sendRUMData();
    });
  }

  sendRUMData() {
    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/rum', JSON.stringify(this.data));
    } else {
      fetch('/api/rum', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.data),
        keepalive: true
      });
    }
  }

  generateSessionId() {
    return 'rum_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }

  getUserInfo() {
    return {
      id: localStorage.getItem('userId') || 'anonymous',
      agent: navigator.userAgent,
      language: navigator.language,
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
    };
  }

  getDeviceInfo() {
    return {
      screen: {
        width: screen.width,
        height: screen.height,
        colorDepth: screen.colorDepth
      },
      viewport: {
        width: window.innerWidth,
        height: window.innerHeight
      },
      pixelRatio: window.devicePixelRatio || 1
    };
  }

  getResourceType(url) {
    if (url.includes('.js')) return 'script';
    if (url.includes('.css')) return 'stylesheet';
    if (url.match(/\.(png|jpg|jpeg|gif|svg|webp)$/)) return 'image';
    if (url.includes('/api/')) return 'api';
    return 'other';
  }
}

// Initialize RUM collection
const rumCollector = new RUMCollector();
```

## Monitoring Service Integration

### Sentry Integration

```javascript
// sentry-config.js
import * as Sentry from '@sentry/react';
import { Integrations } from '@sentry/tracing';

Sentry.init({
  dsn: process.env.REACT_APP_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [
    new Integrations.BrowserTracing({
      tracingOrigins: ['localhost', process.env.REACT_APP_API_URL],
    }),
  ],
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  beforeSend(event) {
    // Filter out development errors
    if (process.env.NODE_ENV === 'development') {
      console.log('Sentry event:', event);
    }
    return event;
  },
});

export default Sentry;
```

### LogRocket Integration

```javascript
// logrocket-config.js
import LogRocket from 'logrocket';
import setupLogRocketReact from 'logrocket-react';

if (process.env.NODE_ENV === 'production') {
  LogRocket.init(process.env.REACT_APP_LOGROCKET_ID);
  setupLogRocketReact(LogRocket);

  // Identify users
  LogRocket.identify('user-id', {
    name: 'User Name',
    email: 'user@example.com',
  });
}

export default LogRocket;
```

## Monitoring Dashboard Setup

### Custom Dashboard Component

```jsx
// MonitoringDashboard.jsx
import React, { useState, useEffect } from 'react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

const MonitoringDashboard = () => {
  const [metrics, setMetrics] = useState({
    performance: [],
    errors: [],
    users: []
  });

  useEffect(() => {
    fetchMetrics();
    const interval = setInterval(fetchMetrics, 30000); // Update every 30 seconds
    return () => clearInterval(interval);
  }, []);

  const fetchMetrics = async () => {
    try {
      const response = await fetch('/api/monitoring/dashboard');
      const data = await response.json();
      setMetrics(data);
    } catch (error) {
      console.error('Failed to fetch metrics:', error);
    }
  };

  return (
    <div className="monitoring-dashboard">
      <h1>Frontend Monitoring Dashboard</h1>
      
      <div className="metrics-grid">
        <div className="metric-card">
          <h3>Performance Metrics</h3>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={metrics.performance}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="timestamp" />
              <YAxis />
              <Tooltip />
              <Line type="monotone" dataKey="lcp" stroke="#8884d8" name="LCP" />
              <Line type="monotone" dataKey="fid" stroke="#82ca9d" name="FID" />
              <Line type="monotone" dataKey="cls" stroke="#ffc658" name="CLS" />
            </LineChart>
          </ResponsiveContainer>
        </div>

        <div className="metric-card">
          <h3>Error Rates</h3>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={metrics.errors}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="timestamp" />
              <YAxis />
              <Tooltip />
              <Line type="monotone" dataKey="errorRate" stroke="#ff7300" name="Error Rate" />
            </LineChart>
          </ResponsiveContainer>
        </div>

        <div className="metric-card">
          <h3>Active Users</h3>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={metrics.users}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="timestamp" />
              <YAxis />
              <Tooltip />
              <Line type="monotone" dataKey="activeUsers" stroke="#387908" name="Active Users" />
            </LineChart>
          </ResponsiveContainer>
        </div>
      </div>
    </div>
  );
};

export default MonitoringDashboard;
```

## Alerting and Notifications

### Alert Configuration

```javascript
// alert-config.js
const alertConfig = {
  performance: {
    lcp: { threshold: 2500, severity: 'warning' },
    fid: { threshold: 100, severity: 'warning' },
    cls: { threshold: 0.1, severity: 'warning' },
    ttfb: { threshold: 600, severity: 'critical' }
  },
  errors: {
    errorRate: { threshold: 0.05, severity: 'critical' },
    jsErrors: { threshold: 10, severity: 'warning' }
  },
  availability: {
    uptime: { threshold: 0.99, severity: 'critical' }
  }
};

class AlertManager {
  constructor(config) {
    this.config = config;
    this.alertHistory = new Map();
  }

  checkMetric(type, metric, value) {
    const threshold = this.config[type]?.[metric];
    if (!threshold) return;

    const alertKey = `${type}_${metric}`;
    const isAlert = value > threshold.threshold;
    const lastAlert = this.alertHistory.get(alertKey);

    if (isAlert && (!lastAlert || Date.now() - lastAlert > 300000)) { // 5 min cooldown
      this.sendAlert(type, metric, value, threshold);
      this.alertHistory.set(alertKey, Date.now());
    }
  }

  sendAlert(type, metric, value, threshold) {
    const alert = {
      type,
      metric,
      value,
      threshold: threshold.threshold,
      severity: threshold.severity,
      timestamp: Date.now()
    };

    // Send to alerting service
    fetch('/api/alerts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(alert)
    });

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.warn('Alert triggered:', alert);
    }
  }
}

const alertManager = new AlertManager(alertConfig);
export default alertManager;
```

## Implementation Checklist

### Phase 1: Basic Monitoring
- [ ] Implement Core Web Vitals tracking
- [ ] Set up basic error tracking
- [ ] Configure Google Analytics
- [ ] Implement performance observer
- [ ] Set up basic alerting

### Phase 2: Advanced Monitoring
- [ ] Implement RUM collection
- [ ] Set up custom analytics
- [ ] Configure Sentry integration
- [ ] Implement user session tracking
- [ ] Set up monitoring dashboard

### Phase 3: Optimization
- [ ] Implement LogRocket for session replay
- [ ] Set up synthetic monitoring
- [ ] Configure advanced alerting
- [ ] Implement A/B testing tracking
- [ ] Set up business metrics tracking

### Phase 4: Analysis & Reporting
- [ ] Create automated reports
- [ ] Set up performance budgets
- [ ] Implement trend analysis
- [ ] Configure stakeholder dashboards
- [ ] Set up data export capabilities

## Best Practices

### Data Privacy
- Implement user consent management
- Anonymize sensitive data
- Comply with GDPR/CCPA requirements
- Provide opt-out mechanisms

### Performance Impact
- Use sampling for high-volume events
- Implement client-side buffering
- Use beacon API for reliable data sending
- Minimize monitoring overhead

### Data Quality
- Validate data before sending
- Implement data deduplication
- Handle network failures gracefully
- Maintain data consistency

### Security
- Encrypt sensitive monitoring data
- Implement proper authentication
- Use HTTPS for all monitoring endpoints
- Sanitize user input in error reports

## Troubleshooting Guide

### Common Issues
1. **High monitoring overhead**: Implement sampling and buffering
2. **Missing data**: Check network connectivity and API endpoints
3. **Inaccurate metrics**: Verify timing calculations and data collection
4. **Alert fatigue**: Adjust thresholds and implement cooldowns

### Debugging Tools
- Browser DevTools Performance tab
- React DevTools Profiler
- Network tab for monitoring requests
- Console for error tracking

## Conclusion

This comprehensive frontend monitoring setup provides visibility into application performance, user experience, and system reliability. Regular review and optimization of monitoring configurations ensure continued effectiveness and actionable insights for the DafnckMachine v3.1 application.