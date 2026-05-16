# Modules

This document describes the main modules of the Mobywatel application.

The system is divided into backend modules, web frontend modules and mobile frontend modules. Each module has a clear responsibility and should be developed as an independent part of the application as much as possible.

The backend is a single deployable monolith with internal Clean Architecture. It exposes two logical API surfaces:

```txt
User API   - used by CitizenMobile and CitizenWeb
Admin API  - used by AdminWeb
```

Application use cases are implemented with CQRS + MediatR.

---

## 1. Module Overview

The Mobywatel system contains the following main modules:

```txt
Authentication Module
Authorization Module
User Module
Citizen Module
Admin Module
Document Module
Document Type Module
Document Status Module
QR Code Module
Activity Log Module
Dashboard Module
Shared Module
```

The modules are used by three client applications:

```txt
Citizen Mobile Application
Citizen Web Application
Admin Web Panel
```

Client applications communicate with the same backend monolith through different API surfaces.

```txt
CitizenMobile  ─┐
CitizenWeb     ├──>  User API  ─┐
                               ├──> Backend Monolith ───> Database
AdminWeb       ─────>  Admin API ┘
```

---

## 2. Backend Modules

Backend modules are implemented mainly inside the following projects:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
```

Each backend module can contain:

- domain entities,
- commands,
- queries,
- MediatR handlers,
- MediatR pipeline behaviors,
- DTOs,
- validators,
- repository interfaces,
- repository implementations,
- API endpoints,
- tests.

---

# 3. Authentication Module

## 3.1 Purpose

The Authentication Module is responsible for user login, logout and token management.

It allows Citizens, Admins and SuperAdmins to access the system using email and password.

---

## 3.2 Main Responsibilities

The module is responsible for:

- validating user credentials,
- generating JWT access tokens,
- generating refresh tokens,
- refreshing access tokens,
- logging out users,
- storing refresh tokens,
- identifying the current user.

---

## 3.3 Backend Location

```txt
Mobywatel.Application/Auth/
Mobywatel.Infrastructure/Authentication/
Mobywatel.Api/Controllers/UserApi/AuthController.cs
Mobywatel.Api/Controllers/AdminApi/AuthController.cs
```

---

## 3.4 Main Backend Elements

### Commands

```txt
Login
RefreshToken
Logout
```

### Queries

```txt
GetCurrentUser
```

### DTOs

```txt
LoginRequestDto
LoginResponseDto
RefreshTokenRequestDto
CurrentUserDto
```

### Services

```txt
JwtTokenService
PasswordHasher
CurrentUserService
```

### Entities

```txt
User
RefreshToken
```

---

## 3.5 API Endpoints

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

---

## 3.6 Used By

```txt
CitizenMobile
CitizenWeb
AdminWeb
```

---

## 3.7 Acceptance Criteria

- User can log in with email and password.
- Invalid login data returns an error.
- Access token is generated after successful login.
- Refresh token can be used to generate a new access token.
- User can log out.
- Protected endpoints require authentication.
- Current user can be identified from the token.

---

# 4. Authorization Module

## 4.1 Purpose

The Authorization Module controls what a logged-in user is allowed to do.

The system uses role-based access control.

---

## 4.2 User Roles

```txt
Citizen
Admin
SuperAdmin
```

---

## 4.3 Main Responsibilities

The module is responsible for:

- protecting citizen endpoints,
- protecting admin endpoints,
- protecting SuperAdmin endpoints,
- checking if a citizen accesses only own data,
- blocking unauthorized operations,
- returning forbidden responses for invalid roles.

---

## 4.4 Backend Location

```txt
Mobywatel.Application/Common/Security/
Mobywatel.Api/Controllers/UserApi/
Mobywatel.Api/Controllers/AdminApi/
Mobywatel.Api/Filters/
Mobywatel.Api/Middlewares/
```

---

## 4.5 Authorization Rules

```txt
Citizen:
- can access own profile,
- can access own documents,
- can access own activity history,
- cannot access admin panel endpoints.

Admin:
- can manage citizens,
- can manage documents,
- can update document statuses,
- can view activity logs,
- cannot manage other administrators.

SuperAdmin:
- can do everything Admin can do,
- can manage administrators,
- can manage document types,
- can view full audit logs,
- can manage selected system dictionaries.
```

---

## 4.6 Used By

```txt
Authentication Module
Citizen Module
Admin Module
Document Module
Document Type Module
Activity Log Module
Dashboard Module
```

---

## 4.7 Acceptance Criteria

- Citizen cannot access admin endpoints.
- Admin cannot access SuperAdmin-only endpoints.
- Citizen cannot access documents of another citizen.
- Admin can access management endpoints.
- SuperAdmin can access system-level endpoints.

---

# 5. User Module

## 5.1 Purpose

The User Module manages common user account data.

A user account can belong to a Citizen, Admin or SuperAdmin.

---

## 5.2 Main Responsibilities

The module is responsible for:

- storing basic user data,
- creating user accounts,
- updating user data,
- activating users,
- deactivating users,
- storing user role,
- connecting user account with Citizen or Admin profile.

---

## 5.3 Backend Location

```txt
Mobywatel.Domain/Entities/User.cs
Mobywatel.Application/Users/
Mobywatel.Infrastructure/Repositories/UserRepository.cs
Mobywatel.Api/Controllers/AdminApi/DashboardController.cs
```

If the project does not have a separate `Users` folder in `Mobywatel.Application`, user use cases can be placed inside:

```txt
Mobywatel.Application/Citizens/
Mobywatel.Application/Admins/
```

---

## 5.4 Main Backend Elements

### Commands

```txt
CreateUser
UpdateUser
ActivateUser
DeactivateUser
```

### Queries

```txt
GetUsers
GetUserDetails
```

### DTOs

```txt
UserDto
UserDetailsDto
CreateUserRequestDto
UpdateUserRequestDto
UpdateUserStatusRequestDto
```

### Entities

```txt
User
Citizen
Admin
```

---

## 5.5 API Endpoints

```txt
GET   /api/admin/users
GET   /api/admin/users/{id}
POST  /api/admin/users
PUT   /api/admin/users/{id}
PATCH /api/admin/users/{id}/status
```

---

## 5.6 Used By

```txt
AdminWeb
Authentication Module
Citizen Module
Admin Module
Activity Log Module
```

---

## 5.7 Acceptance Criteria

- Admin can view user list.
- Admin can view user details.
- Admin can create citizen account.
- Admin can update user data.
- Admin can activate or deactivate user.
- Citizen cannot access user management.
- User operations are saved in activity logs.

---

# 6. Citizen Module

## 6.1 Purpose

The Citizen Module manages citizen-specific data and citizen-facing functionality.

It is used by both the mobile application and citizen web application.

---

## 6.2 Main Responsibilities

The module is responsible for:

- storing citizen profile data,
- returning current citizen profile,
- returning citizen documents,
- returning citizen activity history,
- allowing admins to view citizen details,
- allowing admins to update citizen data.

---

## 6.3 Backend Location

```txt
Mobywatel.Domain/Entities/Citizen.cs
Mobywatel.Application/Citizens/
Mobywatel.Api/Controllers/UserApi/ProfileController.cs
Mobywatel.Api/Controllers/AdminApi/UsersController.cs
```

---

## 6.4 Main Backend Elements

### Commands

```txt
CreateCitizen
UpdateCitizen
```

### Queries

```txt
GetCurrentCitizenProfile
GetCitizenDetails
GetCitizenDocuments
GetCitizenActivityLogs
```

### DTOs

```txt
CitizenDto
CitizenDetailsDto
CreateCitizenRequestDto
UpdateCitizenRequestDto
```

### Entities

```txt
Citizen
User
Document
ActivityLog
```

---

## 6.5 API Endpoints

```txt
GET /api/user/profile
GET /api/user/documents
GET /api/user/activity-logs
GET /api/admin/users/{id}
PUT /api/admin/users/{id}
```

---

## 6.6 Used By

```txt
CitizenMobile
CitizenWeb
AdminWeb
Document Module
Activity Log Module
```

---

## 6.7 Acceptance Criteria

- Citizen can view own profile.
- Citizen can view only own documents.
- Citizen can view own activity history.
- Admin can view citizen details.
- Admin can update citizen data.
- Citizen cannot update protected fields directly.

---

# 7. Admin Module

## 7.1 Purpose

The Admin Module manages administrator-specific functionality.

It is mainly used by the Admin Web Panel.

---

## 7.2 Main Responsibilities

The module is responsible for:

- storing admin profile data,
- allowing SuperAdmin to view admin accounts,
- allowing SuperAdmin to create admin accounts,
- allowing SuperAdmin to update admin accounts,
- allowing SuperAdmin to activate or deactivate admin accounts.

---

## 7.3 Backend Location

```txt
Mobywatel.Domain/Entities/Admin.cs
Mobywatel.Application/Admins/
Mobywatel.Api/Controllers/AdminApi/UsersController.cs
```

If the project does not have a separate `Admins` folder in the Application layer, admin management can be placed inside:

```txt
Mobywatel.Application/Citizens/
Mobywatel.Application/Common/Security/
```

For clearer structure, a separate `Admins` folder is recommended.

---

## 7.4 Main Backend Elements

### Commands

```txt
CreateAdmin
UpdateAdmin
ActivateAdmin
DeactivateAdmin
```

### Queries

```txt
GetAdmins
GetAdminDetails
```

### DTOs

```txt
AdminDto
AdminDetailsDto
CreateAdminRequestDto
UpdateAdminRequestDto
UpdateAdminStatusRequestDto
```

### Entities

```txt
Admin
User
```

---

## 7.5 API Endpoints

```txt
GET   /api/admin/admins
POST  /api/admin/admins
PUT   /api/admin/admins/{id}
PATCH /api/admin/admins/{id}/status
```

---

## 7.6 Used By

```txt
AdminWeb
Authorization Module
Activity Log Module
```

---

## 7.7 Acceptance Criteria

- SuperAdmin can view admin list.
- SuperAdmin can create admin account.
- SuperAdmin can update admin account.
- SuperAdmin can activate or deactivate admin account.
- Admin cannot manage other admin accounts.
- Citizen cannot access admin management.

---

# 8. Document Module

## 8.1 Purpose

The Document Module is the main business module of the system.

It manages documents assigned to citizens.

---

## 8.2 Main Responsibilities

The module is responsible for:

- storing document data,
- creating documents,
- assigning documents to citizens,
- updating document data,
- updating document status,
- deleting or deactivating documents,
- returning document details,
- returning citizen document list,
- returning admin document list.

---

## 8.3 Backend Location

```txt
Mobywatel.Domain/Entities/Document.cs
Mobywatel.Application/Documents/
Mobywatel.Infrastructure/Repositories/DocumentRepository.cs
Mobywatel.Api/Controllers/UserApi/DocumentsController.cs
Mobywatel.Api/Controllers/AdminApi/DocumentsController.cs
```

---

## 8.4 Main Backend Elements

### Commands

```txt
CreateDocument
AssignDocumentToCitizen
UpdateDocument
UpdateDocumentStatus
DeleteDocument
```

### Queries

```txt
GetCitizenDocuments
GetDocumentDetails
GetAllDocuments
```

### DTOs

```txt
DocumentDto
DocumentDetailsDto
CreateDocumentRequestDto
UpdateDocumentRequestDto
UpdateDocumentStatusRequestDto
```

### Entities

```txt
Document
Citizen
DocumentType
DocumentStatus
```

---

## 8.5 API Endpoints

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

---

## 8.6 Used By

```txt
CitizenMobile
CitizenWeb
AdminWeb
Citizen Module
Document Type Module
Document Status Module
QR Code Module
Activity Log Module
```

---

## 8.7 Acceptance Criteria

- Citizen can view only own documents.
- Citizen can view own document details.
- Admin can view all documents.
- Admin can create documents.
- Admin can assign documents to citizens.
- Admin can update document data.
- Admin can update document status.
- Admin can delete or deactivate documents.
- Document operations are saved in activity logs.

---

# 9. Document Type Module

## 9.1 Purpose

The Document Type Module manages available categories of documents.

Examples:

```txt
Identity Card
Driving License
Student Card
Passport
Residence Card
```

---

## 9.2 Main Responsibilities

The module is responsible for:

- storing document types,
- returning document type list,
- creating document types,
- updating document types,
- activating document types,
- deactivating document types,
- preventing inactive document types from being used for new documents.

---

## 9.3 Backend Location

```txt
Mobywatel.Domain/Entities/DocumentType.cs
Mobywatel.Application/DocumentTypes/
Mobywatel.Infrastructure/Persistence/Configurations/DocumentTypeConfiguration.cs
Mobywatel.Api/Controllers/AdminApi/DocumentTypesController.cs
```

---

## 9.4 Main Backend Elements

### Commands

```txt
CreateDocumentType
UpdateDocumentType
ActivateDocumentType
DeactivateDocumentType
```

### Queries

```txt
GetDocumentTypes
GetDocumentTypeDetails
```

### DTOs

```txt
DocumentTypeDto
DocumentTypeDetailsDto
CreateDocumentTypeRequestDto
UpdateDocumentTypeRequestDto
UpdateDocumentTypeStatusRequestDto
```

### Entities

```txt
DocumentType
Document
```

---

## 9.5 API Endpoints

```txt
GET   /api/admin/document-types
GET   /api/admin/document-types/{id}
POST  /api/admin/document-types
PUT   /api/admin/document-types/{id}
PATCH /api/admin/document-types/{id}/status
```

---

## 9.6 Used By

```txt
AdminWeb
Document Module
Activity Log Module
```

---

## 9.7 Acceptance Criteria

- Admin can view document types.
- SuperAdmin can create document types.
- SuperAdmin can update document types.
- SuperAdmin can activate or deactivate document types.
- Inactive document type cannot be used for new documents.
- Existing documents keep their assigned document type.

---

# 10. Document Status Module

## 10.1 Purpose

The Document Status Module defines and controls the status of documents.

---

## 10.2 Available Statuses

```txt
Active
Expired
Blocked
Pending
Rejected
```

---

## 10.3 Status Meaning

```txt
Active   - document is valid and visible as active
Expired  - document is no longer valid
Blocked  - document was blocked by administrator
Pending  - document waits for confirmation or activation
Rejected - document was rejected by administrator
```

---

## 10.4 Main Responsibilities

The module is responsible for:

- defining allowed statuses,
- validating document status changes,
- displaying document status to citizens,
- allowing admins to update document status,
- logging document status changes.

---

## 10.5 Backend Location

```txt
Mobywatel.Domain/Entities/DocumentStatus.cs
Mobywatel.Domain/Enums/DocumentStatusType.cs
Mobywatel.Application/Documents/Commands/UpdateDocumentStatus/
```

---

## 10.6 Main Backend Elements

### Commands

```txt
UpdateDocumentStatus
```

### DTOs

```txt
UpdateDocumentStatusRequestDto
DocumentStatusDto
```

### Entities and Enums

```txt
Document
DocumentStatus
DocumentStatusType
```

---

## 10.7 API Endpoints

```txt
PATCH /api/admin/documents/{id}/status
```

---

## 10.8 Used By

```txt
CitizenMobile
CitizenWeb
AdminWeb
Document Module
Activity Log Module
```

---

## 10.9 Acceptance Criteria

- Admin can update document status.
- Citizen can view document status.
- Invalid status value is rejected.
- Status update is saved in the database.
- Status update creates activity log entry.

---

# 11. QR Code Module

## 11.1 Purpose

The QR Code Module provides QR code data for citizen documents.

In the MVP, QR code is used only as a document preview feature. It does not integrate with real external verification systems.

---

## 11.2 Main Responsibilities

The module is responsible for:

- generating QR code value for a document,
- returning QR code value to frontend,
- allowing citizen to display QR code,
- preventing citizen from accessing QR code of another citizen's document.

---

## 11.3 Backend Location

```txt
Mobywatel.Infrastructure/Services/QrCodeService.cs
Mobywatel.Application/Documents/Queries/GetDocumentQrCode/
Mobywatel.Api/Controllers/UserApi/DocumentsController.cs
```

---

## 11.4 Main Backend Elements

### Queries

```txt
GetDocumentQrCode
```

### DTOs

```txt
DocumentQrCodeDto
```

### Services

```txt
QrCodeService
```

### Entities

```txt
Document
Citizen
```

---

## 11.5 API Endpoints

```txt
GET /api/user/documents/{id}/qr-code
```

---

## 11.6 Used By

```txt
CitizenMobile
CitizenWeb
Document Module
Authorization Module
```

---

## 11.7 Acceptance Criteria

- Citizen can display QR code for own document.
- Citizen cannot display QR code for another citizen's document.
- QR code value is connected with selected document.
- QR code value is generated or returned by backend.
- QR code verification by external systems is not required in MVP.

---

# 12. Activity Log Module

## 12.1 Purpose

The Activity Log Module tracks important actions performed in the system.

It is used for citizen activity history and admin audit preview.

---

## 12.2 Main Responsibilities

The module is responsible for:

- saving login activity,
- saving document view activity,
- saving document creation activity,
- saving document assignment activity,
- saving document status update activity,
- saving user management activity,
- saving document type management activity,
- returning citizen activity history,
- returning admin activity logs,
- returning activity log details.

---

## 12.3 Backend Location

```txt
Mobywatel.Domain/Entities/ActivityLog.cs
Mobywatel.Application/ActivityLogs/
Mobywatel.Infrastructure/Repositories/ActivityLogRepository.cs
Mobywatel.Api/Controllers/UserApi/ActivityLogsController.cs
Mobywatel.Api/Controllers/AdminApi/ActivityLogsController.cs
```

---

## 12.4 Main Backend Elements

### Commands

```txt
CreateActivityLog
```

### Queries

```txt
GetCitizenActivityLogs
GetActivityLogs
GetActivityLogDetails
```

### DTOs

```txt
ActivityLogDto
ActivityLogDetailsDto
CreateActivityLogRequestDto
```

### Entities

```txt
ActivityLog
User
```

---

## 12.5 Example Activity Types

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
AdminCreated
AdminUpdated
```

---

## 12.6 API Endpoints

```txt
GET /api/user/activity-logs
GET /api/admin/activity-logs
GET /api/admin/activity-logs/{id}
```

---

## 12.7 Used By

```txt
CitizenMobile
CitizenWeb
AdminWeb
Authentication Module
User Module
Citizen Module
Document Module
Document Type Module
Admin Module
```

---

## 12.8 Acceptance Criteria

- Important actions create activity logs.
- Citizen can view own activity history.
- Admin can view activity logs.
- SuperAdmin can view full audit logs.
- Activity log contains user, action, description, date and optional IP address.

---

# 13. Dashboard Module

## 13.1 Purpose

The Dashboard Module provides summary data for the Admin Web Panel.

It allows administrators to quickly understand the current system state.

---

## 13.2 Main Responsibilities

The module is responsible for returning:

- total users count,
- total citizens count,
- total documents count,
- active documents count,
- expired documents count,
- blocked documents count,
- recent activity list.

---

## 13.3 Backend Location

```txt
Mobywatel.Application/Dashboard/
Mobywatel.Api/Controllers/AdminApi/AdminsController.cs
```

If the project does not have a separate `Dashboard` folder, dashboard queries can be placed inside:

```txt
Mobywatel.Application/Common/
```

or:

```txt
Mobywatel.Application/Admins/
```

---

## 13.4 Main Backend Elements

### Queries

```txt
GetAdminDashboard
```

### DTOs

```txt
AdminDashboardDto
DashboardStatisticsDto
RecentActivityDto
```

---

## 13.5 API Endpoints

```txt
GET /api/admin/dashboard
```

---

## 13.6 Used By

```txt
AdminWeb
Activity Log Module
Document Module
User Module
Citizen Module
```

---

## 13.7 Acceptance Criteria

- Admin can view dashboard after login.
- Citizen cannot access admin dashboard.
- Dashboard returns basic statistics.
- Dashboard returns recent activity.
- Data is loaded from Admin API.

---

# 14. Shared Module

## 14.1 Purpose

The Shared Module contains reusable elements used by more than one module.

It should remain small and should not contain core business logic.

---

## 14.2 Backend Location

```txt
Mobywatel.Shared/
Mobywatel.Application/Common/
Mobywatel.Domain/Common/
```

---

## 14.3 Main Responsibilities

The module is responsible for:

- shared constants,
- shared result models,
- common exceptions,
- common interfaces,
- base entities,
- helper methods,
- reusable DTO structures.

---

## 14.4 Main Elements

```txt
Constants
Contracts
Helpers
Result
BaseEntity
AuditableEntity
ApplicationException
ValidationException
NotFoundException
UnauthorizedException
ForbiddenException
```

---

## 14.5 Used By

```txt
Authentication Module
Authorization Module
User Module
Citizen Module
Admin Module
Document Module
Document Type Module
Activity Log Module
Dashboard Module
```

---

## 14.6 Acceptance Criteria

- Shared code is reusable.
- Shared module does not contain feature-specific business logic.
- Common exceptions are reused across modules.
- Common response models are consistent.

---

# 15. Citizen Mobile Modules

The Citizen Mobile Application contains modules focused on citizen features.

---

## 15.1 Mobile Authentication Module

### Location

```txt
CitizenMobile/src/api/authApi.ts
CitizenMobile/src/store/authStore.ts
CitizenMobile/src/screens/LoginScreen.tsx
CitizenMobile/src/navigation/AuthNavigator.tsx
```

### Responsibilities

- display login screen,
- send login request,
- store authentication token,
- handle logout,
- protect authenticated screens.

---

## 15.2 Mobile Profile Module

### Location

```txt
CitizenMobile/src/screens/ProfileScreen.tsx
CitizenMobile/src/api/userApi.ts
CitizenMobile/src/store/userStore.ts
```

### Responsibilities

- load citizen profile,
- display citizen profile,
- handle profile loading state,
- handle profile error state.

---

## 15.3 Mobile Documents Module

### Location

```txt
CitizenMobile/src/screens/DocumentsScreen.tsx
CitizenMobile/src/screens/DocumentDetailsScreen.tsx
CitizenMobile/src/api/documentsApi.ts
CitizenMobile/src/store/documentStore.ts
CitizenMobile/src/components/DocumentCard.tsx
CitizenMobile/src/components/StatusBadge.tsx
```

### Responsibilities

- load citizen document list,
- display document cards,
- open document details,
- display document status,
- handle loading and error states.

---

## 15.4 Mobile QR Code Module

### Location

```txt
CitizenMobile/src/screens/QrCodeScreen.tsx
CitizenMobile/src/components/QrCodePreview.tsx
CitizenMobile/src/api/documentsApi.ts
```

### Responsibilities

- load QR code value,
- display QR code,
- handle QR code loading state,
- handle QR code access error.

---

## 15.5 Mobile Activity History Module

### Location

```txt
CitizenMobile/src/screens/ActivityHistoryScreen.tsx
CitizenMobile/src/api/activityApi.ts
```

### Responsibilities

- load citizen activity history,
- display activity list,
- handle empty activity history,
- handle loading and error states.

---

# 16. Citizen Web Modules

The Citizen Web Application contains modules focused on citizen functionality in a browser.

---

## 16.1 Citizen Web Authentication Module

### Location

```txt
CitizenWeb/src/app/features/auth/
CitizenWeb/src/app/core/guards/
CitizenWeb/src/app/core/interceptors/
```

### Responsibilities

- display login page,
- send login request,
- store token,
- protect citizen routes,
- add JWT token to API requests.

---

## 16.2 Citizen Web Dashboard Module

### Location

```txt
CitizenWeb/src/app/features/dashboard/
```

### Responsibilities

- display citizen dashboard,
- show basic citizen information,
- show document shortcuts,
- show recent activity preview.

---

## 16.3 Citizen Web Profile Module

### Location

```txt
CitizenWeb/src/app/features/profile/
```

### Responsibilities

- load citizen profile,
- display profile data,
- handle loading and error states.

---

## 16.4 Citizen Web Documents Module

### Location

```txt
CitizenWeb/src/app/features/documents/
```

### Responsibilities

- load citizen document list,
- display documents,
- open document details,
- display document status,
- display QR code preview if available.

---

## 16.5 Citizen Web Activity History Module

### Location

```txt
CitizenWeb/src/app/features/activity-history/
```

### Responsibilities

- load citizen activity history,
- display activity logs,
- handle empty state,
- handle loading and error states.

---

# 17. Admin Web Modules

The Admin Web Panel contains modules focused on administration.

---

## 17.1 Admin Web Authentication Module

### Location

```txt
AdminWeb/src/app/features/auth/
AdminWeb/src/app/core/guards/
AdminWeb/src/app/core/interceptors/
```

### Responsibilities

- display admin login page,
- send login request,
- verify Admin or SuperAdmin role,
- store token,
- protect admin routes,
- add JWT token to API requests.

---

## 17.2 Admin Web Dashboard Module

### Location

```txt
AdminWeb/src/app/features/dashboard/
```

### Responsibilities

- load dashboard statistics,
- display total users,
- display total citizens,
- display total documents,
- display active/expired/blocked documents,
- display recent activity.

---

## 17.3 Admin Web Users Module

### Location

```txt
AdminWeb/src/app/features/users/
```

### Responsibilities

- display user list,
- display user details,
- create citizen user,
- update user data,
- activate or deactivate users,
- handle user form validation.

---

## 17.4 Admin Web Documents Module

### Location

```txt
AdminWeb/src/app/features/documents/
```

### Responsibilities

- display document list,
- display document details,
- create document,
- assign document to citizen,
- update document data,
- update document status,
- delete or deactivate document.

---

## 17.5 Admin Web Document Types Module

### Location

```txt
AdminWeb/src/app/features/document-types/
```

### Responsibilities

- display document types,
- create document type,
- update document type,
- activate or deactivate document type,
- restrict selected actions to SuperAdmin.

---

## 17.6 Admin Web Activity Logs Module

### Location

```txt
AdminWeb/src/app/features/activity-logs/
```

### Responsibilities

- display activity logs,
- display activity log details,
- filter logs by user, action or date,
- restrict full audit visibility to SuperAdmin if needed.

---

## 17.7 Admin Web System Settings Module

### Location

```txt
AdminWeb/src/app/features/system-settings/
```

### Responsibilities

- display selected system settings,
- manage selected dictionaries,
- restrict access to SuperAdmin.

---

## 17.8 Admin Web Layout Module

### Location

```txt
AdminWeb/src/app/layout/
```

### Responsibilities

- define admin page layout,
- display sidebar,
- display topbar,
- handle navigation,
- display logged-in user information.

---

# 18. Module Dependencies

The modules should follow clear dependency rules.

---

## 18.1 Backend Dependency Direction

```txt
Mobywatel.Api
    ↓
Mobywatel.Application
    ↓
Mobywatel.Domain

Mobywatel.Infrastructure
    ↓
Mobywatel.Application
    ↓
Mobywatel.Domain
```

The Domain layer should not depend on any other backend layer.

---

## 18.2 Module Dependency Summary

```txt
Authentication Module
    depends on User Module, Activity Log Module

Authorization Module
    depends on User Module

Citizen Module
    depends on User Module, Document Module, Activity Log Module

Admin Module
    depends on User Module, Activity Log Module

Document Module
    depends on Citizen Module, Document Type Module, Document Status Module, QR Code Module, Activity Log Module

Document Type Module
    depends on Activity Log Module

Document Status Module
    depends on Document Module, Activity Log Module

QR Code Module
    depends on Document Module

Activity Log Module
    depends on User Module

Dashboard Module
    depends on User Module, Citizen Module, Document Module, Activity Log Module
```

---

# 19. Recommended Implementation Order

The modules should be implemented in this order:

```txt
1. Shared Module
2. User Module
3. Authentication Module
4. Authorization Module
5. Citizen Module
6. Document Type Module
7. Document Status Module
8. Document Module
9. QR Code Module
10. Activity Log Module
11. Dashboard Module
12. Admin Module
13. Citizen Mobile Modules
14. Citizen Web Modules
15. Admin Web Modules
```

This order is recommended because each later module depends on earlier modules.

---

# 20. Module Acceptance Summary

The MVP module implementation is complete when:

- users can authenticate,
- roles control access correctly,
- citizens can view their profiles,
- citizens can view their documents,
- citizens can display QR codes,
- admins can manage users,
- admins can manage documents,
- SuperAdmins can manage admins and document types,
- document statuses can be updated,
- activity logs are created and displayed,
- admin dashboard shows basic statistics,
- CitizenMobile and CitizenWeb communicate with the User API,
- AdminWeb communicates with the Admin API,
- backend code is a monolith with internal Clean Architecture,
- application use cases use CQRS + MediatR,
- modules are separated by responsibility.
