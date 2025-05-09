# Static CDN Implementation Guide

This document explains how the static CDN is implemented in the Komornicka 100 application, including configuration details, security considerations, and usage examples.

## Overview

The Komornicka 100 application uses a dedicated Content Delivery Network (CDN) based on Lighttpd to serve static assets such as:

- GPX route files
- Images
- Downloadable resources

This architecture separates static content delivery from the application logic, providing:
1. Better performance for static assets
2. Reduced load on the main application server
3. Optimized caching based on content type
4. Simplified management of static resources

## Architecture

The CDN consists of:

1. **Lighttpd Server**: A lightweight, high-performance web server optimized for static content
2. **Nginx Proxy**: Handles requests to cdn.komornicka100.pl and applies security policies
3. **Static Directory**: Organized structure for GPX files, images, and downloads
4. **Security Layer**: Implements headers, rate limiting, and other protections

### System Interaction Flow

```
User → Nginx Proxy → Lighttpd Server → Static Files
```

In development:
- CDN is accessible at http://localhost:8888

In production:
- CDN is accessible at https://cdn.komornicka100.pl

## Implementation Details

### Directory Structure

```
static/
├── gpx/          # GPX route files
├── images/       # Images and graphics
├── downloads/    # Downloadable resources (PDFs, etc.)
└── index.html    # Basic landing page
```

### Container Configuration

The CDN is implemented as a Docker container:

- **Container name**: k100-cdn
- **Base image**: sebp/lighttpd
- **Local port**: 8888 (development)
- **Network**: Accessible through the k100 and proxy networks

### Lighttpd Configuration

The Lighttpd server is configured with:

1. **MIME Types**: Proper content types for all supported file formats
2. **Directory Listing**: Clean listing of directory contents
3. **Compression**: For text-based formats (HTML, CSS, JavaScript, XML, JSON)
4. **Caching**: File-type specific cache expiration times
5. **CORS Headers**: Cross-origin resource sharing support

Key configuration settings:

```
# Mime type for GPX files
".gpx" => "application/gpx+xml"

# Caching for static files
$HTTP["url"] =~ "\.(jpg|jpeg|gif|png|svg|css|js|gpx|zip|ico|webp)$" {
  expire.url = ( "" => "access plus 7 days" )
}

# CORS headers
setenv.add-response-header = (
  "Access-Control-Allow-Origin" => "*",
  "Access-Control-Allow-Methods" => "GET, OPTIONS",
  "Access-Control-Allow-Headers" => "Origin, X-Requested-With, Content-Type, Accept, Range",
  "Cache-Control" => "public, max-age=604800"
)
```

### Nginx Configuration

The Nginx proxy:

1. Routes requests to cdn.komornicka100.pl to the Lighttpd container
2. Applies security headers and policies
3. Manages rate limiting
4. Provides custom error pages
5. Configures content-specific caching

Key security headers:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'none'; img-src 'self'; style-src 'self'; script-src 'none'; connect-src 'none'; font-src 'none'; object-src 'none'; media-src 'self'; frame-ancestors 'none'" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

## Security Considerations

The CDN implementation includes several security measures:

### 1. Content Security Policy (CSP)

The main application's CSP has been updated to allow loading resources from the CDN:

```nginx
img-src 'self' data: ... https://cdn.komornicka100.pl;
connect-src 'self' ... https://cdn.komornicka100.pl;
```

The CDN itself has a strict CSP that only allows loading of appropriate resources.

For the CDN domain, inline styles are permitted to allow the index page to render properly:

```nginx
# CDN Content Security Policy with unsafe-inline for styles
add_header Content-Security-Policy "default-src 'none'; img-src 'self'; style-src 'self' 'unsafe-inline'; script-src 'none'; connect-src 'none'; font-src 'none'; object-src 'none'; media-src 'self'; frame-ancestors 'none'" always;
```

> **Note**: For enhanced security, you can replace `'unsafe-inline'` with specific hashes of your inline styles. For example: `style-src 'self' 'sha256-QGTUUZ6ybKBGbNW16iP7fkyh6J/DF+BbXZBOjKVHtOI='`

### 2. Cross-Origin Resource Sharing (CORS)

CORS headers allow the main application to load resources from the CDN domain while preventing unauthorized access from other domains.

### 3. Rate Limiting

```nginx
# Rate limiting with a higher burst for static content
limit_req zone=frontend_limit burst=100 nodelay;
```

This prevents abuse while allowing legitimate traffic spikes.

### 4. HTTP Security Headers

A comprehensive set of security headers protects against common attacks:
- MIME type confusion
- Clickjacking
- XSS attacks
- Information leakage

### 5. Input Validation

The Nginx configuration blocks suspicious query strings and user agents that might indicate attack attempts.

### 6. Cache Control

Different cache policies are applied based on file type:
- Images: 30 days
- GPX files: 7 days
- ZIP files: 7 days

## Usage Guide

### Accessing CDN Resources

Resources on the CDN can be accessed via:
- Development: `http://localhost:8888/{resource-path}`
- Production: `https://cdn.komornicka100.pl/{resource-path}`

### Using CDN URLs in Code

#### In Next.js Components:

```tsx
import { useEffect, useState } from 'react';

export default function RoutePage() {
  const [routeGpx, setRouteGpx] = useState(null);
  
  useEffect(() => {
    // Using the environment variable for CDN URL
    const cdnUrl = process.env.NEXT_PUBLIC_CDN_URL || '';
    
    // Fetch the GPX file from CDN
    fetch(`${cdnUrl}/gpx/komornicka100-route.gpx`)
      .then(response => response.text())
      .then(data => {
        setRouteGpx(data);
      })
      .catch(error => console.error('Error loading route GPX:', error));
  }, []);
  
  // Component rendering...
}
```

#### For Images:

```tsx
import Image from 'next/image';

export function Hero() {
  const cdnUrl = process.env.NEXT_PUBLIC_CDN_URL || '';
  
  return (
    <Image 
      src={`${cdnUrl}/images/hero-banner.jpg`} 
      alt="Komornicka 100"
      width={1200}
      height={600}
    />
  );
}
```

#### For Downloads:

```tsx
export function Downloads() {
  const cdnUrl = process.env.NEXT_PUBLIC_CDN_URL || '';
  
  return (
    <div className="downloads">
      <h3>Downloads</h3>
      <ul>
        <li>
          <a href={`${cdnUrl}/downloads/race-manual.pdf`} download>
            Race Manual (PDF)
          </a>
        </li>
        <li>
          <a href={`${cdnUrl}/gpx/komornicka100-route.gpx`} download>
            Official Route (GPX)
          </a>
        </li>
      </ul>
    </div>
  );
}
```

### Adding Files to the CDN

To add files to the CDN, place them in the appropriate directory in the `static/` folder:

```bash
# Add GPX files
cp your-route.gpx static/gpx/

# Add images
cp logo.png static/images/

# Add downloadable files
cp race-manual.pdf static/downloads/
```

## Configuration Reference

### Environment Variables

```
# Development
NEXT_PUBLIC_CDN_URL=http://localhost:8888

# Production
NEXT_PUBLIC_CDN_URL=https://cdn.komornicka100.pl
```

### File Type Support

The CDN is configured to handle these file types:

| File Extension | MIME Type | Cache Duration |
|----------------|-----------|----------------|
| .html, .htm | text/html | 1 day |
| .css | text/css | 7 days |
| .js | application/javascript | 7 days |
| .jpg, .jpeg | image/jpeg | 30 days |
| .png | image/png | 30 days |
| .gif | image/gif | 30 days |
| .svg | image/svg+xml | 30 days |
| .webp | image/webp | 30 days |
| .ico | image/x-icon | 30 days |
| .gpx | application/gpx+xml | 7 days |
| .json | application/json | 7 days |
| .xml | application/xml | 7 days |
| .zip | application/zip | 7 days |
| .pdf | application/pdf | 7 days |

## Troubleshooting

### Common Issues

1. **CORS errors when accessing CDN resources**:
   - Check that the CORS headers are properly configured in `lighttpd.conf`
   - Verify that the Content Security Policy in the main application allows the CDN domain
   
2. **Content Security Policy blocking inline styles**:
   - If you see errors like: `Refused to apply inline style because it violates the following Content Security Policy directive: "style-src 'self'"`, you need to update the CSP in `nginx/conf.d/cdn.conf`
   - Add `'unsafe-inline'` to the style-src directive: `style-src 'self' 'unsafe-inline'`
   - Or use a specific hash for the inline style: `style-src 'self' 'sha256-[hash]'`
   - Restart Nginx after making changes: `docker compose restart nginx`

2. **CDN not accessible in production**:
   - Verify DNS configuration for cdn.komornicka100.pl
   - Check Nginx configuration in `nginx/conf.d/cdn.conf`
   - Make sure the cdn container is running and healthy

3. **Files not appearing with correct MIME type**:
   - Check MIME type configuration in `lighttpd.conf`
   - Verify file extension is correctly mapped

4. **Slow loading of CDN resources**:
   - Check cache control headers
   - Verify compression is enabled for the file type
   - Check network latency to the CDN server

### Checking Logs

To check CDN logs:

```bash
# View Lighttpd access logs
docker compose logs -f cdn

# View Nginx proxy logs for CDN requests
docker compose exec nginx tail -f /var/log/nginx/access.log | grep cdn.komornicka100.pl
```

## Maintenance

### Regular Maintenance Tasks

1. **Update static content**:
   - Keep GPX files up to date
   - Optimize images for web
   - Update downloadable resources as needed

2. **Security updates**:
   - Keep Lighttpd and Nginx images updated
   - Review security headers periodically
   - Scan for vulnerabilities

3. **Monitoring**:
   - Check access logs for unusual patterns
   - Monitor disk usage for static files
   - Verify cache performance

## Future Improvements

Potential enhancements for the CDN system:

1. **Content replication**: Set up multiple CDN nodes for geographic distribution
2. **Image optimization**: Add automatic image resizing and optimization
3. **Advanced caching**: Implement cache invalidation mechanisms
4. **Access control**: Add authentication for protected resources
5. **Analytics**: Implement detailed access statistics for CDN resources