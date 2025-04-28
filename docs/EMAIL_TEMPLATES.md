# Email Templates System

This document explains the email templates system in the Komornicka 100 application.

## Overview

The email templates system provides:

1. A flexible, maintainable way to manage email content
2. Multi-language support (English and Polish)
3. Variable substitution in templates
4. Consistent formatting across all system emails
5. Easy addition of new email types

## Implementation

### Core Components

The email system consists of three main components:

1. **Templates Definition**: `backend/app/templates/email_templates.py`
   - Contains all email templates in both languages
   - Organized as static methods in language-specific classes

2. **Email Service**: `backend/app/services/email_service.py`
   - Functions for sending different types of emails
   - Handles template selection based on language setting
   - Manages SMTP connection and sending

3. **Configuration**: `backend/app/core/email_config.py`
   - Email-specific configuration settings
   - Loads settings from environment variables

### Directory Structure

```
backend/
├── app/
│   ├── core/
│   │   ├── config.py            # Main app config
│   │   └── email_config.py      # Email-specific config
│   ├── services/
│   │   └── email_service.py     # Email sending functions
│   └── templates/
│       └── email_templates.py   # All email templates
```

## How It Works

### Template Definition

Templates are defined as static methods in language-specific classes:

```python
class EnglishTemplates:
    @staticmethod
    def verification_email() -> EmailTemplate:
        """Email verification template"""
        subject = "{project_name} - Verify Your Email"
        content = """
        <html>
        <body>
            <h2>Hello {first_name},</h2>
            <p>Thank you for registering for the {project_name}!</p>
            <p>Please verify your email address by clicking the link below:</p>
            <p><a href="{verify_url}">Verify Email</a></p>
            <p>This link will expire in 48 hours.</p>
            <p>Best regards,<br>{project_name} Team</p>
        </body>
        </html>
        """
        return EmailTemplate(subject, content)
```

### EmailTemplate Class

The `EmailTemplate` class handles variable substitution and rendering:

```python
class EmailTemplate:
    def __init__(self, subject: str, html_content: str):
        self.subject = subject
        self.html_content = html_content
    
    def render(self, **kwargs) -> Dict[str, str]:
        """Render the template with variables"""
        subject = self.subject
        html_content = self.html_content
        
        # Replace variables in subject and content
        for key, value in kwargs.items():
            placeholder = f"{{{key}}}"
            subject = subject.replace(placeholder, str(value))
            html_content = html_content.replace(placeholder, str(value))
            
        return {
            "subject": subject,
            "html_content": html_content
        }
```

### Language Selection

The system selects templates based on the `EMAIL_TEMPLATES_TRANSLATE` setting:

```python
def get_templates(use_polish: bool = False):
    """Return templates in the specified language"""
    return PolishTemplates if use_polish else EnglishTemplates
```

This setting is loaded from the environment variables in `email_config.py`:

```python
class EmailSettings(BaseSettings):
    EMAIL_TEMPLATES_TRANSLATE: bool = False
    # Other settings...
```

## Available Email Types

The system currently supports these email types:

1. **Verification Email**: Sent when a user registers to verify their email address
2. **Delete Confirmation Email**: Sent when a user requests account deletion
3. **Deletion Complete Email**: Sent after a user's account has been deleted
4. **Strava Connected Email**: Sent when a user successfully connects their Strava account
5. **Activity Verification Email**: Sent when a user's activity is verified
6. **Activity Removed Email**: Sent when a previously verified activity is removed
7. **User Deactivated Email**: Sent when a user account is deactivated by an admin

Each email type has:
- A dedicated template method in both language classes
- A corresponding send function in `email_service.py`

## Sending Emails

### Basic Usage

To send an email, call the appropriate function from `email_service.py`:

```python
from app.services.email_service import send_verification_email

# Send verification email
success = send_verification_email(
    to_email="user@example.com",
    first_name="John",
    verify_url="https://example.com/verify/123/token"
)
```

### With Background Tasks

For web endpoints, use FastAPI's background tasks to send emails asynchronously:

```python
@router.post("/register")
def register_user(
    user_data: UserRegistration,
    background_tasks: BackgroundTasks,
):
    # ... process registration ...
    
    # Send email in background
    background_tasks.add_task(
        send_verification_email,
        user.email,
        user.first_name,
        verify_url
    )
    
    return {"message": "Registration successful"}
```

### From Worker

In the worker process, emails are sent directly:

```python
# Send activity verification email
send_activity_verification_email(
    user.email,
    user.first_name,
    activity.name,
    activity.start_date.isoformat(),
    source_gpx.name
)
```

## Email Template Variables

Common variables used in templates:

| Variable | Description | Example |
|----------|-------------|---------|
| `{project_name}` | Application name | "Komornicka 100" |
| `{first_name}` | User's first name | "John" |
| `{verify_url}` | Email verification URL | "https://example.com/verify/123" |
| `{delete_url}` | Account deletion URL | "https://example.com/delete/123" |
| `{activity_name}` | Name of a verified activity | "Morning Ride" |
| `{activity_date}` | Date of activity | "2025-04-23" |
| `{source_gpx_name}` | Name of the matched route | "Komornicka 100 Loop" |
| `{deactivation_reason}` | Reason for account deactivation | "Admin requested removal" |

## Adding New Email Templates

To add a new email template:

### 1. Define Templates

Add a new static method to both language classes in `email_templates.py`:

```python
# In EnglishTemplates class
@staticmethod
def password_reset_email() -> EmailTemplate:
    """Password reset email template"""
    subject = "{project_name} - Reset Your Password"
    content = """
    <html>
    <body>
        <h2>Hello {first_name},</h2>
        <p>You requested a password reset for your {project_name} account.</p>
        <p>Click the link below to reset your password:</p>
        <p><a href="{reset_url}">Reset Password</a></p>
        <p>This link will expire in 1 hour.</p>
        <p>Best regards,<br>{project_name} Team</p>
    </body>
    </html>
    """
    return EmailTemplate(subject, content)

# Also add to PolishTemplates class with translated content
```

### 2. Create Service Function

Add a new function to `email_service.py`:

```python
def send_password_reset_email(
    to_email: str,
    first_name: str,
    reset_url: str
) -> bool:
    """Send password reset email"""
    # Get correct templates based on language setting
    templates = get_templates(email_settings.EMAIL_TEMPLATES_TRANSLATE)
    
    # Get and render the password reset email template
    template = templates.password_reset_email()
    rendered = template.render(
        project_name=settings.PROJECT_NAME,
        first_name=first_name,
        reset_url=reset_url
    )
    
    # Send the email
    return send_email(to_email, rendered["subject"], rendered["html_content"])
```

### 3. Use the New Function

Call the function from your application code:

```python
# In a router or service
send_password_reset_email(
    user.email,
    user.first_name,
    f"{settings.FRONTEND_URL}/reset-password/{token}"
)
```

## Testing Emails

### Development Environment

In development, emails are sent to [Mailpit](http://localhost:8025) for inspection.

Configure this in `.env`:
```
SMTP_SERVER=mailpit
SMTP_PORT=1025
```

### Testing Templates Directly

You can test how templates render without sending:

```python
from app.templates.email_templates import get_templates
from app.core.config import settings

# Get the templates based on language
templates = get_templates(False)  # False for English, True for Polish

# Render a template
template = templates.verification_email()
rendered = template.render(
    project_name=settings.PROJECT_NAME,
    first_name="Test User",
    verify_url="https://example.com/verify"
)

# Print the rendered content
print(rendered["subject"])
print(rendered["html_content"])
```

### Manual Email Sending Test

Use this script to test email sending:

```python
# test_email.py
from app.services.email_service import send_email
from app.core.config import settings

success = send_email(
    "test@example.com",
    "Test Email",
    "<html><body><h1>Test Email</h1><p>This is a test email.</p></body></html>"
)

print(f"Email sent: {success}")
```

## SMTP Configuration

Configure email sending in `.env`:

```
# Development (using Mailpit)
SMTP_SERVER=mailpit
SMTP_PORT=1025
SMTP_FROM=noreply@komornicka100.pl
SMTP_USERNAME=
SMTP_PASSWORD=

# Production
SMTP_SERVER=smtp.example.com
SMTP_PORT=587
SMTP_FROM=noreply@komornicka100.pl
SMTP_USERNAME=your_username
SMTP_PASSWORD=your_password
```

The application supports:
- Standard SMTP (port 25)
- SMTP with TLS (port 587)
- SMTP with SSL (port 465)

## Language Configuration

Set the language for emails in `.env`:

```
# Use English templates
EMAIL_TEMPLATES_TRANSLATE=false

# Use Polish templates
EMAIL_TEMPLATES_TRANSLATE=true
```

## Best Practices

### Template Design

1. **Keep it simple**: Use basic HTML for maximum compatibility
2. **Include both text and links**: Don't rely solely on buttons or links
3. **Be concise**: Keep email content brief and to the point
4. **Test rendering**: Verify emails render correctly in different clients

### Variables

1. **Document variables**: Comment which variables are expected
2. **Use consistent naming**: Follow the established naming pattern
3. **Provide defaults**: Handle missing variables gracefully

### Translations

1. **Maintain consistency**: Ensure the same variables are used in both languages
2. **Preserve formatting**: Maintain the same HTML structure between languages
3. **Review translations**: Have a native speaker verify translations

## Troubleshooting

### Common Issues

1. **Emails not sending**:
   - Check SMTP settings in `.env`
   - Verify that the SMTP server is accessible
   - Check for errors in the application logs

2. **Variables not rendering**:
   - Ensure variable names match exactly (case-sensitive)
   - Check that the variable is passed to the render function

3. **Formatting issues**:
   - Verify HTML is properly formatted
   - Test the email in different clients

4. **Encoding problems**:
   - Ensure proper UTF-8 encoding is used
   - Be cautious with special characters

### Debugging Tips

1. Enable verbose logging for SMTP in development:
   ```python
   import smtplib
   smtplib.SMTP.debuglevel = 1
   ```

2. Check the application logs for SMTP communication:
   ```bash
   docker compose logs -f api
   ```

3. Inspect emails in Mailpit in development:
   ```
   http://localhost:8025
   ```