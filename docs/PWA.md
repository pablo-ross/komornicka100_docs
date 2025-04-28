# Komornicka 100 PWA Implementation

This document explains the Progressive Web App (PWA) implementation for the Komornicka 100 application, with a focus on solving the issues with deep linking and external URLs.

## Overview

The Komornicka 100 app has been configured as a Progressive Web App, allowing users to install it on their devices and use it offline. The PWA implementation includes:

- Service Worker for caching and offline support
- Web App Manifest for installation and app metadata
- Special handling for deep links and authentication flows

## Key Files

- `public/service-worker.js` - Service worker for caching and offline functionality
- `public/pwa.js` - PWA registration script that handles installation and deep linking
- `public/site.webmanifest` - Web app manifest that defines app metadata and behavior
- `components/PWADebugComponent.tsx` - Debug component for testing PWA functionality
- `pages/pwa-test.tsx` - Test page for PWA functionality

## How it Works

### Deep Linking Solution

The PWA implementation uses a special approach to handle deep links and external URLs:

1. **Critical Paths Exclusion**: The service worker specifically excludes certain paths (like authentication, verification, and Strava auth) from being cached. This ensures these critical flows always go to the network.

2. **Navigation Request Handling**: Special handling for navigation requests ensures deep links work correctly, even when the PWA is installed.

3. **External Link Handling**: The PWA script includes code to detect and handle external links correctly, opening them in the browser instead of within the PWA.

### Authentication Flows

The implementation pays special attention to authentication flows, ensuring links in emails and from Strava work correctly:

- Email verification links (`/email-verify/[userId]/[token]`)
- Account deletion links (`/delete/[userId]/[token]`)
- Strava authentication links (`/strava-auth/[userId]/[token]`)

These paths are treated as critical and are never cached, ensuring they always work as expected.

## Testing PWA Functionality

You can test PWA functionality using the built-in `pwa-test.tsx` page, which includes:

1. **PWA Status Tests**: Check if the PWA is installable, already installed, or running in standalone mode.
2. **Deep Link Tests**: Test deep linking with various parameters.
3. **Cache Tests**: Check cache status and allow clearing the cache.
4. **External Link Tests**: Test how external links are handled.

Access this page at: `/pwa-test`

## Troubleshooting Common Issues

### Deep Links Not Working

If deep links aren't working properly:

1. Check if the link path is included in the `CRITICAL_PATHS` array in `service-worker.js`.
2. Test the link using the PWA Test page to see how it's being handled.
3. Clear the cache using the PWA Test page.

### Installation Issues

If users can't install the PWA:

1. Ensure all required icons are available (see the manifest).
2. Use the PWA Test page to check installability.
3. Verify the manifest is loading correctly.

### Strava Authentication Issues

If Strava authentication isn't working:

1. Ensure `/strava-auth` is in the `CRITICAL_PATHS` array.
2. Test the authentication flow in both browser and PWA mode.
3. Check that the service worker isn't interfering with the authentication.

## PWA Requirements

For the PWA to work correctly, ensure:

1. All icon sizes specified in the manifest are available.
2. The service worker is registered correctly.
3. HTTPS is used in production.
4. The manifest is properly configured.

## Icon Requirements

The following icons are required:

- `/apple-touch-icon.png` (180x180)
- `/favicon-16x16.png` (16x16)
- `/favicon-32x32.png` (32x32)
- `/android-chrome-192x192.png` (192x192)
- `/android-chrome-512x512.png` (512x512)
- `/maskable-icon.png` (512x512) - Should include padding for the "safe zone"

## PWA Capability Configuration

You can configure which PWA capabilities are enabled by editing the `PWA_FEATURES` object in `public/pwa.js`:

```javascript
const PWA_FEATURES = {
  enableInstallPrompt: true,   // Let users install the PWA
  enableOffline: true,         // Support working offline
  enableDeepLinks: true,       // Support external deep links
  enableNotifications: false,  // Push notifications (disabled by default)
};
```

## Manual Testing Workflow

When testing the PWA, follow this workflow:

1. Use `/pwa-test` to check PWA status and capabilities.
2. Test installation by clicking the "Install App" button on the PWA test page.
3. Test deep links by generating and opening them through the test page.
4. Test critical paths (authentication, verification, Strava) in both browser and PWA modes.
5. Use the debug component to diagnose issues.