# API Gateway Implementation with Nginx

This document explains the implementation of an API Gateway using Nginx for the Komornicka 100 application.

## Overview

We've enhanced the existing Nginx configuration to act as a proper API Gateway, implementing:

1. Security headers
2. Rate limiting
3. IP blocking
4. Request validation
5. CORS handling
6. Improved error handling

## Configuration Files

There are two primary configuration files:

1. `nginx.conf` - Main Nginx configuration
2. `default.conf` - Server-specific configuration with routes and security rules

## Security Features Implemented

### 1. Security Headers

The following security headers have been added to all responses:

- **X-Content-Type-Options**: Prevents MIME type sniffing
- **X-Frame-Options**: Prevents clickjacking attacks
- **X-XSS-Protection**: Enables browser XSS protection
- **Referrer-Policy**: Controls information sent in the Referer header
- **Content-Security-Policy**: Restricts resource loading to prevent XSS
- **Strict-Transport-Security**: Forces HTTPS connections

### 2. Rate Limiting

Two rate limiting zones have been implemented:

- **API Limit**: 10 requests per second with burst of 20
- **Frontend Limit**: 20 requests per second with burst of 50

This prevents abuse and helps mitigate DDoS attacks.

### 3. IP Blocking

A geo-based IP blocking mechanism is in place:

```nginx
geo $block {
    default 0;
    # Example: 192.168.1.100 1;
}
```

Add known malicious IPs to this list to block them from accessing your site.

### 4. Request Validation

Request validation is implemented through:

- Blocking suspicious query parameters that might indicate SQL injection or path traversal
- Blocking suspicious user agents associated with scanning tools and bots

### 5. CORS Configuration

CORS headers are added to allow controlled cross-origin requests to your API.

## Maintenance

### Adding Blocked IPs

To block an IP address:

1. Edit the `geo $block` block in `nginx.conf`
2. Add the IP with value 1: `123.45.67.89 1;`
3. Reload Nginx: `docker compose exec nginx nginx -s reload`

### Adjusting Rate Limits

If rate limits are too restrictive or too permissive:

1. Modify the values in the `limit_req_zone` directives in `nginx.conf`
2. Adjust the burst parameters in the `limit_req` directives in `default.conf`
3. Reload Nginx

### Updating Security Headers

To update the Content Security Policy or other security headers:

1. Modify the `add_header` directives in `default.conf`
2. Ensure the CSP allows all legitimate resources your app needs
3. Reload Nginx

## Monitoring and Troubleshooting

### Viewing Logs

Nginx logs are stored in:
- Access logs: `/var/log/nginx/access.log`
- Error logs: `/var/log/nginx/error.log`

View logs with:
```bash
docker compose exec nginx tail -f /var/log/nginx/access.log
docker compose exec nginx tail -f /var/log/nginx/error.log
```

### Testing Security Headers

Use a service like [securityheaders.com](https://securityheaders.com) to test your security headers.

### Testing Rate Limiting

To test rate limiting, use a tool like Apache Bench:

```bash
sudo apt-get install apache2-utils
ab -n 100 -c 10 https://komornicka100.pl/api/health
```

If rate limiting is working, you'll see 429 Too Many Requests responses.

## Future Enhancements

Consider these enhancements to further improve your API Gateway:

1. **JWT Validation**: Validate JWTs at the gateway level
2. **Request/Response Transformation**: Modify requests and responses at the gateway
3. **OAuth Integration**: Add OAuth authentication at the gateway level
4. **Advanced Monitoring**: Integrate with monitoring tools
5. **Custom Error Pages**: Create user-friendly error pages for 403, 404, and 5xx errors

## References

- [Nginx Documentation](https://nginx.org/en/docs/)
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)