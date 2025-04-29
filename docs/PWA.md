# Komornicka 100 PWA Implementation Guide

This document explains the Progressive Web App (PWA) implementation for the Komornicka 100 application, with a focus on offline capabilities, installation, and handling authentication flows.

## Overview

The Komornicka 100 app is configured as a Progressive Web App, allowing users to install it on their devices and use portions of it offline. The implementation uses Next.js with next-pwa to provide PWA functionality.

## Key Files

- `frontend/next.config.js` - Contains the next-pwa configuration
- `frontend/public/manifest.json` - Defines app metadata and installation behavior
- `frontend/public/site.webmanifest` - Alternative manifest format for broader compatibility
- `frontend/components/PWAInstaller.tsx` - Handles app installation prompts
- `frontend/components/PWADebugComponent.tsx` - Debug component for testing PWA functionality
- `frontend/pages/pwa-test.tsx` - Test page for PWA features and diagnostics
- `frontend/pages/_document.tsx` - Contains PWA-related meta tags and link elements
- `frontend/public/offline.html` - Fallback page shown when offline

## PWA Configuration

The app uses `next-pwa` to handle service worker generation and registration. The configuration in `next.config.js` includes:

```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development',
  register: true,
  skipWaiting: true,
  sw: 'sw.js',
  buildExcludes: [/middleware-manifest\.json$/],
  fallbacks: {
    document: '/offline.html'
  },
  cacheOnFrontEndNav: true,
  dynamicStartUrl: true,
  reloadOnOnline: false,
  swSrc: './worker/index.js'
});
```

This configuration:
- Disables the PWA in development mode
- Registers the service worker automatically
- Uses `/offline.html` as a fallback when the user is offline
- Enables caching during front-end navigation
- Uses a custom service worker source file

## Web App Manifest

The app includes two manifest files for maximum compatibility:

1. `manifest.json` - Primary manifest file
2. `site.webmanifest` - Alternative format for broader browser support

Both files define:
- App name and description
- Icons in various sizes for different platforms
- Theme and background colors
- Display mode (standalone)
- Start URL and scope
- Shortcuts for quick access to key features

## PWA Capabilities

The current implementation provides these key capabilities:

- **Installability**: Users can install the app to their home screen
- **Offline Support**: Core pages and assets are cached for offline use
- **App-like Experience**: Runs in standalone mode without browser UI
- **Deep Linking**: Supports navigation to specific pages via URL parameters

## Authentication Flows and PWA

Special consideration has been given to authentication flows:

- Email verification links (`/email-verify/[userId]/[token]`)
- Account deletion links (`/delete/[userId]/[token]`)
- Strava authentication links (`/strava-auth/[userId]/[token]`)

These critical paths are configured to bypass caching to ensure they always work as expected.

## PWA Installation Component

The `PWAInstaller.tsx` component:
- Detects if the app is installable
- Shows an installation prompt on mobile devices
- Only displays in production environments
- Disappears if the app is already installed

## Testing PWA Functionality

You can test PWA functionality using the built-in `pwa-test.tsx` page, which includes:

1. **PWA Status Tests**: Check if the PWA is installable or already installed
2. **Manifest Verification**: Confirms the manifest file is available
3. **Service Worker Checks**: Verifies service worker registration
4. **Deep Link Tests**: Test navigation with URL parameters
5. **Cache Management**: Check cache status and clear caches
6. **External Link Tests**: Test how external links are handled

Access this page at: `/pwa-test`

## PWA Debug Component

The `PWADebugComponent.tsx` component provides detailed diagnostic information:

- Current platform detection
- Service worker status and URL
- Manifest availability
- Installation status
- Icon availability checks

## Required Icons

For full PWA support, the following icons are required:

- `/apple-touch-icon.png` (180×180)
- `/favicon-16x16.png` (16×16)
- `/favicon-32x32.png` (32×32)
- `/favicon.ico` (multiple sizes)
- `/android-chrome-192x192.png` (192×192)
- `/android-chrome-512x512.png` (512×512)
- `/maskable-icon.png` (512×512 with safe zone padding)

## Offline Fallback

The `/offline.html` page provides a graceful experience when users are offline, explaining:
- That the user is currently offline
- How to retry connecting
- Which features may still be available offline

## Troubleshooting Common Issues

### Installation Issues

If users cannot install the PWA:

1. Verify all required icons are present
2. Ensure the manifest is properly configured
3. Check for any console errors related to service worker registration
4. Confirm the app is being served over HTTPS in production

### Service Worker Problems

For service worker issues:

1. Clear the browser cache and service workers
2. Check browser support for service workers
3. Verify the service worker is registering correctly
4. Use the PWA Test page to inspect service worker status

### Deep Link Issues

If deep links aren't working properly:

1. Test links both in browser mode and standalone PWA mode
2. Verify URL structure and parameters
3. Check for any caching issues affecting dynamic content

## PWA Best Practices Implemented

The current implementation follows these PWA best practices:

- **Responsive Design**: Works on all device sizes
- **App-like Interface**: Full-screen UI in standalone mode
- **Custom Offline Page**: Graceful degradation when offline
- **Fast Load Times**: Core assets are cached for quick loading
- **Theme-color Support**: Consistent color scheme in the system UI
- **Installable**: Meets criteria for Add to Home Screen
- **Mobile-First**: Optimized for mobile devices