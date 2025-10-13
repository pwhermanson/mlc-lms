# Progressive Web App (PWA) Specifications

## Overview

This document provides the technical implementation specifications for Progressive Web App capabilities in the MLC LMS platform. It defines the Web App Manifest structure, Service Worker implementation, offline storage schemas, and build configuration required to deliver installable, offline-capable educational experiences.

## Purpose

While [PWA Considerations](./pwa-considerations.md) establishes the strategic capabilities, user experience patterns, and behavioral requirements for PWA features, this document serves as the **technical implementation reference** for developers building the PWA functionality. It specifies:

- Exact manifest.json structure and configuration
- Service Worker file organization and registration code
- Workbox/next-pwa build configurations
- IndexedDB schemas for offline data persistence
- Cache naming conventions and versioning strategies
- Build pipeline integration and deployment requirements
- Testing procedures for PWA features

This specification ensures consistent PWA implementation across development teams and provides a reference for code reviews, testing, and deployment verification.

**Implementation Phase**: PWA features are targeted for **Phase 2** implementation, after core online functionality is stable. See [PWA Considerations - Phase Timing](./pwa-considerations.md#purpose) for rationale.

---

## Web App Manifest Specification

### Manifest File: `/public/manifest.json`

The Web App Manifest configures how the MLC LMS appears when installed as a PWA. This file must be served from the root public directory with `Content-Type: application/manifest+json`.

```json
{
  "name": "Music Learning Community",
  "short_name": "MLC",
  "description": "Interactive music education platform with games, assignments, and progress tracking",
  "start_url": "/?source=pwa",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1e40af",
  "orientation": "any",
  "scope": "/",
  "lang": "en-US",
  "dir": "ltr",
  
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-maskable-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icons/icon-maskable-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  
  "screenshots": [
    {
      "src": "/screenshots/desktop-dashboard.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Student dashboard showing assignments and progress"
    },
    {
      "src": "/screenshots/mobile-game.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Playing a music game on mobile"
    }
  ],
  
  "shortcuts": [
    {
      "name": "Assignments",
      "short_name": "Assignments",
      "description": "View your current assignments",
      "url": "/student/assignments?source=pwa_shortcut",
      "icons": [
        {
          "src": "/icons/shortcut-assignments.png",
          "sizes": "96x96",
          "type": "image/png"
        }
      ]
    },
    {
      "name": "All Games",
      "short_name": "Games",
      "description": "Browse the game library",
      "url": "/games?source=pwa_shortcut",
      "icons": [
        {
          "src": "/icons/shortcut-games.png",
          "sizes": "96x96",
          "type": "image/png"
        }
      ]
    },
    {
      "name": "Messages",
      "short_name": "Messages",
      "description": "Check your messages",
      "url": "/messages?source=pwa_shortcut",
      "icons": [
        {
          "src": "/icons/shortcut-messages.png",
          "sizes": "96x96",
          "type": "image/png"
        }
      ]
    }
  ],
  
  "categories": ["education", "music"],
  
  "prefer_related_applications": false,
  
  "iarc_rating_id": "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7"
}
```

### Manifest Configuration Details

**Display Modes**:
- `standalone`: Preferred mode - looks and behaves like native app (no browser chrome)
- Fallback chain: `standalone` → `minimal-ui` → `browser`

**Theme and Background Colors**:
- `theme_color: #1e40af`: Blue-600 from brand palette (see [UX Design System](../01-ux-design-system/components-overview.md))
- `background_color: #ffffff`: White background for splash screen
- Must match CSS to prevent color flash on launch

**Icon Requirements**:
- **Purpose: any**: Standard app icons for home screen
- **Purpose: maskable**: Safe area for Android adaptive icons
  - Design guideline: Critical content within 80% center circle
  - 20% margin around edges may be masked
- All icons must be PNG format (no SVG support in manifest)
- Sizes: 72, 96, 128, 144, 152, 192, 384, 512 for comprehensive device coverage

**Screenshots** (Android Chrome/Edge):
- Used in installation prompt and app store listings
- `form_factor: wide` for desktop (min 1280x720)
- `form_factor: narrow` for mobile (typical phone aspect ratios)
- Maximum 8 screenshots supported

**Shortcuts** (Android/Desktop):
- Long-press app icon or right-click to access
- Maximum 4 shortcuts recommended
- Query parameters (`?source=pwa_shortcut`) enable analytics tracking

**IARC Rating**:
- International Age Rating Coalition certificate
- Required for Play Store listing (future consideration)
- Obtain from [https://www.globalratings.com/](https://www.globalratings.com/)

### HTML Head Integration

Add to `/pages/_document.tsx` (Next.js custom Document):

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        {/* PWA Manifest */}
        <link rel="manifest" href="/manifest.json" />
        
        {/* Theme Color */}
        <meta name="theme-color" content="#1e40af" />
        
        {/* iOS-specific meta tags */}
        <meta name="apple-mobile-web-app-capable" content="yes" />
        <meta name="apple-mobile-web-app-status-bar-style" content="default" />
        <meta name="apple-mobile-web-app-title" content="MLC" />
        
        {/* iOS App Icons (fallback for older iOS) */}
        <link rel="apple-touch-icon" sizes="180x180" href="/icons/apple-touch-icon.png" />
        <link rel="apple-touch-icon" sizes="152x152" href="/icons/icon-152x152.png" />
        <link rel="apple-touch-icon" sizes="144x144" href="/icons/icon-144x144.png" />
        <link rel="apple-touch-icon" sizes="120x120" href="/icons/apple-touch-icon-120x120.png" />
        
        {/* iOS Splash Screens (optional, generated by next-pwa) */}
        <link
          rel="apple-touch-startup-image"
          media="screen and (device-width: 430px) and (device-height: 932px) and (-webkit-device-pixel-ratio: 3)"
          href="/splash/iPhone_15_Pro_Max__iPhone_15_Plus__iPhone_14_Pro_Max_portrait.png"
        />
        
        {/* Favicon */}
        <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
        <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
        
        {/* Microsoft Tiles */}
        <meta name="msapplication-TileColor" content="#1e40af" />
        <meta name="msapplication-TileImage" content="/icons/icon-144x144.png" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

---

## Service Worker Implementation

### Next.js PWA Configuration

Install `next-pwa` for Service Worker generation with Workbox:

```bash
npm install next-pwa
```

### Next.js Config: `next.config.js`

```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development',
  register: true,
  skipWaiting: true,
  scope: '/',
  sw: 'sw.js',
  
  // Workbox options
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.(?:gstatic)\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'google-fonts-webfonts',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 365 * 24 * 60 * 60 // 1 year
        }
      }
    },
    {
      urlPattern: /^https:\/\/fonts\.(?:googleapis)\.com\/.*/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'google-fonts-stylesheets',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 7 * 24 * 60 * 60 // 1 week
        }
      }
    },
    {
      urlPattern: /\.(?:eot|otf|ttc|ttf|woff|woff2|font.css)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-font-assets',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 7 * 24 * 60 * 60 // 1 week
        }
      }
    },
    {
      urlPattern: /\.(?:jpg|jpeg|gif|png|svg|ico|webp)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-image-assets',
        expiration: {
          maxEntries: 64,
          maxAgeSeconds: 30 * 24 * 60 * 60 // 30 days
        }
      }
    },
    {
      urlPattern: /\.(?:mp3|wav|ogg|m4a)$/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'static-audio-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 30 * 24 * 60 * 60 // 30 days
        },
        cacheableResponse: {
          statuses: [0, 200]
        }
      }
    },
    {
      urlPattern: /\.(?:js)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-js-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60 // 24 hours
        }
      }
    },
    {
      urlPattern: /\.(?:css|less)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-style-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60 // 24 hours
        }
      }
    },
    {
      urlPattern: /^\/api\/games\/.*/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'api-games',
        expiration: {
          maxEntries: 50,
          maxAgeSeconds: 60 * 60 // 1 hour
        },
        networkTimeoutSeconds: 10
      }
    },
    {
      urlPattern: /^\/api\/assignments\/.*/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'api-assignments',
        expiration: {
          maxEntries: 50,
          maxAgeSeconds: 5 * 60 // 5 minutes
        },
        networkTimeoutSeconds: 10
      }
    },
    {
      urlPattern: /\.(?:html)$/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'pages',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60 // 24 hours
        }
      }
    },
    {
      urlPattern: /^\/games\/[^/]+\/$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'game-pages',
        expiration: {
          maxEntries: 50,
          maxAgeSeconds: 7 * 24 * 60 * 60 // 7 days
        }
      }
    }
  ],
  
  // Exclude from pre-caching
  buildExcludes: [/middleware-manifest\.json$/],
  
  // Additional options
  publicExcludes: ['!noprecache/**/*'],
  
  fallbacks: {
    document: '/offline',
    image: '/images/fallback-image.png',
    audio: '/audio/fallback-silence.mp3'
  }
})

module.exports = withPWA({
  reactStrictMode: true,
  swcMinify: true,
  // ... other Next.js config
})
```

### Caching Strategy Patterns

**Handler Types** (Workbox strategies from [Caching Strategy](../18-architecture/caching-strategy.md)):

1. **CacheFirst** - For static, immutable assets
   - Check cache first, fall back to network
   - Use for: Fonts, audio files, versioned assets
   - Fastest but may serve stale content

2. **NetworkFirst** - For dynamic, frequently-updated content
   - Try network first, fall back to cache
   - Use for: API responses, HTML pages, real-time data
   - Ensures freshness when online

3. **StaleWhileRevalidate** - Balance of speed and freshness
   - Return cached version immediately, update cache in background
   - Use for: Images, stylesheets, JavaScript, game metadata
   - Best user experience for semi-static content

4. **NetworkOnly** - Always fetch from network (no caching)
   - Use for: User authentication, score submissions, sensitive data
   - Not included in config - bypasses Service Worker

5. **CacheOnly** - Only return cached content (no network)
   - Use for: Offline-only features, pre-cached offline pages
   - Requires explicit cache population

### Cache Naming Convention

**Pattern**: `mlc-lms-v{MAJOR}-{CACHE_TYPE}-{ENVIRONMENT}`

Examples:
- `mlc-lms-v1-game-assets-production`
- `mlc-lms-v1-api-games-staging`
- `mlc-lms-v2-static-images-production`

**Versioning Rules**:
- Increment MAJOR version for breaking changes requiring cache clear
- Cache names include version to enable clean migration
- Old caches automatically deleted on Service Worker activation

---

## Offline Data Storage

### IndexedDB Schema

IndexedDB stores structured offline data for game progress, assignments, and queue items. Use `idb` library for Promise-based API:

```bash
npm install idb
```

### Database: `mlc-offline-db` (Version 1)

```typescript
// lib/offline-db.ts
import { openDB, DBSchema, IDBPDatabase } from 'idb';

// Database schema definition
interface MLCOfflineDB extends DBSchema {
  'offline-queue': {
    key: string;
    value: OfflineQueueItem;
    indexes: {
      'by-status': string;
      'by-type': string;
      'by-timestamp': Date;
    };
  };
  'offline-games': {
    key: string;
    value: OfflineGameData;
    indexes: {
      'by-downloaded': Date;
      'by-last-accessed': Date;
    };
  };
  'offline-assignments': {
    key: string;
    value: OfflineAssignment;
    indexes: {
      'by-student': string;
      'by-downloaded': Date;
    };
  };
  'pwa-state': {
    key: string;
    value: PWAStateData;
  };
}

// Data types (matching PWA Considerations data model)
interface OfflineQueueItem {
  id: string;
  type: 'score_submission' | 'progress_update' | 'message_send' | 'assignment_complete';
  payload: any;
  timestamp: Date;
  retryCount: number;
  maxRetries: number;
  status: 'queued' | 'syncing' | 'success' | 'failed';
  errorMessage?: string;
}

interface OfflineGameData {
  gameId: string;
  gameMetadata: {
    title: string;
    description: string;
    category: string;
    difficulty: string;
  };
  assetsDownloaded: boolean;
  assetUrls: string[];
  downloadedAt: Date;
  lastAccessedAt: Date;
  sizeBytes: number;
}

interface OfflineAssignment {
  assignmentId: string;
  studentId: string;
  gameIds: string[];
  assignmentMetadata: {
    title: string;
    dueDate: Date | null;
    teacherName: string;
  };
  downloadedAt: Date;
  expiresAt: Date | null;
}

interface PWAStateData {
  key: string;
  value: any;
  updatedAt: Date;
}

// Open database with version management
const DB_NAME = 'mlc-offline-db';
const DB_VERSION = 1;

let dbPromise: Promise<IDBPDatabase<MLCOfflineDB>> | null = null;

export function getDB(): Promise<IDBPDatabase<MLCOfflineDB>> {
  if (!dbPromise) {
    dbPromise = openDB<MLCOfflineDB>(DB_NAME, DB_VERSION, {
      upgrade(db, oldVersion, newVersion, transaction) {
        // Version 1: Initial schema
        if (oldVersion < 1) {
          // Offline Queue store
          const queueStore = db.createObjectStore('offline-queue', {
            keyPath: 'id'
          });
          queueStore.createIndex('by-status', 'status');
          queueStore.createIndex('by-type', 'type');
          queueStore.createIndex('by-timestamp', 'timestamp');

          // Offline Games store
          const gamesStore = db.createObjectStore('offline-games', {
            keyPath: 'gameId'
          });
          gamesStore.createIndex('by-downloaded', 'downloadedAt');
          gamesStore.createIndex('by-last-accessed', 'lastAccessedAt');

          // Offline Assignments store
          const assignmentsStore = db.createObjectStore('offline-assignments', {
            keyPath: 'assignmentId'
          });
          assignmentsStore.createIndex('by-student', 'studentId');
          assignmentsStore.createIndex('by-downloaded', 'downloadedAt');

          // PWA State store (key-value pairs)
          db.createObjectStore('pwa-state', {
            keyPath: 'key'
          });
        }
        
        // Future version migrations
        // if (oldVersion < 2) { /* Version 2 changes */ }
      },
      blocked() {
        console.warn('IndexedDB upgrade blocked - close other tabs');
      },
      blocking() {
        console.warn('IndexedDB blocking - new version available');
      },
      terminated() {
        console.error('IndexedDB connection terminated unexpectedly');
        dbPromise = null; // Reset to allow reconnection
      }
    });
  }
  return dbPromise;
}

// Helper functions for offline queue
export async function addToOfflineQueue(item: Omit<OfflineQueueItem, 'id' | 'timestamp' | 'retryCount' | 'status'>): Promise<string> {
  const db = await getDB();
  const id = `queue-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  const queueItem: OfflineQueueItem = {
    ...item,
    id,
    timestamp: new Date(),
    retryCount: 0,
    status: 'queued'
  };
  await db.add('offline-queue', queueItem);
  return id;
}

export async function getOfflineQueue(): Promise<OfflineQueueItem[]> {
  const db = await getDB();
  return db.getAllFromIndex('offline-queue', 'by-status', 'queued');
}

export async function updateQueueItemStatus(id: string, status: OfflineQueueItem['status'], errorMessage?: string): Promise<void> {
  const db = await getDB();
  const item = await db.get('offline-queue', id);
  if (item) {
    item.status = status;
    if (errorMessage) {
      item.errorMessage = errorMessage;
    }
    await db.put('offline-queue', item);
  }
}

export async function clearCompletedQueueItems(): Promise<void> {
  const db = await getDB();
  const completedItems = await db.getAllFromIndex('offline-queue', 'by-status', 'success');
  const tx = db.transaction('offline-queue', 'readwrite');
  await Promise.all(completedItems.map(item => tx.store.delete(item.id)));
  await tx.done;
}

// Helper functions for offline games
export async function saveOfflineGame(gameData: OfflineGameData): Promise<void> {
  const db = await getDB();
  await db.put('offline-games', gameData);
}

export async function getOfflineGame(gameId: string): Promise<OfflineGameData | undefined> {
  const db = await getDB();
  return db.get('offline-games', gameId);
}

export async function getAllOfflineGames(): Promise<OfflineGameData[]> {
  const db = await getDB();
  return db.getAll('offline-games');
}

export async function deleteOfflineGame(gameId: string): Promise<void> {
  const db = await getDB();
  await db.delete('offline-games', gameId);
}

export async function updateGameAccessTime(gameId: string): Promise<void> {
  const db = await getDB();
  const game = await db.get('offline-games', gameId);
  if (game) {
    game.lastAccessedAt = new Date();
    await db.put('offline-games', game);
  }
}

// Helper functions for offline assignments
export async function saveOfflineAssignment(assignment: OfflineAssignment): Promise<void> {
  const db = await getDB();
  await db.put('offline-assignments', assignment);
}

export async function getOfflineAssignments(studentId: string): Promise<OfflineAssignment[]> {
  const db = await getDB();
  return db.getAllFromIndex('offline-assignments', 'by-student', studentId);
}

// Helper functions for PWA state
export async function setPWAState(key: string, value: any): Promise<void> {
  const db = await getDB();
  await db.put('pwa-state', {
    key,
    value,
    updatedAt: new Date()
  });
}

export async function getPWAState(key: string): Promise<any> {
  const db = await getDB();
  const record = await db.get('pwa-state', key);
  return record?.value;
}
```

### Storage Quota Management

```typescript
// lib/storage-quota.ts

export interface StorageQuotaInfo {
  usage: number;
  quota: number;
  percentUsed: number;
  available: number;
}

export async function getStorageQuota(): Promise<StorageQuotaInfo> {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    const estimate = await navigator.storage.estimate();
    const usage = estimate.usage || 0;
    const quota = estimate.quota || 0;
    const percentUsed = quota > 0 ? (usage / quota) * 100 : 0;
    const available = quota - usage;
    
    return {
      usage,
      quota,
      percentUsed,
      available
    };
  }
  
  // Fallback for browsers without Storage API
  return {
    usage: 0,
    quota: 0,
    percentUsed: 0,
    available: 0
  };
}

export async function requestPersistentStorage(): Promise<boolean> {
  if ('storage' in navigator && 'persist' in navigator.storage) {
    return await navigator.storage.persist();
  }
  return false;
}

export async function isPersistentStorageGranted(): Promise<boolean> {
  if ('storage' in navigator && 'persisted' in navigator.storage) {
    return await navigator.storage.persisted();
  }
  return false;
}

// Evict least recently used games when quota exceeded
export async function evictLRUGames(targetBytes: number): Promise<number> {
  const games = await getAllOfflineGames();
  
  // Sort by last accessed (oldest first)
  games.sort((a, b) => a.lastAccessedAt.getTime() - b.lastAccessedAt.getTime());
  
  let freedBytes = 0;
  for (const game of games) {
    if (freedBytes >= targetBytes) break;
    
    await deleteOfflineGame(game.gameId);
    // Also clear Service Worker cache for this game
    if ('caches' in window) {
      const cacheKeys = await caches.keys();
      for (const key of cacheKeys) {
        const cache = await caches.open(key);
        for (const url of game.assetUrls) {
          await cache.delete(url);
        }
      }
    }
    
    freedBytes += game.sizeBytes;
  }
  
  return freedBytes;
}
```

---

## PWA Installation Detection and Prompts

### Install Prompt Component

```tsx
// components/PWAInstallPrompt.tsx
import { useState, useEffect } from 'react';
import { X } from 'lucide-react';

interface BeforeInstallPromptEvent extends Event {
  prompt: () => Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

export function PWAInstallPrompt() {
  const [deferredPrompt, setDeferredPrompt] = useState<BeforeInstallPromptEvent | null>(null);
  const [showPrompt, setShowPrompt] = useState(false);
  const [isInstalled, setIsInstalled] = useState(false);
  
  useEffect(() => {
    // Check if already installed
    if (window.matchMedia('(display-mode: standalone)').matches) {
      setIsInstalled(true);
      return;
    }
    
    // Check if user previously dismissed
    const dismissedAt = localStorage.getItem('pwa-install-dismissed');
    if (dismissedAt) {
      const daysSinceDismissal = (Date.now() - parseInt(dismissedAt)) / (1000 * 60 * 60 * 24);
      if (daysSinceDismissal < 30) {
        return; // Don't show again for 30 days
      }
    }
    
    // Listen for beforeinstallprompt event
    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e as BeforeInstallPromptEvent);
      
      // Show prompt after user has visited a few times
      const visitCount = parseInt(localStorage.getItem('visit-count') || '0') + 1;
      localStorage.setItem('visit-count', visitCount.toString());
      
      if (visitCount >= 2) {
        setTimeout(() => setShowPrompt(true), 5000); // Show after 5 seconds
      }
    };
    
    window.addEventListener('beforeinstallprompt', handler);
    
    return () => window.removeEventListener('beforeinstallprompt', handler);
  }, []);
  
  const handleInstallClick = async () => {
    if (!deferredPrompt) return;
    
    // Show native install prompt
    deferredPrompt.prompt();
    
    // Wait for user choice
    const { outcome } = await deferredPrompt.userChoice;
    
    if (outcome === 'accepted') {
      console.log('PWA installed');
      // Track analytics
      if (window.gtag) {
        window.gtag('event', 'pwa_installed', {
          method: 'custom_prompt'
        });
      }
    } else {
      console.log('PWA installation dismissed');
    }
    
    // Clear the prompt
    setDeferredPrompt(null);
    setShowPrompt(false);
  };
  
  const handleDismiss = () => {
    setShowPrompt(false);
    localStorage.setItem('pwa-install-dismissed', Date.now().toString());
    
    // Track analytics
    if (window.gtag) {
      window.gtag('event', 'pwa_install_dismissed', {
        dismissal_count: parseInt(localStorage.getItem('pwa-dismiss-count') || '0') + 1
      });
    }
    
    const dismissCount = parseInt(localStorage.getItem('pwa-dismiss-count') || '0') + 1;
    localStorage.setItem('pwa-dismiss-count', dismissCount.toString());
  };
  
  // Don't render if installed or not showing
  if (isInstalled || !showPrompt || !deferredPrompt) {
    return null;
  }
  
  return (
    <div className="fixed bottom-4 left-4 right-4 md:left-auto md:right-4 md:max-w-md bg-white border border-gray-200 rounded-lg shadow-lg p-4 z-50">
      <button
        onClick={handleDismiss}
        className="absolute top-2 right-2 text-gray-400 hover:text-gray-600"
        aria-label="Dismiss"
      >
        <X className="w-4 h-4" />
      </button>
      
      <div className="flex items-start gap-3">
        <img src="/icons/icon-96x96.png" alt="MLC" className="w-12 h-12 rounded-lg" />
        <div className="flex-1">
          <h3 className="font-semibold text-gray-900 mb-1">
            Install MLC App
          </h3>
          <p className="text-sm text-gray-600 mb-3">
            Install for faster access and offline practice
          </p>
          <div className="flex gap-2">
            <button
              onClick={handleInstallClick}
              className="px-4 py-2 bg-blue-600 text-white text-sm font-medium rounded-md hover:bg-blue-700"
            >
              Install
            </button>
            <button
              onClick={handleDismiss}
              className="px-4 py-2 bg-gray-100 text-gray-700 text-sm font-medium rounded-md hover:bg-gray-200"
            >
              Not Now
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### iOS Installation Instructions

```tsx
// components/IOSInstallPrompt.tsx
import { useState, useEffect } from 'react';
import { Share, Plus, X } from 'lucide-react';

export function IOSInstallPrompt() {
  const [showPrompt, setShowPrompt] = useState(false);
  
  useEffect(() => {
    // Detect iOS
    const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) && !(window as any).MSStream;
    
    // Check if already installed
    const isInstalled = window.matchMedia('(display-mode: standalone)').matches;
    
    // Check if user previously dismissed
    const dismissedAt = localStorage.getItem('ios-install-dismissed');
    const shouldShow = dismissedAt 
      ? (Date.now() - parseInt(dismissedAt)) / (1000 * 60 * 60 * 24) >= 30
      : true;
    
    if (isIOS && !isInstalled && shouldShow) {
      setTimeout(() => setShowPrompt(true), 10000); // Show after 10 seconds
    }
  }, []);
  
  if (!showPrompt) return null;
  
  return (
    <div className="fixed bottom-4 left-4 right-4 bg-white border border-gray-200 rounded-lg shadow-lg p-4 z-50">
      <button
        onClick={() => {
          setShowPrompt(false);
          localStorage.setItem('ios-install-dismissed', Date.now().toString());
        }}
        className="absolute top-2 right-2 text-gray-400 hover:text-gray-600"
        aria-label="Dismiss"
      >
        <X className="w-4 h-4" />
      </button>
      
      <h3 className="font-semibold text-gray-900 mb-2">
        Install MLC on Your iPhone
      </h3>
      
      <ol className="space-y-2 text-sm text-gray-600">
        <li className="flex items-center gap-2">
          <span className="flex-shrink-0 w-6 h-6 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center text-xs font-semibold">
            1
          </span>
          <span>
            Tap the <Share className="inline w-4 h-4 mx-1" /> Share button below
          </span>
        </li>
        <li className="flex items-center gap-2">
          <span className="flex-shrink-0 w-6 h-6 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center text-xs font-semibold">
            2
          </span>
          <span>
            Scroll and tap <Plus className="inline w-4 h-4 mx-1" /> "Add to Home Screen"
          </span>
        </li>
        <li className="flex items-center gap-2">
          <span className="flex-shrink-0 w-6 h-6 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center text-xs font-semibold">
            3
          </span>
          <span>Tap "Add" to confirm</span>
        </li>
      </ol>
    </div>
  );
}
```

---

## Build and Deployment Configuration

### Environment Variables

Add to `.env.local` and production environment:

```bash
# PWA Configuration
NEXT_PUBLIC_PWA_ENABLED=true
NEXT_PUBLIC_APP_NAME="Music Learning Community"
NEXT_PUBLIC_APP_SHORT_NAME="MLC"
NEXT_PUBLIC_THEME_COLOR=#1e40af

# Service Worker versioning
NEXT_PUBLIC_SW_VERSION=1.0.0

# Analytics for PWA events
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX
```

### Build Scripts

Update `package.json`:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "build:pwa": "NODE_ENV=production NEXT_PUBLIC_PWA_ENABLED=true next build",
    "start": "next start",
    "lint": "next lint",
    "pwa:validate": "node scripts/validate-pwa.js",
    "pwa:generate-icons": "node scripts/generate-pwa-icons.js"
  }
}
```

### PWA Validation Script

Create `scripts/validate-pwa.js`:

```javascript
const fs = require('fs');
const path = require('path');

// Validate manifest.json
function validateManifest() {
  const manifestPath = path.join(__dirname, '../public/manifest.json');
  
  if (!fs.existsSync(manifestPath)) {
    console.error('❌ manifest.json not found');
    return false;
  }
  
  const manifest = JSON.parse(fs.readFileSync(manifestPath, 'utf8'));
  
  const required = ['name', 'short_name', 'start_url', 'display', 'icons'];
  const missing = required.filter(field => !manifest[field]);
  
  if (missing.length > 0) {
    console.error(`❌ Missing required manifest fields: ${missing.join(', ')}`);
    return false;
  }
  
  // Validate icons
  const iconSizes = manifest.icons.map(icon => icon.sizes);
  const requiredSizes = ['192x192', '512x512'];
  const hasRequired = requiredSizes.every(size => iconSizes.includes(size));
  
  if (!hasRequired) {
    console.error(`❌ Missing required icon sizes: ${requiredSizes.join(', ')}`);
    return false;
  }
  
  console.log('✅ manifest.json is valid');
  return true;
}

// Validate Service Worker
function validateServiceWorker() {
  const swPath = path.join(__dirname, '../public/sw.js');
  
  if (!fs.existsSync(swPath)) {
    console.warn('⚠️  sw.js not found (generated during build)');
    return true; // Not an error during development
  }
  
  console.log('✅ Service Worker file exists');
  return true;
}

// Validate PWA icons
function validateIcons() {
  const iconsDir = path.join(__dirname, '../public/icons');
  
  if (!fs.existsSync(iconsDir)) {
    console.error('❌ /public/icons directory not found');
    return false;
  }
  
  const requiredIcons = [
    'icon-192x192.png',
    'icon-512x512.png',
    'icon-maskable-192x192.png',
    'icon-maskable-512x512.png'
  ];
  
  const missing = requiredIcons.filter(icon => 
    !fs.existsSync(path.join(iconsDir, icon))
  );
  
  if (missing.length > 0) {
    console.error(`❌ Missing required icons: ${missing.join(', ')}`);
    return false;
  }
  
  console.log('✅ All required PWA icons present');
  return true;
}

// Run all validations
function main() {
  console.log('🔍 Validating PWA configuration...\n');
  
  const results = [
    validateManifest(),
    validateServiceWorker(),
    validateIcons()
  ];
  
  const allValid = results.every(r => r === true);
  
  if (allValid) {
    console.log('\n✅ PWA configuration is valid');
    process.exit(0);
  } else {
    console.log('\n❌ PWA configuration has errors');
    process.exit(1);
  }
}

main();
```

### Deployment Checklist

**Pre-Deployment**:
- [ ] Run `npm run pwa:validate` to verify configuration
- [ ] Test Service Worker registration in production build locally
- [ ] Verify manifest.json is accessible at `/manifest.json`
- [ ] Test offline functionality with Chrome DevTools (Network → Offline)
- [ ] Verify install prompt appears on supported browsers
- [ ] Test on actual iOS device (Safari) and Android device (Chrome)

**Deployment**:
- [ ] Set `NEXT_PUBLIC_PWA_ENABLED=true` in production environment
- [ ] Verify HTTPS is enabled (required for Service Workers)
- [ ] Set proper Cache-Control headers for Service Worker file:
  ```
  sw.js: Cache-Control: no-cache, no-store, must-revalidate
  ```
- [ ] Verify manifest.json has correct `start_url` for production domain
- [ ] Test PWA installation on production domain

**Post-Deployment**:
- [ ] Use [Lighthouse PWA audit](https://web.dev/measure/) to verify PWA criteria
- [ ] Test Service Worker updates by deploying new version
- [ ] Monitor PWA installation analytics
- [ ] Track offline usage patterns
- [ ] Monitor Service Worker errors in production logs

---

## Testing Requirements

### Service Worker Testing

```typescript
// __tests__/pwa/service-worker.test.ts
describe('Service Worker', () => {
  it('should register successfully', async () => {
    const registration = await navigator.serviceWorker.register('/sw.js');
    expect(registration).toBeDefined();
    expect(registration.scope).toBe('https://example.com/');
  });
  
  it('should cache critical assets on install', async () => {
    const cache = await caches.open('mlc-lms-v1-static');
    const cachedURLs = (await cache.keys()).map(req => req.url);
    
    expect(cachedURLs).toContain(expect.stringContaining('/manifest.json'));
    expect(cachedURLs).toContain(expect.stringContaining('/offline'));
  });
  
  it('should respond with cached assets when offline', async () => {
    // Simulate offline
    await navigator.serviceWorker.ready;
    
    const response = await fetch('/icons/icon-192x192.png');
    expect(response.ok).toBe(true);
    expect(response.headers.get('x-cache')).toBe('HIT');
  });
});
```

### IndexedDB Testing

```typescript
// __tests__/pwa/offline-storage.test.ts
import { getDB, addToOfflineQueue, getOfflineQueue } from '@/lib/offline-db';

describe('Offline Storage', () => {
  beforeEach(async () => {
    const db = await getDB();
    await db.clear('offline-queue');
  });
  
  it('should add items to offline queue', async () => {
    const id = await addToOfflineQueue({
      type: 'score_submission',
      payload: { gameId: 'test', score: 100 },
      maxRetries: 3
    });
    
    expect(id).toBeDefined();
    
    const queue = await getOfflineQueue();
    expect(queue).toHaveLength(1);
    expect(queue[0].type).toBe('score_submission');
  });
  
  it('should persist data across page reloads', async () => {
    await addToOfflineQueue({
      type: 'progress_update',
      payload: { progress: 75 },
      maxRetries: 3
    });
    
    // Simulate page reload
    const db = await getDB();
    const queue = await db.getAll('offline-queue');
    
    expect(queue).toHaveLength(1);
  });
});
```

### PWA Installation Testing

Use Playwright for automated PWA testing:

```typescript
// __tests__/e2e/pwa-install.spec.ts
import { test, expect } from '@playwright/test';

test.describe('PWA Installation', () => {
  test('should show install prompt on supported browsers', async ({ page, context }) => {
    await page.goto('/');
    
    // Wait for beforeinstallprompt event
    const promptShown = await page.evaluate(() => {
      return new Promise((resolve) => {
        window.addEventListener('beforeinstallprompt', () => resolve(true));
        setTimeout(() => resolve(false), 5000);
      });
    });
    
    expect(promptShown).toBe(true);
  });
  
  test('should install as PWA', async ({ page, context }) => {
    await context.grantPermissions(['storage-access']);
    await page.goto('/');
    
    // Trigger install
    await page.evaluate(() => {
      (window as any).deferredPrompt?.prompt();
    });
    
    // Verify standalone mode
    const isStandalone = await page.evaluate(() => 
      window.matchMedia('(display-mode: standalone)').matches
    );
    
    // Note: This requires browser support for PWA in headless mode
    // May need manual testing on actual devices
  });
  
  test('should work offline after installation', async ({ page, context }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    // Go offline
    await context.setOffline(true);
    
    // Navigate to a cached page
    await page.goto('/games');
    
    // Should still load from cache
    await expect(page.locator('h1')).toContainText('Games');
  });
});
```

---

## Supporting Documents Referenced

This PWA specifications document draws from the following source documents:

- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility requirements informing PWA API support and Service Worker capabilities across Chrome, Firefox, Edge, Opera, Safari on desktop and mobile platforms
- [MLC_LearningUI_wireframes_201210.pdf](../00-ORIG-CONTEXT/MLC_LearningUI_wireframes_201210.pdf) — Mobile interface wireframes informing PWA design patterns, icon requirements, and offline UX considerations
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game implementation details informing offline game asset caching strategies and storage requirements

---

## Dependencies

### Core Dependencies
- **[PWA Considerations](./pwa-considerations.md)** - Strategic PWA capabilities, behaviors, and UX patterns (this document implements those strategies)
- **[Caching Strategy](../18-architecture/caching-strategy.md)** - Multi-layer caching architecture including Service Worker cache (Layer 1)
- **[Technology Stack](../00-foundations/techstack.md)** - Next.js framework and infrastructure supporting PWA implementation
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)** - Browser support for Service Workers, IndexedDB, and PWA APIs

### Feature Dependencies
- **[Game Model](../08-games-registry-and-authoring/game-model.md)** - Game metadata structure for offline caching
- **[Assignment Model](../07-content-architecture/assignment-model.md)** - Assignment data structure for offline storage
- **[Event Model](../15-analytics-and-reporting/event-model.md)** - Telemetry events for PWA installation and usage tracking

### Technical Dependencies
- **next-pwa** (npm package) - Next.js PWA plugin with Workbox integration
- **idb** (npm package) - Promise-based IndexedDB wrapper
- **Workbox** - Google's Service Worker library (via next-pwa)
- **Service Workers API** - Core PWA functionality
- **IndexedDB API** - Client-side structured data storage
- **Cache API** - Service Worker cache management
- **Storage API** - Quota estimation and persistence

---

## Open Questions

### Implementation Details

- **Q**: Should we use custom Service Worker or rely entirely on next-pwa generation?
  - **Consideration**: Custom SW provides more control but increases maintenance burden
  - **Recommendation**: Start with next-pwa, add custom SW only if needed for specific features

- **Q**: What's the optimal cache size limit per user?
  - **Current thinking**: 200MB default, with warnings at 150MB
  - **Issue**: Varies by device - need device-aware quota management

- **Q**: Should we pre-cache all game assets or cache on-demand?
  - **Options**: 
    1. Pre-cache during installation (large initial download)
    2. Cache on first play (progressive enhancement)
    3. Hybrid: Pre-cache essential, on-demand for optional
  - **Recommendation**: Option 3 - hybrid approach

### Testing and Quality

- **Q**: How to automate PWA testing in CI/CD pipeline?
  - **Challenge**: Service Workers require HTTPS, hard to test in headless browsers
  - **Options**: Use Playwright with local HTTPS or deploy to staging
  
- **Q**: How to handle Service Worker updates in middle of game session?
  - **Current behavior**: Wait for game completion before activating new SW
  - **Needs testing**: Verify no data loss during update

### Analytics and Monitoring

- **Q**: What PWA metrics should we track?
  - **Current**: Installation rate, offline usage, cache hit rate
  - **Additional**: Time-to-interactive in PWA mode, update adoption rate
  
- **Q**: How to debug Service Worker issues in production?
  - **Tools**: Remote debugging, error logging to server, Chrome DevTools
  - **Need**: Centralized Service Worker error reporting

---

## Related Documentation

- [PWA Considerations](./pwa-considerations.md) - Strategic PWA capabilities and behaviors
- [Responsive Behaviors](./responsive-behaviors.md) - Interaction patterns in PWA standalone mode
- [Caching Strategy](../18-architecture/caching-strategy.md) - Multi-layer caching including Service Worker
- [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md) - PWA API support across browsers
- [Technology Stack](../00-foundations/techstack.md) - Next.js and PWA tooling
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game metadata for offline caching
- [Assignment Model](../07-content-architecture/assignment-model.md) - Assignment data for offline storage
