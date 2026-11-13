# I-ITSM — Integrated IT Service Management System

A full-stack IT Service Management application with a Spring Boot backend and React frontend.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (Client)                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  React 18 + Vite 5 SPA (port 5173)                      │    │
│  │  - Pages: Login, Register, Employee, Technician, Admin  │    │
│  │  - Auth: HTTP Basic via sessionStorage                  │    │
│  │  - API: fetch() with Vite proxy to backend              │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP/REST + Basic Auth
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Spring Boot 4.1.1 Backend (port 8080)        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Security: Spring Security + HTTP Basic + JSON errors   │    │
│  │  Data: Spring Data JPA + Hibernate + PostgreSQL         │    │
│  │  Build: Maven 3.9+ (wrapper included)                   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
IITSM_CLEAN_INTEGRATION/
├── backend/                    # Spring Boot 4.1.1, Java 21
│   ├── src/main/java/i_itsm/
│   │   ├── config/             # SecurityConfig, DataInitializer
│   │   ├── controller/         # REST endpoints
│   │   ├── model/              # JPA entities
│   │   ├── repository/         # Spring Data repositories
│   │   ├── service/            # Business logic
│   │   └── exception/          # Custom exceptions
│   ├── src/main/resources/
│   │   └── application.properties
│   ├── pom.xml
│   └── mvnw, mvnw.cmd
│
├── frontend/                   # React 18.3.1, Vite 5.4.11
│   ├── src/
│   │   ├── api/                # API client (realApi.js, mockApi.js)
│   │   ├── auth/               # AuthContext, ProtectedRoute
│   │   ├── components/         # Shared UI components
│   │   └── pages/              # Page components by role
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
│
├── documentation/              # Technical documentation
│   ├── API_COMPATIBILITY_MATRIX.md
│   ├── INTEGRATION_ARCHITECTURE.md
│   └── LOCAL_SETUP.md
│
├── integration_reports/        # Integration reports
│   └── INTEGRATION_COMPLETION_REPORT.md
│
└── README.md                   # This file
```

## Quick Start

### Prerequisites
- **Java 21** (LTS) — via [SDKMAN](https://sdkman.io/): `sdk install java 21.0.13.0+10-amzn`
- **Node.js 18+** and **npm 9+**
- **Docker** (for PostgreSQL)
- **Git**

### 1. Start PostgreSQL
```bash
docker run -d --name itsm-postgres \
  -e POSTGRES_DB=itsm_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=malay@sql \
  -p 5432:5432 postgres:16
```

### 2. Start Backend (Terminal 1)
```bash
cd backend
source ~/.sdkman/bin/sdkman-init.sh
./mvnw spring-boot:run
```
Backend runs on **http://localhost:8080**

### 3. Start Frontend (Terminal 2)
```bash
cd frontend
npm install
npm run dev
```
Frontend runs on **http://localhost:5173** (proxies `/api` to backend)

### 4. Access Application
Open **http://localhost:5173** and login:

| Role | Username | Password |
|------|----------|----------|
| Administrator | `admin` | `admin123` |
| Technician | `technician` | `tech123` |
| Employee | `employee` | `employee123` |

## Features

### Employee Workflow
- ✅ Login / Register
- ✅ Create incidents (auto priority & SLA calculation)
- ✅ View own incidents (`/api/incidents/mine`)
- ✅ View incident details with SLA status
- ✅ Notifications (in-app + simulated email)

### Technician Workflow
- ✅ Login
- ✅ View assigned incidents
- ✅ Update status: OPEN → IN_PROGRESS → RESOLVED
- ✅ Workload visibility & SLA violation tracking

### Administrator Workflow
- ✅ Dashboard with statistics & charts
- ✅ View all incidents with 6 filters
- ✅ Assign incidents (SKILL / LEAST_LOADED strategies)
- ✅ Technician management (CRUD, skills, availability)
- ✅ Audit log with `performedBy` tracking
- ✅ SLA monitoring

## API Endpoints

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/api/me` | Basic | Any | Current user info |
| POST | `/api/auth/register` | None | Public | Register employee |
| GET | `/api/incidents` | Basic | All | List all incidents |
| GET | `/api/incidents/mine` | Basic | Employee | Own incidents |
| GET | `/api/incidents/search` | Basic | All | Filtered search |
| POST | `/api/incidents` | Basic | All | Create incident |
| PUT | `/api/incidents/{id}/status` | Basic | Tech, Admin | Update status |
| GET | `/api/technicians` | Basic | Tech, Admin | List technicians |
| POST | `/api/technicians` | Basic | Tech, Admin | Create technician |
| PUT | `/api/technicians/{id}` | Basic | Tech, Admin | Update technician |
| POST | `/api/assignments` | Basic | Tech, Admin | Assign incident |
| GET | `/api/sla/{id}` | Basic | All | SLA status |
| GET | `/api/notifications` | Basic | All | All notifications |
| PUT | `/api/notifications/{id}/read` | Basic | All | Mark read |
| GET | `/api/audit/{id}` | Basic | Admin | Audit history |

## Testing

### Backend
```bash
cd backend
./mvnw test              # Unit tests (21 tests)
./mvnw clean verify      # Full build + tests
```

### Frontend
```bash
cd frontend
npm run build            # TypeScript check + production build
```

## Configuration

### Backend (`backend/src/main/resources/application.properties`)
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/itsm_db
spring.datasource.username=postgres
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.open-in-view=false
```

### Frontend (`frontend/vite.config.js`)
```javascript
server: {
  port: 5173,
  proxy: { '/api': 'http://localhost:8080' }
}
```

## Security

- **Authentication**: HTTP Basic (credentials in sessionStorage)
- **Authorization**: Role-based (EMPLOYEE, TECHNICIAN, ADMINISTRATOR)
- **Password Storage**: BCrypt encoded
- **Error Responses**: JSON format for 401/403/400
- **CORS**: Configured for `http://localhost:5173` with credentials

## Documentation

- [`documentation/API_COMPATIBILITY_MATRIX.md`](documentation/API_COMPATIBILITY_MATRIX.md) — Complete API mapping
- [`documentation/INTEGRATION_ARCHITECTURE.md`](documentation/INTEGRATION_ARCHITECTURE.md) — System architecture
- [`documentation/LOCAL_SETUP.md`](documentation/LOCAL_SETUP.md) — Detailed setup guide
- [`integration_reports/INTEGRATION_COMPLETION_REPORT.md`](integration_reports/INTEGRATION_COMPLETION_REPORT.md) — Integration summary

## License

Internal project — I-ITSM Incident Management System