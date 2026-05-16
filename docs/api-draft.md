# API Draft

This document describes the initial REST API draft for the Mobywatel MVP.

The backend is one deployable monolith. It exposes two logical REST API surfaces:

```txt
User API   - used by Citizen Mobile Application and Citizen Web Application
Admin API  - used by Admin Web Panel
```

Both API surfaces use the same internal Clean Architecture, Application layer, Domain layer, Infrastructure layer and database. Application use cases are implemented with CQRS + MediatR.

```txt
CitizenMobile  ─┐
CitizenWeb     ├──>  User API  ─┐
                               ├──> Backend Monolith ───> PostgreSQL Database
AdminWeb       ─────>  Admin API ┘
```

## 1. API Overview

User API base URL for local development:

```txt
https://localhost:5001/api/user
```

Admin API base URL for local development:

```txt
https://localhost:5001/api/admin
```

Alternative Docker/internal URLs:

```txt
http://backend-api:8080/api/user
http://backend-api:8080/api/admin
```

The backend exposes REST APIs from one monolith.

Main endpoint groups:

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

Endpoint paths should use the `/api/user/*` prefix for citizen-facing operations and the `/api/admin/*` prefix for administration operations.

All API responses should use a consistent response format.

Success response example:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Example"
  },
  "message": null,
  "errors": []
}
```

Error response example:

```json
{
  "success": false,
  "data": null,
  "message": "Validation failed.",
  "errors": [
    {
      "field": "email",
      "message": "Email is required."
    }
  ]
}
```

## 2. Authentication

The API uses JWT authentication.

Authenticated requests must contain:

```txt
Authorization: Bearer <access_token>
```

The system supports three roles:

```txt
Citizen
Admin
SuperAdmin
```

Citizen can access only own profile, documents, QR codes and activity history.

Admin can manage citizens, users, documents and view activity logs.

SuperAdmin can do everything Admin can do and can also manage admins, document types and system-level data.

## 3. HTTP Status Codes

The API should use standard HTTP status codes.

| Status Code | Meaning |
|---:|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

## 4. Auth API

Base path:

```txt
/api/user/auth
/api/admin/auth
```

The examples below show the User API path. Admin authentication uses the same endpoint names under `/api/admin/auth`.

### 4.1 Login

Endpoint:

```txt
POST /api/user/auth/login
```

Access:

```txt
Public
```

Description:

Authenticates a user using email and password. CitizenMobile and CitizenWeb use `/api/user/auth/login`. AdminWeb uses `/api/admin/auth/login`.

Request:

```json
{
  "email": "citizen@mobywatel.local",
  "password": "Password123!"
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt_access_token",
    "refreshToken": "refresh_token",
    "expiresAt": "2026-05-05T12:00:00Z",
    "user": {
      "id": "uuid",
      "email": "citizen@mobywatel.local",
      "firstName": "Jan",
      "lastName": "Kowalski",
      "role": "Citizen"
    }
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
```

### 4.2 Refresh Token

Endpoint:

```txt
POST /api/user/auth/refresh-token
```

Access:

```txt
Public
```

Description:

Generates a new access token using a valid refresh token.

Request:

```json
{
  "refreshToken": "refresh_token"
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "accessToken": "new_jwt_access_token",
    "refreshToken": "new_refresh_token",
    "expiresAt": "2026-05-05T12:30:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
```

### 4.3 Logout

Endpoint:

```txt
POST /api/user/auth/logout
```

Access:

```txt
Authenticated
```

Description:

Logs out the current user and revokes the refresh token.

Request:

```json
{
  "refreshToken": "refresh_token"
}
```

Response `204 No Content`:

```txt
No response body
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
```

### 4.4 Get Current User

Endpoint:

```txt
GET /api/user/auth/me
```

Access:

```txt
Authenticated
```

Description:

Returns information about the currently authenticated user.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "citizen@mobywatel.local",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "role": "Citizen",
    "isActive": true
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
```

## 5. User Profile API

Base path:

```txt
/api/user
/api/admin/users
```

### 5.1 Get Current Citizen Profile

Endpoint:

```txt
GET /api/user/profile
```

Access:

```txt
Citizen
```

Description:

Returns the profile of the currently logged-in citizen.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "email": "citizen@mobywatel.local",
    "pesel": "99010112345",
    "dateOfBirth": "1999-01-01",
    "address": {
      "street": "Main Street 1",
      "city": "Warsaw",
      "postalCode": "00-001",
      "country": "Poland"
    }
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 5.2 Get Current Citizen Documents

Endpoint:

```txt
GET /api/user/documents
```

Access:

```txt
Citizen
```

Description:

Returns documents assigned to the currently logged-in citizen.

Response `200 OK`:

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "documentType": {
        "id": "uuid",
        "name": "Identity Card",
        "code": "ID_CARD"
      },
      "documentNumber": "ABC123456",
      "issueDate": "2024-01-01",
      "expirationDate": "2034-01-01",
      "status": "Active"
    }
  ],
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 5.3 Get Current Citizen Activity Logs

Endpoint:

```txt
GET /api/user/activity-logs
```

Access:

```txt
Citizen
```

Description:

Returns activity history for the currently logged-in citizen.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| page | integer | No | Page number |
| pageSize | integer | No | Number of items per page |
| action | string | No | Filter by action type |

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "action": "UserLoggedIn",
        "description": "User logged in.",
        "createdAt": "2026-05-05T10:00:00Z"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "totalItems": 1
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 5.4 Get Citizen By Id

Endpoint:

```txt
GET /api/admin/users/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns citizen profile details by citizen ID. This endpoint is used by AdminWeb.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "email": "citizen@mobywatel.local",
    "pesel": "99010112345",
    "dateOfBirth": "1999-01-01",
    "address": {
      "street": "Main Street 1",
      "city": "Warsaw",
      "postalCode": "00-001",
      "country": "Poland"
    },
    "createdAt": "2026-05-05T10:00:00Z",
    "updatedAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 5.5 Update Citizen

Endpoint:

```txt
PUT /api/admin/users/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Updates selected citizen data.

Request:

```json
{
  "firstName": "Jan",
  "lastName": "Kowalski",
  "pesel": "99010112345",
  "dateOfBirth": "1999-01-01",
  "address": {
    "street": "Main Street 1",
    "city": "Warsaw",
    "postalCode": "00-001",
    "country": "Poland"
  }
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "pesel": "99010112345",
    "dateOfBirth": "1999-01-01",
    "address": {
      "street": "Main Street 1",
      "city": "Warsaw",
      "postalCode": "00-001",
      "country": "Poland"
    }
  },
  "message": "Citizen updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

## 6. Admin API

Base path:

```txt
/api/admin
```

### 6.1 Get Admin Dashboard

Endpoint:

```txt
GET /api/admin/dashboard
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns basic dashboard statistics for the admin panel.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "statistics": {
      "totalUsers": 50,
      "totalCitizens": 45,
      "totalDocuments": 120,
      "activeDocuments": 100,
      "expiredDocuments": 15,
      "blockedDocuments": 5
    },
    "recentActivity": [
      {
        "id": "uuid",
        "userFullName": "Jan Kowalski",
        "action": "DocumentViewed",
        "description": "Document was viewed.",
        "createdAt": "2026-05-05T10:00:00Z"
      }
    ]
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 6.2 Get Users

Endpoint:

```txt
GET /api/admin/users
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns a paginated list of users.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| page | integer | No | Page number |
| pageSize | integer | No | Number of items per page |
| search | string | No | Search by email, first name or last name |
| role | string | No | Filter by role |
| isActive | boolean | No | Filter by active status |

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "email": "citizen@mobywatel.local",
        "firstName": "Jan",
        "lastName": "Kowalski",
        "role": "Citizen",
        "isActive": true,
        "createdAt": "2026-05-05T10:00:00Z"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "totalItems": 1
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 6.3 Get User Details

Endpoint:

```txt
GET /api/admin/users/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns user details by user ID.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "citizen@mobywatel.local",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "role": "Citizen",
    "isActive": true,
    "citizenId": "uuid",
    "adminId": null,
    "createdAt": "2026-05-05T10:00:00Z",
    "updatedAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 6.4 Create Citizen User

Endpoint:

```txt
POST /api/admin/users
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Creates a new citizen user account.

Request:

```json
{
  "email": "new.citizen@mobywatel.local",
  "password": "Password123!",
  "firstName": "Anna",
  "lastName": "Nowak",
  "pesel": "99010112345",
  "dateOfBirth": "1999-01-01",
  "address": {
    "street": "Main Street 1",
    "city": "Warsaw",
    "postalCode": "00-001",
    "country": "Poland"
  }
}
```

Response `201 Created`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "new.citizen@mobywatel.local",
    "firstName": "Anna",
    "lastName": "Nowak",
    "role": "Citizen",
    "isActive": true,
    "citizenId": "uuid"
  },
  "message": "Citizen user created successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
409 Conflict
```

### 6.5 Update User

Endpoint:

```txt
PUT /api/admin/users/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Updates common user account data.

Request:

```json
{
  "email": "citizen.updated@mobywatel.local",
  "firstName": "Jan",
  "lastName": "Kowalski"
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "citizen.updated@mobywatel.local",
    "firstName": "Jan",
    "lastName": "Kowalski",
    "role": "Citizen",
    "isActive": true
  },
  "message": "User updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

### 6.6 Update User Status

Endpoint:

```txt
PATCH /api/admin/users/{id}/status
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Activates or deactivates a user account.

Request:

```json
{
  "isActive": false
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isActive": false
  },
  "message": "User status updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

## 7. Admin Management API

Base path:

```txt
/api/admin/admins
```

### 7.1 Get Admins

Endpoint:

```txt
GET /api/admin/admins
```

Access:

```txt
SuperAdmin
```

Description:

Returns a list of administrator accounts.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| page | integer | No | Page number |
| pageSize | integer | No | Number of items per page |
| search | string | No | Search by email, first name or last name |
| isActive | boolean | No | Filter by active status |

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "userId": "uuid",
        "email": "admin@mobywatel.local",
        "firstName": "Admin",
        "lastName": "User",
        "role": "Admin",
        "position": "System Administrator",
        "isActive": true
      }
    ],
    "page": 1,
    "pageSize": 20,
    "totalItems": 1
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 7.2 Create Admin

Endpoint:

```txt
POST /api/admin/admins
```

Access:

```txt
SuperAdmin
```

Description:

Creates a new Admin account.

Request:

```json
{
  "email": "new.admin@mobywatel.local",
  "password": "Password123!",
  "firstName": "Admin",
  "lastName": "Nowak",
  "position": "System Administrator"
}
```

Response `201 Created`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "email": "new.admin@mobywatel.local",
    "firstName": "Admin",
    "lastName": "Nowak",
    "role": "Admin",
    "position": "System Administrator",
    "isActive": true
  },
  "message": "Admin created successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
409 Conflict
```

### 7.3 Update Admin

Endpoint:

```txt
PUT /api/admin/admins/{id}
```

Access:

```txt
SuperAdmin
```

Description:

Updates selected admin account.

Request:

```json
{
  "firstName": "Admin",
  "lastName": "Updated",
  "position": "Senior System Administrator"
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "firstName": "Admin",
    "lastName": "Updated",
    "position": "Senior System Administrator"
  },
  "message": "Admin updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

### 7.4 Update Admin Status

Endpoint:

```txt
PATCH /api/admin/admins/{id}/status
```

Access:

```txt
SuperAdmin
```

Description:

Activates or deactivates selected admin account.

Request:

```json
{
  "isActive": false
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isActive": false
  },
  "message": "Admin status updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

## 8. Documents API

Base path:

```txt
/api/admin/documents
```

### 8.1 Get Documents

Endpoint:

```txt
GET /api/admin/documents
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns a paginated list of all documents. This endpoint is used by AdminWeb.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| page | integer | No | Page number |
| pageSize | integer | No | Number of items per page |
| search | string | No | Search by document number |
| status | string | No | Filter by document status |
| documentTypeId | uuid | No | Filter by document type |
| citizenId | uuid | No | Filter by citizen |

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "citizenId": "uuid",
        "citizenFullName": "Jan Kowalski",
        "documentType": {
          "id": "uuid",
          "name": "Identity Card",
          "code": "ID_CARD"
        },
        "documentNumber": "ABC123456",
        "issueDate": "2024-01-01",
        "expirationDate": "2034-01-01",
        "status": "Active"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "totalItems": 1
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 8.2 Get Document Details

Endpoint:

```txt
GET /api/admin/documents/{id}
```

Access:

```txt
Citizen
Admin
SuperAdmin
```

Description:

Returns document details.

Access rules:

```txt
Citizen can access only own document.
Admin can access all documents.
SuperAdmin can access all documents.
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "citizen": {
      "id": "uuid",
      "fullName": "Jan Kowalski",
      "pesel": "99010112345"
    },
    "documentType": {
      "id": "uuid",
      "name": "Identity Card",
      "code": "ID_CARD"
    },
    "documentNumber": "ABC123456",
    "issueDate": "2024-01-01",
    "expirationDate": "2034-01-01",
    "status": "Active",
    "createdAt": "2026-05-05T10:00:00Z",
    "updatedAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 8.3 Create Document

Endpoint:

```txt
POST /api/admin/documents
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Creates a new document and assigns it to a citizen.

Request:

```json
{
  "citizenId": "uuid",
  "documentTypeId": "uuid",
  "documentNumber": "ABC123456",
  "issueDate": "2024-01-01",
  "expirationDate": "2034-01-01",
  "status": "Active"
}
```

Response `201 Created`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "citizenId": "uuid",
    "documentTypeId": "uuid",
    "documentNumber": "ABC123456",
    "issueDate": "2024-01-01",
    "expirationDate": "2034-01-01",
    "status": "Active"
  },
  "message": "Document created successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

### 8.4 Update Document

Endpoint:

```txt
PUT /api/admin/documents/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Updates document data. This endpoint can also be used to assign or reassign a document to a citizen.

Request:

```json
{
  "citizenId": "uuid",
  "documentTypeId": "uuid",
  "documentNumber": "ABC123456",
  "issueDate": "2024-01-01",
  "expirationDate": "2034-01-01"
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "citizenId": "uuid",
    "documentTypeId": "uuid",
    "documentNumber": "ABC123456",
    "issueDate": "2024-01-01",
    "expirationDate": "2034-01-01",
    "status": "Active"
  },
  "message": "Document updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

### 8.5 Update Document Status

Endpoint:

```txt
PATCH /api/admin/documents/{id}/status
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Updates document status.

Allowed statuses:

```txt
Active
Expired
Blocked
Pending
Rejected
```

Request:

```json
{
  "status": "Blocked",
  "reason": "Document blocked by administrator."
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "Blocked"
  },
  "message": "Document status updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

### 8.6 Delete Document

Endpoint:

```txt
DELETE /api/admin/documents/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Deletes or deactivates a document. The MVP can use soft delete instead of physical delete.

Response `204 No Content`:

```txt
No response body
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 8.7 Get Document QR Code

Endpoint:

```txt
GET /api/user/documents/{id}/qr-code
```

Access:

```txt
Citizen
Admin
SuperAdmin
```

Description:

Returns QR code value for selected document.

Access rules:

```txt
Citizen can access QR code only for own document.
Admin can access all document QR codes.
SuperAdmin can access all document QR codes.
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "documentId": "uuid",
    "qrCodeValue": "MOBYWATEL:DOCUMENT:uuid:ABC123456",
    "generatedAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

## 9. Document Types API

Base path:

```txt
/api/admin/document-types
```

### 9.1 Get Document Types

Endpoint:

```txt
GET /api/admin/document-types
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns available document types.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| includeInactive | boolean | No | Whether inactive types should be returned |

Response `200 OK`:

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Identity Card",
      "code": "ID_CARD",
      "description": "Basic identity document.",
      "isActive": true
    }
  ],
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 9.2 Get Document Type Details

Endpoint:

```txt
GET /api/admin/document-types/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns details of selected document type.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Identity Card",
    "code": "ID_CARD",
    "description": "Basic identity document.",
    "isActive": true,
    "createdAt": "2026-05-05T10:00:00Z",
    "updatedAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

### 9.3 Create Document Type

Endpoint:

```txt
POST /api/admin/document-types
```

Access:

```txt
SuperAdmin
```

Description:

Creates a new document type.

Request:

```json
{
  "name": "Student Card",
  "code": "STUDENT_CARD",
  "description": "Student identification document."
}
```

Response `201 Created`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Student Card",
    "code": "STUDENT_CARD",
    "description": "Student identification document.",
    "isActive": true
  },
  "message": "Document type created successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
409 Conflict
```

### 9.4 Update Document Type

Endpoint:

```txt
PUT /api/admin/document-types/{id}
```

Access:

```txt
SuperAdmin
```

Description:

Updates existing document type.

Request:

```json
{
  "name": "Student Card",
  "description": "Updated student identification document."
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Student Card",
    "code": "STUDENT_CARD",
    "description": "Updated student identification document.",
    "isActive": true
  },
  "message": "Document type updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

### 9.5 Update Document Type Status

Endpoint:

```txt
PATCH /api/admin/document-types/{id}/status
```

Access:

```txt
SuperAdmin
```

Description:

Activates or deactivates selected document type. Inactive document types cannot be used for new documents. Existing documents keep their assigned document type.

Request:

```json
{
  "isActive": false
}
```

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isActive": false
  },
  "message": "Document type status updated successfully.",
  "errors": []
}
```

Possible error responses:

```txt
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

## 10. Activity Logs API

Base path:

```txt
/api/admin/activity-logs
```

### 10.1 Get Activity Logs

Endpoint:

```txt
GET /api/admin/activity-logs
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns system activity logs. Admin can view standard activity logs. SuperAdmin can view full audit logs.

Query parameters:

| Name | Type | Required | Description |
|---|---|---:|---|
| page | integer | No | Page number |
| pageSize | integer | No | Number of items per page |
| userId | uuid | No | Filter by user |
| action | string | No | Filter by action |
| from | date | No | Start date |
| to | date | No | End date |

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "userId": "uuid",
        "userFullName": "Jan Kowalski",
        "action": "DocumentStatusUpdated",
        "description": "Document status changed to Blocked.",
        "ipAddress": "127.0.0.1",
        "createdAt": "2026-05-05T10:00:00Z"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "totalItems": 1
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
```

### 10.2 Get Activity Log Details

Endpoint:

```txt
GET /api/admin/activity-logs/{id}
```

Access:

```txt
Admin
SuperAdmin
```

Description:

Returns details of selected activity log entry.

Response `200 OK`:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "userFullName": "Jan Kowalski",
    "userRole": "Admin",
    "action": "DocumentStatusUpdated",
    "description": "Document status changed to Blocked.",
    "ipAddress": "127.0.0.1",
    "metadata": {
      "documentId": "uuid",
      "previousStatus": "Active",
      "newStatus": "Blocked"
    },
    "createdAt": "2026-05-05T10:00:00Z"
  },
  "message": null,
  "errors": []
}
```

Possible error responses:

```txt
401 Unauthorized
403 Forbidden
404 Not Found
```

## 11. DTO Draft

Auth DTOs:

```txt
LoginRequestDto
LoginResponseDto
RefreshTokenRequestDto
RefreshTokenResponseDto
CurrentUserDto
```

User DTOs:

```txt
UserDto
UserDetailsDto
CreateCitizenUserRequestDto
UpdateUserRequestDto
UpdateUserStatusRequestDto
```

Citizen DTOs:

```txt
CitizenDto
CitizenDetailsDto
UpdateCitizenRequestDto
CitizenAddressDto
```

Admin DTOs:

```txt
AdminDto
AdminDetailsDto
CreateAdminRequestDto
UpdateAdminRequestDto
UpdateAdminStatusRequestDto
```

Document DTOs:

```txt
DocumentDto
DocumentDetailsDto
CreateDocumentRequestDto
UpdateDocumentRequestDto
UpdateDocumentStatusRequestDto
DocumentQrCodeDto
```

Document Type DTOs:

```txt
DocumentTypeDto
DocumentTypeDetailsDto
CreateDocumentTypeRequestDto
UpdateDocumentTypeRequestDto
UpdateDocumentTypeStatusRequestDto
```

Activity Log DTOs:

```txt
ActivityLogDto
ActivityLogDetailsDto
```

Dashboard DTOs:

```txt
AdminDashboardDto
DashboardStatisticsDto
RecentActivityDto
```

Common DTOs:

```txt
ApiResponse<T>
PagedResponse<T>
ValidationErrorDto
```

## 12. Enum Draft

UserRole:

```txt
Citizen
Admin
SuperAdmin
```

DocumentStatus:

```txt
Active
Expired
Blocked
Pending
Rejected
```

ActivityLogAction:

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

## 13. Validation Rules

Login validation:

```txt
Email is required.
Email must have valid format.
Password is required.
```

Create user validation:

```txt
Email is required.
Email must be unique.
Password is required.
Password must meet security rules.
First name is required.
Last name is required.
Role is assigned by endpoint logic, not by client.
```

Create citizen validation:

```txt
PESEL is required.
PESEL must be unique.
Date of birth is required.
Address fields should be valid.
```

Create document validation:

```txt
CitizenId is required.
Citizen must exist.
DocumentTypeId is required.
Document type must exist.
Document type must be active.
DocumentNumber is required.
DocumentNumber must be unique.
IssueDate is required.
ExpirationDate is required.
ExpirationDate must be later than IssueDate.
Status must be valid.
```

Update document status validation:

```txt
Status is required.
Status must be one of allowed values.
Reason is optional.
```

Create document type validation:

```txt
Name is required.
Code is required.
Code must be unique.
Description is optional.
```

## 14. Authorization Matrix

| Endpoint | Citizen | Admin | SuperAdmin |
|---|---:|---:|---:|
| `POST /api/user/auth/login` | Yes | Yes | Yes |
| `POST /api/user/auth/refresh-token` | Yes | Yes | Yes |
| `POST /api/user/auth/logout` | Yes | Yes | Yes |
| `GET /api/user/auth/me` | Yes | Yes | Yes |
| `GET /api/user/profile` | Yes | No | No |
| `GET /api/user/documents` | Yes | No | No |
| `GET /api/user/activity-logs` | Yes | No | No |
| `GET /api/admin/users/{id}` | No | Yes | Yes |
| `PUT /api/admin/users/{id}` | No | Yes | Yes |
| `GET /api/admin/dashboard` | No | Yes | Yes |
| `GET /api/admin/users` | No | Yes | Yes |
| `GET /api/admin/users/{id}` | No | Yes | Yes |
| `POST /api/admin/users` | No | Yes | Yes |
| `PUT /api/admin/users/{id}` | No | Yes | Yes |
| `PATCH /api/admin/users/{id}/status` | No | Yes | Yes |
| `GET /api/admin/admins` | No | No | Yes |
| `POST /api/admin/admins` | No | No | Yes |
| `PUT /api/admin/admins/{id}` | No | No | Yes |
| `PATCH /api/admin/admins/{id}/status` | No | No | Yes |
| `GET /api/admin/documents` | No | Yes | Yes |
| `GET /api/admin/documents/{id}` | Own only | Yes | Yes |
| `POST /api/admin/documents` | No | Yes | Yes |
| `PUT /api/admin/documents/{id}` | No | Yes | Yes |
| `PATCH /api/admin/documents/{id}/status` | No | Yes | Yes |
| `DELETE /api/admin/documents/{id}` | No | Yes | Yes |
| `GET /api/user/documents/{id}/qr-code` | Own only | Yes | Yes |
| `GET /api/admin/document-types` | No | Yes | Yes |
| `GET /api/admin/document-types/{id}` | No | Yes | Yes |
| `POST /api/admin/document-types` | No | No | Yes |
| `PUT /api/admin/document-types/{id}` | No | No | Yes |
| `PATCH /api/admin/document-types/{id}/status` | No | No | Yes |
| `GET /api/admin/activity-logs` | No | Yes | Yes |
| `GET /api/admin/activity-logs/{id}` | No | Yes | Yes |

## 15. MVP Endpoint Summary

```txt
POST   /api/user/auth/login
POST   /api/user/auth/refresh-token
POST   /api/user/auth/logout
GET    /api/user/auth/me

GET    /api/user/profile
GET    /api/user/documents
GET    /api/user/activity-logs
GET    /api/admin/users/{id}
PUT    /api/admin/users/{id}

GET    /api/admin/dashboard
GET    /api/admin/users
GET    /api/admin/users/{id}
POST   /api/admin/users
PUT    /api/admin/users/{id}
PATCH  /api/admin/users/{id}/status

GET    /api/admin/admins
POST   /api/admin/admins
PUT    /api/admin/admins/{id}
PATCH  /api/admin/admins/{id}/status

GET    /api/admin/documents
GET    /api/admin/documents/{id}
POST   /api/admin/documents
PUT    /api/admin/documents/{id}
PATCH  /api/admin/documents/{id}/status
DELETE /api/admin/documents/{id}
GET    /api/user/documents/{id}/qr-code

GET    /api/admin/document-types
GET    /api/admin/document-types/{id}
POST   /api/admin/document-types
PUT    /api/admin/document-types/{id}
PATCH  /api/admin/document-types/{id}/status

GET    /api/admin/activity-logs
GET    /api/admin/activity-logs/{id}
```

## 16. Open Questions

The following decisions can be finalized during implementation:

```txt
Should document delete be hard delete or soft delete?
Should citizens receive 403 or 404 when trying to access another citizen's document?
Should refresh tokens be rotated on every refresh?
Should activity logs store metadata as JSON?
Should Admin see all activity logs or only selected operational logs?
Should QR code value be stored in database or generated dynamically?
Should document numbers be globally unique or unique per document type?
```

## 17. Final API Summary

The Mobywatel MVP backend is one monolith with two logical API surfaces: User API for Citizen Mobile Application and Citizen Web Application, and Admin API for Admin Web Panel.

The API supports authentication, authorization, citizen profile, citizen documents, QR code preview, activity history, admin dashboard, user management, admin management, document management, document type management and activity logs.

The most important API rule is:

```txt
Security and business logic must always be enforced by the backend.
```

Frontend and mobile applications may hide unavailable actions in the UI, but the backend must always verify role permissions and ownership rules in controllers, policies and MediatR application handlers.
