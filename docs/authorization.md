# Authorization

This document describes the authorization concept for the Mobywatel MVP.

Authorization defines what an authenticated user is allowed to do in the system. Authentication answers the question: **who is the user?** Authorization answers the question: **what is the user allowed to access?**

The Mobywatel MVP uses role-based authorization with additional ownership checks for citizen data.

---

## 1. Authorization Goal

The goal of authorization is to protect system resources from unauthorized access.

The authorization layer must ensure that:

- only authenticated users can access protected endpoints,
- Citizens can access only their own data,
- Admins can manage citizens and documents,
- SuperAdmins can manage administrators and system-level data,
- backend rules are always enforced regardless of frontend behavior,
- unauthorized access attempts return consistent HTTP responses.

Frontend applications may hide unavailable actions in the UI, but the backend must always verify permissions.

---

## 2. Authorization Model

The MVP uses:

```txt
ASP.NET Core Identity
JWT Bearer Authentication
Role-Based Authorization
Ownership-Based Authorization
```

Authorization is based on three main roles:

```txt
Citizen
Admin
SuperAdmin
```

The roles are stored using ASP.NET Core Identity roles.

Relevant Identity tables:

```txt
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetRoleClaims
AspNetUserClaims
```

Domain-specific profile data is stored separately in:

```txt
Citizens
Admins
```

---

## 3. User Roles

## 3.1 Citizen

A Citizen is a regular user of the citizen mobile application and citizen web application.

A Citizen can:

- view own profile,
- view own documents,
- view own document details,
- display QR code for own document,
- view own activity history,
- log out,
- refresh access token.

A Citizen cannot:

- access admin panel endpoints,
- view other citizens' profiles,
- view other citizens' documents,
- create documents,
- update documents,
- assign documents,
- manage users,
- manage document types,
- manage administrators,
- view system-wide activity logs.

---

## 3.2 Admin

An Admin is a user of the admin web panel.

An Admin can:

- access the admin dashboard,
- view user list,
- view user details,
- create citizen accounts,
- update citizen data,
- activate or deactivate citizen users,
- view document list,
- view document details,
- create documents,
- assign documents to citizens,
- update document data,
- update document status,
- delete or deactivate documents,
- view activity logs,
- view document types.

An Admin cannot:

- manage other administrators,
- create admin accounts,
- update admin accounts,
- deactivate admin accounts,
- create document types if this action is reserved for SuperAdmin,
- update document types if this action is reserved for SuperAdmin,
- access SuperAdmin-only system settings.

---

## 3.3 SuperAdmin

A SuperAdmin has the highest access level in the system.

A SuperAdmin can:

- do everything an Admin can do,
- view admin accounts,
- create admin accounts,
- update admin accounts,
- activate or deactivate admin accounts,
- create document types,
- update document types,
- activate or deactivate document types,
- view full activity logs,
- manage selected system dictionaries or settings.

The SuperAdmin role is mainly used to demonstrate extended role-based authorization in the MVP.

---

## 4. Authentication and Authorization Flow

## 4.1 Login Flow

1. User sends login request with email and password.
2. Backend validates credentials using ASP.NET Core Identity.
3. Backend checks whether the user account is active.
4. Backend loads user roles from Identity.
5. Backend creates a JWT access token.
6. JWT contains user identity and role claims.
7. Backend returns access token and refresh token.
8. Client sends the JWT in later requests.

Example authorization header:

```txt
Authorization: Bearer <access_token>
```

---

## 4.2 Request Authorization Flow

For every protected request:

1. Client sends JWT access token in the `Authorization` header.
2. Backend validates token signature and expiration.
3. Backend reads user ID and role claims from the token.
4. Backend checks endpoint role requirements.
5. Backend checks ownership rules if needed.
6. Backend either allows or rejects the request.

---

## 5. JWT Claims

The JWT access token should contain only the claims needed for authorization and user identification.

Recommended claims:

```txt
sub             - Identity user ID
email           - user email
role            - user role: Citizen, Admin or SuperAdmin
jti             - JWT unique identifier
iat             - issued at timestamp
exp             - expiration timestamp
```

Optional claims:

```txt
citizen_id      - citizen profile ID, only for Citizen users
admin_id        - admin profile ID, only for Admin and SuperAdmin users
full_name       - display name, optional
```

Recommended approach:

- always include Identity user ID in `sub`,
- include role claim for role-based policies,
- load domain profile IDs from the database when needed,
- avoid storing sensitive personal data inside JWT.

---

## 6. Authorization Policies

Authorization should use named policies in ASP.NET Core where possible.

Recommended policies:

```txt
CitizenOnly
AdminOnly
SuperAdminOnly
AdminOrSuperAdmin
AuthenticatedUser
```

Example policy meaning:

| Policy | Allowed Roles |
|---|---|
| `AuthenticatedUser` | Citizen, Admin, SuperAdmin |
| `CitizenOnly` | Citizen |
| `AdminOnly` | Admin |
| `SuperAdminOnly` | SuperAdmin |
| `AdminOrSuperAdmin` | Admin, SuperAdmin |

In ASP.NET Core, these policies can be configured in the API project.

Example concept:

```csharp
options.AddPolicy("CitizenOnly", policy =>
    policy.RequireRole("Citizen"));

options.AddPolicy("AdminOrSuperAdmin", policy =>
    policy.RequireRole("Admin", "SuperAdmin"));

options.AddPolicy("SuperAdminOnly", policy =>
    policy.RequireRole("SuperAdmin"));
```

---

## 7. Ownership-Based Authorization

Role checks are not enough for citizen data.

A Citizen must only access resources that belong to their own account.

Ownership checks are required for:

```txt
GET /api/citizens/me
GET /api/citizens/me/documents
GET /api/citizens/me/activity-logs
GET /api/documents/{id}
GET /api/documents/{id}/qr-code
```

For example, a Citizen can call:

```txt
GET /api/documents/{id}
```

but only if the document belongs to the currently authenticated citizen.

The backend should verify this using the database:

```txt
CurrentUser.IdentityUserId
        ↓
Citizens.UserId
        ↓
Documents.CitizenId
```

If the document does not belong to the citizen, the backend must reject the request.

Recommended response:

```txt
403 Forbidden
```

Alternative response:

```txt
404 Not Found
```

For the MVP, `403 Forbidden` is clearer because it directly communicates that the user is authenticated but not allowed to access the resource.

---

## 8. Endpoint Authorization Matrix

| Endpoint | Citizen | Admin | SuperAdmin |
|---|---:|---:|---:|
| `POST /api/auth/login` | Yes | Yes | Yes |
| `POST /api/auth/refresh-token` | Yes | Yes | Yes |
| `POST /api/auth/logout` | Yes | Yes | Yes |
| `GET /api/auth/me` | Yes | Yes | Yes |
| `GET /api/citizens/me` | Yes | No | No |
| `GET /api/citizens/me/documents` | Yes | No | No |
| `GET /api/citizens/me/activity-logs` | Yes | No | No |
| `GET /api/citizens/{id}` | No | Yes | Yes |
| `PUT /api/citizens/{id}` | No | Yes | Yes |
| `GET /api/admin/dashboard` | No | Yes | Yes |
| `GET /api/admin/users` | No | Yes | Yes |
| `GET /api/admin/users/{id}` | No | Yes | Yes |
| `POST /api/admin/users` | No | Yes | Yes |
| `PUT /api/admin/users/{id}` | No | Yes | Yes |
| `PATCH /api/admin/users/{id}/status` | No | Yes | Yes |
| `GET /api/super-admin/admins` | No | No | Yes |
| `POST /api/super-admin/admins` | No | No | Yes |
| `PUT /api/super-admin/admins/{id}` | No | No | Yes |
| `PATCH /api/super-admin/admins/{id}/status` | No | No | Yes |
| `GET /api/documents` | No | Yes | Yes |
| `GET /api/documents/{id}` | Own only | Yes | Yes |
| `POST /api/documents` | No | Yes | Yes |
| `PUT /api/documents/{id}` | No | Yes | Yes |
| `PATCH /api/documents/{id}/status` | No | Yes | Yes |
| `DELETE /api/documents/{id}` | No | Yes | Yes |
| `GET /api/documents/{id}/qr-code` | Own only | Yes | Yes |
| `GET /api/document-types` | No | Yes | Yes |
| `GET /api/document-types/{id}` | No | Yes | Yes |
| `POST /api/document-types` | No | No | Yes |
| `PUT /api/document-types/{id}` | No | No | Yes |
| `PATCH /api/document-types/{id}/status` | No | No | Yes |
| `GET /api/activity-logs` | No | Yes | Yes |
| `GET /api/activity-logs/{id}` | No | Yes | Yes |

---

## 9. Controller-Level Authorization

Authorization should be applied at the controller or endpoint level.

Example concepts:

```csharp
[Authorize]
[ApiController]
[Route("api/auth")]
public class AuthController : ControllerBase
{
}
```

```csharp
[Authorize(Policy = "CitizenOnly")]
[HttpGet("me")]
public async Task<IActionResult> GetCurrentCitizenProfile()
{
}
```

```csharp
[Authorize(Policy = "AdminOrSuperAdmin")]
[HttpGet("users")]
public async Task<IActionResult> GetUsers()
{
}
```

```csharp
[Authorize(Policy = "SuperAdminOnly")]
[HttpPost("admins")]
public async Task<IActionResult> CreateAdmin()
{
}
```

However, controller attributes should not replace business-level checks. Ownership rules must still be verified inside application logic or authorization services.

---

## 10. Application-Level Authorization

Some authorization rules require database checks and should be handled in the Application layer.

Examples:

- checking whether a document belongs to the current citizen,
- checking whether a citizen profile belongs to the current user,
- checking whether an Admin is not trying to manage another Admin,
- checking whether a document type is active before creating a document,
- checking whether a user account is active before allowing login.

Recommended location:

```txt
Mobywatel.Application/Common/Security/
Mobywatel.Application/Common/Interfaces/
Mobywatel.Infrastructure/Services/CurrentUserService.cs
```

Recommended services:

```txt
ICurrentUserService
IAuthorizationService
IOwnershipChecker
```

Example ownership check concept:

```csharp
public async Task<bool> IsDocumentOwnedByCurrentCitizenAsync(Guid documentId)
{
    var currentUserId = currentUserService.UserId;

    return await dbContext.Documents
        .AnyAsync(document =>
            document.Id == documentId &&
            document.Citizen.UserId == currentUserId);
}
```

---

## 11. Frontend Authorization

Frontend applications should also apply authorization checks, but only for user experience.

Frontend authorization is not a security boundary.

## 11.1 Citizen Mobile Application

The mobile application should:

- allow only authenticated Citizen users to access citizen screens,
- store access token securely,
- attach JWT token to API requests,
- redirect unauthenticated users to login,
- hide unavailable actions.

Protected screens:

```txt
DashboardScreen
ProfileScreen
DocumentsScreen
DocumentDetailsScreen
QrCodeScreen
ActivityHistoryScreen
```

---

## 11.2 Citizen Web Application

The citizen web application should:

- protect citizen routes using Angular guards,
- allow access only to users with Citizen role,
- attach JWT using HTTP interceptor,
- redirect unauthorized users to login or access denied page.

Example protected routes:

```txt
/profile
/documents
/documents/:id
/activity-history
```

---

## 11.3 Admin Web Application

The admin web panel should:

- protect admin routes using Angular guards,
- allow access only to Admin and SuperAdmin users,
- hide SuperAdmin-only navigation items for Admin users,
- attach JWT using HTTP interceptor,
- redirect unauthorized users to access denied page.

Example protected routes:

```txt
/dashboard
/users
/documents
/document-types
/activity-logs
/system-settings
/super-admin/admins
```

---

## 12. HTTP Response Rules

Authorization errors should use standard HTTP status codes.

| Status Code | Meaning | Example |
|---:|---|---|
| 401 | Unauthorized | Missing, invalid or expired token |
| 403 | Forbidden | User is authenticated but has no permission |
| 404 | Not Found | Resource does not exist |

Recommended behavior:

```txt
401 Unauthorized
```

Use when:

- token is missing,
- token is invalid,
- token is expired,
- user is not authenticated.

```txt
403 Forbidden
```

Use when:

- user is authenticated,
- user has wrong role,
- Citizen tries to access another citizen's data,
- Admin tries to access SuperAdmin-only endpoint.

```txt
404 Not Found
```

Use when:

- resource does not exist,
- resource was deleted or deactivated and should not be visible.

---

## 13. Activity Logging and Authorization

Important authorization-related events should be logged.

Recommended logged events:

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

Optional security events:

```txt
AccessDenied
InvalidLoginAttempt
TokenRefreshed
RefreshTokenRevoked
```

For MVP, it is enough to log main business actions. Security-specific events can be added later.

Activity logs should not store sensitive information such as passwords, raw tokens or full personal identifiers beyond what is necessary for the demo.

---

## 14. Refresh Token Authorization

Refresh tokens are used to obtain new access tokens.

The backend should verify that:

- refresh token exists,
- refresh token belongs to the user,
- refresh token is not expired,
- refresh token has not been revoked,
- user account is still active.

Recommended table:

```txt
RefreshTokens
```

Recommended fields:

```txt
Id
UserId
Token
ExpiresAt
RevokedAt
CreatedAt
CreatedByIp
RevokedByIp
```

Recommended MVP decision:

```txt
Use a custom RefreshTokens table instead of only AspNetUserTokens.
```

Reason:

- easier token revocation,
- easier refresh token rotation,
- clearer session history,
- better control over JWT-based authentication.

---

## 15. Authorization Implementation Order

Recommended implementation order:

```txt
1. Configure ASP.NET Core Identity
2. Seed roles: Citizen, Admin, SuperAdmin
3. Seed default SuperAdmin, Admin and Citizen accounts
4. Configure JWT Bearer authentication
5. Add role claims to JWT
6. Configure authorization policies
7. Protect controllers and endpoints
8. Implement CurrentUserService
9. Implement ownership checks for citizen documents
10. Add frontend route guards and HTTP interceptors
11. Add authorization tests
```

---

## 16. Authorization Test Scenarios

The MVP should include tests for the most important authorization rules.

Recommended test scenarios:

```txt
Citizen can access own profile.
Citizen cannot access admin dashboard.
Citizen can access own documents.
Citizen cannot access another citizen's document.
Citizen cannot create a document.
Admin can access admin dashboard.
Admin can create citizen account.
Admin can create document.
Admin can update document status.
Admin cannot create another admin account.
Admin cannot create document type if endpoint is SuperAdmin-only.
SuperAdmin can create admin account.
SuperAdmin can create document type.
Missing token returns 401 Unauthorized.
Invalid role returns 403 Forbidden.
Invalid ownership returns 403 Forbidden.
```

---

## 17. Open Questions

The following decisions can be finalized during implementation:

```txt
Should Citizen receive 403 or 404 when accessing another citizen's document?
Should refresh tokens be rotated on every refresh?
Should security events like failed login attempts be stored in ActivityLogs?
Should Admin see all activity logs or only operational logs?
Should SuperAdmin have all permissions through role checks only or additional policies?
Should role names be stored as constants in Mobywatel.Shared?
```

Recommended MVP answers:

```txt
Use 403 for invalid ownership.
Rotate refresh tokens on every refresh if time allows.
Log main business actions first.
Let Admin see standard activity logs.
Let SuperAdmin see full audit logs.
Store role names as constants.
```

---

## 18. Final Authorization Summary

The Mobywatel MVP uses ASP.NET Core Identity, JWT access tokens and role-based authorization.

The system has three roles:

```txt
Citizen
Admin
SuperAdmin
```

Citizen users can access only their own profile, documents, QR codes and activity history.

Admin users can manage citizens, users, documents, document statuses and view operational logs.

SuperAdmin users can additionally manage administrators, document types and selected system-level data.

The most important rule is:

```txt
Authorization must always be enforced by the backend.
```

Frontend guards and UI visibility are useful for user experience, but they do not replace backend authorization checks.
