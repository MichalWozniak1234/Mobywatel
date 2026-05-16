# Use Cases

This document describes the main use cases for the Mobywatel application.

The system supports three main user roles:

- Citizen,
- Admin,
- SuperAdmin.

The application consists of:

- Citizen Mobile Application,
- Citizen Web Application,
- Admin Web Panel,
- Backend Monolith,
- User API,
- Admin API,
- Relational Database.

The main purpose of the system is to allow citizens to view their digital documents and allow administrators to manage users, documents, document types, statuses and activity logs.

Citizen use cases are exposed through the User API. Administration use cases are exposed through the Admin API. Both API surfaces use the same backend monolith, internal Clean Architecture and Application layer implemented with CQRS + MediatR.

---

## 1. Actors

## 1.1 Citizen

A Citizen is a regular user of the system.

The Citizen can use:

- mobile application,
- citizen web application.

The Citizen can:

- log in,
- view own profile,
- view own documents,
- open document details,
- check document status,
- display document QR code,
- view own activity history.

The Citizen cannot:

- create documents,
- edit documents,
- assign documents,
- manage users,
- access admin panel.

---

## 1.2 Admin

An Admin is a user of the web administration panel.

The Admin can:

- log in to the admin panel,
- view admin dashboard,
- manage citizens,
- manage citizen documents,
- assign documents to citizens,
- update document statuses,
- view activity logs.

The Admin cannot:

- access another admin's management features,
- manage SuperAdmin accounts,
- change system-level configuration reserved for SuperAdmin.

---

## 1.3 SuperAdmin

A SuperAdmin has the highest access level in the system.

The SuperAdmin can:

- do everything an Admin can do,
- manage administrators,
- manage document types,
- manage selected system dictionaries,
- view full audit logs,
- configure selected system data.

The SuperAdmin role is used for advanced system management.

---

## 2. Use Case Overview

The main use cases are divided into the following groups:

```txt
Authentication
Citizen Profile
Citizen Documents
QR Code
Activity History
Admin Dashboard
User Management
Document Management
Document Type Management
Activity Logs
System Administration
```

---

# 3. Authentication Use Cases

---

## UC-01: Citizen Login

## Actor

Citizen

## Goal

The Citizen wants to log in to the mobile application or citizen web application.

## Preconditions

- Citizen account exists in the system.
- Citizen account is active.
- Citizen knows valid email and password.
- User API is available.

## Main Flow

1. Citizen opens the mobile application or citizen web application.
2. Citizen enters email and password.
3. Application sends login request to the User API.
4. Backend validates the credentials.
5. Backend checks if the user account is active.
6. Backend generates JWT access token.
7. Backend returns authentication data to the application.
8. Application stores the token.
9. Citizen is redirected to the dashboard.

## Alternative Flow

### Invalid credentials

1. Citizen enters invalid email or password.
2. Backend rejects the login request.
3. Application displays an error message.

### Inactive account

1. Citizen enters valid credentials.
2. Backend detects that the account is inactive.
3. Backend rejects the login request.
4. Application displays an account status error.

## Postconditions

- Citizen is authenticated.
- Citizen can access protected citizen features.
- Login activity is saved in activity logs.

## Related API Endpoint

```txt
POST /api/user/auth/login
```

---

## UC-02: Admin Login

## Actor

Admin

## Goal

The Admin wants to log in to the admin web panel.

## Preconditions

- Admin account exists in the system.
- Admin account is active.
- Admin knows valid email and password.
- Admin API is available.

## Main Flow

1. Admin opens the admin web panel.
2. Admin enters email and password.
3. Admin web panel sends login request to the Admin API.
4. Backend validates the credentials.
5. Backend checks if the user has Admin or SuperAdmin role.
6. Backend generates JWT access token.
7. Backend returns authentication data to the admin web panel.
8. Admin web panel stores the token.
9. Admin is redirected to the admin dashboard.

## Alternative Flow

### User is not an admin

1. User enters valid credentials.
2. Backend detects that the user does not have Admin or SuperAdmin role.
3. Backend rejects access to admin panel.
4. Admin web panel displays an access denied message.

## Postconditions

- Admin is authenticated.
- Admin can access admin panel features.
- Login activity is saved in activity logs.

## Related API Endpoint

```txt
POST /api/admin/auth/login
```

---

## UC-03: Logout

## Actor

Citizen, Admin, SuperAdmin

## Goal

The user wants to log out from the application.

## Preconditions

- User is logged in.
- User has a valid authentication token.

## Main Flow

1. User clicks the logout button.
2. Application sends logout request to the relevant User API or Admin API.
3. Backend invalidates or revokes the refresh token.
4. Application removes local authentication data.
5. User is redirected to the login page.

## Postconditions

- User is logged out.
- User cannot access protected pages without logging in again.

## Related API Endpoint

```txt
POST /api/user/auth/logout
POST /api/admin/auth/logout
```

---

## UC-04: Refresh Access Token

## Actor

Citizen, Admin, SuperAdmin

## Goal

The application wants to refresh the access token without forcing the user to log in again.

## Preconditions

- User is logged in.
- Refresh token exists.
- Refresh token is valid and not expired.

## Main Flow

1. Application detects that the access token is expired or close to expiration.
2. Application sends refresh token request to the relevant User API or Admin API.
3. Backend validates the refresh token.
4. Backend generates a new access token.
5. Backend returns a new access token to the application.
6. Application continues the user session.

## Alternative Flow

### Invalid refresh token

1. Application sends an invalid refresh token.
2. Backend rejects the request.
3. Application logs the user out.
4. User is redirected to the login page.

## Postconditions

- User session continues with a new access token.

## Related API Endpoint

```txt
POST /api/user/auth/refresh-token
POST /api/admin/auth/refresh-token
```

---

# 4. Citizen Profile Use Cases

---

## UC-05: View Own Profile

## Actor

Citizen

## Goal

The Citizen wants to view personal profile data.

## Preconditions

- Citizen is logged in.
- Citizen account is active.
- Citizen profile exists in the database.

## Main Flow

1. Citizen opens the profile page.
2. Application sends request for current citizen profile.
3. Backend identifies the current user from the token.
4. Backend loads citizen profile from the database.
5. Backend returns profile data.
6. Application displays profile information.

## Postconditions

- Citizen sees own profile data.

## Related API Endpoint

```txt
GET /api/user/profile
```

---

## UC-06: Admin Views Citizen Profile

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view citizen profile details.

## Preconditions

- Admin is logged in.
- Admin has permission to view citizen data.
- Citizen exists in the database.

## Main Flow

1. Admin opens the user list.
2. Admin selects a citizen.
3. Admin web panel sends request for citizen details.
4. Backend verifies Admin or SuperAdmin role.
5. Backend loads citizen profile.
6. Backend returns citizen data.
7. Admin web panel displays citizen details.

## Alternative Flow

### Citizen not found

1. Admin selects a citizen that does not exist.
2. Backend returns not found response.
3. Admin web panel displays an error message.

## Postconditions

- Admin sees selected citizen profile.

## Related API Endpoint

```txt
GET /api/admin/users/{id}
```

---

## UC-07: Admin Updates Citizen Profile

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to update citizen profile data.

## Preconditions

- Admin is logged in.
- Admin has permission to update citizen data.
- Citizen exists in the database.

## Main Flow

1. Admin opens citizen details.
2. Admin edits selected citizen data.
3. Admin submits the form.
4. Admin web panel sends update request to the Admin API.
5. Backend validates input data.
6. Backend updates citizen profile in the database.
7. Backend saves activity log.
8. Backend returns updated citizen data.
9. Admin web panel displays success message.

## Alternative Flow

### Invalid data

1. Admin submits invalid data.
2. Backend returns validation errors.
3. Admin web panel displays validation messages.

## Postconditions

- Citizen profile is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/users/{id}
```

---

# 5. Citizen Document Use Cases

---

## UC-08: View Own Document List

## Actor

Citizen

## Goal

The Citizen wants to view a list of assigned documents.

## Preconditions

- Citizen is logged in.
- Citizen account is active.
- Citizen has access to the mobile application or citizen web application.

## Main Flow

1. Citizen opens the documents page.
2. Application sends request for current citizen documents.
3. Backend identifies the citizen from the token.
4. Backend loads documents assigned to the citizen.
5. Backend returns document list.
6. Application displays documents.

## Alternative Flow

### No documents assigned

1. Backend returns an empty document list.
2. Application displays information that no documents are available.

## Postconditions

- Citizen sees own document list.
- Optional document list view activity may be saved.

## Related API Endpoint

```txt
GET /api/user/documents
```

---

## UC-09: View Document Details

## Actor

Citizen

## Goal

The Citizen wants to view details of a selected document.

## Preconditions

- Citizen is logged in.
- Document exists.
- Document belongs to the current citizen.

## Main Flow

1. Citizen opens the document list.
2. Citizen selects a document.
3. Application sends request for document details.
4. Backend validates that the document belongs to the current citizen.
5. Backend loads document details.
6. Backend saves document view activity log.
7. Backend returns document details.
8. Application displays document details.

## Alternative Flow

### Document does not belong to citizen

1. Citizen tries to open a document that belongs to another user.
2. Backend denies access.
3. Application displays access denied message.

### Document not found

1. Citizen tries to open a document that does not exist.
2. Backend returns not found response.
3. Application displays error message.

## Postconditions

- Citizen sees selected document details.
- Document view activity is saved.

## Related API Endpoint

```txt
GET /api/user/documents/{id}
```

---

## UC-10: Check Document Status

## Actor

Citizen

## Goal

The Citizen wants to check whether a document is active, expired, blocked, pending or rejected.

## Preconditions

- Citizen is logged in.
- Document exists.
- Document belongs to the current citizen.

## Main Flow

1. Citizen opens document details.
2. Application displays the current document status returned by the backend.
3. Citizen can see the document status clearly.

## Postconditions

- Citizen knows the current status of the document.

## Related API Endpoint

```txt
GET /api/user/documents/{id}
```

---

# 6. QR Code Use Cases

---

## UC-11: Display Document QR Code

## Actor

Citizen

## Goal

The Citizen wants to display a QR code connected with a selected document.

## Preconditions

- Citizen is logged in.
- Document exists.
- Document belongs to the current citizen.
- Document has generated QR code value or can receive one from the backend.

## Main Flow

1. Citizen opens document details.
2. Citizen selects QR code preview.
3. Application sends request for document QR code data.
4. Backend verifies that the document belongs to the current citizen.
5. Backend returns QR code value.
6. Application generates and displays QR code.

## Alternative Flow

### Unauthorized document access

1. Citizen tries to open QR code for a document that belongs to another citizen.
2. Backend denies access.
3. Application displays access denied message.

## Postconditions

- Citizen sees QR code for own document.

## Related API Endpoint

```txt
GET /api/user/documents/{id}/qr-code
```

---

# 7. Activity History Use Cases

---

## UC-12: Citizen Views Own Activity History

## Actor

Citizen

## Goal

The Citizen wants to view activity history related to own account.

## Preconditions

- Citizen is logged in.
- Activity logs exist or can return an empty list.

## Main Flow

1. Citizen opens activity history page.
2. Application sends request for current citizen activity logs.
3. Backend identifies current citizen from the token.
4. Backend loads activity logs related to the citizen.
5. Backend returns activity history.
6. Application displays activity history.

## Alternative Flow

### No activity logs

1. Backend returns an empty list.
2. Application displays information that no activity history is available.

## Postconditions

- Citizen sees own activity history.

## Related API Endpoint

```txt
GET /api/user/activity-logs
```

---

# 8. Admin Dashboard Use Cases

---

## UC-13: View Admin Dashboard

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view basic system statistics and recent activity.

## Preconditions

- Admin is logged in.
- Admin has access to admin panel.

## Main Flow

1. Admin opens the admin dashboard.
2. Admin web panel sends dashboard data request.
3. Backend verifies Admin or SuperAdmin role.
4. Backend calculates dashboard statistics.
5. Backend loads recent activity.
6. Backend returns dashboard data.
7. Admin web panel displays dashboard.

## Dashboard data may include

```txt
Total users count
Total citizens count
Total documents count
Active documents count
Expired documents count
Recent activity list
```

## Postconditions

- Admin sees system overview.

## Related API Endpoint

```txt
GET /api/admin/dashboard
```

---

# 9. User Management Use Cases

---

## UC-14: View User List

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view users registered in the system.

## Preconditions

- Admin is logged in.
- Admin has permission to view users.

## Main Flow

1. Admin opens the users page.
2. Admin web panel sends request for user list.
3. Backend verifies Admin or SuperAdmin role.
4. Backend loads users from the database.
5. Backend returns user list.
6. Admin web panel displays users.

## Alternative Flow

### No users found

1. Backend returns an empty list.
2. Admin web panel displays information that no users are available.

## Postconditions

- Admin sees user list.

## Related API Endpoint

```txt
GET /api/admin/users
```

---

## UC-15: View User Details

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view details of a selected user.

## Preconditions

- Admin is logged in.
- Selected user exists.

## Main Flow

1. Admin opens the users page.
2. Admin selects a user.
3. Admin web panel sends request for user details.
4. Backend verifies Admin or SuperAdmin role.
5. Backend loads selected user details.
6. Backend returns user details.
7. Admin web panel displays user details.

## Alternative Flow

### User not found

1. Admin selects a user that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Admin sees selected user details.

## Related API Endpoint

```txt
GET /api/admin/users/{id}
```

---

## UC-16: Create Citizen Account

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to create a new citizen account.

## Preconditions

- Admin is logged in.
- Admin has permission to create citizen accounts.
- Email does not already exist.

## Main Flow

1. Admin opens create user form.
2. Admin enters citizen account data.
3. Admin submits the form.
4. Admin web panel sends create user request to the Admin API.
5. Backend validates input data.
6. Backend checks if email is unique.
7. Backend hashes the password.
8. Backend creates user with Citizen role.
9. Backend creates citizen profile.
10. Backend saves activity log.
11. Backend returns created user data.
12. Admin web panel displays success message.

## Alternative Flow

### Email already exists

1. Admin enters an email already used in the system.
2. Backend rejects the request.
3. Admin web panel displays validation error.

### Invalid data

1. Admin submits invalid data.
2. Backend returns validation errors.
3. Admin web panel displays validation messages.

## Postconditions

- New citizen account is created.
- Activity log is saved.

## Related API Endpoint

```txt
POST /api/admin/users
```

---

## UC-17: Update User Data

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to update selected user data.

## Preconditions

- Admin is logged in.
- User exists.
- Admin has permission to update user data.

## Main Flow

1. Admin opens user details.
2. Admin edits user data.
3. Admin submits changes.
4. Admin web panel sends update request to Admin API.
5. Backend validates input data.
6. Backend updates user data.
7. Backend saves activity log.
8. Backend returns updated data.
9. Admin web panel displays success message.

## Alternative Flow

### User not found

1. Admin tries to update a user that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### Invalid data

1. Admin submits invalid data.
2. Backend returns validation errors.
3. Admin web panel displays validation messages.

## Postconditions

- User data is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/users/{id}
```

---

## UC-18: Activate or Deactivate User

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to activate or deactivate a user account.

## Preconditions

- Admin is logged in.
- User exists.
- Admin has permission to change user status.

## Main Flow

1. Admin opens user details or user list.
2. Admin selects activate or deactivate action.
3. Admin web panel sends status update request.
4. Backend verifies permission.
5. Backend updates user status.
6. Backend saves activity log.
7. Backend returns success response.
8. Admin web panel displays updated status.

## Alternative Flow

### User not found

1. Admin selects a user that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- User status is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PATCH /api/admin/users/{id}/status
```

---

# 10. Document Management Use Cases

---

## UC-19: Admin Views Document List

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view all documents in the system.

## Preconditions

- Admin is logged in.
- Admin has permission to view documents.

## Main Flow

1. Admin opens documents page.
2. Admin web panel sends request for document list.
3. Backend verifies Admin or SuperAdmin role.
4. Backend loads documents from the database.
5. Backend returns document list.
6. Admin web panel displays documents.

## Alternative Flow

### No documents found

1. Backend returns an empty list.
2. Admin web panel displays information that no documents are available.

## Postconditions

- Admin sees document list.

## Related API Endpoint

```txt
GET /api/admin/documents
```

---

## UC-20: Admin Views Document Details

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view document details.

## Preconditions

- Admin is logged in.
- Document exists.
- Admin has permission to view documents.

## Main Flow

1. Admin opens document list.
2. Admin selects a document.
3. Admin web panel sends request for document details.
4. Backend verifies Admin or SuperAdmin role.
5. Backend loads document details.
6. Backend returns document details.
7. Admin web panel displays document details.

## Alternative Flow

### Document not found

1. Admin selects a document that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Admin sees document details.

## Related API Endpoint

```txt
GET /api/admin/documents/{id}
```

---

## UC-21: Create Document

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to create a new document.

## Preconditions

- Admin is logged in.
- Admin has permission to create documents.
- Selected document type exists.
- Selected citizen exists.

## Main Flow

1. Admin opens create document form.
2. Admin enters document data.
3. Admin selects document type.
4. Admin selects citizen.
5. Admin submits the form.
6. Admin web panel sends create document request.
7. Backend validates input data.
8. Backend checks if citizen exists.
9. Backend checks if document type exists and is active.
10. Backend generates QR code value.
11. Backend creates document in the database.
12. Backend saves activity log.
13. Backend returns created document.
14. Admin web panel displays success message.

## Alternative Flow

### Invalid document type

1. Admin selects inactive or non-existing document type.
2. Backend rejects the request.
3. Admin web panel displays validation error.

### Citizen not found

1. Admin selects citizen that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Document is created.
- Document is assigned to a citizen.
- QR code value is generated.
- Activity log is saved.

## Related API Endpoint

```txt
POST /api/admin/documents
```

---

## UC-22: Assign Document to Citizen

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to assign a document to a selected citizen.

## Preconditions

- Admin is logged in.
- Document exists.
- Citizen exists.
- Admin has permission to assign documents.

## Main Flow

1. Admin opens document details.
2. Admin selects citizen.
3. Admin confirms assignment.
4. Admin web panel sends assignment request.
5. Backend verifies permission.
6. Backend checks if document exists.
7. Backend checks if citizen exists.
8. Backend assigns document to citizen.
9. Backend saves activity log.
10. Backend returns success response.
11. Admin web panel displays updated assignment.

## Alternative Flow

### Document not found

1. Admin selects document that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### Citizen not found

1. Admin selects citizen that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Document is assigned to selected citizen.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/documents/{id}
```

---

## UC-23: Update Document Data

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to update document data.

## Preconditions

- Admin is logged in.
- Document exists.
- Admin has permission to update documents.

## Main Flow

1. Admin opens document details.
2. Admin edits document data.
3. Admin submits changes.
4. Admin web panel sends update request.
5. Backend validates input data.
6. Backend updates document in the database.
7. Backend saves activity log.
8. Backend returns updated document data.
9. Admin web panel displays success message.

## Alternative Flow

### Document not found

1. Admin tries to update a document that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### Invalid data

1. Admin submits invalid document data.
2. Backend returns validation errors.
3. Admin web panel displays validation messages.

## Postconditions

- Document data is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/documents/{id}
```

---

## UC-24: Update Document Status

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to update the status of a document.

## Preconditions

- Admin is logged in.
- Document exists.
- Admin has permission to update document status.
- New status value is valid.

## Main Flow

1. Admin opens document details or document list.
2. Admin selects new document status.
3. Admin confirms status update.
4. Admin web panel sends status update request.
5. Backend validates status value.
6. Backend updates document status in the database.
7. Backend saves activity log.
8. Backend returns updated document.
9. Admin web panel displays new document status.

## Possible statuses

```txt
Active
Expired
Blocked
Pending
Rejected
```

## Alternative Flow

### Invalid status

1. Admin selects invalid status.
2. Backend rejects the request.
3. Admin web panel displays validation error.

### Document not found

1. Admin tries to update a document that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Document status is updated.
- Citizen can see the updated status.
- Activity log is saved.

## Related API Endpoint

```txt
PATCH /api/admin/documents/{id}/status
```

---

## UC-25: Delete Document

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to delete a document from the system.

## Preconditions

- Admin is logged in.
- Document exists.
- Admin has permission to delete documents.

## Main Flow

1. Admin opens document list or document details.
2. Admin selects delete document action.
3. Admin confirms deletion.
4. Admin web panel sends delete request.
5. Backend verifies permission.
6. Backend deletes document or marks it as deleted.
7. Backend saves activity log.
8. Backend returns success response.
9. Admin web panel removes document from the list.

## Alternative Flow

### Document not found

1. Admin tries to delete a document that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Document is deleted or marked as deleted.
- Activity log is saved.

## Related API Endpoint

```txt
DELETE /api/admin/documents/{id}
```

---

# 11. Document Type Use Cases

---

## UC-26: View Document Type List

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view available document types.

## Preconditions

- Admin is logged in.
- Admin has permission to view document types.

## Main Flow

1. Admin opens document types page.
2. Admin web panel sends request for document types.
3. Backend verifies Admin or SuperAdmin role.
4. Backend loads document types.
5. Backend returns document type list.
6. Admin web panel displays document types.

## Alternative Flow

### No document types found

1. Backend returns an empty list.
2. Admin web panel displays information that no document types are available.

## Postconditions

- Admin sees document type list.

## Related API Endpoint

```txt
GET /api/admin/document-types
```

---

## UC-27: Create Document Type

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to create a new document type.

## Preconditions

- SuperAdmin is logged in.
- Document type code does not already exist.

## Main Flow

1. SuperAdmin opens document types page.
2. SuperAdmin selects create document type.
3. SuperAdmin enters document type data.
4. SuperAdmin submits the form.
5. Admin web panel sends create request.
6. Backend verifies SuperAdmin role.
7. Backend validates input data.
8. Backend checks if document type code is unique.
9. Backend creates document type.
10. Backend saves activity log.
11. Backend returns created document type.
12. Admin web panel displays success message.

## Alternative Flow

### Code already exists

1. SuperAdmin enters a code that already exists.
2. Backend rejects the request.
3. Admin web panel displays validation error.

### User is not SuperAdmin

1. Admin tries to create document type.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- New document type is created.
- Activity log is saved.

## Related API Endpoint

```txt
POST /api/admin/document-types
```

---

## UC-28: Update Document Type

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to update existing document type data.

## Preconditions

- SuperAdmin is logged in.
- Document type exists.

## Main Flow

1. SuperAdmin opens document type details.
2. SuperAdmin edits document type data.
3. SuperAdmin submits changes.
4. Admin web panel sends update request.
5. Backend verifies SuperAdmin role.
6. Backend validates input data.
7. Backend updates document type.
8. Backend saves activity log.
9. Backend returns updated document type.
10. Admin web panel displays success message.

## Alternative Flow

### Document type not found

1. SuperAdmin tries to update a document type that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### User is not SuperAdmin

1. Admin tries to update document type.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- Document type is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/document-types/{id}
```

---

## UC-29: Deactivate Document Type

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to deactivate a document type.

## Preconditions

- SuperAdmin is logged in.
- Document type exists.

## Main Flow

1. SuperAdmin opens document types page.
2. SuperAdmin selects deactivate action.
3. SuperAdmin confirms deactivation.
4. Admin web panel sends status update request.
5. Backend verifies SuperAdmin role.
6. Backend deactivates document type.
7. Backend saves activity log.
8. Backend returns success response.
9. Admin web panel displays updated document type status.

## Alternative Flow

### Document type not found

1. SuperAdmin tries to deactivate a document type that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### Document type is used by existing documents

1. Document type is already used by existing documents.
2. Backend allows deactivation for new documents only.
3. Existing documents keep their assigned document type.

## Postconditions

- Document type is deactivated.
- New documents cannot use this document type.
- Existing documents keep their assigned type.
- Activity log is saved.

## Related API Endpoint

```txt
PATCH /api/admin/document-types/{id}/status
```

---

# 12. Activity Log Use Cases

---

## UC-30: Admin Views Activity Logs

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view system activity logs.

## Preconditions

- Admin is logged in.
- Admin has permission to view activity logs.

## Main Flow

1. Admin opens activity logs page.
2. Admin web panel sends request for activity logs.
3. Backend verifies Admin or SuperAdmin role.
4. Backend loads activity logs.
5. Backend returns activity logs.
6. Admin web panel displays logs.

## Alternative Flow

### No activity logs found

1. Backend returns an empty list.
2. Admin web panel displays information that no activity logs are available.

## Postconditions

- Admin sees activity logs.

## Related API Endpoint

```txt
GET /api/admin/activity-logs
```

---

## UC-31: Admin Views Activity Log Details

## Actor

Admin, SuperAdmin

## Goal

The Admin wants to view details of a selected activity log.

## Preconditions

- Admin is logged in.
- Activity log exists.

## Main Flow

1. Admin opens activity logs page.
2. Admin selects activity log entry.
3. Admin web panel sends request for activity log details.
4. Backend verifies Admin or SuperAdmin role.
5. Backend loads activity log details.
6. Backend returns activity log details.
7. Admin web panel displays activity log details.

## Alternative Flow

### Activity log not found

1. Admin selects activity log that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

## Postconditions

- Admin sees selected activity log details.

## Related API Endpoint

```txt
GET /api/admin/activity-logs/{id}
```

---

# 13. SuperAdmin Use Cases

---

## UC-32: View Admin List

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to view all administrator accounts.

## Preconditions

- SuperAdmin is logged in.

## Main Flow

1. SuperAdmin opens admin management page.
2. Admin web panel sends request for admin list.
3. Backend verifies SuperAdmin role.
4. Backend loads admin accounts.
5. Backend returns admin list.
6. Admin web panel displays administrators.

## Alternative Flow

### User is not SuperAdmin

1. Admin tries to open admin management page.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- SuperAdmin sees admin list.

## Related API Endpoint

```txt
GET /api/admin/admins
```

---

## UC-33: Create Admin Account

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to create a new administrator account.

## Preconditions

- SuperAdmin is logged in.
- Email does not already exist.

## Main Flow

1. SuperAdmin opens admin management page.
2. SuperAdmin selects create admin account.
3. SuperAdmin enters admin data.
4. SuperAdmin submits the form.
5. Admin web panel sends create request.
6. Backend verifies SuperAdmin role.
7. Backend validates input data.
8. Backend checks if email is unique.
9. Backend hashes password.
10. Backend creates user with Admin role.
11. Backend creates admin profile.
12. Backend saves activity log.
13. Backend returns created admin data.
14. Admin web panel displays success message.

## Alternative Flow

### Email already exists

1. SuperAdmin enters an email already used in the system.
2. Backend rejects the request.
3. Admin web panel displays validation error.

### User is not SuperAdmin

1. Admin tries to create another admin account.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- New admin account is created.
- Activity log is saved.

## Related API Endpoint

```txt
POST /api/admin/admins
```

---

## UC-34: Update Admin Account

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to update administrator account data.

## Preconditions

- SuperAdmin is logged in.
- Admin account exists.

## Main Flow

1. SuperAdmin opens admin details.
2. SuperAdmin edits admin data.
3. SuperAdmin submits changes.
4. Admin web panel sends update request.
5. Backend verifies SuperAdmin role.
6. Backend validates input data.
7. Backend updates admin account.
8. Backend saves activity log.
9. Backend returns updated admin data.
10. Admin web panel displays success message.

## Alternative Flow

### Admin account not found

1. SuperAdmin tries to update admin that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### User is not SuperAdmin

1. Admin tries to update another admin account.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- Admin account is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PUT /api/admin/admins/{id}
```

---

## UC-35: Activate or Deactivate Admin Account

## Actor

SuperAdmin

## Goal

The SuperAdmin wants to activate or deactivate an admin account.

## Preconditions

- SuperAdmin is logged in.
- Admin account exists.

## Main Flow

1. SuperAdmin opens admin list.
2. SuperAdmin selects activate or deactivate action.
3. SuperAdmin confirms status change.
4. Admin web panel sends status update request.
5. Backend verifies SuperAdmin role.
6. Backend updates admin account status.
7. Backend saves activity log.
8. Backend returns success response.
9. Admin web panel displays updated admin status.

## Alternative Flow

### Admin account not found

1. SuperAdmin selects admin account that does not exist.
2. Backend returns not found response.
3. Admin web panel displays error message.

### User is not SuperAdmin

1. Admin tries to change admin account status.
2. Backend denies access.
3. Admin web panel displays access denied message.

## Postconditions

- Admin account status is updated.
- Activity log is saved.

## Related API Endpoint

```txt
PATCH /api/admin/admins/{id}/status
```

---

# 14. Access Control Summary

The table below summarizes which actor can perform selected use cases.

| Use Case | Citizen | Admin | SuperAdmin |
|---|---:|---:|---:|
| Login | Yes | Yes | Yes |
| Logout | Yes | Yes | Yes |
| View own profile | Yes | No | No |
| View own documents | Yes | No | No |
| View document QR code | Yes | No | No |
| View own activity history | Yes | No | No |
| View admin dashboard | No | Yes | Yes |
| View users | No | Yes | Yes |
| Create citizen account | No | Yes | Yes |
| Update citizen data | No | Yes | Yes |
| Create document | No | Yes | Yes |
| Assign document to citizen | No | Yes | Yes |
| Update document status | No | Yes | Yes |
| View document types | No | Yes | Yes |
| Create document type | No | No | Yes |
| Update document type | No | No | Yes |
| Manage admin accounts | No | No | Yes |
| View activity logs | No | Yes | Yes |
| View full audit logs | No | No | Yes |

---

# 15. Use Case Priority

The use cases should be implemented in the following order.

## Priority 1 — Authentication

```txt
UC-01 Citizen Login
UC-02 Admin Login
UC-03 Logout
UC-04 Refresh Access Token
```

## Priority 2 — Citizen Core Features

```txt
UC-05 View Own Profile
UC-08 View Own Document List
UC-09 View Document Details
UC-10 Check Document Status
UC-11 Display Document QR Code
UC-12 Citizen Views Own Activity History
```

## Priority 3 — Admin Core Features

```txt
UC-13 View Admin Dashboard
UC-14 View User List
UC-15 View User Details
UC-16 Create Citizen Account
UC-17 Update User Data
UC-18 Activate or Deactivate User
```

## Priority 4 — Document Management

```txt
UC-19 Admin Views Document List
UC-20 Admin Views Document Details
UC-21 Create Document
UC-22 Assign Document to Citizen
UC-23 Update Document Data
UC-24 Update Document Status
UC-25 Delete Document
```

## Priority 5 — Document Types and Logs

```txt
UC-26 View Document Type List
UC-27 Create Document Type
UC-28 Update Document Type
UC-29 Deactivate Document Type
UC-30 Admin Views Activity Logs
UC-31 Admin Views Activity Log Details
```

## Priority 6 — SuperAdmin

```txt
UC-32 View Admin List
UC-33 Create Admin Account
UC-34 Update Admin Account
UC-35 Activate or Deactivate Admin Account
```

---

# 16. Final Summary

The use cases define the main behavior of the Mobywatel system.

The Citizen focuses on viewing personal documents and activity history.

The Admin focuses on managing users, citizens and documents.

The SuperAdmin focuses on system-level management, including administrators and document types.

All use cases are served by one backend monolith. Citizen-facing use cases are available through the User API, administration use cases are available through the Admin API, and both use the same business logic and database.
