# Worker Job Schedules

This document describes all scheduled jobs in the Komornicka 100 worker service.

## Overview

The worker service runs on a schedule-based system using the Python `schedule` library. All jobs respect the master switch `K100_ENABLED` and most jobs only run during operating hours (06:00 - 22:00 CEST/CET).

## Master Controls

### K100_ENABLED Setting
- **Environment Variable**: `K100_ENABLED`
- **Default**: `true`
- **Purpose**: Master switch to enable/disable all worker operations
- **Behavior**: When `false`, worker runs but performs no operations

### Operating Hours
- **Active Hours**: 06:00 - 22:00 (local time, respects TZ setting)
- **Timezone**: Configurable via `TZ` environment variable (default: `Europe/Warsaw`)
- **Exceptions**: Monthly reports can run outside operating hours

## Scheduled Jobs

### 1. Activity Processing
- **Schedule**: Every 2 hours at :21 minutes
- **Function**: `job()`
- **Operating Hours**: ✅ Respects 06:00-22:00 limit
- **Purpose**: 
  - Fetches new Strava activities for all eligible users
  - Verifies activities against source GPX routes
  - Records verified activities in the database
  - Updates user activity check timestamps

### 2. Daily Strava Verification
- **Schedule**: Daily at 08:30
- **Function**: `daily_strava_check_job()`
- **Operating Hours**: ✅ Respects 06:00-22:00 limit
- **Purpose**: 
  - Checks user Strava connections are still valid
  - Checks activity availability on Strava
  - Deactivates users with invalid tokens
  - Revokes activities no longer available

### 3. Strava Reminders
- **Schedule**: Every hour at :11 minutes
- **Function**: `strava_reminder_job()`
- **Operating Hours**: ✅ Respects 06:00-22:00 limit
- **Purpose**:
  - Sends reminder emails to users who haven't connected Strava
  - Implements tiered reminder system (30 minutes, 1 day, 7 days)

### 4. Activity Attempts Cleanup
- **Schedule**: Weekly on Mondays at 03:15
- **Function**: `cleanup_activity_attempts_job()`
- **Operating Hours**: ✅ Respects 06:00-22:00 limit
- **Purpose**:
  - Cleans up old activity attempt records
  - Removes short activities older than 7 days
  - Removes normal activities older than 180 days
  - Manages database size

### 5. Monthly Reports
- **Schedule**: Daily at 11:00 (runs only on 1st day of month)
- **Function**: `monthly_report_job_wrapper()`
- **Operating Hours**: ❌ Can run outside operating hours
- **Purpose**:
  - Generates monthly activity reports
  - Creates Excel attachments
  - Sends reports to configured recipients
  - Only executes on the 1st day of each month

## Schedule Configuration

### Time Format
All times use 24-hour format and are interpreted in the local timezone (TZ setting).

### Examples from Code
```python
# Every 2 hours at :21 minutes
schedule.every(2).hours.at(":21").do(job)

# Daily at 08:30
schedule.every().day.at("08:30").do(daily_strava_check_job)

# Every hour at :11 minutes  
schedule.every().hour.at(":11").do(strava_reminder_job)

# Weekly on Monday at 03:15
schedule.every().monday.at("03:15").do(cleanup_activity_attempts_job)

# Daily at 11:00 (with monthly filter)
schedule.every().day.at("11:00").do(monthly_report_job_wrapper)
```

## Manual Execution Scripts

The following scripts are available for manual execution:

### Activity Processing
- **Script**: `worker/scripts/run_strava_checker.py`
- **Purpose**: Manually run Strava verification checks
- **Options**: `--users-only`, `--activities-only`

### Monthly Reports
- **Script**: `worker/scripts/run_monthly_report.py`
- **Purpose**: Manually generate monthly reports
- **Options**: `--year`, `--month`, `--prev-month`

### GPX Verification Testing
- **Script**: `worker/scripts/test_gpx_verification.py`
- **Purpose**: Test GPX verification logic
- **Options**: `--test-file`, `--source-file`, `--min-score`

### Database Cleanup
- **Script**: `worker/scripts/cleanup_short_activities.py`
- **Purpose**: Manually clean up short activities
- **Options**: `--threshold`, `--days`, `--dry-run`

### Webhook Testing
- **Script**: `worker/scripts/test_webhook.py`
- **Purpose**: Test webhook functionality
- **Options**: `--type`, `--url`

## Environment Variables Affecting Schedules

### Time Settings
- `TZ`: Timezone for schedule interpretation (default: `Europe/Warsaw`)

### Reminder Settings
- `STRAVA_REMINDER_FIRST_MINUTES`: First reminder delay (default: 30 minutes)
- `STRAVA_REMINDER_SECOND_DAYS`: Second reminder delay (default: 1 day)
- `STRAVA_REMINDER_THIRD_DAYS`: Third reminder delay (default: 7 days)

### Cleanup Settings
- `CLEANUP_SHORT_DAYS`: Days to keep short activities (default: 7)
- `CLEANUP_NORMAL_DAYS`: Days to keep normal activities (default: 180)
- `CLEANUP_SHORT_THRESHOLD`: Fraction of min distance for short activities (default: 0.75)

### Report Settings
- `MONTHLY_REPORT_RECIPIENTS`: Comma-separated email list for monthly reports

## Startup Behavior

### Previous Behavior (Issue)
- Worker would run all activity processing immediately on startup
- This could cause heavy load and duplicate processing

### Current Behavior (Fixed)
- Worker starts and only logs the configured schedule
- No jobs run immediately on startup
- All jobs wait for their scheduled time
- First activity processing will occur at the next :21 minute mark of an even hour

## Monitoring and Logging

### Log Levels
- **Development**: DEBUG level logging
- **Production**: INFO level logging

### Log Files
- Main worker log: `worker/logs/worker.log`
- Rotating logs: Daily rotation, 14 days retention
- Console output: All logs also go to stdout/stderr

### Log Messages
Each job logs:
- Start and completion messages
- Success/failure status
- Processing statistics (items processed, errors, etc.)
- Operating hours checks
- K100_ENABLED status checks

## Troubleshooting

### Worker Not Running Jobs
1. Check `K100_ENABLED` environment variable
2. Verify current time is within operating hours (06:00-22:00)
3. Check worker logs for error messages
4. Verify database connectivity
5. Check API connectivity (worker → backend API)

### Jobs Running Too Frequently
- Check system time and timezone settings
- Verify schedule configuration in logs
- Look for multiple worker instances

### Jobs Not Running at Expected Times
- Verify timezone settings (`TZ` environment variable)
- Check if system clock is synchronized
- Review worker logs for schedule setup messages