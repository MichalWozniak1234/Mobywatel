# Database Concept

This document describes the database concept for the Mobywatel MVP.

The database is designed for a student/demo digital citizen document platform. The system allows citizens to view their digital documents, while administrators manage users, documents, document types, statuses and activity logs.

The backend uses:

```txt
ASP.NET Core Web API
ASP.NET Core Identity
Entity Framework Core
MediatR
PostgreSQL
Docker Compose
```

The database is accessed only by the backend monolith. Mobile and web applications communicate with the database indirectly through User API or Admin API.

```txt
CitizenMobile  ─┐
CitizenWeb     ├──>  User API  ─┐
                               ├──> Backend Monolith ───> PostgreSQL Database
AdminWeb       ─────>  Admin API ┘
```

---

## 1. Database Goals

The database should support the MVP functionality:

- user authentication,
- role-based authorization,
- citizen profile management,
- admin profile management,
- document management,
- document type management,
- document status tracking,
- QR code preview,
- activity logging,
- refresh token management,
- seed data for local demo.

The database should remain simple, relational and easy to explain during project presentation.

---

## 2. Database Technology

The MVP database will use:

```txt
PostgreSQL
```

The database will be managed from the backend using:

```txt
Entity Framework Core
```

Authentication tables will be created using:

```txt
ASP.NET Core Identity
```

Local development database should run in Docker Compose.

Example database service:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: mobywatel-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: mobywatel_db
      POSTGRES_USER: mobywatel_user
      POSTGRES_PASSWORD: mobywatel_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## 3. Database Scope

Because the project uses ASP.NET Core Identity, the database contains two groups of tables:

```txt
Identity tables
Domain tables
```

Identity tables are responsible for users, roles, claims, logins and tokens.

Domain tables are responsible for business data such as citizens, admins, documents, document types and activity logs.

---

## 4. Planned Tables

The database should contain **13 tables** in the MVP version.

### 4.1 Identity Tables

```txt
1. AspNetUsers
2. AspNetRoles
3. AspNetUserRoles
4. AspNetUserClaims
5. AspNetRoleClaims
6. AspNetUserLogins
7. AspNetUserTokens
```

These tables are created and managed by ASP.NET Core Identity.

### 4.2 Domain Tables

```txt
8. Citizens
9. Admins
10. Documents
11. DocumentTypes
12. ActivityLogs
13. RefreshTokens
```

These tables are created for the business logic of the Mobywatel application.

---

## 5. Why ASP.NET Core Identity Is Used

ASP.NET Core Identity is used because the project needs standard authentication and role management.

Identity provides ready support for:

- user accounts,
- password hashing,
- password validation,
- roles,
- user-role assignment,
- claims,
- external login support if needed in the future,
- security-related user fields.

The system still uses JWT tokens for API authentication, but user accounts and roles are stored through Identity tables.

---

## 6. Table Overview

## 6.1 AspNetUsers

Stores application user accounts.

This table replaces the previously planned custom `Users` table.

Used by:

```txt
Citizen
Admin
SuperAdmin
```

Example important columns:

```txt
Id
UserName
NormalizedUserName
Email
NormalizedEmail
EmailConfirmed
PasswordHash
SecurityStamp
ConcurrencyStamp
PhoneNumber
PhoneNumberConfirmed
TwoFactorEnabled
LockoutEnd
LockoutEnabled
AccessFailedCount
FirstName
LastName
IsActive
CreatedAt
UpdatedAt
```

Recommended custom columns:

```txt
FirstName
LastName
IsActive
CreatedAt
UpdatedAt
```

Role should not be stored as a custom column in `AspNetUsers`. Roles should be assigned through `AspNetRoles` and `AspNetUserRoles`.

---

## 6.2 AspNetRoles

Stores available system roles.

Required roles:

```txt
Citizen
Admin
SuperAdmin
```

Example columns:

```txt
Id
Name
NormalizedName
ConcurrencyStamp
```

---

## 6.3 AspNetUserRoles

Join table between users and roles.

This table defines which user has which role.

Example relationship:

```txt
AspNetUsers * ─── * AspNetRoles
```

Implemented through:

```txt
AspNetUserRoles
```

For the MVP, each user should normally have one role only.

---

## 6.4 AspNetUserClaims

Stores claims assigned directly to users.

This table is part of Identity.

For the MVP, this table can remain unused, but it should stay in the schema because Identity creates it by default.

---

## 6.5 AspNetRoleClaims

Stores claims assigned to roles.

This table is part of Identity.

For the MVP, role-based authorization is enough, so this table can remain unused.

---

## 6.6 AspNetUserLogins

Stores external login provider data.

Examples of external providers could be Google or Microsoft.

For the MVP, external login is out of scope, so this table can remain unused.

---

## 6.7 AspNetUserTokens

Stores Identity user tokens.

For JWT refresh tokens, the project should use a separate `RefreshTokens` table instead of relying on `AspNetUserTokens`, because custom refresh token management is clearer and easier to control in an API-based system.

---

# 7. Domain Tables

## 7.1 Citizens

Stores citizen-specific profile data.

Each citizen is connected to one Identity user.

Relationship:

```txt
AspNetUsers 1 ─── 0..1 Citizens
```

Recommended columns:

```txt
Id uuid PK
UserId text FK -> AspNetUsers.Id
Pesel varchar(11) UNIQUE NOT NULL
DateOfBirth date NOT NULL
Street varchar(200)
City varchar(100)
PostalCode varchar(20)
Country varchar(100)
CreatedAt timestamp NOT NULL
UpdatedAt timestamp NULL
```

Notes:

- `UserId` connects citizen profile to login account.
- `Pesel` should be unique.
- Address is stored directly in `Citizens` for MVP simplicity.
- A separate `Addresses` table is not needed in the MVP.

---

## 7.2 Admins

Stores administrator-specific profile data.

Each admin is connected to one Identity user.

Relationship:

```txt
AspNetUsers 1 ─── 0..1 Admins
```

Recommended columns:

```txt
Id uuid PK
UserId text FK -> AspNetUsers.Id
Position varchar(100)
CreatedAt timestamp NOT NULL
UpdatedAt timestamp NULL
```

Notes:

- Admin and SuperAdmin accounts both use `AspNetUsers`.
- The role is assigned through Identity roles.
- `Admins` stores only additional admin profile data.

---

## 7.3 DocumentTypes

Stores available document categories.

Examples:

```txt
Identity Card
Driving License
Student Card
Passport
Residence Card
```

Relationship:

```txt
DocumentTypes 1 ─── * Documents
```

Recommended columns:

```txt
Id uuid PK
Name varchar(100) NOT NULL
Code varchar(50) UNIQUE NOT NULL
Description varchar(500) NULL
IsActive boolean NOT NULL
CreatedAt timestamp NOT NULL
UpdatedAt timestamp NULL
```

Notes:

- `Code` should be unique.
- Inactive document types cannot be used for new documents.
- Existing documents keep their assigned document type.

---

## 7.4 Documents

Stores documents assigned to citizens.

Relationship:

```txt
Citizens 1 ─── * Documents
DocumentTypes 1 ─── * Documents
```

Recommended columns:

```txt
Id uuid PK
CitizenId uuid FK -> Citizens.Id
DocumentTypeId uuid FK -> DocumentTypes.Id
DocumentNumber varchar(100) NOT NULL
IssueDate date NOT NULL
ExpirationDate date NOT NULL
Status varchar(30) NOT NULL
QrCodeValue varchar(500) NULL
IsDeleted boolean NOT NULL
CreatedAt timestamp NOT NULL
UpdatedAt timestamp NULL
```

Allowed document statuses:

```txt
Active
Expired
Blocked
Pending
Rejected
```

Notes:

- `DocumentNumber` should be unique.
- `ExpirationDate` must be later than `IssueDate`.
- `Status` can be stored as string or enum conversion.
- `QrCodeValue` can be stored in this table for MVP simplicity.
- `IsDeleted` supports soft delete.
- A separate `DocumentStatuses` table is not required for MVP.
- A separate `QrCodes` table is not required for MVP.

---

## 7.5 ActivityLogs

Stores important system actions.

Used for:

- citizen activity history,
- admin activity log preview,
- audit-like tracking for SuperAdmin.

Relationship:

```txt
AspNetUsers 1 ─── * ActivityLogs
```

Recommended columns:

```txt
Id uuid PK
UserId text FK -> AspNetUsers.Id NULL
Action varchar(100) NOT NULL
Description varchar(1000) NOT NULL
IpAddress varchar(100) NULL
Metadata jsonb NULL
CreatedAt timestamp NOT NULL
```

Example actions:

```txt
UserLoggedIn
UserLoggedOut
DocumentViewed
DocumentCreated
DocumentUpdated
DocumentDeleted
DocumentAssigned
DocumentStatusUpdated
UserCreated
UserUpdated
UserStatusChanged
DocumentTypeCreated
DocumentTypeUpdated
DocumentTypeStatusChanged
AdminCreated
AdminUpdated
AdminStatusChanged
```

Notes:

- `Metadata` should use PostgreSQL `jsonb`.
- Sensitive data should not be stored in log descriptions.
- Logs should be append-only in normal application flow.

---

## 7.6 RefreshTokens

Stores refresh tokens used by the JWT authentication flow.

Even though Identity has `AspNetUserTokens`, a separate `RefreshTokens` table is recommended for API refresh tokens.

Relationship:

```txt
AspNetUsers 1 ─── * RefreshTokens
```

Recommended columns:

```txt
Id uuid PK
UserId text FK -> AspNetUsers.Id
Token varchar(500) UNIQUE NOT NULL
ExpiresAt timestamp NOT NULL
RevokedAt timestamp NULL
CreatedAt timestamp NOT NULL
CreatedByIp varchar(100) NULL
RevokedByIp varchar(100) NULL
ReplacedByToken varchar(500) NULL
```

Notes:

- Refresh tokens should be revocable.
- Refresh token rotation is recommended.
- Expired or revoked tokens should not be accepted.
- `Token` should be unique.

---

# 8. Entity Relationship Summary

Main relationships:

```txt
AspNetUsers 1 ─── 0..1 Citizens
AspNetUsers 1 ─── 0..1 Admins
AspNetUsers 1 ─── * RefreshTokens
AspNetUsers 1 ─── * ActivityLogs

AspNetUsers * ─── * AspNetRoles
AspNetUserRoles connects AspNetUsers and AspNetRoles

Citizens 1 ─── * Documents
DocumentTypes 1 ─── * Documents
```

Simplified ERD:

```txt
AspNetRoles
     ▲
     │
AspNetUserRoles
     │
     ▼
AspNetUsers
  │      │       │          │
  │      │       │          └── * ActivityLogs
  │      │       └──────────── * RefreshTokens
  │      └──────────────────── 0..1 Admins
  └─────────────────────────── 0..1 Citizens
                                      │
                                      └── * Documents ─── 1 DocumentTypes
```

---

# 9. Constraints

Recommended constraints:

## 9.1 AspNetUsers

```txt
Email should be unique.
Email is required.
PasswordHash is managed by Identity.
IsActive should default to true.
```

## 9.2 Citizens

```txt
UserId is required and unique.
Pesel is required and unique.
DateOfBirth is required.
```

## 9.3 Admins

```txt
UserId is required and unique.
```

## 9.4 DocumentTypes

```txt
Name is required.
Code is required and unique.
IsActive defaults to true.
```

## 9.5 Documents

```txt
CitizenId is required.
DocumentTypeId is required.
DocumentNumber is required and unique.
IssueDate is required.
ExpirationDate is required.
ExpirationDate must be later than IssueDate.
Status is required.
IsDeleted defaults to false.
```

## 9.6 ActivityLogs

```txt
Action is required.
Description is required.
CreatedAt is required.
```

## 9.7 RefreshTokens

```txt
UserId is required.
Token is required and unique.
ExpiresAt is required.
CreatedAt is required.
```

---

# 10. Indexes

Recommended indexes for MVP:

```txt
AspNetUsers.Email
AspNetUsers.NormalizedEmail
AspNetUsers.NormalizedUserName
AspNetRoles.NormalizedName
Citizens.UserId
Citizens.Pesel
Admins.UserId
Documents.CitizenId
Documents.DocumentTypeId
Documents.DocumentNumber
Documents.Status
Documents.IsDeleted
DocumentTypes.Code
DocumentTypes.IsActive
ActivityLogs.UserId
ActivityLogs.Action
ActivityLogs.CreatedAt
RefreshTokens.UserId
RefreshTokens.Token
RefreshTokens.ExpiresAt
```

These indexes support common queries such as:

- login by email,
- checking user roles,
- loading citizen profile,
- loading citizen documents,
- filtering documents by status,
- loading activity history,
- validating refresh tokens.

---

# 11. Delete Strategy

## 11.1 Documents

For documents, the MVP should use soft delete:

```txt
Documents.IsDeleted = true
```

Reason:

- safer than physical delete,
- keeps historical data for activity logs,
- easier to explain in admin/audit context.

## 11.2 Users

Users should not be physically deleted in MVP.

Instead, deactivate account:

```txt
AspNetUsers.IsActive = false
```

Reason:

- keeps account history,
- avoids breaking relations with logs and documents,
- supports admin status management.

## 11.3 Document Types

Document types should not be deleted if they are used by existing documents.

Instead, deactivate them:

```txt
DocumentTypes.IsActive = false
```

Existing documents keep their assigned type.

---

# 12. Seed Data

The database should include seed data for local testing and presentation.

Required seed data:

```txt
Roles
Default SuperAdmin account
Default Admin account
Default Citizen account
Default document types
Example citizen profile
Example documents
```

## 12.1 Roles

```txt
Citizen
Admin
SuperAdmin
```

## 12.2 Users

Example users:

```txt
superadmin@mobywatel.local
admin@mobywatel.local
citizen@mobywatel.local
```

Each user should have:

```txt
Email
PasswordHash
FirstName
LastName
IsActive = true
Assigned role
```

## 12.3 Document Types

Example document types:

```txt
Identity Card
Driving License
Student Card
Passport
Residence Card
```

## 12.4 Example Documents

Example documents should be assigned to the example citizen.

Possible examples:

```txt
Identity Card - Active
Driving License - Active
Student Card - Pending
```

---

# 13. Entity Framework Core Notes

The backend should use a custom Identity user class.

Example concept:

```csharp
public class ApplicationUser : IdentityUser
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}
```

The database context should inherit from Identity DbContext.

Example concept:

```csharp
public class MobywatelDbContext : IdentityDbContext<ApplicationUser>
{
    public DbSet<Citizen> Citizens { get; set; }
    public DbSet<Admin> Admins { get; set; }
    public DbSet<Document> Documents { get; set; }
    public DbSet<DocumentType> DocumentTypes { get; set; }
    public DbSet<ActivityLog> ActivityLogs { get; set; }
    public DbSet<RefreshToken> RefreshTokens { get; set; }
}
```

Recommended location:

```txt
Mobywatel.Infrastructure/Persistence/MobywatelDbContext.cs
```

Entity configurations should be placed in:

```txt
Mobywatel.Infrastructure/Persistence/Configurations/
```

Suggested configuration files:

```txt
CitizenConfiguration.cs
AdminConfiguration.cs
DocumentConfiguration.cs
DocumentTypeConfiguration.cs
ActivityLogConfiguration.cs
RefreshTokenConfiguration.cs
```

---

# 14. Database and API Mapping

The database supports the following API endpoint groups exposed by the backend monolith:

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

## 14.1 Auth

Uses:

```txt
AspNetUsers
AspNetRoles
AspNetUserRoles
RefreshTokens
ActivityLogs
```

## 14.2 Citizens

Uses:

```txt
AspNetUsers
Citizens
Documents
ActivityLogs
```

## 14.3 Admin Users

Uses:

```txt
AspNetUsers
AspNetRoles
AspNetUserRoles
Citizens
Admins
ActivityLogs
```

## 14.4 Documents

Uses:

```txt
Documents
Citizens
DocumentTypes
ActivityLogs
```

## 14.5 Document Types

Uses:

```txt
DocumentTypes
ActivityLogs
```

## 14.6 Activity Logs

Uses:

```txt
ActivityLogs
AspNetUsers
```

---

# 15. What Is Not Needed in MVP

The following tables are not required in the MVP:

```txt
Addresses
DocumentStatuses
QrCodes
Permissions
Files
Notifications
Settings
AuditMetadata
DocumentVerificationSessions
ExternalProviders
Payments
```

Reasons:

- addresses can be stored in `Citizens`,
- document status can be stored in `Documents.Status`,
- QR code value can be stored in `Documents.QrCodeValue` or generated dynamically,
- permissions are covered by Identity roles,
- files and real document scans are outside MVP scope,
- notifications are outside MVP scope,
- real verification sessions are outside MVP scope,
- external government integrations are outside MVP scope.

---

# 16. Open Decisions

The following decisions can be finalized during implementation:

```txt
Should refresh tokens be rotated on every refresh?
Should DocumentNumber be globally unique or unique per DocumentType?
Should QrCodeValue be stored in Documents or generated dynamically?
Should ActivityLogs.Metadata always be jsonb?
Should Admin see all activity logs or only operational logs?
Should Citizens receive 403 or 404 when accessing another citizen's document?
Should soft-deleted documents remain visible to SuperAdmin?
```

Recommended MVP choices:

```txt
Refresh tokens should be rotated.
DocumentNumber should be globally unique.
QrCodeValue can be stored in Documents.
ActivityLogs.Metadata should be jsonb.
Admin can see standard logs; SuperAdmin can see full logs.
Unauthorized citizen access should return 403.
Soft-deleted documents can remain visible only in admin/audit context.
```

---

# 17. Final Database Summary

The Mobywatel MVP database should contain **13 tables** when ASP.NET Core Identity is used.

```txt
1. AspNetUsers
2. AspNetRoles
3. AspNetUserRoles
4. AspNetUserClaims
5. AspNetRoleClaims
6. AspNetUserLogins
7. AspNetUserTokens
8. Citizens
9. Admins
10. Documents
11. DocumentTypes
12. ActivityLogs
13. RefreshTokens
```

This structure is enough for:

- Identity-based login,
- password hashing,
- roles,
- JWT authentication,
- refresh tokens,
- citizen profiles,
- admin profiles,
- document management,
- document types,
- document statuses,
- QR code preview,
- activity logs,
- dashboard statistics.

The database remains simple, relational and suitable for the MVP scope.
