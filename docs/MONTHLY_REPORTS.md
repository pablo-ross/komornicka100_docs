# Monthly Reports

This document describes the monthly report functionality in the Komornicka 100 worker system.

## Overview

The monthly report system automatically generates and sends comprehensive reports about verified cycling activities on the first day of each month at 11:00 AM. The report covers all activities from the previous month.

## Features

- **Automated scheduling**: Reports are sent automatically on the 1st day of each month at 11:00 AM
- **Bilingual support**: Reports are generated in English or Polish based on the `EMAIL_TEMPLATES_TRANSLATE` setting
- **Excel attachments**: Detailed activity data is included as an Excel (.xlsx) file
- **Multiple recipients**: Support for sending reports to multiple email addresses
- **Comprehensive statistics**: Including total activities, unique participants, distance, and time
- **Route breakdown**: Activities grouped by route with detailed listings

## Configuration

### Environment Variables

Add the following to your `.env` file:

```bash
# Monthly report recipients (comma-separated email addresses)
MONTHLY_REPORT_RECIPIENTS=admin@example.com,manager@example.com,reports@example.com

# If no recipients are configured, reports will not be sent
```

### Dependencies

The monthly report functionality requires the `openpyxl` library for Excel generation. This is already included in the updated `requirements.txt`.

## Report Content

### Email Content

The email includes:
- **Summary statistics**:
  - Total verified activities
  - Number of unique participants
  - Total distance covered (km)
  - Total time spent cycling (hours)
- **Activities by route**: Breakdown of activities grouped by route
- **Excel attachment**: Detailed data for further analysis

### Excel Attachment

The Excel file contains the following columns:
- Activity ID
- User ID
- First Name
- Last Name
- Email
- Activity Name
- Distance (km)
- Duration (hours)
- Start Date
- Route Name
- Similarity Score
- Verified At
- Strava Activity ID

## Manual Testing

### Test Scripts

#### Complete Test Suite
```bash
# Run comprehensive tests
python scripts/test_monthly_report.py

# Test with specific month
python scripts/test_monthly_report.py --year 2025 --month 3

# Test with custom recipients
python scripts/test_monthly_report.py --recipients "test@example.com"

# Test without Excel attachment
python scripts/test_monthly_report.py --no-excel
```

#### Manual Report Generation
```bash
# Generate report for previous month
python scripts/run_monthly_report.py

# Generate report for specific month
python scripts/run_monthly_report.py --year 2025 --month 3
```

### Testing Checklist

Before deploying, test the following:

1. **API Connection**: Verify the worker can fetch monthly activity data
2. **Excel Generation**: Confirm Excel files are created with correct data
3. **Email Sending**: Test email delivery with and without attachments
4. **Language Support**: Test both English and Polish templates
5. **Error Handling**: Test with months that have no activities

## Scheduling

The monthly report is scheduled using the existing worker scheduling system:

- **Frequency**: First day of each month at 11:00 AM
- **Time zone**: Uses the system timezone (configurable via `TZ` environment variable)
- **Dependencies**: Requires K100_ENABLED=true and configured recipients

## API Endpoint

The monthly report uses the following API endpoint:

```
GET /api/activities/monthly/{year}/{month}
```

This endpoint requires API key authentication and returns all verified activities for the specified month with full user and route information.

## Troubleshooting

### Common Issues

1. **No reports sent**:
   - Check `MONTHLY_REPORT_RECIPIENTS` is configured
   - Verify `K100_ENABLED=true`
   - Check worker logs for errors

2. **Excel generation fails**:
   - Ensure `openpyxl` is installed: `pip install openpyxl==3.1.2`
   - Check for sufficient disk space

3. **Email delivery issues**:
   - Verify SMTP settings in `.env`
   - Check email server logs
   - Test with simpler email content first

4. **API connection fails**:
   - Verify `WORKER_API_KEY` is configured correctly
   - Check API service is running and accessible
   - Test API endpoint manually

### Debug Mode

Enable debug logging for more detailed information:

```bash
python scripts/test_monthly_report.py --debug
```

### Log Files

Monthly report activities are logged to:
- `logs/worker.log` (main worker log)
- `logs/monthly_report.log` (specific to monthly reports)
- `logs/test_monthly_report.log` (test script logs)

## Implementation Details

### Key Files

- `monthly_report.py`: Main monthly report functionality
- `worker.py`: Scheduling integration
- `settings.py`: Configuration management
- `scripts/test_monthly_report.py`: Comprehensive test suite
- `scripts/run_monthly_report.py`: Manual execution script

### Integration Points

The monthly report integrates with:
- **Worker scheduler**: For automatic execution
- **Email service**: Using existing SMTP configuration
- **API service**: For activity data retrieval
- **Template system**: For multilingual support

## Future Enhancements

Potential improvements for future versions:
- CSV export option alongside Excel
- Customizable report templates
- Weekly or quarterly report options
- Dashboard integration for report status
- Report delivery confirmation tracking