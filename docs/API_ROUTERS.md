# API Routers Documentation

This document provides an overview of the API endpoints available in the Komornicka 100 application.

## Table of Contents

- [Activities Router](#activities-router)
- [Auth Router](#auth-router)
- [Placeholder Router](#placeholder-router)
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

## Placeholder Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/placeholder/{width}/{height}` | GET | Generate a placeholder image with specified dimensions | Unknown | No |

## Strava Router

| Endpoint | Method | Description | Return Type | Requires Auth |
|---------|--------|-------------|-------------|---------------|
| `/strava/auth/{user_id}/{token}` | GET | Handle Strava OAuth callback with authorization code | Dict[str | No |
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

