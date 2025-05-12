# Komornicka 100 Admin API Documentation

This document provides comprehensive documentation for the administrative API endpoints available in the Komornicka 100 application.

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Creating an Admin Account](#creating-an-admin-account)
- [Dashboard API](#dashboard-api)
- [User Management API](#user-management-api)
- [Activity Management API](#activity-management-api)
- [Route Management API](#route-management-api)

## Overview

The Admin API provides a comprehensive set of endpoints for managing the Komornicka 100 application. These endpoints are secured and only accessible to authenticated administrators. The API is organized into several logical sections:

- **Dashboard**: Statistics and overview data
- **User Management**: Create, read, update, and deactivate users
- **Activity Management**: View and manage user activities
- **Route Management**: Manage GPX routes for the application

## Authentication

The Admin API uses JWT (JSON Web Token) authentication with access and refresh tokens.

### Login Flow

1. **Obtain Token**:
   ```
   POST /api/admin/auth/token
   ```
   This endpoint accepts form data with username and password, and returns access and refresh tokens.

2. **Refresh Token**:
   ```
   POST /api/admin/auth/refresh
   ```
   This endpoint allows you to obtain a new access token using a valid refresh token.

3. **Using Tokens**:
   For all admin endpoints, include the access token in the Authorization header:
   ```
   Authorization: Bearer <access_token>
   ```

## Creating an Admin Account

Admin accounts can be created using a command-line script. To create a new admin account, run:

```bash
docker compose exec api python scripts/create_admin.py username password
```

or in production:

```bash
docker compose -f docker-compose.prod.yml exec api python scripts/create_admin.py username password
```

Replace `username` and `password` with your desired credentials.

## Dashboard API

### Get Dashboard Statistics

```
GET /api/admin/dashboard/stats
```

Retrieves an overview of system statistics including:
- User counts (total, active, verified, with Strava connection)
- Activity counts (total, last 24h, last 7d, last 30d)
- Route counts (total, active)
- POI counts (total, active)

#### Response example:

```json
{
  "users": {
    "total": 150,
    "active": 120,
    "email_verified": 110,
    "strava_connected": 95
  },
  "activities": {
    "total": 500,
    "last_24h": 15,
    "last_7d": 95,
    "last_30d": 350
  },
  "routes": {
    "total": 5,
    "active": 4
  },
  "pois": {
    "total": 25,
    "active": 22
  },
  "timestamp": "2025-05-11T18:30:00.000Z"
}
```

## User Management API

### List Users

```
GET /api/admin/users
```

Retrieves a paginated list of users with optional filtering.

#### Query Parameters:

- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)
- `active_only`: Filter for active users only (default: false)
- `strava_connected`: Filter by Strava connection status (optional)
- `email_verified`: Filter by email verification status (optional)

### Get User Details

```
GET /api/admin/users/{user_id}
```

Retrieves detailed information about a specific user.

#### Path Parameters:

- `user_id`: UUID of the user

#### Response includes:

- User profile information
- Activity count
- Active tokens
- Recent audit logs

### Search Users

```
POST /api/admin/users/search
```

Search for users with various criteria.

#### Query Parameters:

- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)

#### Request Body:

```json
{
  "email": "string or null",
  "name": "string or null",
  "strava_id": "string or null",
  "strava_username": "string or null",
  "is_active": "boolean or null",
  "is_email_verified": "boolean or null",
  "is_strava_connected": "boolean or null",
  "created_after": "datetime or null",
  "created_before": "datetime or null"
}
```

### Deactivate User

```
POST /api/admin/users/{user_id}/deactivate
```

Deactivate a user account.

#### Path Parameters:

- `user_id`: UUID of the user

#### Query Parameters:

- `reason`: Reason for deactivation (required)

### Reactivate User

```
POST /api/admin/users/{user_id}/reactivate
```

Reactivate a previously deactivated user account.

#### Path Parameters:

- `user_id`: UUID of the user

### Disconnect Strava

```
POST /api/admin/users/{user_id}/disconnect-strava
```

Disconnect a user's Strava integration.

#### Path Parameters:

- `user_id`: UUID of the user

#### Query Parameters:

- `reason`: Reason for disconnection (required)

## Activity Management API

### List Activities

```
GET /api/admin/activities
```

Retrieves a paginated list of verified activities with optional filtering.

#### Query Parameters:

- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)
- `route_id`: Filter by route ID (optional)
- `user_id`: Filter by user ID (optional)
- `from_date`: Filter activities after this date (ISO format, optional)
- `to_date`: Filter activities before this date (ISO format, optional)
- `min_similarity`: Minimum similarity score (0-1, optional)
- `max_similarity`: Maximum similarity score (0-1, optional)

### Get Activity Details

```
GET /api/admin/activities/{activity_id}
```

Retrieves detailed information about a specific activity.

#### Path Parameters:

- `activity_id`: UUID of the activity

#### Response includes:

- Activity details (name, date, distance, etc.)
- User information
- Route information
- GPS path data (if available)
- Verification history

### Revoke Activity

```
POST /api/admin/activities/{activity_id}/revoke
```

Revoke a previously verified activity.

#### Path Parameters:

- `activity_id`: UUID of the activity

#### Query Parameters:

- `reason`: Reason for revocation (required)

### Get Activity Statistics

```
GET /api/admin/activities/stats
```

Retrieves overall activity statistics.

#### Query Parameters:

- `route_id`: Filter by route ID (optional)
- `from_date`: Start date in ISO format (optional)
- `to_date`: End date in ISO format (optional)

#### Response includes:

- Total activities count
- Unique users count
- Average similarity score
- Activities by route
- Activities by date (daily counts)

## Route Management API

### List Routes

```
GET /api/admin/sourcegpxs
```

Retrieves a list of all routes.

#### Query Parameters:

- `include_inactive`: Whether to include inactive routes (default: false)

### Get Route Details

```
GET /api/admin/sourcegpxs/{route_id}
```

Retrieves detailed information about a specific route.

#### Path Parameters:

- `route_id`: UUID of the route

#### Response includes:

- Route details (name, distance, etc.)
- Usage statistics
- Path data for visualization

### Activate Route

```
POST /api/admin/sourcegpxs/{route_id}/activate
```

Activate an inactive route.

#### Path Parameters:

- `route_id`: UUID of the route

### Deactivate Route

```
POST /api/admin/sourcegpxs/{route_id}/deactivate
```

Deactivate an active route.

#### Path Parameters:

- `route_id`: UUID of the route

### Upload Route

```
POST /api/admin/sourcegpxs/upload
```

Upload a new GPX route file.

#### Query Parameters:

- `name`: Route name (optional, defaults to filename)
- `description`: Route description (optional)

#### Request Body:

- Multipart form data with a file field containing the GPX file