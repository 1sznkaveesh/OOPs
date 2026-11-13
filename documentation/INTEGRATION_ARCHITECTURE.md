# Integration Architecture — I-ITSM

## Overview

This document describes the architecture of the integrated I-ITSM application, combining a Spring Boot 4.1.1 backend with a React 18.3.1 frontend.

## System Components

### Backend (Spring Boot 4.1.1, Java 21)
- **Framework**: Spring Boot 4.1.1
- **Language**: Java 21
- **Database**: PostgreSQL 16 (via JDBC)
- **ORM**: Spring Data JPA / Hibernate 7.4.5
- **Security**: Spring Security 6.x with HTTP Basic authentication
- **Build**: Maven 3.9+ (via wrapper)
- **Port**: 8080

### Frontend (React 18.3.1, Vite 5.4.11)
- **Framework**: React 18.3.1
- **Build Tool**: Vite 5.4.11
- **Routing**: React Router 6.28.0
- **State Management**: React Context (AuthContext)
- **HTTP Client**: Native fetch API
- **PWA**: Vite PWA plugin
- **Port**: 5173 (dev), served by backend in production

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (Client)                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  React SPA (Vite dev server on :5173)                   │    │
│  │  - Pages: Login, Register, Employee, Technician, Admin  │    │
│  │  - Components: IncidentTable, Navbar, Badges            │    │
│  │  - Auth: HTTP Basic via sessionStorage                  │    │
│  │  - API: fetch() with /api proxy                         │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP/REST + Basic Auth
                           │ (Vite proxy /api → :8080 in dev)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Spring Boot Backend (:8080)                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Security Layer                                          │    │
│  │  - HTTP Basic Authentication                             │    │
│  │  - Role-based Authorization (EMPLOYEE, TECHNICIAN, ADMIN)│    │
│  │  - CORS configured for localhost:5173                    │    │
│  │  - JSON error responses for 401/403/400                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  REST Controllers                                        │    │
│  │  - AuthController: /api/me, /api/auth/register          │    │
│  │  - IncidentController: /api/incidents/*                 │    │
│  │  - AssignmentController: /api/assignments               │    │
│  │  - TechnicianController: /api/technicians/*             │    │
│  │  - NotificationController: /api/notifications/*         │    │
│  │  - SLAController: /api/sla/*                            │    │
│  │  - AuditController: /api/audit/*                        │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Service Layer                                           │    │
│  │  - IncidentService, AssignmentService, etc.             │    │
│  │  - Business logic: priority calculation, SLA, audit     │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Data Layer                                              │    │
│  │  - JPA Repositories (Spring Data)                       │    │
│  │  - Entities: Incident, User, Technician, Notification,  │    │
│  │    AuditRecord, SLAPolicy                                │    │
│  │  - PostgreSQL database                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Data Flow

### Authentication Flow
1. User submits credentials on Login page
2. Frontend stores `Authorization: Basic base64(user:pass)` in sessionStorage
3. Frontend calls `GET /api/me` to get user info (role, technicianId)
4. Backend validates credentials via Spring Security's `CustomUserDetailsService`
5. Backend returns `{username, role, technicianId}` 
6. Frontend stores user object and redirects based on role

### Incident Creation Flow (Employee)
1. Employee fills incident form (title, description, category, impact, urgency, criticality)
2. Frontend sends `POST /api/incidents` with incident data
3. Backend validates, calculates priority (P1-P4), sets SLA deadline
4. Backend saves incident with `reportedBy` = current user
5. Backend creates audit record (`INCIDENT_CREATED`) and notifications
6. Backend returns created incident with ID
7. Frontend navigates to incident detail page

### Incident Assignment Flow (Admin)
1. Admin views all incidents with filters
2. Admin clicks "Assign" on an incident, selects strategy (SKILL/LEAST_LOADED)
3. Frontend sends `POST /api/assignments?incidentId=X&strategy=Y`
4. Backend finds suitable technician, updates incident and technician workload
5. Backend creates audit record (`TECHNICIAN_ASSIGNED`) and notifications
6. Backend returns assigned technician
7. Frontend shows success message and refreshes list

### Status Update Flow (Technician)
1. Technician views assigned incidents
2. Technician clicks "Start work" or "Mark resolved"
3. Frontend sends `PUT /api/incidents/{id}/status?status=NEW_STATUS`
4. Backend validates transition, updates incident, adjusts technician workload
5. Backend creates audit record (`STATUS_UPDATED`) and notifications
6. Backend returns updated incident
7. Frontend refreshes display

## Security Model

### Roles
| Role | Permissions |
|------|-------------|
| EMPLOYEE | Create incidents, view own incidents, view all incidents (read), view notifications |
| TECHNICIAN | All employee permissions + update incident status, view assigned incidents, view technicians |
| ADMINISTRATOR | All permissions + manage technicians, view audit logs, assign incidents |

### Authentication
- HTTP Basic Auth (username/password)
- Credentials stored in sessionStorage (base64 encoded)
- Passwords BCrypt-hashed in database
- No JWT tokens, no sessions

### Authorization
- Spring Security `@PreAuthorize` equivalent via `HttpSecurity` configuration
- Method-level security not used; all authorization at filter chain level
- CORS configured to only allow frontend origin with credentials

## Database Schema

### Core Tables
- `users` - Authentication (username, password_hash, role)
- `incidents` - Incident records with foreign keys to users (reported_by) and technicians (assigned_technician_id)
- `technicians` - Technician profiles (name, skills[], active_incident_count, active)
- `technician_skills` - Join table for technician skills
- `notifications` - Notification log (incident_id, message, channel, read, created_at)
- `audit_record` - Audit trail (incident_id, action, details, performed_by, timestamp)

### Key Relationships
- User 1:N Incident (reportedBy)
- Technician 1:N Incident (assignedTechnician) 
- Technician N:M Skill (via technician_skills)
- Incident 1:N Notification
- Incident 1:N AuditRecord

## API Contract

See `API_COMPATIBILITY_MATRIX.md` for detailed endpoint mapping.

### Key Endpoints
| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | /api/me | Basic | Any | Current user info |
| POST | /api/auth/register | None | Public | Register new employee |
| GET | /api/incidents | Basic | All | List all incidents |
| GET | /api/incidents/mine | Basic | Employee | List own incidents |
| GET | /api/incidents/search | Basic | All | Search incidents |
| POST | /api/incidents | Basic | All | Create incident |
| PUT | /api/incidents/{id}/status | Basic | Tech, Admin | Update status |
| GET | /api/technicians | Basic | Tech, Admin | List technicians |
| POST | /api/technicians | Basic | Tech, Admin | Create technician |
| PUT | /api/technicians/{id} | Basic | Tech, Admin | Update technician |
| POST | /api/assignments | Basic | Tech, Admin | Assign incident |
| GET | /api/sla/{id} | Basic | All | SLA status |
| GET | /api/notifications | Basic | All | All notifications |
| PUT | /api/notifications/{id}/read | Basic | All | Mark notification read |
| GET | /api/audit/{id} | Basic | Admin | Audit history |

## Development Workflow

### Backend
```bash
cd backend
./mvnw spring-boot:run     # Start on :8080
./mvnw test                # Run tests
./mvnw clean verify        # Full build + test
```

### Frontend
```bash
cd frontend
npm install                # Install dependencies
npm run dev                # Start Vite on :5173 (proxies /api to :8080)
npm run build              # Production build to dist/
```

### Database
```bash
docker run -d --name itsm-postgres \
  -e POSTGRES_DB=itsm_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=malay@sql \
  -p 5432:5432 postgres:16
```

## Production Deployment

1. Build frontend: `npm run build` → outputs to `frontend/dist/`
2. Configure backend to serve static files from `frontend/dist/`
3. Set production database credentials via environment variables
4. Update CORS allowed origins for production domain
5. Use reverse proxy (nginx) for SSL termination and static file serving

## Configuration

### Backend (application.properties)
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/itsm_db
spring.datasource.username=postgres
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.open-in-view=false
```

### Frontend (vite.config.js)
```javascript
server: {
  port: 5173,
  proxy: { '/api': 'http://localhost:8080' }
}
```

### Environment Variables
- `DB_PASSWORD` - Database password (not committed)
- `SPRING_PROFILES_ACTIVE` - Spring profile (dev/prod)

---

*Generated as part of I-ITSM Clean Full-Stack Integration*