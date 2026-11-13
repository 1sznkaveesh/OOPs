# Local Setup Guide — I-ITSM

## Prerequisites

### Required Software
| Tool | Version | Purpose |
|------|---------|---------|
| Java | 21 (LTS) | Backend runtime & compilation |
| Maven | 3.9+ | Backend build (wrapper provided) |
| Node.js | 18+ | Frontend runtime |
| npm | 9+ | Frontend package manager |
| Docker | 20+ | PostgreSQL database |
| Git | 2.30+ | Version control |

### Java Installation
```bash
# Using SDKMAN (recommended)
curl -s "https://get.sdkman.io" | bash
source ~/.sdkman/bin/sdkman-init.sh
sdk install java 21.0.13.0+10-amzn
sdk use java 21.0.13.0+10-amzn

# Verify
java -version
javac -version
```

## Quick Start

### 1. Start PostgreSQL Database
```bash
docker run -d --name itsm-postgres \
  -e POSTGRES_DB=itsm_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=malay@sql \
  -p 5432:5432 \
  postgres:16

# Verify
docker ps | grep itsm-postgres
```

### 2. Start Backend
```bash
cd ~/IITSM_CLEAN_INTEGRATION/backend
source ~/.sdkman/bin/sdkman-init.sh
./mvnw spring-boot:run

# Backend will be available at http://localhost:8080
# Default users created automatically:
#   admin / admin123 (ADMINISTRATOR)
#   technician / tech123 (TECHNICIAN)
#   employee / employee123 (EMPLOYEE)
```

### 3. Start Frontend
```bash
cd ~/IITSM_CLEAN_INTEGRATION/frontend
npm install
npm run dev

# Frontend will be available at http://localhost:5173
# Vite proxies /api/* to http://localhost:8080
```

### 4. Access Application
Open http://localhost:5173 in browser and log in with:
- **Admin**: admin / admin123
- **Technician**: technician / tech123  
- **Employee**: employee / employee123

## Detailed Setup

### Backend Configuration

#### Database Connection
Edit `backend/src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/itsm_db
spring.datasource.username=postgres
spring.datasource.password=malay@sql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.open-in-view=false
```

For production, use environment variables:
```bash
export DB_PASSWORD=your_secure_password
# application.properties:
spring.datasource.password=${DB_PASSWORD}
```

#### CORS Configuration
For production, update `SecurityConfig.java`:
```java
config.setAllowedOrigins(List.of("https://your-production-domain.com"));
```

### Frontend Configuration

#### API Base URL
The frontend uses Vite's proxy in development. For production, create `.env.production`:
```
VITE_API_BASE_URL=https://api.your-domain.com
```

Update `realApi.js` to use this:
```javascript
const API_BASE = import.meta.env.VITE_API_BASE_URL || ''
```

#### Build for Production
```bash
cd frontend
npm run build
# Output in dist/ - serve with nginx or Spring Boot static resources
```

## Running Tests

### Backend Tests
```bash
cd backend
./mvnw test              # Unit tests only
./mvnw clean verify      # Full build + tests
```

### Frontend Checks
```bash
cd frontend
npm run build            # TypeScript check + production build
# (No test runner configured in original project)
```

## Default Data

On first backend startup, these users are created automatically:

| Username | Password | Role |
|----------|----------|------|
| admin | admin123 | ADMINISTRATOR |
| technician | tech123 | TECHNICIAN |
| employee | employee123 | EMPLOYEE |

Technicians must be created by admin via the Technicians page.

## Ports

| Service | Port | URL |
|---------|------|-----|
| Frontend (Vite) | 5173 | http://localhost:5173 |
| Backend (Spring Boot) | 8080 | http://localhost:8080 |
| PostgreSQL | 5432 | jdbc:postgresql://localhost:5432/itsm_db |

## Troubleshooting

### Backend Won't Start
1. Check Java version: `java -version` (must be 21)
2. Check PostgreSQL is running: `docker ps | grep postgres`
3. Check port 8080 is free: `lsof -i :8080`
4. Check database credentials in `application.properties`

### Frontend Can't Connect to Backend
1. Verify backend is running on :8080
2. Check Vite proxy config in `vite.config.js`
3. Check browser console for CORS errors
4. Verify CORS allowed origins in `SecurityConfig.java`

### Database Issues
```bash
# Reset database (WARNING: deletes all data)
docker rm -f itsm-postgres
docker run -d --name itsm-postgres \
  -e POSTGRES_DB=itsm_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=malay@sql \
  -p 5432:5432 postgres:16
```

### Build Failures
```bash
# Backend clean build
cd backend && ./mvnw clean compile

# Frontend clean install
cd frontend && rm -rf node_modules package-lock.json && npm install
```

## Environment Variables

### Backend
| Variable | Default | Description |
|----------|---------|-------------|
| DB_PASSWORD | malay@sql | PostgreSQL password |
| SPRING_PROFILES_ACTIVE | (none) | Spring profile |
| SERVER_PORT | 8080 | HTTP port |

### Frontend
| Variable | Default | Description |
|----------|---------|-------------|
| VITE_API_BASE_URL | (empty) | Production API base URL |
| PORT | 5173 | Dev server port |

## IDE Setup

### IntelliJ IDEA / VS Code
1. Open `backend/` as Maven project
2. Open `frontend/` as Node.js project
3. Configure Java 21 SDK
4. Enable annotation processing for Lombok (if used)

## Production Checklist

- [ ] Change default passwords
- [ ] Use strong database password via environment variable
- [ ] Configure HTTPS/TLS
- [ ] Set CORS allowed origins to production domain
- [ ] Configure PostgreSQL for production (backups, connection pooling)
- [ ] Build frontend for production (`npm run build`)
- [ ] Serve frontend static files via nginx or Spring Boot
- [ ] Set up logging and monitoring
- [ ] Configure Spring Boot actuator for health checks

## Support

For issues:
1. Check application logs: `backend/target/spring.log` or console output
2. Check browser developer tools (Network, Console tabs)
3. Verify database connectivity
4. Review this guide and `API_COMPATIBILITY_MATRIX.md`

---

*Generated as part of I-ITSM Clean Full-Stack Integration*