# MVP Scope

This document describes the MVP scope for the Mobywatel application.

The goal of the MVP is to build the first working version of the system that demonstrates the main product idea: citizens can access their digital documents, and administrators can manage users, documents and document statuses through one backend monolith with separate User API and Admin API surfaces.

---

## 1. MVP Goal

The MVP version of Mobywatel should prove that the system can work as a simple digital citizen document platform.

The MVP must include:

- one backend monolith,
- User API for citizen-facing clients,
- Admin API for administration clients,
- relational database,
- citizen mobile application,
- citizen web application,
- admin web panel,
- authentication and authorization,
- basic document management,
- activity logging,
- Docker-based local development setup.

The MVP does not need to integrate with real government systems. It is a student/demo application focused on architecture, functionality and presentation.

---

## 2. MVP Users

The MVP supports three main user roles:

```txt
Citizen
Admin
SuperAdmin
```

### 2.1 Citizen

A citizen is the main end user of the mobile and citizen web application.

In the MVP, the citizen can:

- log in,
- view personal profile,
- view assigned documents,
- open document details,
- check document status,
- display QR code connected with a document,
- view activity history.

The citizen cannot create or edit documents.

### 2.2 Admin

An admin is the user of the web administration panel.

In the MVP, the admin can:

- log in,
- view dashboard,
- view user list,
- view user details,
- create citizen accounts,
- update citizen data,
- view document list,
- create documents,
- assign documents to citizens,
- update document statuses,
- view activity logs.

The admin cannot manage other administrators.

### 2.3 SuperAdmin

A super admin has the highest access level.

In the MVP, the super admin can:

- do everything an admin can do,
- manage admin accounts,
- manage document types,
- view full activity logs,
- manage selected system dictionaries.

The SuperAdmin role is mainly intended to show role-based access control.

---

## 3. MVP System Components

The MVP consists of the following components:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
CitizenMobile
CitizenWeb
AdminWeb
Database
Docker
```

### 3.1 Backend Monolith

The backend monolith is the central part of the system. It is deployed as one ASP.NET Core application and exposes two logical REST API surfaces.

Responsibilities:

- expose User API endpoints,
- expose Admin API endpoints,
- authenticate users,
- authorize users by role,
- manage users,
- manage citizens,
- manage documents,
- manage document types,
- save activity logs,
- return data to mobile and web applications,
- expose Swagger/OpenAPI documentation.

CitizenMobile and CitizenWeb use the User API. AdminWeb uses the Admin API. Both API surfaces use the same Application, Domain, Infrastructure and database.

### 3.2 Database

The MVP uses a relational database.

The database stores:

- users,
- citizens,
- admins,
- documents,
- document types,
- document statuses,
- refresh tokens,
- activity logs.

The database is accessed only by the backend monolith.

### 3.3 Citizen Mobile Application

The citizen mobile application is the main citizen-facing application.

MVP screens:

```txt
LoginScreen
DashboardScreen
ProfileScreen
DocumentsScreen
DocumentDetailsScreen
QrCodeScreen
ActivityHistoryScreen
```

The mobile app allows the citizen to quickly access personal documents.

### 3.4 Citizen Web Application

The citizen web application provides browser access for citizens.

MVP pages:

```txt
Login
Dashboard
Profile
Documents
Document Details
Activity History
```

The citizen web application uses the same User API as the mobile app.

### 3.5 Admin Web Panel

The admin web panel is used by administrators.

MVP pages:

```txt
Admin Login
Dashboard
Users
User Details
Documents
Document Details
Create Document
Document Types
Activity Logs
System Settings
```

The admin panel is used to manage data available to citizens.

---

## 4. MVP Functional Scope

This section defines what should be included in the first version of the application.

---

### 4.1 Authentication

Authentication is required for all user types.

Included in MVP:

```txt
User login
JWT access token generation
Refresh token generation
Logout
Protected endpoints
Role-based authorization
```

Required endpoints:

```txt
POST /api/user/auth/login
POST /api/user/auth/refresh-token
POST /api/user/auth/logout
GET  /api/user/auth/me

POST /api/admin/auth/login
POST /api/admin/auth/refresh-token
POST /api/admin/auth/logout
GET  /api/admin/auth/me
```

Acceptance criteria:

- User can log in using email and password.
- Backend returns access token after successful login.
- Protected endpoints require valid token.
- User role is included in authorization logic.
- Citizen cannot access admin endpoints.
- Admin cannot access SuperAdmin-only endpoints.
- Invalid login data returns an error response.

---

### 4.2 Citizen Profile

Citizen profile allows the citizen to see basic personal data.

Included in MVP:

```txt
View current citizen profile
View citizen personal data
View address data
```

Required endpoints:

```txt
GET /api/user/profile
GET /api/admin/users/{id}
PUT /api/admin/users/{id}
```

Acceptance criteria:

- Citizen can view only own profile.
- Admin can view citizen profiles.
- Admin can update citizen data.
- Citizen cannot update protected fields directly.
- Profile data is loaded from the database.

---

### 4.3 User Management

User management is available in the admin panel.

Included in MVP:

```txt
View user list
View user details
Create citizen user
Update user data
Activate or deactivate user
```

Required endpoints:

```txt
GET   /api/admin/users
GET   /api/admin/users/{id}
POST  /api/admin/users
PUT   /api/admin/users/{id}
PATCH /api/admin/users/{id}/status
```

Acceptance criteria:

- Admin can display list of users.
- Admin can open user details.
- Admin can create a citizen account.
- Admin can update user data.
- Admin can activate or deactivate a user.
- Citizen cannot access user management endpoints.

---

### 4.4 Admin Management

Admin management is available only for SuperAdmin.

Included in MVP:

```txt
View admin list
Create admin account
Update admin account
Activate or deactivate admin account
```

Required endpoints:

```txt
GET   /api/admin/admins
POST  /api/admin/admins
PUT   /api/admin/admins/{id}
PATCH /api/admin/admins/{id}/status
```

Acceptance criteria:

- SuperAdmin can view admin list.
- SuperAdmin can create admin accounts.
- SuperAdmin can update admin data.
- SuperAdmin can deactivate admin accounts.
- Admin cannot manage other admins.
- Citizen cannot access admin management endpoints.

---

### 4.5 Document Management

Document management is the core feature of the MVP.

Included in MVP:

```txt
Citizen document list
Citizen document details
Admin document list
Admin document details
Create document
Assign document to citizen
Update document status
Delete document
```

Required endpoints:

```txt
GET    /api/user/documents
GET    /api/user/documents/{id}
GET    /api/admin/documents
GET    /api/admin/documents/{id}
POST   /api/admin/documents
PUT    /api/admin/documents/{id}
PATCH  /api/admin/documents/{id}/status
DELETE /api/admin/documents/{id}
```

Acceptance criteria:

- Citizen can view only own documents.
- Citizen can open document details.
- Admin can view all documents.
- Admin can create a document.
- Admin can assign document to a citizen.
- Admin can update document status.
- Admin can delete document.
- Document status is visible to citizen.
- All document operations are saved in the database.

---

### 4.6 Document Types

Document types define available types of documents.

Included in MVP:

```txt
View document type list
Create document type
Update document type
Deactivate document type
```

Example document types:

```txt
Identity Card
Driving License
Student Card
Passport
Residence Card
```

Required endpoints:

```txt
GET   /api/admin/document-types
GET   /api/admin/document-types/{id}
POST  /api/admin/document-types
PUT   /api/admin/document-types/{id}
PATCH /api/admin/document-types/{id}/status
```

Acceptance criteria:

- Admin can view document types.
- SuperAdmin can create document types.
- SuperAdmin can update document types.
- SuperAdmin can deactivate document types.
- Deactivated document types cannot be used for new documents.
- Existing documents keep their assigned type.

---

### 4.7 Document Status

Document status shows the current state of a document.

Included in MVP:

```txt
Active status
Expired status
Blocked status
Pending status
Rejected status
```

Status meaning:

```txt
Active   - document is valid and visible as active
Expired  - document is no longer valid
Blocked  - document was blocked by administrator
Pending  - document waits for confirmation or activation
Rejected - document was rejected by administrator
```

Required endpoint:

```txt
PATCH /api/admin/documents/{id}/status
```

Acceptance criteria:

- Admin can update document status.
- Citizen can see document status.
- Status update is saved in the database.
- Status update creates activity log entry.
- Invalid status value is rejected.

---

### 4.8 QR Code Preview

The MVP includes basic QR code preview for citizen documents.

Included in MVP:

```txt
Generate QR code value for document
Display QR code in mobile app
Display QR code in citizen web app
```

Required endpoints:

```txt
GET /api/user/documents/{id}/qr-code
```

Acceptance criteria:

- Citizen can open QR code for own document.
- Citizen cannot open QR code for another citizen document.
- QR code is connected with selected document.
- QR code value is generated by backend.
- QR code does not need to be verified by external systems in MVP.

---

### 4.9 Activity Logs

Activity logs track important actions in the system.

Included in MVP:

```txt
Save login activity
Save document view activity
Save document creation activity
Save document assignment activity
Save document status update activity
Save user management activity
Display citizen activity history
Display admin activity logs
```

Required endpoints:

```txt
GET /api/user/activity-logs
GET /api/admin/activity-logs
GET /api/admin/activity-logs/{id}
```

Example activity types:

```txt
UserLoggedIn
DocumentViewed
DocumentCreated
DocumentAssigned
DocumentStatusUpdated
UserCreated
UserUpdated
UserStatusChanged
DocumentTypeCreated
DocumentTypeUpdated
```

Acceptance criteria:

- Important actions create activity log entries.
- Citizen can view own activity history.
- Admin can view activity logs related to managed data.
- SuperAdmin can view all activity logs.
- Activity log contains date, user, action and description.

---

### 4.10 Admin Dashboard

The admin dashboard gives basic system overview.

Included in MVP:

```txt
Total users count
Total citizens count
Total documents count
Active documents count
Expired documents count
Recent activity list
```

Required endpoint:

```txt
GET /api/admin/dashboard
```

Acceptance criteria:

- Admin can open dashboard after login.
- Dashboard displays basic statistics.
- Dashboard displays recent activity.
- Data is loaded from Admin API.
- Citizen cannot access dashboard endpoint.

---

## 5. MVP Frontend Scope

### 5.1 Citizen Mobile MVP

The mobile application should be simple and focused on document access.

Screens included in MVP:

```txt
LoginScreen
DashboardScreen
ProfileScreen
DocumentsScreen
DocumentDetailsScreen
QrCodeScreen
ActivityHistoryScreen
```

Mobile user flow:

```txt
1. Citizen opens mobile app.
2. Citizen logs in.
3. Citizen sees dashboard.
4. Citizen opens document list.
5. Citizen selects a document.
6. Citizen sees document details and status.
7. Citizen opens QR code.
8. Citizen can view activity history.
```

Acceptance criteria:

- Mobile app connects to User API.
- Citizen can log in.
- Citizen can view own profile.
- Citizen can view own documents.
- Citizen can view document details.
- Citizen can display QR code.
- Citizen can view activity history.
- Mobile app handles loading and error states.

---

### 5.2 Citizen Web MVP

The citizen web application provides the same basic functionality as mobile, but in browser form.

Pages included in MVP:

```txt
Login
Dashboard
Profile
Documents
Document Details
QR Code
Activity History
```

Acceptance criteria:

- Citizen web app connects to User API.
- Citizen can log in.
- Citizen can view dashboard.
- Citizen can view own profile.
- Citizen can view own documents.
- Citizen can open document details.
- Citizen can display QR code.
- Citizen can view activity history.

---

### 5.3 Admin Web MVP

The admin web panel provides management functionality.

Pages included in MVP:

```txt
Login
Dashboard
Users
User Details
Create User
Documents
Document Details
Create Document
Edit Document
Document Types
Activity Logs
System Settings
```

Acceptance criteria:

- Admin can log in.
- Admin can view dashboard.
- Admin can view user list.
- Admin can create citizen users.
- Admin can update citizen users.
- Admin can view document list.
- Admin can create documents.
- Admin can assign documents to citizens.
- Admin can update document status.
- Admin can view activity logs.
- SuperAdmin can manage document types and admin accounts.

---

## 6. MVP Backend Scope

The backend MVP should include the following modules:

```txt
Auth
Citizens
Users
Admins
Documents
DocumentTypes
ActivityLogs
Dashboard
```

### 6.1 Backend Commands

Planned backend commands:

```txt
Login
RefreshToken
Logout
CreateCitizen
UpdateCitizen
CreateDocument
AssignDocumentToCitizen
UpdateDocument
UpdateDocumentStatus
DeleteDocument
CreateDocumentType
UpdateDocumentType
DeactivateDocumentType
CreateAdmin
UpdateAdmin
DeactivateUser
```

### 6.2 Backend Queries

Planned backend queries:

```txt
GetCurrentUser
GetCitizenProfile
GetCitizenDocuments
GetDocumentDetails
GetAllDocuments
GetUsers
GetUserDetails
GetDocumentTypes
GetActivityLogs
GetCitizenActivityLogs
GetAdminDashboard
GetAdmins
```

---

## 7. MVP Database Scope

The MVP database should contain the following tables:

```txt
Users
Citizens
Admins
Documents
DocumentTypes
ActivityLogs
RefreshTokens
```

### 7.1 Users

Stores common user account data.

Example fields:

```txt
Id
Email
PasswordHash
FirstName
LastName
Role
IsActive
CreatedAt
UpdatedAt
```

### 7.2 Citizens

Stores citizen-specific data.

Example fields:

```txt
Id
UserId
Pesel
DateOfBirth
Street
City
PostalCode
Country
CreatedAt
UpdatedAt
```

### 7.3 Admins

Stores admin-specific data.

Example fields:

```txt
Id
UserId
Position
CreatedAt
UpdatedAt
```

### 7.4 Documents

Stores citizen documents.

Example fields:

```txt
Id
CitizenId
DocumentTypeId
DocumentNumber
IssueDate
ExpirationDate
Status
QrCodeValue
CreatedAt
UpdatedAt
```

### 7.5 DocumentTypes

Stores available document types.

Example fields:

```txt
Id
Name
Code
Description
IsActive
CreatedAt
UpdatedAt
```

### 7.6 ActivityLogs

Stores system activity.

Example fields:

```txt
Id
UserId
Action
Description
IpAddress
CreatedAt
```

### 7.7 RefreshTokens

Stores refresh tokens.

Example fields:

```txt
Id
UserId
Token
ExpiresAt
RevokedAt
CreatedAt
```

---

## 8. MVP Seed Data

The MVP should include initial seed data.

Required seed data:

```txt
Default roles
Default SuperAdmin account
Default Admin account
Default document types
Example citizen account
Example documents
```

Example roles:

```txt
Citizen
Admin
SuperAdmin
```

Example document types:

```txt
Identity Card
Driving License
Student Card
Passport
```

Example users:

```txt
superadmin@mobywatel.local
admin@mobywatel.local
citizen@mobywatel.local
```

---

## 9. MVP API Scope

The MVP backend should expose two logical API surfaces.

```txt
User API
  /api/user/auth
  /api/user/profile
  /api/user/documents
  /api/user/activity-logs

Admin API
  /api/admin/auth
  /api/admin/dashboard
  /api/admin/users
  /api/admin/admins
  /api/admin/documents
  /api/admin/document-types
  /api/admin/activity-logs
```

### 9.1 Auth Endpoints

```txt
POST /api/user/auth/login
POST /api/user/auth/refresh-token
POST /api/user/auth/logout
GET  /api/user/auth/me

POST /api/admin/auth/login
POST /api/admin/auth/refresh-token
POST /api/admin/auth/logout
GET  /api/admin/auth/me
```

### 9.2 Citizen Endpoints

```txt
GET /api/user/profile
GET /api/user/documents
GET /api/user/documents/{id}
GET /api/user/documents/{id}/qr-code
GET /api/user/activity-logs
```

### 9.3 Admin Endpoints

```txt
GET   /api/admin/dashboard
GET   /api/admin/users
GET   /api/admin/users/{id}
POST  /api/admin/users
PUT   /api/admin/users/{id}
PATCH /api/admin/users/{id}/status
```

### 9.4 SuperAdmin Endpoints

```txt
GET   /api/admin/admins
POST  /api/admin/admins
PUT   /api/admin/admins/{id}
PATCH /api/admin/admins/{id}/status
```

### 9.5 Document Endpoints

```txt
GET    /api/admin/documents
GET    /api/admin/documents/{id}
POST   /api/admin/documents
PUT    /api/admin/documents/{id}
PATCH  /api/admin/documents/{id}/status
DELETE /api/admin/documents/{id}
```

### 9.6 Document Type Endpoints

```txt
GET   /api/admin/document-types
GET   /api/admin/document-types/{id}
POST  /api/admin/document-types
PUT   /api/admin/document-types/{id}
PATCH /api/admin/document-types/{id}/status
```

### 9.7 Activity Log Endpoints

```txt
GET /api/admin/activity-logs
GET /api/admin/activity-logs/{id}
```

---

## 10. Out of Scope for MVP

The following features are not included in the MVP:

```txt
Integration with real government systems
Real identity verification
Trusted profile integration
Biometric authentication
Payment services
Legal document validation
Production-level security certification
External document registry integration
Push notifications
Offline mode
Advanced analytics
Advanced QR code verification
Document expiration reminders
Email notifications
SMS notifications
Multi-language support
Mobile app store release
Real personal data processing
```

These features can be considered in future versions.

---

## 11. MVP Non-Functional Requirements

### 11.1 Architecture

The backend should be a monolith with internal Clean Architecture.

Required backend layers:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
```

Acceptance criteria:

- Domain layer does not depend on other layers.
- Application layer contains use cases.
- Application layer uses CQRS + MediatR.
- Infrastructure layer contains database and technical implementations.
- API layer exposes separate User API and Admin API endpoints.
- Business logic is not placed directly inside controllers.

### 11.2 Database

The MVP should use a relational database.

Acceptance criteria:

- Database runs locally.
- Database can be started using Docker.
- Entity Framework migrations are available.
- Seed data can be inserted.
- Backend connects to database successfully.

### 11.3 Docker

Docker should simplify local development.

Acceptance criteria:

- Backend can run using Docker.
- Database can run using Docker.
- Admin web can run using Docker.
- Citizen web can run using Docker.
- Project can be started using `docker-compose`.

### 11.4 API Documentation

The backend should expose API documentation.

Acceptance criteria:

- Swagger/OpenAPI is available.
- Main endpoints are documented.
- Request and response models are visible.
- Another developer can test endpoints from Swagger.

### 11.5 Error Handling

The backend should handle errors consistently.

Acceptance criteria:

- Invalid login returns proper error.
- Missing token returns unauthorized response.
- Invalid role returns forbidden response.
- Not found resource returns not found response.
- Validation errors return clear messages.

### 11.6 Security

The MVP should include basic security mechanisms.

Acceptance criteria:

- Passwords are hashed.
- API uses JWT authentication.
- Endpoints are protected by role.
- Citizen can access only own documents.
- Admin endpoints are not available to citizens.
- Important actions are logged.

---

## 12. MVP Success Criteria

The MVP is successful when:

```txt
Backend monolith runs correctly.
Database runs in Docker.
Swagger documentation is available.
Citizen can log in.
Citizen can view own profile.
Citizen can view own documents.
Citizen can open document details.
Citizen can display QR code.
Citizen can view activity history.
Admin can log in.
Admin can view dashboard.
Admin can manage users.
Admin can manage documents.
Admin can update document statuses.
SuperAdmin can manage document types.
Activity logs are saved and displayed.
Citizen mobile app communicates with User API.
Citizen web app communicates with User API.
Admin web panel communicates with Admin API.
Project can be started by another person using documentation.
```

---

## 13. MVP Development Priority

The MVP should be implemented in the following order.

### Priority 1 — Backend Foundation

```txt
Create solution and backend projects
Configure Clean Architecture dependencies
Configure CQRS and MediatR
Create domain entities
Configure database context
Configure migrations
Configure Docker database
Configure Swagger
```

### Priority 2 — Authentication

```txt
Create user model
Implement password hashing
Implement JWT token service
Implement login endpoint
Implement refresh token endpoint
Implement authorization by role
Seed default users
```

### Priority 3 — Documents

```txt
Create document entities
Create document type entities
Create document commands
Create document queries
Create document endpoints
Create document status update
Create QR code value generation
```

### Priority 4 — Citizen Features

```txt
Get citizen profile
Get citizen documents
Get document details
Get QR code
Get citizen activity history
```

### Priority 5 — Admin Features

```txt
Admin dashboard
User management
Document management
Document type management
Activity logs
```

### Priority 6 — Frontend Applications

```txt
Citizen mobile login
Citizen mobile documents
Citizen mobile document details
Citizen mobile QR code
Citizen web login
Citizen web documents
Admin web login
Admin web dashboard
Admin web user management
Admin web document management
```

### Priority 7 — Testing and Documentation

```txt
Add backend tests
Add API endpoint tests
Write README.md
Write architecture documentation
Write API documentation
Write database documentation
```

---

## 14. Final MVP Summary

The MVP version of Mobywatel is a working demo system with one backend monolith, internal Clean Architecture, User API, Admin API and three client applications.

The citizen can use mobile or web application to view personal documents and activity history.

The administrator can use the admin web panel to manage users, documents, document types and activity logs.

The SuperAdmin role demonstrates extended permissions and system-level management.

The MVP focuses on clean structure, working features, CQRS + MediatR use cases and clear separation between citizen functionality and administration functionality.
