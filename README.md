# Form Manager

Full-stack form management system with admin approval workflow and Azure Blob Storage file upload support.

## About

Form Manager is a comprehensive web application for creating, managing, and collecting form submissions. The system allows administrators to build custom forms with various field types (text, textarea, select, file upload, etc.) and collect responses from users. 

**Key capabilities:**
- **Form Creation**: Admins can create dynamic forms with custom fields, validation rules, and file upload capabilities
- **Public Submission**: Users can submit forms without registration through public links
- **Submission Management**: Admins can view, filter, and manage all form submissions
- **File Storage**: Uploaded files are securely stored in Azure Blob Storage
- **Admin Approval System**: New admin registrations require approval from super administrators before access is granted
- **Multi-language Interface**: Supports English and Ukrainian languages
- **Role-Based Access**: Three-tier permission system (Super Admin, Admin, Regular User)

The system is built with a clean architecture pattern, ensuring maintainability and scalability. The backend provides a RESTful API, while the frontend offers an intuitive interface for form management and submission viewing.

## Architecture

Clean architecture with layered structure:

- **Domain** - domain models and business logic
- **Application** - use cases and handlers (MediatR pattern)
- **Infrastructure** - repositories, Azure Blob Storage client
- **API** - FastAPI endpoints

## Tech Stack

### Backend
- Python 3.12
- FastAPI
- SQLAlchemy 2 (async)
- PostgreSQL
- Azure Blob Storage
- MediatR (mediatr_py)
- Dependency Injection (dependency-injector)
- Alembic for migrations
- JWT authentication

### Frontend
- React 18
- TypeScript
- Ant Design
- React Query (TanStack Query)
- React Router
- i18next (English/Ukrainian)

## Quick Start

### Prerequisites

- Docker and Docker Compose
- Azure Blob Storage account and connection string

### Setup

1. Clone and configure:

```bash
cd FManager
cp .env.example .env
```

2. Edit `.env` file with your settings:

```env
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/fmanager_db
AZURE_STORAGE_CONNECTION_STRING=your_azure_connection_string_here
AZURE_STORAGE_CONTAINER_NAME=forms-files
DEBUG=True
```

**Important:** You need an Azure Storage Account connection string. Get it from Azure Portal → Storage Account → Access Keys.

3. Start services:

```bash
docker-compose up --build
```

Applications will be available at:
- Backend API: http://localhost:8000
- Frontend: http://localhost:3000

4. Run migrations:

```bash
docker-compose exec backend alembic upgrade head
```

5. Create super admin (optional):

Super admins must be created manually in the database. To create one programmatically:

```python
# Create a script or use database client
# Set is_super_admin=True, is_approved=True, is_admin=True for a user
```

Default super admin credentials (if created):
- Email: `admin@mail.com`
- Password: `admin`

## Database Migrations

### Create a new migration:

```bash
docker-compose exec backend alembic revision --autogenerate -m "description"
```

### Apply migrations:

```bash
docker-compose exec backend alembic upgrade head
```

### Rollback migration:

```bash
docker-compose exec backend alembic downgrade -1
```

## Project Structure

```
FManager/
├── backend/
│   ├── app/
│   │   ├── api/              # FastAPI endpoints
│   │   ├── application/      # Use cases and handlers
│   │   ├── core/             # Config, database, DI container
│   │   ├── domain/           # Domain models
│   │   └── infrastructure/   # Repositories, external services
│   ├── alembic/              # Database migrations
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── api/              # API client
│   │   ├── components/       # React components
│   │   ├── pages/            # Page components
│   │   └── locales/          # i18n translations
│   ├── Dockerfile
│   └── package.json
└── docker-compose.yml
```

## Key Features

- **Admin Approval Workflow**: New admin registrations require super admin approval
- **Role-Based Access Control**: Super admins, admins, and regular users with different permissions
- **Form Management**: Create, edit, delete forms with various field types (text, textarea, select, file upload, etc.)
- **Form Submissions**: Public form submission endpoint with file upload support
- **Azure Blob Storage**: Files stored securely in Azure Blob Storage
- **Multi-language Support**: English and Ukrainian translations
- **Async Operations**: Full async/await support throughout backend
- **Clean Architecture**: Layered architecture with dependency injection
- **MediatR Pattern**: Use cases handled via MediatR pattern
- **JWT Authentication**: Secure token-based authentication

## API Endpoints

### Authentication
- `POST /api/v1/auth/register` - Register new admin (requires super admin approval)
- `POST /api/v1/auth/login` - Login user

### Forms
- `POST /api/v1/forms?creator_id={uuid}` - Create form (admin only)
- `GET /api/v1/forms/{form_id}` - Get form details
- `PUT /api/v1/forms/{form_id}` - Update form (admin only)
- `DELETE /api/v1/forms/{form_id}` - Delete form (admin only)
- `GET /api/v1/admin/{creator_id}/forms` - Get all forms by creator (admin only)
- `POST /api/v1/forms/{form_id}/submit` - Submit form (public, no auth required)

### Submissions
- `GET /api/v1/admin/{admin_id}/submissions` - Get all submissions for admin (admin only)
- `GET /api/v1/forms/{form_id}/submissions` - Get submissions for specific form (admin only)
- `DELETE /api/v1/submissions/{submission_id}` - Delete submission (admin only)

### Users
- `POST /api/v1/users` - Create user (admin only)
- `GET /api/v1/users/{user_id}` - Get user details
- `PUT /api/v1/users/{user_id}` - Update user (admin only)
- `DELETE /api/v1/users/{user_id}` - Delete user (admin only)
- `GET /api/v1/admin/{admin_id}/users` - Get users by admin (admin only)

### Super Admin
- `GET /api/v1/super-admin/unapproved-admins` - Get pending admin registrations (super admin only)
- `POST /api/v1/super-admin/admins/{user_id}/approve` - Approve admin registration (super admin only)
- `POST /api/v1/super-admin/admins/{user_id}/reject` - Reject admin registration (super admin only)

## User Roles & Permissions

### Super Admin
- Can approve/reject admin registrations
- Full access to all features
- Created manually in database (`is_super_admin=True`)

### Admin
- Can create and manage forms
- Can view submissions for their forms
- Must be approved by super admin before first login
- Registration creates account with `is_approved=False`

### Regular Users
- Created automatically when submitting forms
- No login access to admin panel
- Can submit public forms

## Local Development

### Backend

1. Create virtual environment:

```bash
cd backend
python3.12 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Run migrations:

```bash
alembic upgrade head
```

4. Start server:

```bash
uvicorn app.main:app --reload
```

### Frontend

1. Install dependencies:

```bash
cd frontend
npm install
```

2. Start development server:

```bash
npm start
```

Frontend will be available at http://localhost:3000

## Environment Variables

Required environment variables:

```env
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/fmanager_db
AZURE_STORAGE_CONNECTION_STRING=your_azure_connection_string_here
AZURE_STORAGE_CONTAINER_NAME=forms-files
SECRET_KEY=your-secret-key-for-jwt-tokens
DEBUG=True
```

## Creating Super Admin

Super admins must be created manually in the database. You can:

1. Use a database client (pgAdmin, DBeaver, etc.)
2. Create a Python script to insert a super admin user
3. Use SQL directly:

```sql
UPDATE users 
SET is_super_admin = TRUE, is_approved = TRUE, is_admin = TRUE 
WHERE email = 'admin@mail.com';
```

**Important**: Super admins should have strong passwords and be created securely.
