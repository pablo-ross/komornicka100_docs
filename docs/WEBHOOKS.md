# Webhook Integration Guide

Komornicka 100 supports webhooks that allow you to receive real-time notifications about important events in the system. This document explains how to configure and use webhooks.

## Overview

Webhooks are HTTP callbacks that are triggered when specific events occur in the Komornicka 100 system. They allow you to integrate with external systems such as:

- Automation platforms (n8n, Make, Zapier)
- Notification services
- Custom applications
- Analytics systems

## Supported Webhook Events

The following events trigger webhooks:

| Event | Description | Webhook Type |
|-------|-------------|--------------|
| User Registration | Triggered when a user registers or reactivates their account | `user_registered` |
| Strava Connection | Triggered when a user connects their Strava account | `user_connected_strava` |
| User Deactivation | Triggered when a user is deactivated | `user_deactivated` |
| Activity Approval | Triggered when a Strava activity is approved | `strava_activity_approved` |
| Activity Revocation | Triggered when an approved activity is revoked | `strava_activity_revoked` |

## Configuration

Webhooks are configured through the main `.env` file in the root directory of the project. 

1. Use the example file `.env.example` to create your configuration:
   ```bash
   cp .env.example .env
   ```

2. Edit the file to add your webhook URLs:
   ```
   WEBHOOK_USER_REGISTERED=https://your-webhook-url.com/path
   WEBHOOK_USER_CONNECTED_STRAVA=https://your-webhook-url.com/path
   WEBHOOK_USER_DEACTIVATED=https://your-webhook-url.com/path
   WEBHOOK_STRAVA_ACTIVITY_APPROVED=https://your-webhook-url.com/path
   WEBHOOK_STRAVA_ACTIVITY_REVOKED=https://your-webhook-url.com/path
   ```

3. If your webhook endpoints require authentication, configure the credentials:
   ```
   WEBHOOK_BASIC_AUTH_USER=your_username
   WEBHOOK_BASIC_AUTH_PASS=your_password
   ```

4. Restart the application for the changes to take effect.

## Webhook Payload Format

All webhooks follow a consistent payload format:

```json
{
  "webhook_type": "event_type",
  "data": {
    // Event-specific data...
  }
}
```

### User Registered Payload

```json
{
  "webhook_type": "user_registered",
  "data": {
    "user_id": "uuid-string",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "age": 35,
    "event": "new_registration", // or "reactivated"
    "registered_at": "2025-04-23T10:15:30.123Z"
  }
}
```

### User Connected Strava Payload

```json
{
  "webhook_type": "user_connected_strava",
  "data": {
    "user_id": "uuid-string",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "strava_id": "12345678",
    "strava_username": "johndoe",
    "connected_at": "2025-04-23T10:15:30.123Z"
  }
}
```

### User Deactivated Payload

```json
{
  "webhook_type": "user_deactivated",
  "data": {
    "user_id": "uuid-string",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "strava_id": "12345678",
    "strava_username": "johndoe",
    "reason": "User requested account deletion",
    "deactivated_at": "2025-04-23T10:15:30.123Z"
  }
}
```

### Activity Approved Payload

```json
{
  "webhook_type": "strava_activity_approved",
  "data": {
    "user_id": "uuid-string",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "activity_id": "uuid-string",
    "strava_activity_id": "1234567890",
    "activity_name": "Morning Ride",
    "activity_date": "2025-04-23T08:30:00.000Z",
    "distance_km": 100.5,
    "duration_seconds": 12345,
    "route_name": "Komornicka 100 Loop",
    "similarity_score": 0.95,
    "verified_at": "2025-04-23T10:15:30.123Z"
  }
}
```

### Activity Revoked Payload

```json
{
  "webhook_type": "strava_activity_revoked",
  "data": {
    "activity_id": "uuid-string",
    "user_id": "uuid-string",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "activity_name": "Morning Ride",
    "activity_date": "2025-04-23T08:30:00.000Z",
    "strava_activity_id": "1234567890",
    "route_name": "Komornicka 100 Loop",
    "reason": "Activity no longer visible on Strava",
    "revoked_at": "2025-04-23T14:20:15.456Z"
  }
}
```

## Testing Webhooks

You can test the webhook functionality using the included test script:

```bash
./.scripts/test-webhooks.sh
```

This will send test payloads to all configured webhook endpoints.

To test specific webhooks:

```bash
# Test just the user registration webhook
./.scripts/test-webhooks.sh --registered

# Test just the activity approval webhook
./.scripts/test-webhooks.sh --approved

# Test multiple specific webhooks
./.scripts/test-webhooks.sh --registered --connected
```

## Webhook Response Handling

Your webhook endpoint should:

1. Respond with a 2xx HTTP status code (200-299) to acknowledge receipt
2. Process the webhook asynchronously if complex processing is required
3. Handle duplicates (use the event timestamp to deduplicate)

If a webhook fails to deliver (non-2xx response), the system will not retry by default.

## Security Considerations

1. Use HTTPS for all webhook endpoints
2. Configure basic authentication in the `.env` file
3. Validate the webhook data on your receiving end
4. Consider implementing a shared secret if your endpoint requires additional verification

## Integrating with Common Platforms

### n8n

1. Create a new workflow in n8n
2. Add a "Webhook" trigger node
3. Configure it as a "Passive Webhook" and copy the URL
4. Paste the URL into your `.env` file
5. Use JSON parsing nodes to extract the webhook data

### Zapier

1. Create a new Zap
2. Choose "Webhook" as the trigger
3. Select "Catch Hook" and copy the webhook URL
4. Paste the URL into your `.env` file
5. Configure the action steps to process the webhook data

### Custom Applications

If you're building a custom application to receive webhooks:

1. Create an endpoint that accepts POST requests
2. Parse the JSON payload
3. Validate the webhook data
4. Process the event accordingly
5. Return a 200 OK response

## Troubleshooting

If you're having issues with webhooks:

1. Check your webhook logs in the Komornicka 100 application (`docker compose logs worker`)
2. Verify the `.env` file contains the correct URLs
3. Ensure your webhook endpoint is accessible from the Komornicka 100 server
4. Test the webhooks using the test script
5. Check the network connectivity between the server and your webhook endpoint

For any additional questions, please contact the system administrator.