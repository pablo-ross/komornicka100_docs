# Komornicka 100 - Developer Guide

This guide provides detailed information for developers working on the Komornicka 100 application - a cycling contest app that integrates with Strava to verify rides against predefined routes.

## Architecture Overview

The application follows a modern microservices-inspired architecture:

1. **Frontend**: Next.js application with React components and Tailwind CSS for styling
2. **Backend API**: FastAPI service providing RESTful endpoints for the frontend
3. **Worker**: Background service for checking Strava activities and performing route verification
4. **Database**: PostgreSQL database for data persistence
5. **PgAdmin**: Web interface for database management and queries
6. **Mailpit**: Development mail server for testing email functionality
7. **NGINX**: (Production only) Reverse proxy with SSL termination

### System Interaction Flow

```
User → Frontend → Backend API ← → Database
                     ↑
                     ↓
                   Worker → Strava API
                     ↓
                   Emails
```

## Development Environment Setup

### Prerequisites

- Docker and Docker Compose
- Git
- Node.js 18+ (for local frontend development)
- Python 3.11+ (for local backend development)
- Strava API credentials (see [STRAVA_SETUP.md](STRAVA_SETUP.md))

### Initial Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/pablo-ross/komornicka100.git
   cd komornicka100
   ```

2. Run the setup script to prepare the environment:
   ```bash
   chmod +x .scripts/setup.sh
   ./.scripts/setup.sh
   ```

3. Configure environment settings:
   - Copy `.env.example` to `.env` and update values
   - Copy `frontend/.env.local.example` to `frontend/.env.local` and update values

4. Add GPX route files to the `gpx/` directory:
   - Routes should be in standard GPX format
   - Files must have `.gpx` extension
   - The filename is stored in the database but the route name is taken from inside the GPX file

5. Setup git pre-commit hook for code quality:
   ```bash
   chmod +x .githooks/pre-commit
   git config --local core.hooksPath ./githooks
   ```

6. Start the development environment:
   ```bash
   ./.scripts/run-dev.sh
   ```

7. Initialize GPX files in the database:
   ```bash
   ./.scripts/init-gpx-files.sh
   ```

### Local Development

#### Running the Application

The repository includes convenience scripts for various tasks:

- `./.scripts/run-dev.sh` - Start development environment
- `./.scripts/run-prod.sh` - Start production environment
- `./.scripts/init-gpx-files.sh` - Initialize GPX files in database
- `./.scripts/monitor-system.sh` - Monitor system resources
- `./.scripts/manage-users.sh` - User management utilities
- `./.scripts/test-webhooks.sh` - Test webhook functionality
- `./.scripts/setup-logging.sh` - Configure logging system

#### Directory Structure

The application follows a modular structure:

- `backend/` - FastAPI application
  - `app/` - Main application code
    - `core/` - Core configuration and setup
    - `models.py` - Database models
    - `routers/` - API endpoint definitions
    - `services/` - Business logic services
    - `templates/` - Email templates
    - `utils/` - Utility functions
  - `scripts/` - Maintenance and utility scripts
  - `tests/` - Test cases

- `frontend/` - Next.js application
  - `components/` - React components
  - `pages/` - Next.js pages (routes)
  - `hooks/` - Custom React hooks
  - `public/` - Static assets
  - `styles/` - CSS and styling
  - `translations/` - Internationalization files

- `worker/` - Background processing
  - `strava_checker/` - Strava checking components
  - `verification/` - Route verification logic
  - `scripts/` - Worker utility scripts

## Development Workflows

### Backend Development (FastAPI)

The backend provides API endpoints and business logic using FastAPI:

1. **Key Files**:
   - `app/main.py` - Application entry point
   - `app/routers/` - API endpoints by feature area
   - `app/services/` - Business logic services
   - `app/core/config.py` - Configuration settings

2. **Making Changes**:
   - Development server auto-reloads when code changes
   - Access the API docs at `http://localhost:8000/docs`
   - API routes follow RESTful conventions

3. **Adding New Endpoints**:
   ```python
   @router.get("/items/{item_id}")
   async def read_item(
       item_id: str,
       db: Session = Depends(get_db)
   ):
       # Your logic here
       return {"item_id": item_id}
   ```

4. **Adding Services**:
   - Create new files in `app/services/` for distinct functionality
   - Follow the existing pattern of pure functions with clear inputs/outputs
   - Update `__init__.py` to expose the new service functions

### Frontend Development (Next.js)

The frontend is built with Next.js and React:

1. **Key Directories**:
   - `pages/` - Routes and page components
   - `components/` - Reusable React components
   - `hooks/` - Custom React hooks
   - `utils/` - Utility functions

2. **Making Changes**:
   - Development server auto-reloads when code changes
   - Access the frontend at `http://localhost:3000`

3. **Adding New Pages**:
   - Create a new file in `pages/` directory
   - Use existing components from `components/` directory

4. **Styling**:
   - The project uses Tailwind CSS
   - Global styles are in `styles/globals.css`
   - Component-specific styles use Tailwind classes

### Worker Development

The worker processes Strava activities in the background:

1. **Key Files**:
   - `worker.py` - Main worker script
   - `verification/` - Activity verification logic
   - `strava_checker/` - Strava connection checking

2. **Testing Changes**:
   ```bash
   # Restart the worker after changes
   docker compose restart worker
   
   # View worker logs
   docker compose logs -f worker
   
   # Run a specific check manually
   docker compose exec worker python scripts/run_strava_checker.py --activities-only
   ```

### Database Operations

The application uses SQLAlchemy with PostgreSQL:

1. **Database Models**:
   - Models are defined in `backend/app/models.py`
   - The database schema is auto-created from these models

2. **Database Access**:
   - Access PgAdmin at `http://localhost:5050`
   - Login with credentials from your `.env` file
   - Connect to the database server

3. **Running Database Scripts**:
   ```bash
   # Clean database
   docker compose exec api python scripts/clean_database.py --table users
   
   # Fix leaderboard
   docker compose exec api python scripts/fix_leaderboard.py
   
   # List users
   docker compose exec api python scripts/list_users.py
   ```

## Testing and Debugging

### Email Testing

Test emails with Mailpit in development:

1. Access the Mailpit interface at `http://localhost:8025`
2. All emails sent by the application will be captured here
3. View HTML content, text content, and attachments

### API Testing

Test API endpoints using Swagger UI:

1. Access the interactive documentation at `http://localhost:8000/docs`
2. Try out endpoints directly from the browser
3. View request/response details and models

### Debugging

Multiple debugging options are available:

1. **Logs**:
   ```bash
   # View logs for specific services
   docker compose logs -f api
   docker compose logs -f worker
   docker compose logs -f frontend
   
   # View specific log files
   docker compose exec api cat logs/app.log
   ```

2. **Debug Panel**:
   - The application includes a debug panel component for Strava issues
   - Access debug information at `/strava-diagnostic`

3. **API Debug Middleware**:
   - Debug middleware logs detailed request/response info in development mode
   - See `backend/app/middleware/debug_middleware.py` for implementation

4. **Test Scripts**:
   - Use `backend/tests/test_strava_api.py` to test Strava API directly
   - Use `backend/scripts/debug_logging.py` for logging diagnostics

## Extending the Application

### Adding New Features

Follow these steps to add new features:

1. **Plan the data model**:
   - Update `backend/app/models.py` with new database models
   - Consider relationships with existing models

2. **Create backend endpoints**:
   - Add a new router file in `backend/app/routers/`
   - Define endpoints with appropriate HTTP methods
   - Add the router to `main.py`

3. **Implement business logic**:
   - Create service functions in `backend/app/services/`
   - Keep services focused on specific functionality

4. **Add frontend components**:
   - Create reusable components in `frontend/components/`
   - Add new pages in `frontend/pages/`
   - Connect to backend API using fetch or axios

5. **Update documentation**:
   - Add documentation for new features
   - Update existing guides as needed

### Internationalization

The application supports multiple languages:

1. **Email Templates**:
   - Email templates have both English and Polish versions
   - Controlled by `EMAIL_TEMPLATES_TRANSLATE` in `.env`
   - See `backend/app/templates/email_templates.py`

2. **Frontend Translations**:
   - Use the translation hook in `frontend/hooks/useTranslation.ts`
   - Add translations in `frontend/translations/pl.ts`

## Production Deployment

### Deployment Options

1. **Using Run Script**:
   ```bash
   ./.scripts/run-prod.sh
   ```
   The script provides options for:
   - Basic mode: Direct service access
   - NGINX mode: With reverse proxy and SSL

2. **Manual Deployment**:
   ```bash
   docker compose -f docker-compose.prod.yml up -d
   ```

### Production Considerations

1. **SSL/TLS**:
   - For NGINX setup, place certificates in `nginx/ssl/`:
     - `cert.pem` - Certificate file
     - `key.pem` - Private key file

2. **Environment Variables**:
   - Update `.env` with production settings:
     - Set `ENVIRONMENT=production`
     - Configure real SMTP server settings
     - Set actual Strava API credentials
     - Update webhook URLs if needed

3. **Database Backups**:
   - The system includes automatic PostgreSQL backups
   - Configure backup settings in `.env`:
     ```
     BACKUP_SCHEDULE=@daily
     BACKUP_KEEP_DAYS=7
     ```

4. **Logs Rotation**:
   - Production uses logrotate for managing log files
   - See `logrotate/logrotate.conf` for configuration

5. **Cloudflare Integration**:
   - For Cloudflare Tunnel, provide `CLOUDFLARE_TUNNEL_TOKEN` in `.env`

## Webhooks System

The application includes a webhook system to notify external systems about events:

1. **Supported Events**:
   - `user_registered` - New user registration
   - `user_connected_strava` - User connected Strava account
   - `user_deactivated` - User account deactivated
   - `strava_activity_approved` - Activity verified against a route
   - `strava_activity_revoked` - Previously approved activity revoked

2. **Configuration**:
   - Configure webhook URLs in `.env`:
     ```
     WEBHOOK_USER_REGISTERED=https://your-webhook-url.com/path
     ```
   - Set authentication if needed:
     ```
     WEBHOOK_BASIC_AUTH_USER=username
     WEBHOOK_BASIC_AUTH_PASS=password
     ```

3. **Testing Webhooks**:
   ```bash
   ./.scripts/test-webhooks.sh
   ```

See [WEBHOOKS.md](WEBHOOKS.md) for detailed webhook documentation.

## Troubleshooting

### Common Issues

1. **Database Connection Issues**:
   - Check database credentials in `.env`
   - Verify that the database container is running
   - Check for PostgreSQL errors in logs

2. **Strava API Issues**:
   - Verify your Strava API credentials
   - Test the connection with `backend/tests/test_strava_api.py`
   - Check [STRAVA_DEBUG.md](STRAVA_DEBUG.md) for detailed debugging steps

3. **Email Sending Failures**:
   - Check SMTP settings in `.env`
   - Verify email templates in `app/templates/email_templates.py`
   - Use Mailpit for testing in development

4. **Worker Not Processing Activities**:
   - Check if worker container is running
   - Verify worker logs for errors
   - Test activity verification manually

5. **Frontend Build Issues**:
   - Clear Next.js cache: `docker compose exec frontend rm -rf .next`
   - Check for JavaScript errors in the browser console
   - Verify that API URLs are correctly configured

### Logging

The application uses structured logging with different levels:

1. **Development**: More verbose logging with DEBUG level
2. **Production**: INFO level and above

Logs are stored in:
- `backend/logs/` - API logs
- `worker/logs/` - Worker logs
- `nginx/logs/` - NGINX logs (production only)

### Getting Help

If you encounter issues not covered here:

1. Check the container logs for error messages
2. Review the documentation for the specific component
3. Examine the actual code implementation
4. Consult with the project maintainers