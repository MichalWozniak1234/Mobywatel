# Backlog

This document defines the initial product and technical backlog for the Mobywatel MVP.

The backlog is based on the current project scope, modules, API draft, database concept and authorization concept.

---

## 1. Backlog Goal

The goal of this backlog is to organize the MVP implementation into clear, prioritized work items.

The MVP should deliver a working demo system where:

- Citizens can log in and view their digital documents.
- Citizens can view document details, status, QR code and activity history.
- Admins can manage citizen users and documents.
- SuperAdmins can manage administrators and document types.
- The backend is one monolith with User API and Admin API surfaces.
- Application use cases use CQRS + MediatR.
- The database uses PostgreSQL and ASP.NET Core Identity.
- The system can be started locally using Docker.

---

## 2. Priority Levels

The backlog uses the following priority levels:

| Priority | Meaning |
|---|---|
| P0 | Required foundation for the system to run |
| P1 | Core MVP functionality |
| P2 | Important supporting functionality |
| P3 | Nice-to-have MVP improvement |
| Later | Outside current MVP scope |

---

## 3. MVP Epics

The MVP backlog is divided into the following epics:

```txt
E01 - Project Setup and Architecture
E02 - Database and Persistence
E03 - Authentication
E04 - Authorization
E05 - Citizen Profile
E06 - Document Types
E07 - Documents
E08 - QR Code Preview
E09 - Activity Logs
E10 - Admin Dashboard
E11 - User Management
E12 - Admin Management
E13 - Citizen Mobile Application
E14 - Citizen Web Application
E15 - Admin Web Panel
E16 - Docker and Local Development
E17 - Testing
E18 - Documentation
```

---

# E01 - Project Setup and Architecture

## Goal

Create the base solution structure and prepare the backend monolith for internal Clean Architecture.

## User Value

A clean structure makes the project easier to implement, test and present.

## Stories and Tasks

### BL-001 - Create solution structure

**Priority:** P0  
**Type:** Technical task

Create the main solution and backend projects:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
```

Acceptance criteria:

- Solution builds successfully.
- Projects reference each other according to Clean Architecture rules.
- Domain layer does not depend on Application, Infrastructure or API.
- `Mobywatel.Api` contains separated User API and Admin API controller folders.
- The backend is deployable as one monolith.

---

### BL-002 - Configure dependency injection

**Priority:** P0  
**Type:** Technical task

Add dependency injection setup for Application and Infrastructure layers.

Acceptance criteria:

- `Mobywatel.Application` has `DependencyInjection.cs`.
- `Mobywatel.Infrastructure` has `DependencyInjection.cs`.
- API project registers required services.
- MediatR is registered for Application handlers.
- Common MediatR pipeline behaviors can be registered from Application.

---

### BL-003 - Configure common response model

**Priority:** P1  
**Type:** Technical task

Create common API response models.

Planned models:

```txt
ApiResponse<T>
PagedResponse<T>
ValidationErrorDto
```

Acceptance criteria:

- Successful responses use a consistent format.
- Validation errors use a consistent format.
- Controllers or handlers can return shared result models.

---

### BL-004 - Configure global error handling

**Priority:** P1  
**Type:** Technical task

Create middleware or filters for handling common exceptions.

Acceptance criteria:

- Validation errors return `400 Bad Request`.
- Unauthorized requests return `401 Unauthorized`.
- Forbidden requests return `403 Forbidden`.
- Missing resources return `404 Not Found`.
- Unexpected errors return `500 Internal Server Error`.

---

# E02 - Database and Persistence

## Goal

Create the PostgreSQL database model using Entity Framework Core and ASP.NET Core Identity.

## User Value

The system can store users, roles, citizens, admins, documents, document types, activity logs and refresh tokens.

## Stories and Tasks

### BL-005 - Configure PostgreSQL connection

**Priority:** P0  
**Type:** Technical task

Configure Entity Framework Core with PostgreSQL.

Acceptance criteria:

- Backend can connect to PostgreSQL.
- Connection string is stored in configuration.
- Database provider is registered in Infrastructure.

---

### BL-006 - Configure ASP.NET Core Identity entities

**Priority:** P0  
**Type:** Technical task

Configure ASP.NET Core Identity for users and roles.

Main Identity tables:

```txt
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetRoleClaims
AspNetUserLogins
AspNetUserTokens
```

Acceptance criteria:

- Identity user entity is configured.
- Identity role entity is configured.
- Roles `Citizen`, `Admin`, `SuperAdmin` can be created.
- Identity tables are included in migrations.

---

### BL-007 - Create domain entities

**Priority:** P0  
**Type:** Technical task

Create domain entities:

```txt
Citizen
Admin
Document
DocumentType
ActivityLog
RefreshToken
```

Acceptance criteria:

- Entities are placed in `Mobywatel.Domain`.
- Entities contain required fields.
- Entities use GUID primary keys.
- Entities include audit fields where needed.

---

### BL-008 - Configure EF Core entity mappings

**Priority:** P0  
**Type:** Technical task

Create configuration classes for domain entities.

Acceptance criteria:

- Citizens table has relation to `AspNetUsers`.
- Admins table has relation to `AspNetUsers`.
- Documents table has relation to Citizens and DocumentTypes.
- ActivityLogs table has relation to `AspNetUsers`.
- RefreshTokens table has relation to `AspNetUsers`.
- Required indexes and unique constraints are configured.

---

### BL-009 - Create initial migration

**Priority:** P0  
**Type:** Technical task

Generate the first EF Core migration.

Acceptance criteria:

- Migration creates Identity tables.
- Migration creates domain tables.
- Migration can be applied to local PostgreSQL.
- Database schema matches `database-concept.md`.

---

### BL-010 - Add seed data

**Priority:** P1  
**Type:** Technical task

Add initial seed data for demo and testing.

Seed data:

```txt
Roles
Default SuperAdmin account
Default Admin account
Example Citizen account
Default document types
Example documents
```

Acceptance criteria:

- The system creates default roles.
- Demo accounts are created.
- Demo document types are created.
- Example citizen has at least one document.

---

# E03 - Authentication

## Goal

Allow Citizens, Admins and SuperAdmins to log in, refresh tokens and log out.

## User Value

Users can securely access their features according to their account role.

## Stories and Tasks

### BL-011 - Implement login endpoint

**Priority:** P1  
**Type:** User story

As a user, I want to log in using email and password so that I can access the application.

Endpoint:

```txt
POST /api/user/auth/login
```

Acceptance criteria:

- Valid credentials return JWT access token.
- Response includes refresh token.
- Response includes user role.
- Invalid credentials return `401 Unauthorized`.
- Inactive account cannot log in.
- Login activity is saved.

---

### BL-012 - Implement JWT token service

**Priority:** P1  
**Type:** Technical task

Create a service responsible for generating JWT access tokens.

Acceptance criteria:

- Token contains user id.
- Token contains email.
- Token contains role claim.
- Token expiration is configurable.
- JWT validation is configured in API.

---

### BL-013 - Implement refresh token flow

**Priority:** P1  
**Type:** User story

As a logged-in user, I want my access token to be refreshed so that I do not need to log in repeatedly.

Endpoint:

```txt
POST /api/user/auth/refresh-token
```

Acceptance criteria:

- Valid refresh token returns new access token.
- Expired refresh token is rejected.
- Revoked refresh token is rejected.
- Refresh token is stored in `RefreshTokens` table.

---

### BL-014 - Implement logout endpoint

**Priority:** P1  
**Type:** User story

As a user, I want to log out so that my session is ended.

Endpoint:

```txt
POST /api/user/auth/logout
```

Acceptance criteria:

- Refresh token is revoked.
- User can no longer use revoked refresh token.
- Endpoint requires authentication.
- Logout activity is saved.

---

### BL-015 - Implement current user endpoint

**Priority:** P1  
**Type:** User story

As a logged-in user, I want to check my current account data.

Endpoint:

```txt
GET /api/user/auth/me
```

Acceptance criteria:

- Endpoint returns current user id, email, name, role and status.
- Endpoint requires authentication.
- Missing token returns `401 Unauthorized`.

---

# E04 - Authorization

## Goal

Protect endpoints using roles and ownership rules.

## User Value

Each user can access only the data and features intended for their role.

## Stories and Tasks

### BL-016 - Configure role-based policies

**Priority:** P1  
**Type:** Technical task

Create authorization policies:

```txt
CitizenOnly
AdminOnly
AdminOrSuperAdmin
SuperAdminOnly
AuthenticatedUser
```

Acceptance criteria:

- Citizen endpoints require Citizen role.
- Admin endpoints require Admin or SuperAdmin role.
- SuperAdmin endpoints require SuperAdmin role.
- Invalid role returns `403 Forbidden`.

---

### BL-017 - Implement document ownership checks

**Priority:** P1  
**Type:** Technical task

Citizens must only access their own documents.

Acceptance criteria:

- Citizen can open own document.
- Citizen cannot open another citizen's document.
- Citizen can open QR code only for own document.
- Admin and SuperAdmin can access all documents.

---

### BL-018 - Implement citizen ownership checks

**Priority:** P1  
**Type:** Technical task

Citizens must only access their own profile and activity history.

Acceptance criteria:

- Citizen can access `GET /api/user/profile`.
- Citizen cannot access admin citizen-management endpoints.
- Admin and SuperAdmin can view citizen details.

---

# E05 - Citizen Profile

## Goal

Allow citizens to view their profile and admins to manage citizen data.

## User Value

Citizens can see their personal data, while admins can maintain citizen records.

## Stories and Tasks

### BL-019 - Get current citizen profile

**Priority:** P1  
**Type:** User story

As a Citizen, I want to view my profile.

Endpoint:

```txt
GET /api/user/profile
```

Acceptance criteria:

- Citizen receives own profile.
- Profile includes name, email, PESEL, date of birth and address.
- Endpoint requires Citizen role.

---

### BL-020 - Get citizen details by id

**Priority:** P1  
**Type:** User story

As an Admin, I want to view citizen details.

Endpoint:

```txt
GET /api/admin/users/{id}
```

Acceptance criteria:

- Admin can view citizen details.
- SuperAdmin can view citizen details.
- Citizen cannot use this endpoint.
- Missing citizen returns `404 Not Found`.

---

### BL-021 - Update citizen profile

**Priority:** P2  
**Type:** User story

As an Admin, I want to update citizen data.

Endpoint:

```txt
PUT /api/admin/users/{id}
```

Acceptance criteria:

- Admin can update selected citizen fields.
- PESEL must stay unique.
- Invalid data returns validation errors.
- Update creates activity log entry.

---

# E06 - Document Types

## Goal

Manage available document categories.

## User Value

Admins can assign correct document types, and SuperAdmins can manage system dictionaries.

## Stories and Tasks

### BL-022 - Get document type list

**Priority:** P1  
**Type:** User story

As an Admin, I want to view available document types.

Endpoint:

```txt
GET /api/admin/document-types
```

Acceptance criteria:

- Admin can view document types.
- SuperAdmin can view document types.
- Inactive types can be optionally included.

---

### BL-023 - Create document type

**Priority:** P2  
**Type:** User story

As a SuperAdmin, I want to create a document type.

Endpoint:

```txt
POST /api/admin/document-types
```

Acceptance criteria:

- Only SuperAdmin can create document type.
- Code must be unique.
- Name is required.
- New document type is active by default.
- Activity log is saved.

---

### BL-024 - Update document type

**Priority:** P2  
**Type:** User story

As a SuperAdmin, I want to update a document type.

Endpoint:

```txt
PUT /api/admin/document-types/{id}
```

Acceptance criteria:

- Only SuperAdmin can update document type.
- Missing document type returns `404 Not Found`.
- Activity log is saved.

---

### BL-025 - Activate or deactivate document type

**Priority:** P2  
**Type:** User story

As a SuperAdmin, I want to activate or deactivate document types.

Endpoint:

```txt
PATCH /api/admin/document-types/{id}/status
```

Acceptance criteria:

- Only SuperAdmin can change status.
- Inactive document type cannot be used for new documents.
- Existing documents keep their assigned type.
- Activity log is saved.

---

# E07 - Documents

## Goal

Implement the core document management functionality.

## User Value

Citizens can view digital documents, and admins can manage them.

## Stories and Tasks

### BL-026 - Get current citizen documents

**Priority:** P1  
**Type:** User story

As a Citizen, I want to view my assigned documents.

Endpoint:

```txt
GET /api/user/documents
```

Acceptance criteria:

- Citizen sees only own documents.
- Response includes document type, number, dates and status.
- Empty list is returned if no documents exist.

---

### BL-027 - Get document details

**Priority:** P1  
**Type:** User story

As a user with access, I want to view document details.

Endpoint:

```txt
GET /api/user/documents/{id}
GET /api/admin/documents/{id}
```

Acceptance criteria:

- Citizen can view own document details.
- Citizen cannot view another citizen's document.
- Admin can view all documents.
- SuperAdmin can view all documents.
- Document view activity is saved.

---

### BL-028 - Get all documents for admin

**Priority:** P1  
**Type:** User story

As an Admin, I want to view all documents.

Endpoint:

```txt
GET /api/admin/documents
```

Acceptance criteria:

- Admin and SuperAdmin can view paginated document list.
- Citizen cannot access this endpoint.
- Filtering by status, document type and citizen is supported.

---

### BL-029 - Create document

**Priority:** P1  
**Type:** User story

As an Admin, I want to create a document and assign it to a citizen.

Endpoint:

```txt
POST /api/admin/documents
```

Acceptance criteria:

- Admin and SuperAdmin can create document.
- Citizen cannot create document.
- Citizen must exist.
- Document type must exist and be active.
- Document number must be unique.
- Expiration date must be later than issue date.
- QR code value is generated or saved.
- Activity log is saved.

---

### BL-030 - Update document

**Priority:** P2  
**Type:** User story

As an Admin, I want to update document data.

Endpoint:

```txt
PUT /api/admin/documents/{id}
```

Acceptance criteria:

- Admin and SuperAdmin can update document.
- Citizen cannot update document.
- Document type must be active if changed.
- Citizen must exist if reassigned.
- Activity log is saved.

---

### BL-031 - Update document status

**Priority:** P1  
**Type:** User story

As an Admin, I want to update document status.

Endpoint:

```txt
PATCH /api/admin/documents/{id}/status
```

Allowed statuses:

```txt
Active
Expired
Blocked
Pending
Rejected
```

Acceptance criteria:

- Admin and SuperAdmin can update status.
- Invalid status is rejected.
- Status update is visible to citizen.
- Activity log is saved with previous and new status.

---

### BL-032 - Delete or deactivate document

**Priority:** P2  
**Type:** User story

As an Admin, I want to delete or deactivate a document.

Endpoint:

```txt
DELETE /api/admin/documents/{id}
```

Acceptance criteria:

- Admin and SuperAdmin can delete or soft-delete document.
- Citizen cannot delete document.
- Missing document returns `404 Not Found`.
- Activity log is saved.

---

# E08 - QR Code Preview

## Goal

Allow citizens to display QR code preview for their documents.

## User Value

The QR code makes the document preview more realistic for the demo.

## Stories and Tasks

### BL-033 - Generate QR code value

**Priority:** P1  
**Type:** Technical task

Create service for QR code value generation.

Acceptance criteria:

- QR code value is connected with document id and document number.
- QR code is not integrated with external verification systems.
- Service can be reused by document creation and QR endpoint.

---

### BL-034 - Get document QR code

**Priority:** P1  
**Type:** User story

As a Citizen, I want to display QR code for my document.

Endpoint:

```txt
GET /api/user/documents/{id}/qr-code
```

Acceptance criteria:

- Citizen can get QR code only for own document.
- Admin and SuperAdmin can get QR code for all documents.
- Missing document returns `404 Not Found`.
- Unauthorized ownership access returns `403 Forbidden` or `404 Not Found` depending on final decision.

---

# E09 - Activity Logs

## Goal

Track important actions in the system.

## User Value

Citizens can see their activity history, and admins can inspect system activity.

## Stories and Tasks

### BL-035 - Create activity log service

**Priority:** P1  
**Type:** Technical task

Create service for saving activity logs.

Acceptance criteria:

- Service stores user id, action, description, IP address and metadata.
- Service can be called from application handlers.
- Sensitive data is not stored in descriptions.

---

### BL-036 - Get citizen activity history

**Priority:** P1  
**Type:** User story

As a Citizen, I want to view my activity history.

Endpoint:

```txt
GET /api/user/activity-logs
```

Acceptance criteria:

- Citizen sees only own activity logs.
- Pagination is supported.
- Filtering by action is supported.

---

### BL-037 - Get activity logs for admin

**Priority:** P2  
**Type:** User story

As an Admin, I want to view activity logs.

Endpoint:

```txt
GET /api/admin/activity-logs
```

Acceptance criteria:

- Admin and SuperAdmin can view activity logs.
- Pagination is supported.
- Filters by user, action and date range are supported.
- SuperAdmin can view full audit logs.

---

### BL-038 - Get activity log details

**Priority:** P2  
**Type:** User story

As an Admin, I want to view activity log details.

Endpoint:

```txt
GET /api/admin/activity-logs/{id}
```

Acceptance criteria:

- Admin and SuperAdmin can view details.
- Missing log returns `404 Not Found`.
- Details include metadata if available.

---

# E10 - Admin Dashboard

## Goal

Provide system summary for the admin panel.

## User Value

Admins can quickly understand system state.

## Stories and Tasks

### BL-039 - Get admin dashboard data

**Priority:** P2  
**Type:** User story

As an Admin, I want to view dashboard statistics.

Endpoint:

```txt
GET /api/admin/dashboard
```

Acceptance criteria:

- Admin and SuperAdmin can access dashboard.
- Citizen cannot access dashboard.
- Dashboard returns total users, citizens and documents.
- Dashboard returns active, expired and blocked document counts.
- Dashboard returns recent activity.

---

# E11 - User Management

## Goal

Allow Admins to manage citizen user accounts.

## User Value

Admins can create and maintain citizen accounts.

## Stories and Tasks

### BL-040 - Get user list

**Priority:** P1  
**Type:** User story

As an Admin, I want to view users.

Endpoint:

```txt
GET /api/admin/users
```

Acceptance criteria:

- Admin and SuperAdmin can view users.
- Citizen cannot view users.
- Pagination is supported.
- Search and filters are supported.

---

### BL-041 - Get user details

**Priority:** P1  
**Type:** User story

As an Admin, I want to view selected user details.

Endpoint:

```txt
GET /api/admin/users/{id}
```

Acceptance criteria:

- Admin and SuperAdmin can view user details.
- Missing user returns `404 Not Found`.
- Details include citizen or admin profile id if available.

---

### BL-042 - Create citizen user

**Priority:** P1  
**Type:** User story

As an Admin, I want to create a citizen account.

Endpoint:

```txt
POST /api/admin/users
```

Acceptance criteria:

- Admin and SuperAdmin can create Citizen account.
- Email must be unique.
- Password must meet Identity rules.
- Citizen profile is created together with Identity user.
- Activity log is saved.

---

### BL-043 - Update user data

**Priority:** P2  
**Type:** User story

As an Admin, I want to update user account data.

Endpoint:

```txt
PUT /api/admin/users/{id}
```

Acceptance criteria:

- Admin and SuperAdmin can update common user data.
- Email must remain unique.
- Activity log is saved.

---

### BL-044 - Activate or deactivate user

**Priority:** P2  
**Type:** User story

As an Admin, I want to activate or deactivate a user account.

Endpoint:

```txt
PATCH /api/admin/users/{id}/status
```

Acceptance criteria:

- Admin and SuperAdmin can change user status.
- Deactivated user cannot log in.
- Activity log is saved.

---

# E12 - Admin Management

## Goal

Allow SuperAdmin to manage administrator accounts.

## User Value

The project demonstrates role-based access control with elevated permissions.

## Stories and Tasks

### BL-045 - Get admin list

**Priority:** P2  
**Type:** User story

As a SuperAdmin, I want to view administrator accounts.

Endpoint:

```txt
GET /api/admin/admins
```

Acceptance criteria:

- Only SuperAdmin can view admin list.
- Admin cannot manage other admins.
- Pagination and search are supported.

---

### BL-046 - Create admin account

**Priority:** P2  
**Type:** User story

As a SuperAdmin, I want to create an Admin account.

Endpoint:

```txt
POST /api/admin/admins
```

Acceptance criteria:

- Only SuperAdmin can create Admin account.
- Identity user is created with Admin role.
- Admin profile is created.
- Activity log is saved.

---

### BL-047 - Update admin account

**Priority:** P3  
**Type:** User story

As a SuperAdmin, I want to update admin data.

Endpoint:

```txt
PUT /api/admin/admins/{id}
```

Acceptance criteria:

- Only SuperAdmin can update admin account.
- Missing admin returns `404 Not Found`.
- Activity log is saved.

---

### BL-048 - Activate or deactivate admin account

**Priority:** P3  
**Type:** User story

As a SuperAdmin, I want to activate or deactivate admin accounts.

Endpoint:

```txt
PATCH /api/admin/admins/{id}/status
```

Acceptance criteria:

- Only SuperAdmin can change admin status.
- Deactivated admin cannot log in.
- Activity log is saved.

---

# E13 - Citizen Mobile Application

## Goal

Create the citizen-facing mobile application.

## User Value

Citizens can quickly access documents from a mobile device.

## Stories and Tasks

### BL-049 - Create mobile app structure

**Priority:** P2  
**Type:** Technical task

Create React Native project structure.

Acceptance criteria:

- App has navigation structure.
- API client is configured.
- Auth store is configured.

---

### BL-050 - Implement mobile login

**Priority:** P2  
**Type:** User story

As a Citizen, I want to log in from the mobile app.

Acceptance criteria:

- Login screen sends request to backend.
- Token is stored securely enough for MVP.
- Invalid login shows error.
- Successful login redirects to dashboard.

---

### BL-051 - Implement mobile document list

**Priority:** P2  
**Type:** User story

As a Citizen, I want to view my documents in the mobile app.

Acceptance criteria:

- Documents are loaded from backend.
- Document cards show document type, number and status.
- Loading, error and empty states are handled.

---

### BL-052 - Implement mobile document details and QR code

**Priority:** P2  
**Type:** User story

As a Citizen, I want to view document details and QR code.

Acceptance criteria:

- Details screen shows document data.
- QR code screen displays QR code value.
- Unauthorized access errors are handled.

---

### BL-053 - Implement mobile profile and activity history

**Priority:** P3  
**Type:** User story

As a Citizen, I want to view my profile and activity history.

Acceptance criteria:

- Profile screen loads current citizen profile.
- Activity screen loads citizen activity logs.
- Empty states are handled.

---

# E14 - Citizen Web Application

## Goal

Create browser access for citizen functionality.

## User Value

Citizens can access the same document features from a web browser.

## Stories and Tasks

### BL-054 - Create CitizenWeb Angular structure

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Angular app is created.
- Routing is configured.
- Auth guard is configured.
- API interceptor attaches JWT token.

---

### BL-055 - Implement CitizenWeb login

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Login page works with User API.
- Invalid credentials show error.
- Successful login redirects to dashboard.

---

### BL-056 - Implement CitizenWeb documents

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Citizen can view document list.
- Citizen can open document details.
- Citizen can open QR code preview.

---

### BL-057 - Implement CitizenWeb profile and activity history

**Priority:** P3  
**Type:** User story

Acceptance criteria:

- Citizen can view profile.
- Citizen can view activity history.
- Loading and error states are handled.

---

# E15 - Admin Web Panel

## Goal

Create the administration web panel.

## User Value

Admins can manage users, documents, document types and activity logs.

## Stories and Tasks

### BL-058 - Create AdminWeb Angular structure

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Angular admin app is created.
- Admin layout is created.
- Sidebar and topbar are created.
- Admin route guards are configured.

---

### BL-059 - Implement AdminWeb login

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Admin and SuperAdmin can log in.
- Citizen cannot access admin panel.
- Role is checked after login.

---

### BL-060 - Implement Admin dashboard

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Dashboard displays backend statistics.
- Recent activity is displayed.

---

### BL-061 - Implement user management UI

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Admin can view user list.
- Admin can view user details.
- Admin can create citizen user.
- Admin can update user status.

---

### BL-062 - Implement document management UI

**Priority:** P2  
**Type:** User story

Acceptance criteria:

- Admin can view document list.
- Admin can view document details.
- Admin can create document.
- Admin can update document status.

---

### BL-063 - Implement document type management UI

**Priority:** P3  
**Type:** User story

Acceptance criteria:

- Admin can view document types.
- SuperAdmin can create and update document types.
- Admin cannot perform SuperAdmin-only actions.

---

### BL-064 - Implement activity logs UI

**Priority:** P3  
**Type:** User story

Acceptance criteria:

- Admin can view activity logs.
- Admin can open activity log details.
- Filters are available.

---

# E16 - Docker and Local Development

## Goal

Make the project easy to run locally.

## User Value

Another developer can start the project without complex setup.

## Stories and Tasks

### BL-065 - Create Docker Compose for PostgreSQL

**Priority:** P0  
**Type:** Technical task

Acceptance criteria:

- PostgreSQL starts with Docker Compose.
- Database name, user and password are configured.
- Data volume is configured.

---

### BL-066 - Add backend Dockerfile

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Backend can be built as Docker image.
- Backend can connect to PostgreSQL container.

---

### BL-067 - Add local setup instructions

**Priority:** P1  
**Type:** Documentation task

Acceptance criteria:

- README explains how to start PostgreSQL.
- README explains how to run migrations.
- README explains demo credentials.

---

# E17 - Testing

## Goal

Verify backend business rules, authorization and API behavior.

## User Value

The MVP is more stable and easier to present.

## Stories and Tasks

### BL-068 - Add domain tests

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Document status logic is tested.
- Value object validation is tested if implemented.

---

### BL-069 - Add application tests

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Login use case is tested.
- Create document use case is tested.
- Update document status use case is tested.
- Citizen ownership checks are tested.

---

### BL-070 - Add API authorization tests

**Priority:** P2  
**Type:** Technical task

Acceptance criteria:

- Citizen cannot access admin endpoints.
- Admin cannot access SuperAdmin endpoints.
- Citizen cannot access another citizen's document.

---

# E18 - Documentation

## Goal

Keep project documentation clear and useful.

## User Value

The project can be understood, reviewed and started by another person.

## Stories and Tasks

### BL-071 - Finalize architecture documentation

**Priority:** P2  
**Type:** Documentation task

Acceptance criteria:

- Architecture docs describe backend layers.
- Dependency rules are explained.
- Project structure is documented.

---

### BL-072 - Finalize API documentation

**Priority:** P2  
**Type:** Documentation task

Acceptance criteria:

- Endpoint groups are documented.
- Request and response examples are documented.
- Authorization rules are documented.

---

### BL-073 - Finalize database documentation

**Priority:** P2  
**Type:** Documentation task

Acceptance criteria:

- Database tables are documented.
- Relationships are documented.
- Constraints and indexes are documented.
- Seed data is documented.

---

### BL-074 - Finalize authorization documentation

**Priority:** P2  
**Type:** Documentation task

Acceptance criteria:

- Roles are documented.
- Policies are documented.
- Ownership checks are documented.
- Endpoint access matrix is documented.

---

# 4. Recommended Implementation Order

The backlog should be implemented in this order:

```txt
1. BL-001 Create solution structure
2. BL-002 Configure dependency injection
3. BL-005 Configure PostgreSQL connection
4. BL-006 Configure ASP.NET Core Identity entities
5. BL-007 Create domain entities
6. BL-008 Configure EF Core entity mappings
7. BL-009 Create initial migration
8. BL-010 Add seed data
9. BL-011 Implement login endpoint
10. BL-012 Implement JWT token service
11. BL-013 Implement refresh token flow
12. BL-014 Implement logout endpoint
13. BL-015 Implement current user endpoint
14. BL-016 Configure role-based policies
15. BL-017 Implement document ownership checks
16. BL-019 Get current citizen profile
17. BL-022 Get document type list
18. BL-026 Get current citizen documents
19. BL-027 Get document details
20. BL-029 Create document
21. BL-031 Update document status
22. BL-033 Generate QR code value
23. BL-034 Get document QR code
24. BL-035 Create activity log service
25. BL-036 Get citizen activity history
26. BL-040 Get user list
27. BL-042 Create citizen user
28. BL-039 Get admin dashboard data
29. Frontend and mobile features
30. Tests and final documentation
```

---

# 5. MVP Definition of Done

The MVP backlog is complete when:

- Backend solution builds successfully.
- PostgreSQL database runs locally using Docker Compose.
- EF Core migrations create the expected database schema.
- ASP.NET Core Identity is used for users and roles.
- JWT login works for Citizen, Admin and SuperAdmin.
- Refresh token and logout flows work.
- Role-based authorization is enforced.
- Citizen ownership checks are enforced.
- Citizen can view profile, documents, document details, QR code and activity history.
- Admin can view dashboard, users, documents and activity logs.
- Admin can create citizens and documents.
- Admin can update document statuses.
- SuperAdmin can manage admins and document types.
- Activity logs are saved for important actions.
- Swagger/OpenAPI is available.
- Citizen mobile app communicates with User API.
- Citizen web app communicates with User API.
- Admin web panel communicates with Admin API.
- README explains how to start and test the project.

---

# 6. Out of Scope Backlog Items

The following items are outside the MVP and should not be implemented now:

```txt
Real government system integration
Trusted profile integration
Real identity verification
Biometric authentication
Payment services
Push notifications
SMS notifications
Email notifications
Offline mobile mode
Production-level security certification
Advanced QR code verification
External document registry integration
Advanced analytics dashboard
Multi-language support
Mobile app store release
```

---

# 7. Final Backlog Summary

The highest priority is to build the backend foundation first: solution structure, database, Identity, authentication, authorization and document functionality.

After the backend is stable, the frontend applications should consume the existing API.

The MVP should remain focused on a small but complete system: citizens view documents, admins manage data, SuperAdmins manage system-level resources, and all important actions are logged.
