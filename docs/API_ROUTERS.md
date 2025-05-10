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
| `/maps/bounds` | GET | Get bounds for all active source GPX routes | Unknown | No |
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
| `/pois/{poi_id}` | PUT | Update an existing POI | Dict[str | Yes |
| `/pois/{poi_id}` | DELETE | Delete a POI | Dict[str | Yes |
| `/pois/{poi_id}` | GET | Get detailed information about a specific POI | Dict[str | No |
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
| `/activities/user/{user_id}/recent` | GET | No description | List[Dict[str | No |
| `/users/active-strava` | GET | Get all active users with Strava connection | List[Dict[str | Yes |
| `/users/deactivate/{user_id}` | POST | No description | Dict[str | Yes |
| `/users/delete/{user_id}/{token}` | GET | Confirm user deletion with token | Dict[str | No |
| `/users/leaderboard` | GET | Get leaderboard with top users | List[Dict[str | No |
| `/users/process-strava-reminders` | POST | Process and send reminders for users with incomplete Strava authorization | Dict[str | Yes |
| `/users/register` | POST | Register a new user | Any | No |
| `/users/status/{user_id}` | GET | Check the status of a user, including Strava connection status | Dict[str | No |
| `/users/unregister` | POST | Request to unregister and delete user data | Dict[str | No |
| `/users/verify/{user_id}/{token}` | GET | Verify user email with token | Dict[str | No |

