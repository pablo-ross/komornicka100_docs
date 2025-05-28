# API Routers Documentation

This document provides an overview of the API endpoints available in the Komornicka 100 application.

## Table of Contents

- [Activities Router](#activities-router)
- [Auth Router](#auth-router)
- [Maps Router](#maps-router)
- [Placeholder Router](#placeholder-router)
- [Pois Router](#pois-router)
- [Strava Router](#strava-router)
- [Users Router](#users-router)

### Admin Routers

- [Activities Admin Router](#admin_activities-router)
- [Auditlogs Admin Router](#admin_auditlogs-router)
- [Auth Admin Router](#admin_auth-router)
- [Dashboard Admin Router](#admin_dashboard-router)
- [Maps Admin Router](#admin_maps-router)
- [Pois Admin Router](#admin_pois-router)
- [Sourcegpxs Admin Router](#admin_sourcegpxs-router)
- [Users Admin Router](#admin_users-router)

## Activities Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/activities/check/{strava_activity_id}` | GET | Check if a Strava activity exists and its verification status | Dict[str | No |
| `/activities/latest` | GET | Get the latest verified activities from active users | Unknown | No |
| `/activities/leaderboard` | GET | Get leaderboard with top users by activity count | Unknown | No |
| `/activities/revoke/{activity_id}` | POST | No description | Dict[str | Yes |
| `/activities/user/{user_id}` | GET | Get all verified activities for a user | Unknown | No |
| `/activities/{activity_id}` | GET | Get detailed information about a specific activity | Unknown | No |
| `/activities/{activity_id}/streams` | GET | Get GPS streams data for a specific activity | Unknown | No |

## Auth Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/auth/settings` | GET | Get public application settings for the frontend | Unknown | No |

## Maps Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/maps/bounds` | GET | Get bounds for all active source GPX routes | Unknown | Yes |
| `/maps/cache/clear` | POST | Clear the path cache | Unknown | Yes |
| `/maps/source/by-filename/{filename}/path` | GET | Get the path data for a specific source GPX by filename | Unknown | No |
| `/maps/source/{source_id}/path` | GET | Get the path data for a specific source GPX | Unknown | No |
| `/maps/sources` | GET | Get all active source GPX routes for maps | Unknown | No |

## Placeholder Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/placeholder/{width}/{height}` | GET | Generate a placeholder image with specified dimensions | Unknown | No |

## Pois Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/pois` | POST | Create a new POI | Dict[str | Yes |
| `/pois/bounds` | POST | No description | List[Dict[str | No |
| `/pois/cache/clear` | POST | Clear the POIs route cache | Dict[str | Yes |
| `/pois/cache/stats` | GET | Get statistics about the POIs route cache | Dict[str | Yes |
| `/pois/list` | GET | No description | List[Dict[str | No |
| `/pois/source/by-filename/{filename}` | GET | No description | List[Dict[str | No |
| `/pois/source/{source_id}` | GET | No description | List[Dict[str | No |
| `/pois/webhook/update-gpx` | POST | Trigger an update of GPX files with POIs | Dict[str | Yes |
| `/pois/{poi_id}` | GET | Get detailed information about a specific POI | Dict[str | No |
| `/pois/{poi_id}` | PUT | Update an existing POI | Dict[str | Yes |
| `/pois/{poi_id}` | DELETE | Delete a POI | Dict[str | Yes |
| `/pois/{poi_id}/deactivate` | PATCH | Deactivate a POI | Dict[str | Yes |

## Strava Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/strava/auth/{user_id}/{token}` | GET | Handle Strava OAuth callback with authorization code | Dict[str | No |
| `/strava/deactivate/{user_id}` | POST | No description | Dict[str | Yes |
| `/strava/token/{user_id}` | GET | Get a fresh Strava access token for a user | Dict[str | Yes |
| `/strava/webhook` | POST | Handle Strava webhook events (not implemented in this version) | Dict[str | No |
| `/strava/webhook` | GET | Verify Strava webhook subscription | Dict[str | No |

## Users Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/activities/user/{user_id}/recent` | GET | Get all verified activities for a user within the specified time period | List[Dict[str | No |
| `/users/active-strava` | GET | Get all active users with Strava connection | List[Dict[str | Yes |
| `/users/deactivate/{user_id}` | POST | Deactivate a user's account | Dict[str | Yes |
| `/users/delete/{user_id}/{token}` | GET | Confirm user deletion with token | Dict[str | No |
| `/users/leaderboard` | GET | Get leaderboard with top users | List[Dict[str | No |
| `/users/process-strava-reminders` | POST | Process and send reminders for users with incomplete Strava authorization | Dict[str | Yes |
| `/users/register` | POST | Register a new user | Any | No |
| `/users/status/{user_id}` | GET | Check the status of a user, including Strava connection status | Dict[str | Yes |
| `/users/unregister` | POST | Request to unregister and delete user data | Dict[str | No |
| `/users/verify/{user_id}/{token}` | GET | Verify user email with token | Dict[str | No |

# Admin API Routers

These routes are used for administrative purposes and require authentication.

## Activities Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/activities/stats` | GET | No description | Unknown | Yes |
| `/activities/{activity_id}` | GET | Get detailed information about a specific activity | Unknown | Yes |
| `/activities/{activity_id}/revoke` | POST | No description | Unknown | Yes |

## Auditlogs Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/auditlogs/event-types` | GET | Get a list of all unique event types in the audit logs | Unknown | Yes |
| `/auditlogs/search` | POST | No description | Unknown | Yes |
| `/auditlogs/{log_id}` | GET | Get detailed information about a specific audit log | Unknown | Yes |

## Auth Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/auth/logout` | POST | Logout the current user by clearing all auth cookies | Unknown | Yes |
| `/auth/refresh` | POST | No description | Unknown | Yes |
| `/auth/token` | POST | No description | Unknown | Yes |
| `/auth/validate` | GET | Validate current session token | Unknown | Yes |

## Dashboard Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/dashboard/stats` | GET | Get dashboard statistics for admin panel | Unknown | Yes |

## Maps Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/maps/cache/clear` | POST | No description | Unknown | Yes |

## Pois Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/pois/cache/clear` | POST | No description | Unknown | Yes |
| `/pois/cache/stats` | GET | No description | Unknown | Yes |
| `/pois/webhook/update-gpx` | POST | No description | Unknown | Yes |
| `/pois/{poi_id}` | PUT | No description | Unknown | Yes |
| `/pois/{poi_id}` | DELETE | No description | Unknown | Yes |
| `/pois/{poi_id}/deactivate` | PATCH | No description | Unknown | Yes |

## Sourcegpxs Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/sourcegpxs/upload` | POST | No description | Unknown | Yes |
| `/sourcegpxs/{route_id}` | GET | Get detailed information about a specific route | Unknown | Yes |
| `/sourcegpxs/{route_id}/activate` | POST | Activate a route | Unknown | Yes |
| `/sourcegpxs/{route_id}/deactivate` | POST | Deactivate a route | Unknown | Yes |

## Users Admin Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/users/search` | POST | No description | Unknown | Yes |
| `/users/{user_id}` | GET | Get detailed information about a specific user | Unknown | Yes |
| `/users/{user_id}/deactivate` | POST | No description | Unknown | Yes |
| `/users/{user_id}/disconnect-strava` | POST | No description | Unknown | Yes |
| `/users/{user_id}/reactivate` | POST | Reactivate a previously deactivated user account | Unknown | Yes |

