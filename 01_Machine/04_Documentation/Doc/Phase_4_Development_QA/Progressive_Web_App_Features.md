# Progressive Web App Features - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Progressive Web App Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive guidance for implementing Progressive Web App (PWA) features in DafnckMachine v3.1, enabling native-like experiences across all platforms with offline capabilities, push notifications, and enhanced performance.

## PWA Core Principles

### 1. Progressive Enhancement
- **Base Functionality**: Core features work on all browsers
- **Enhanced Features**: Advanced capabilities for supporting browsers
- **Graceful Degradation**: Fallbacks for unsupported features
- **Feature Detection**: Runtime capability checking

### 2. Responsive Design
- **Mobile-First**: Optimized for mobile devices
- **Adaptive Layouts**: Responsive across all screen sizes
- **Touch-Friendly**: Optimized for touch interactions
- **Cross-Platform**: Consistent experience across devices

### 3. Connectivity Independence
- **Offline Support**: Core functionality without network
- **Background Sync**: Data synchronization when online
- **Cache Strategies**: Intelligent resource caching
- **Network Resilience**: Graceful handling of poor connections

## Service Worker Implementation

### Service Worker Registration

```typescript
// src/serviceWorker/registration.ts
export interface ServiceWorkerConfig {
  onSuccess?: (registration: ServiceWorkerRegistration) => void;
  onUpdate?: (registration: ServiceWorkerRegistration) => void;
  onOfflineReady?: () => void;
}

export const registerServiceWorker = async (config?: ServiceWorkerConfig) => {
  if ('serviceWorker' in navigator) {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/'
      });

      registration.addEventListener('updatefound', () => {
        const installingWorker = registration.installing;
        if (installingWorker) {
          installingWorker.addEventListener('statechange', () => {
            if (installingWorker.state === 'installed') {
              if (navigator.serviceWorker.controller) {
                config?.onUpdate?.(registration);
              } else {
                config?.onOfflineReady?.();
                config?.onSuccess?.(registration);
              }
            }
          });
        }
      });

      return registration;
    } catch (error) {
      console.error('Service Worker registration failed:', error);
      throw error;
    }
  }
};
```

### Service Worker Core Logic

```typescript
// public/sw.js
const CACHE_NAME = 'dafnck-machine-v3.1.0';
const STATIC_CACHE = 'static-v1';
const DYNAMIC_CACHE = 'dynamic-v1';

const STATIC_ASSETS = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json',
  '/offline.html'
];

// Install Event
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE)
      .then(cache => cache.addAll(STATIC_ASSETS))
      .then(() => self.skipWaiting())
  );
});

// Activate Event
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then(cacheNames => {
        return Promise.all(
          cacheNames
            .filter(cacheName => cacheName !== STATIC_CACHE && cacheName !== DYNAMIC_CACHE)
            .map(cacheName => caches.delete(cacheName))
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch Event with Cache Strategies
self.addEventListener('fetch', (event) => {
  const { request } = event;
  
  // API requests - Network First
  if (request.url.includes('/api/')) {
    event.respondWith(networkFirst(request));
  }
  // Static assets - Cache First
  else if (request.destination === 'script' || request.destination === 'style') {
    event.respondWith(cacheFirst(request));
  }
  // Pages - Stale While Revalidate
  else if (request.mode === 'navigate') {
    event.respondWith(staleWhileRevalidate(request));
  }
  // Default - Network First
  else {
    event.respondWith(networkFirst(request));
  }
});
```

### Cache Strategies

```typescript
// Cache Strategy Implementations
async function cacheFirst(request) {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    return cachedResponse;
  }
  
  try {
    const networkResponse = await fetch(request);
    const cache = await caches.open(DYNAMIC_CACHE);
    cache.put(request, networkResponse.clone());
    return networkResponse;
  } catch (error) {
    return caches.match('/offline.html');
  }
}

async function networkFirst(request) {
  try {
    const networkResponse = await fetch(request);
    const cache = await caches.open(DYNAMIC_CACHE);
    cache.put(request, networkResponse.clone());
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    return cachedResponse || caches.match('/offline.html');
  }
}

async function staleWhileRevalidate(request) {
  const cachedResponse = await caches.match(request);
  
  const fetchPromise = fetch(request).then(networkResponse => {
    const cache = caches.open(DYNAMIC_CACHE);
    cache.then(c => c.put(request, networkResponse.clone()));
    return networkResponse;
  });
  
  return cachedResponse || fetchPromise;
}
```

## Web App Manifest

### Manifest Configuration

```json
{
  "name": "DafnckMachine v3.1",
  "short_name": "DafnckMachine",
  "description": "Advanced AI Agent System for Development Workflows",
  "start_url": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#1976d2",
  "background_color": "#ffffff",
  "scope": "/",
  "lang": "en-US",
  "dir": "ltr",
  "categories": ["productivity", "development", "business"],
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "shortcuts": [
    {
      "name": "Dashboard",
      "short_name": "Dashboard",
      "description": "Access main dashboard",
      "url": "/dashboard",
      "icons": [
        {
          "src": "/icons/dashboard-96x96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Tasks",
      "short_name": "Tasks",
      "description": "Manage tasks",
      "url": "/tasks",
      "icons": [
        {
          "src": "/icons/tasks-96x96.png",
          "sizes": "96x96"
        }
      ]
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/desktop-1280x720.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/mobile-375x667.png",
      "sizes": "375x667",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ]
}
```

## Offline Functionality

### Offline Detection

```typescript
// src/hooks/useOnlineStatus.ts
import { useState, useEffect } from 'react';

export const useOnlineStatus = () => {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
};
```

### Offline Storage

```typescript
// src/utils/offlineStorage.ts
interface OfflineData {
  id: string;
  data: any;
  timestamp: number;
  action: 'create' | 'update' | 'delete';
}

class OfflineStorageManager {
  private dbName = 'DafnckMachineOffline';
  private version = 1;
  private db: IDBDatabase | null = null;

  async init(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        
        if (!db.objectStoreNames.contains('offlineActions')) {
          const store = db.createObjectStore('offlineActions', { keyPath: 'id' });
          store.createIndex('timestamp', 'timestamp', { unique: false });
        }
      };
    });
  }

  async storeOfflineAction(action: OfflineData): Promise<void> {
    if (!this.db) await this.init();
    
    const transaction = this.db!.transaction(['offlineActions'], 'readwrite');
    const store = transaction.objectStore('offlineActions');
    
    return new Promise((resolve, reject) => {
      const request = store.add(action);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async getOfflineActions(): Promise<OfflineData[]> {
    if (!this.db) await this.init();
    
    const transaction = this.db!.transaction(['offlineActions'], 'readonly');
    const store = transaction.objectStore('offlineActions');
    
    return new Promise((resolve, reject) => {
      const request = store.getAll();
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async clearOfflineActions(): Promise<void> {
    if (!this.db) await this.init();
    
    const transaction = this.db!.transaction(['offlineActions'], 'readwrite');
    const store = transaction.objectStore('offlineActions');
    
    return new Promise((resolve, reject) => {
      const request = store.clear();
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}

export const offlineStorage = new OfflineStorageManager();
```

## Background Sync

### Background Sync Implementation

```typescript
// src/utils/backgroundSync.ts
export class BackgroundSyncManager {
  private static instance: BackgroundSyncManager;
  
  static getInstance(): BackgroundSyncManager {
    if (!BackgroundSyncManager.instance) {
      BackgroundSyncManager.instance = new BackgroundSyncManager();
    }
    return BackgroundSyncManager.instance;
  }

  async registerSync(tag: string): Promise<void> {
    if ('serviceWorker' in navigator && 'sync' in window.ServiceWorkerRegistration.prototype) {
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register(tag);
    }
  }

  async scheduleDataSync(data: any): Promise<void> {
    await offlineStorage.storeOfflineAction({
      id: `sync-${Date.now()}`,
      data,
      timestamp: Date.now(),
      action: 'create'
    });
    
    await this.registerSync('data-sync');
  }
}

// In Service Worker (sw.js)
self.addEventListener('sync', (event) => {
  if (event.tag === 'data-sync') {
    event.waitUntil(syncOfflineData());
  }
});

async function syncOfflineData() {
  try {
    const offlineActions = await getOfflineActionsFromIndexedDB();
    
    for (const action of offlineActions) {
      await fetch('/api/sync', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(action)
      });
    }
    
    await clearOfflineActionsFromIndexedDB();
  } catch (error) {
    console.error('Background sync failed:', error);
  }
}
```

## Push Notifications

### Push Notification Setup

```typescript
// src/utils/pushNotifications.ts
export class PushNotificationManager {
  private vapidPublicKey = process.env.REACT_APP_VAPID_PUBLIC_KEY;

  async requestPermission(): Promise<NotificationPermission> {
    if (!('Notification' in window)) {
      throw new Error('This browser does not support notifications');
    }

    const permission = await Notification.requestPermission();
    return permission;
  }

  async subscribeToPush(): Promise<PushSubscription | null> {
    if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
      return null;
    }

    const registration = await navigator.serviceWorker.ready;
    
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array(this.vapidPublicKey!)
    });

    // Send subscription to server
    await fetch('/api/push/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(subscription)
    });

    return subscription;
  }

  private urlBase64ToUint8Array(base64String: string): Uint8Array {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
  }
}

// Push event handler in Service Worker
self.addEventListener('push', (event) => {
  const options = {
    body: event.data?.text() || 'New notification',
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    vibrate: [100, 50, 100],
    data: {
      dateOfArrival: Date.now(),
      primaryKey: 1
    },
    actions: [
      {
        action: 'explore',
        title: 'View',
        icon: '/icons/checkmark.png'
      },
      {
        action: 'close',
        title: 'Close',
        icon: '/icons/xmark.png'
      }
    ]
  };

  event.waitUntil(
    self.registration.showNotification('DafnckMachine', options)
  );
});
```

## App Installation

### Install Prompt Management

```typescript
// src/hooks/useInstallPrompt.ts
import { useState, useEffect } from 'react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

export const useInstallPrompt = () => {
  const [installPrompt, setInstallPrompt] = useState<BeforeInstallPromptEvent | null>(null);
  const [isInstallable, setIsInstallable] = useState(false);

  useEffect(() => {
    const handleBeforeInstallPrompt = (e: BeforeInstallPromptEvent) => {
      e.preventDefault();
      setInstallPrompt(e);
      setIsInstallable(true);
    };

    const handleAppInstalled = () => {
      setInstallPrompt(null);
      setIsInstallable(false);
    };

    window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt as EventListener);
    window.addEventListener('appinstalled', handleAppInstalled);

    return () => {
      window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt as EventListener);
      window.removeEventListener('appinstalled', handleAppInstalled);
    };
  }, []);

  const promptInstall = async () => {
    if (!installPrompt) return false;

    await installPrompt.prompt();
    const { outcome } = await installPrompt.userChoice;
    
    setInstallPrompt(null);
    setIsInstallable(false);
    
    return outcome === 'accepted';
  };

  return { isInstallable, promptInstall };
};
```

### Install Banner Component

```tsx
// src/components/InstallBanner.tsx
import React from 'react';
import { useInstallPrompt } from '../hooks/useInstallPrompt';

export const InstallBanner: React.FC = () => {
  const { isInstallable, promptInstall } = useInstallPrompt();

  if (!isInstallable) return null;

  const handleInstall = async () => {
    const accepted = await promptInstall();
    if (accepted) {
      console.log('App installed successfully');
    }
  };

  return (
    <div className="install-banner">
      <div className="install-banner__content">
        <h3>Install DafnckMachine</h3>
        <p>Get the full app experience with offline access and notifications</p>
      </div>
      <div className="install-banner__actions">
        <button onClick={handleInstall} className="btn btn-primary">
          Install App
        </button>
      </div>
    </div>
  );
};
```

## Performance Optimization

### Resource Preloading

```typescript
// src/utils/resourcePreloader.ts
export class ResourcePreloader {
  private preloadedResources = new Set<string>();

  preloadCriticalResources(): void {
    const criticalResources = [
      '/static/css/main.css',
      '/static/js/bundle.js',
      '/icons/icon-192x192.png'
    ];

    criticalResources.forEach(resource => {
      this.preloadResource(resource);
    });
  }

  preloadResource(url: string, type: 'script' | 'style' | 'image' = 'script'): void {
    if (this.preloadedResources.has(url)) return;

    const link = document.createElement('link');
    link.rel = 'preload';
    link.href = url;
    
    switch (type) {
      case 'script':
        link.as = 'script';
        break;
      case 'style':
        link.as = 'style';
        break;
      case 'image':
        link.as = 'image';
        break;
    }

    document.head.appendChild(link);
    this.preloadedResources.add(url);
  }

  preloadRoute(route: string): void {
    // Preload route-specific resources
    import(`../pages${route}`).catch(error => {
      console.warn(`Failed to preload route ${route}:`, error);
    });
  }
}

export const resourcePreloader = new ResourcePreloader();
```

## Testing PWA Features

### PWA Testing Suite

```typescript
// src/__tests__/pwa.test.ts
import { render, screen } from '@testing-library/react';
import { InstallBanner } from '../components/InstallBanner';

// Mock service worker
const mockServiceWorker = {
  register: jest.fn(),
  ready: Promise.resolve({
    pushManager: {
      subscribe: jest.fn()
    }
  })
};

Object.defineProperty(navigator, 'serviceWorker', {
  value: mockServiceWorker,
  writable: true
});

describe('PWA Features', () => {
  describe('Service Worker', () => {
    it('should register service worker', async () => {
      const { registerServiceWorker } = await import('../serviceWorker/registration');
      
      await registerServiceWorker();
      
      expect(mockServiceWorker.register).toHaveBeenCalledWith('/sw.js', {
        scope: '/'
      });
    });
  });

  describe('Install Banner', () => {
    it('should not show when not installable', () => {
      render(<InstallBanner />);
      
      expect(screen.queryByText('Install DafnckMachine')).not.toBeInTheDocument();
    });

    it('should show install prompt when installable', () => {
      // Mock beforeinstallprompt event
      const mockEvent = new Event('beforeinstallprompt');
      window.dispatchEvent(mockEvent);
      
      render(<InstallBanner />);
      
      expect(screen.getByText('Install DafnckMachine')).toBeInTheDocument();
    });
  });

  describe('Offline Detection', () => {
    it('should detect online status', () => {
      Object.defineProperty(navigator, 'onLine', {
        value: true,
        writable: true
      });

      const { useOnlineStatus } = require('../hooks/useOnlineStatus');
      const { result } = renderHook(() => useOnlineStatus());
      
      expect(result.current).toBe(true);
    });
  });
});
```

## Deployment Considerations

### PWA Deployment Checklist

- [ ] **HTTPS Required**: PWA features require secure context
- [ ] **Service Worker**: Properly configured and registered
- [ ] **Web App Manifest**: Valid manifest.json with required fields
- [ ] **Icons**: Complete icon set for all platforms
- [ ] **Offline Fallback**: Offline page for network failures
- [ ] **Cache Strategy**: Appropriate caching for different resource types
- [ ] **Performance**: Lighthouse PWA score > 90
- [ ] **Accessibility**: WCAG 2.1 AA compliance
- [ ] **Cross-Browser**: Testing across major browsers
- [ ] **Mobile Testing**: Testing on actual devices

### Performance Metrics

```typescript
// src/utils/performanceMetrics.ts
export class PWAPerformanceMetrics {
  measureInstallTime(): void {
    window.addEventListener('appinstalled', () => {
      const installTime = performance.now();
      this.sendMetric('pwa_install_time', installTime);
    });
  }

  measureOfflineUsage(): void {
    window.addEventListener('offline', () => {
      const offlineStart = Date.now();
      
      const handleOnline = () => {
        const offlineDuration = Date.now() - offlineStart;
        this.sendMetric('pwa_offline_duration', offlineDuration);
        window.removeEventListener('online', handleOnline);
      };
      
      window.addEventListener('online', handleOnline);
    });
  }

  private sendMetric(name: string, value: number): void {
    if ('navigator' in window && 'sendBeacon' in navigator) {
      navigator.sendBeacon('/api/metrics', JSON.stringify({
        name,
        value,
        timestamp: Date.now()
      }));
    }
  }
}
```

## Best Practices

### 1. Service Worker Management
- **Update Strategy**: Implement proper update mechanisms
- **Cache Invalidation**: Clear old caches on updates
- **Error Handling**: Graceful fallbacks for service worker failures
- **Performance**: Optimize service worker execution

### 2. Offline Experience
- **Core Functionality**: Ensure essential features work offline
- **Data Sync**: Implement background synchronization
- **User Feedback**: Clear offline/online status indicators
- **Storage Management**: Efficient use of browser storage

### 3. Installation Experience
- **Timing**: Show install prompt at appropriate moments
- **Value Proposition**: Clear benefits of installation
- **Onboarding**: Guide users through PWA features
- **Fallbacks**: Graceful degradation for non-supporting browsers

### 4. Performance Optimization
- **Resource Loading**: Optimize critical resource loading
- **Cache Efficiency**: Implement effective caching strategies
- **Bundle Size**: Minimize JavaScript bundle sizes
- **Lazy Loading**: Load non-critical resources on demand

## Maintenance and Updates

### Update Management
- **Version Control**: Semantic versioning for PWA updates
- **Cache Busting**: Automatic cache invalidation on updates
- **User Notification**: Inform users of available updates
- **Rollback Strategy**: Plan for update rollbacks if needed

### Monitoring
- **Usage Analytics**: Track PWA feature adoption
- **Performance Monitoring**: Monitor offline/online performance
- **Error Tracking**: Track service worker and PWA errors
- **User Feedback**: Collect feedback on PWA experience

This comprehensive PWA implementation guide ensures DafnckMachine v3.1 delivers a native-like experience across all platforms with robust offline capabilities and enhanced user engagement. 