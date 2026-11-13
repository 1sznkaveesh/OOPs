# API Compatibility Matrix — I-ITSM Integration

**Generated:** 2026-11-13  
**Backend:** Spring Boot 4.1.1, Java 21, PostgreSQL  
**Frontend:** React 18.3.1, Vite 5.4.11, React Router 6.28.0  
**Auth:** HTTP Basic (Spring Security)

---

## Legend

| Status | Meaning |
|--------|---------|
| ✅ Compatible | Frontend call matches backend implementation exactly |
| ⚠️ Minor mismatch | Works but with caveats (fallbacks, different field names, etc.) |
| ❌ Incompatible | Frontend call will fail or produce incorrect results |
| 🔴 Missing | Backend endpoint does not exist |

---

## 1. Authentication & User Identity

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 1.1 | `realApi.js` / `login()` | `login(username, password)` | GET (via fetch) | `/api/me` | **None** | **Missing** | Yes | Any | 🔴 **Missing** | Backend has no `/api/me`. Frontend falls back to guessing role from username. **Fix:** Implement `GET /api/me` returning `{username, role, technicianId?}`. |
| 1.2 | `realApi.js` / `login()` | `login()` fallback | GET | `/api/incidents` | `IncidentController` | `GET /api/incidents` | Yes | Any | ⚠️ **Fallback** | Used only when `/api/me` fails. Proves credentials work but doesn't return user info. |
| 1.3 | `realApi.js` / `register()` | `register(body)` | POST | `/api/auth/register` | **None** | **Missing** | No | Public | 🔴 **Missing** | Frontend Register page calls this. **Fix:** Implement `POST /api/auth/register` creating User with EMPLOYEE role. |
| 1.4 | `AuthContext.jsx` | `logout()` | — | — | — | — | — | — | ✅ Compatible | Client-side only (clears sessionStorage). |

---

## 2. Incidents — Read Operations

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 2.1 | `realApi.js` / `getIncidents()` | `getIncidents()` | GET | `/api/incidents` | `IncidentController` | `GET /api/incidents` | Yes | EMPLOYEE, TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Returns `List<Incident>`. Frontend expects array. |
| 2.2 | `realApi.js` / `searchIncidents(params)` | `searchIncidents(params)` | GET | `/api/incidents/search?...` | `IncidentController` | `GET /api/incidents/search` | Yes | EMPLOYEE, TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Query params: `incidentId`, `priority`, `status`, `category`, `technicianId`, `title`. All match. |
| 2.3 | `IncidentDetail.jsx` | `searchIncidents({incidentId})` | GET | `/api/incidents/search?incidentId=X` | `IncidentController` | `GET /api/incidents/search` | Yes | Any | ✅ Compatible | Used to fetch single incident by ID. Returns array, frontend takes `[0]`. |
| 2.4 | `MyIncidents.jsx` | `getIncidents()` | GET | `/api/incidents` | `IncidentController` | `GET /api/incidents` | Yes | EMPLOYEE | ⚠️ **Minor** | **TODO in code:** "after backend adds GET /api/incidents/mine". Currently returns ALL incidents, frontend filters client-side by title only. No user-incident link exists in backend. |

---

## 3. Incidents — Write Operations

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 3.1 | `realApi.js` / `createIncident(body)` | `createIncident(body)` | POST | `/api/incidents` | `IncidentController` | `POST /api/incidents` | Yes | EMPLOYEE, TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Request body maps to `Incident` entity. Validation: title, description, category, impact, urgency, businessCriticality required. Backend calculates priority & SLA. |
| 3.2 | `realApi.js` / `updateStatus(id, status)` | `updateStatus(id, status)` | PUT | `/api/incidents/{id}/status?status=X` | `IncidentController` | `PUT /api/incidents/{id}/status` | Yes | TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Status values: OPEN, IN_PROGRESS, RESOLVED, CLOSED. Backend validates. |

---

## 4. Technicians

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 4.1 | `realApi.js` / `getTechnicians()` | `getTechnicians()` | GET | `/api/technicians` | `TechnicianController` | `GET /api/technicians` | Yes | TECHNICIAN, ADMINISTRATOR (per SecurityConfig) | ✅ Compatible | Returns `List<Technician>`. Skills as `Set<String>`. |
| 4.2 | `realApi.js` / `createTechnician(body)` | `createTechnician(body)` | POST | `/api/technicians` | `TechnicianController` | `POST /api/technicians` | Yes | TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Body: `{name, skills[], active, activeIncidentCount}`. Backend ignores `id`. |
| 4.3 | `realApi.js` / `updateTechnician(id, body)` | `updateTechnician(id, body)` | PUT | `/api/technicians/{id}` | `TechnicianController` | `PUT /api/technicians/{id}` | Yes | TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Updates name, skills, activeIncidentCount, active. |

---

## 5. Assignments

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 5.1 | `realApi.js` / `assignIncident(incidentId, strategy)` | `assignIncident()` | POST | `/api/assignments?incidentId=X&strategy=Y` | `AssignmentController` | `POST /api/assignments` | Yes | TECHNICIAN, ADMINISTRATOR | ✅ Compatible | Strategy: `SKILL` or `LEAST_LOADED`. Returns assigned `Technician`. |

---

## 6. SLA

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 6.1 | `realApi.js` / `getSla(id)` | `getSla(id)` | GET | `/api/sla/{id}` | `SLAController` | `GET /api/sla/{incidentId}` | Yes | Any authenticated | ✅ Compatible | Returns `{incidentId, priority, slaDeadline, slaViolated}`. Frontend expects `slaDeadline` as ISO string. |

---

## 7. Notifications

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 7.1 | `realApi.js` / `getNotifications()` | `getNotifications()` | GET | `/api/notifications` | `NotificationController` | `GET /api/notifications` | Yes | Any authenticated | ✅ Compatible | Returns `List<Notification>` (all notifications). Frontend expects array. |
| 7.2 | `Notifications.jsx` | `markRead(id)` | PUT | `/api/notifications/{id}/read` | `NotificationController` | `PUT /api/notifications/{notificationId}/read` | Yes | Any authenticated | ✅ Compatible | Returns updated `Notification`. |

---

## 8. Audit

| # | Frontend File/Component | Frontend Function | HTTP Method | Requested URL | Backend Controller | Backend Endpoint | Auth Required | Required Role | Compatibility | Notes / Required Modification |
|---|------------------------|-------------------|-------------|---------------|-------------------|------------------|---------------|---------------|---------------|-------------------------------|
| 8.1 | `realApi.js` / `getAudit(incidentId)` | `getAudit(incidentId)` | GET | `/api/audit/{incidentId}` | `AuditController` | `GET /api/audit/{incidentId}` | Yes | ADMINISTRATOR | ✅ Compatible | Returns `List<AuditRecord>`. Frontend expects `performedBy` field (backend doesn't populate yet). |

---

## 9. Field-Level Compatibility — Incident

| Frontend Field | Backend Field | Type Match | Notes |
|----------------|---------------|------------|-------|
| `id` | `id` | ✅ Long | |
| `title` | `title` | ✅ String | |
| `description` | `description` | ✅ String | |
| `category` | `category` | ✅ String | Values: NETWORK, HARDWARE, SOFTWARE, DATABASE, SECURITY, OTHER |
| `priority` | `priority` | ✅ String | Values: P1, P2, P3, P4 (backend calculated) |
| `status` | `status` | ✅ String | Values: OPEN, IN_PROGRESS, RESOLVED, CLOSED |
| `impact` | `impact` | ✅ String | Values: HIGH, MEDIUM, LOW |
| `urgency` | `urgency` | ✅ String | Values: HIGH, MEDIUM, LOW |
| `businessCriticality` | `businessCriticality` | ✅ String | Values: HIGH, MEDIUM, LOW |
| `createdAt` | `createdAt` | ✅ LocalDateTime → ISO string | Jackson serializes as ISO-8601. Frontend `new Date(i.createdAt)` works. |
| `resolvedAt` | `resolvedAt` | ✅ LocalDateTime → ISO string | Nullable. |
| `slaDeadline` | `slaDeadline` | ✅ LocalDateTime → ISO string | |
| `slaViolated` | `slaViolated` | ✅ boolean | |
| `assignedTechnician` | `assignedTechnician` | ✅ Technician object | EAGER fetch. Contains `id`, `name`, `skills[]`, `activeIncidentCount`, `active`. |

---

## 10. Field-Level Compatibility — Technician

| Frontend Field | Backend Field | Type Match | Notes |
|----------------|---------------|------------|-------|
| `id` | `id` | ✅ Long | |
| `name` | `name` | ✅ String | |
| `skills` | `skills` | ✅ Set<String> → JSON array | |
| `activeIncidentCount` | `activeIncidentCount` | ✅ int | |
| `active` | `active` | ✅ boolean | |

---

## 11. Field-Level Compatibility — Notification

| Frontend Field | Backend Field | Type Match | Notes |
|----------------|---------------|------------|-------|
| `id` | `id` | ✅ Long | |
| `incidentId` | `incidentId` | ✅ Long | |
| `message` | `message` | ✅ String | |
| `channel` | `channel` | ✅ String | IN_APP, EMAIL |
| `createdAt` | `createdAt` | ✅ LocalDateTime → ISO string | |
| `read` | `read` (DB: `is_read`) | ✅ boolean | Jackson maps `isRead()` → `read`. |

---

## 12. Field-Level Compatibility — AuditRecord

| Frontend Field | Backend Field | Type Match | Notes |
|----------------|---------------|------------|-------|
| `id` | `id` | ✅ Long | |
| `incidentId` | `incidentId` | ✅ Long | |
| `action` | `action` | ✅ String | e.g., INCIDENT_CREATED, STATUS_UPDATED, TECHNICIAN_ASSIGNED |
| `details` | `details` | ✅ String | |
| `timestamp` | `timestamp` | ✅ LocalDateTime → ISO string | |
| `performedBy` | **Missing** | ❌ | Frontend expects this (see `AuditLog.jsx:24`). Backend doesn't store it. |

---

## 13. Enum Value Compatibility

| Enum | Frontend Values | Backend Values | Match |
|------|-----------------|----------------|-------|
| User Role | EMPLOYEE, TECHNICIAN, ADMINISTRATOR | EMPLOYEE, TECHNICIAN, ADMINISTRATOR (stored in User.role) | ✅ |
| Incident Status | OPEN, IN_PROGRESS, RESOLVED, CLOSED | OPEN, IN_PROGRESS, RESOLVED, CLOSED | ✅ |
| Priority | P1, P2, P3, P4 | P1, P2, P3, P4 | ✅ |
| Impact/Urgency/Criticality | HIGH, MEDIUM, LOW | HIGH, MEDIUM, LOW | ✅ |
| Category | NETWORK, HARDWARE, SOFTWARE, DATABASE, SECURITY, OTHER | Same (validated in frontend only) | ✅ |
| Assignment Strategy | SKILL, LEAST_LOADED | SKILL, LEAST_LOADED (case-insensitive) | ✅ |
| Notification Channel | IN_APP, EMAIL | IN_APP, EMAIL (case-insensitive) | ✅ |

---

## 14. Error Handling Compatibility

| HTTP Status | Backend Response | Frontend Handling (`errorMessage` in `client.js`) | Compatible? |
|-------------|------------------|---------------------------------------------------|-------------|
| 400 | `{fieldName: "message"}` (validation) or `{"error": "msg"}` | Checks `e.data` object, joins values | ✅ |
| 401 | Spring Security default (HTML) | Returns "Wrong username or password." | ⚠️ Backend returns HTML, not JSON. Frontend may not parse correctly. |
| 403 | Spring Security default (HTML) | Returns "You are not allowed to do this." | ⚠️ Same as 401. |
| 404 | Spring default (HTML) | Falls through to generic message | ⚠️ |
| 500 | Spring default (HTML) | Falls through to generic message | ⚠️ |

**Note:** Spring Security with HTTP Basic returns HTML error pages by default. Frontend expects JSON. Need to configure `AuthenticationEntryPoint` and `AccessDeniedHandler` to return JSON.

---

## 15. Pagination & Search

| Aspect | Frontend Expectation | Backend Implementation | Compatible? |
|--------|---------------------|------------------------|-------------|
| `GET /api/incidents` | Array of all incidents | Returns `List<Incident>` (all) | ✅ (but no pagination — may be large) |
| `GET /api/incidents/search` | Array of filtered incidents | Returns `List<Incident>` (all matching) | ✅ (no pagination) |
| Sorting | Frontend sorts client-side (e.g., `AllIncidents.jsx:70` newest first) | No server-side sorting | ✅ Works but inefficient for large datasets |

---

## 16. CORS & Development Proxy

| Aspect | Configuration | Status |
|--------|---------------|--------|
| Frontend dev server | Vite on `http://localhost:5173` | ✅ |
| Backend server | Spring Boot on `http://localhost:8080` | ✅ |
| Vite proxy | `/api` → `http://localhost:8080` (in `vite.config.js`) | ✅ Works in dev |
| Backend CORS | **Not configured** | ⚠️ Needed for production or non-proxied calls |

---

## 17. Summary of Required Backend Changes

| Priority | Change | Description |
|----------|--------|-------------|
| **Critical** | Add `GET /api/me` | Return authenticated user info: `{username, role, technicianId?}` |
| **Critical** | Add `POST /api/auth/register` | Create new User with EMPLOYEE role, hash password |
| **Critical** | JSON error responses for 401/403/404/500 | Configure `AuthenticationEntryPoint` + `AccessDeniedHandler` |
| **High** | Add `performedBy` to `AuditRecord` | Track who performed each action |
| **High** | Implement `GET /api/incidents/mine` | For employee to see only their incidents (requires user-incident link) |
| **Medium** | Add CORS configuration | For production deployment |
| **Medium** | Link User to Incident | Add `reportedBy` field to Incident referencing User |
| **Low** | Pagination for incident lists | Pageable responses for large datasets |

---

## 18. Summary of Required Frontend Changes

| Priority | Change | Description |
|----------|--------|-------------|
| **Critical** | Handle missing `/api/me` gracefully | Already has fallback, but should use real endpoint when available |
| **Critical** | Update `register` call to use real endpoint | Once backend implements it |
| **Medium** | Remove `DEMO_TECHNICIAN_ID` fallback | Use `technicianId` from `/api/me` |
| **Medium** | Handle `performedBy` in audit log | Once backend adds it |

---

## 19. Verification Checklist

- [ ] Backend builds and tests pass (`./mvnw clean verify`) ✅
- [ ] Frontend builds (`npm run build`) ✅
- [ ] Database schema creates correctly ✅
- [ ] Default users created (admin, technician, employee) ✅
- [ ] All enum values match between frontend and backend ✅
- [ ] Field names match for all entities ✅
- [ ] HTTP methods and paths match ✅
- [ ] Authentication mechanism aligned (HTTP Basic) ✅
- [ ] Role names match (EMPLOYEE, TECHNICIAN, ADMINISTRATOR) ✅
- [ ] Error response format compatible ⚠️ (needs JSON errors)
- [ ] CORS configured for production ⚠️ (not yet)
- [ ] `/api/me` endpoint exists ❌ (missing)
- [ ] `/api/auth/register` endpoint exists ❌ (missing)
- [ ] `performedBy` in audit records ❌ (missing)
- [ ] User-Incident relationship exists ❌ (missing)

---

*End of API Compatibility Matrix*