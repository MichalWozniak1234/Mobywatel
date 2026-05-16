# Tech Decisions

This document describes the main technology decisions for the Mobywatel project.

The purpose of this document is to explain why selected technologies, architecture patterns and development tools are used in the project. It also defines the initial technical direction for the MVP version.

---

# 1. Project Context

Mobywatel is a digital citizen document application.

The system consists of:

- backend monolith,
- User API,
- Admin API,
- citizen web application,
- admin web application,
- citizen mobile application,
- relational database,
- Docker-based local development environment.

The main goal of the MVP is to build a working demo system where citizens can view their digital documents and administrators can manage users, documents, document types, statuses and activity logs.

The system uses one backend monolith with two logical API surfaces.

```txt
Citizen Mobile App  ─┐
Citizen Web App     ├──>  User API  ─┐
                                      ├──> Backend Monolith ───> PostgreSQL Database
Admin Web App       ─────>  Admin API ┘
```

---

# 2. Main Technology Stack

## 2.1 Backend

The backend will be implemented using:

```txt
.NET
C#
ASP.NET Core Web API
Entity Framework Core
MediatR
JWT Authentication
Swagger / OpenAPI
```

## 2.2 Web Frontend

The web applications will be implemented using:

```txt
Angular
TypeScript
HTML
SCSS / CSS
```

The project contains two web applications:

```txt
CitizenWeb
AdminWeb
```

## 2.3 Mobile Application

The mobile application will be implemented using:

```txt
React Native
TypeScript
```

The mobile application is:

```txt
CitizenMobile
```

## 2.4 Database

The database will be:

```txt
PostgreSQL
```

The database will run locally using:

```txt
Docker Compose
```

## 2.5 Local Development Environment

The local development environment will use:

```txt
Docker
Docker Compose
```

Docker Compose will be used mainly to start the PostgreSQL database and, if needed, additional services such as backend and frontend containers.

---

# 3. Decision: Backend in .NET and C#

## Decision

The backend monolith will be implemented using .NET and C#.

## Reason

.NET is a good choice for building structured backend applications with clear separation of layers. It supports REST APIs, authentication, dependency injection, Entity Framework Core, validation, testing and OpenAPI documentation.

C# is a strongly typed language, which helps reduce runtime errors and makes the application easier to maintain as the number of modules grows.

## Consequences

Positive consequences:

- strong typing,
- good support for Clean Architecture,
- built-in dependency injection,
- good integration with Entity Framework Core,
- good support for JWT authentication,
- easy Swagger/OpenAPI integration,
- good testing ecosystem.

Negative consequences:

- developers need basic knowledge of .NET project structure,
- backend setup may be heavier than a simple scripting-based API,
- Entity Framework Core migrations must be managed carefully.

---

# 4. Decision: Backend Monolith with Two API Surfaces

## Decision

The backend will be implemented as one deployable monolith. Inside this monolith, the API layer will expose two logical REST API surfaces:

```txt
User API
Admin API
```

## Reason

The MVP does not need distributed services. A monolith keeps local development, deployment, database access and testing simpler while still allowing clean internal separation.

User-facing use cases and admin use cases have different authorization rules and different client applications, so their HTTP endpoints should be separated at the API boundary.

## Consequences

Positive consequences:

- one backend deployment,
- one database and one transaction boundary,
- no network calls between backend modules,
- clear separation between citizen and admin endpoints,
- simpler local development and presentation.

Negative consequences:

- strong discipline is required to avoid mixing admin and citizen controllers,
- the monolith must keep internal module boundaries clear,
- future extraction into separate services would require deliberate refactoring.

---

# 5. Decision: ASP.NET Core Web API

## Decision

The backend monolith will expose REST APIs using ASP.NET Core Web API.

## Reason

The system needs HTTP APIs used by three clients:

```txt
CitizenMobile -> User API
CitizenWeb    -> User API
AdminWeb      -> Admin API
```

REST API is simple, understandable and suitable for the MVP scope. It allows mobile and web applications to communicate with the same backend monolith using HTTP endpoints.

## Consequences

Positive consequences:

- one backend monolith for all clients,
- separate User API and Admin API endpoint surfaces,
- clear endpoint structure,
- easy testing using Swagger,
- simple integration with Angular and React Native,
- easier future extension.

Negative consequences:

- API contracts must be kept stable,
- frontend applications depend on backend endpoint availability,
- authorization rules must be enforced consistently on the backend.

---

# 6. Decision: Internal Clean Architecture for Backend

## Decision

The backend monolith will follow internal Clean Architecture.

The planned backend layers are:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
```

## Reason

The project contains multiple business modules, including authentication, citizens, users, documents, document types, activity logs and dashboard. Clean Architecture helps separate business logic from infrastructure and API controllers.

This structure makes the backend easier to test, maintain and extend without splitting it into multiple deployable services.

## Layer Responsibilities

### Mobywatel.Api

Responsible for:

- controllers,
- request handling,
- authentication configuration,
- middleware,
- filters,
- Swagger configuration.

### Mobywatel.Application

Responsible for:

- use cases,
- commands,
- queries,
- DTOs,
- validation,
- application interfaces.

### Mobywatel.Domain

Responsible for:

- entities,
- enums,
- value objects,
- domain rules,
- base domain classes.

### Mobywatel.Infrastructure

Responsible for:

- database access,
- Entity Framework Core DbContext,
- repositories,
- migrations,
- authentication services,
- technical services.

### Mobywatel.Shared

Responsible for:

- shared constants,
- shared contracts,
- helper classes,
- common result models.

## Consequences

Positive consequences:

- clear separation of concerns,
- easier unit testing,
- business logic is not placed inside controllers,
- infrastructure can be changed more easily,
- project structure is easier to understand.

Negative consequences:

- more projects and folders are required,
- simple features require more files,
- developers must follow dependency rules.

---

# 7. Decision: CQRS and MediatR in Application Layer

## Decision

The Application layer will use CQRS with MediatR.

Commands will represent write operations. Queries will represent read operations. Each command or query will have a dedicated handler.

```txt
CreateDocumentCommand
CreateDocumentCommandHandler
GetCitizenDocumentsQuery
GetCitizenDocumentsQueryHandler
```

## Reason

The project has clear use cases for authentication, citizen profile, document management, admin management and activity logs. CQRS keeps use cases explicit and prevents controllers from becoming business-logic containers.

MediatR gives a simple in-process dispatch mechanism inside the monolith and supports pipeline behaviors for validation, authorization, logging and exception handling.

## Consequences

Positive consequences:

- controllers stay thin,
- commands and queries map directly to use cases,
- validation and authorization can be handled consistently,
- handlers are easy to unit test,
- no distributed messaging is required for the MVP.

Negative consequences:

- simple operations require command/query and handler files,
- developers must keep command and query responsibilities separate,
- MediatR pipeline behavior order must be configured carefully.

---

# 8. Decision: Entity Framework Core

## Decision

Entity Framework Core will be used as the ORM for database access.

## Reason

The MVP uses a relational database and has clearly defined entities such as users, citizens, admins, documents, document types, activity logs and refresh tokens.

Entity Framework Core simplifies database access, migrations and entity mapping in .NET applications.

## Consequences

Positive consequences:

- easier database access from C#,
- strongly typed queries,
- database migrations,
- good integration with PostgreSQL,
- entity configuration can be separated into configuration classes.

Negative consequences:

- generated SQL should be monitored for complex queries,
- migrations must be reviewed before applying,
- developers need to understand tracking and relationships.

---

# 9. Decision: PostgreSQL Database

## Decision

The MVP database will use PostgreSQL.

## Reason

PostgreSQL is a reliable relational database suitable for structured application data. The system needs relational data between users, citizens, admins, documents, document types, activity logs and refresh tokens.

PostgreSQL works well with Entity Framework Core and can be easily started locally using Docker Compose.

## Database Scope

The MVP database should include the following main tables:

```txt
Users
Citizens
Admins
Documents
DocumentTypes
ActivityLogs
RefreshTokens
```

## Consequences

Positive consequences:

- strong relational data model,
- good support for constraints and indexes,
- easy local setup with Docker,
- good compatibility with .NET and EF Core,
- suitable for future growth.

Negative consequences:

- developers must manage database migrations,
- local environment requires Docker or a local PostgreSQL instance,
- database schema changes must be coordinated with backend code.

---

# 10. Decision: PostgreSQL in Docker Compose

## Decision

PostgreSQL will be started locally using Docker Compose.

## Reason

Docker Compose makes local development easier because every developer can run the same database setup without installing PostgreSQL manually.

This also supports the MVP requirement that the project can be started by another person using documentation.

## Planned Docker Compose Service

Example service:

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

## Consequences

Positive consequences:

- repeatable local database setup,
- no need to install PostgreSQL manually,
- easy reset of local data,
- compatible with backend development and testing.

Negative consequences:

- Docker must be installed,
- developers need basic Docker Compose knowledge,
- local data is stored in Docker volumes and must be managed intentionally.

---

# 11. Decision: Angular for Web Applications

## Decision

The citizen web application and admin web panel will be implemented using Angular.

The web applications are:

```txt
CitizenWeb
AdminWeb
```

## Reason

Angular is suitable for structured web applications with routing, forms, services, guards, interceptors and feature modules. The admin panel especially benefits from Angular because it contains many data-management screens.

Using Angular for both web applications keeps the web stack consistent.

## Consequences

Positive consequences:

- consistent web technology stack,
- strong TypeScript support,
- built-in routing,
- good support for forms and validation,
- guards and interceptors fit authentication needs,
- good structure for admin dashboards and management pages.

Negative consequences:

- Angular has more boilerplate than smaller frontend libraries,
- developers need to understand Angular modules, routing and services,
- two Angular applications require consistent shared conventions.

---

# 12. Decision: Separate Citizen Web and Admin Web Applications

## Decision

The project will contain two separate Angular web applications:

```txt
CitizenWeb
AdminWeb
```

## Reason

Citizen functionality and admin functionality have different users, permissions, layouts and screens.

Separating these applications makes the responsibilities clearer:

- CitizenWeb focuses on citizen document access,
- AdminWeb focuses on management and administration.

## Consequences

Positive consequences:

- clearer separation between citizen and admin features,
- different layouts can be implemented independently,
- admin-only functionality is isolated,
- easier to reason about permissions on frontend routes.

Negative consequences:

- some code may be duplicated,
- shared UI or API types may need to be copied or extracted later,
- both apps must be configured and maintained.

---

# 13. Decision: React Native for Mobile Application

## Decision

The citizen mobile application will be implemented using React Native.

## Reason

React Native allows building a mobile application using TypeScript and reusable component-based architecture. It is suitable for a citizen-facing mobile app focused on login, profile, document list, document details, QR code preview and activity history.

## Consequences

Positive consequences:

- one mobile codebase,
- TypeScript support,
- component-based UI,
- good fit for document preview screens,
- easier API integration with the User API.

Negative consequences:

- mobile-specific setup may be required,
- platform-specific issues may appear,
- app store release is outside the MVP scope.

---

# 14. Decision: TypeScript on Frontend and Mobile

## Decision

Angular and React Native code will use TypeScript.

## Reason

TypeScript improves code safety and makes API contracts easier to model on the frontend and mobile sides. It helps avoid common runtime errors when working with DTOs returned by the backend.

## Consequences

Positive consequences:

- typed frontend models,
- safer API calls,
- better editor support,
- easier refactoring,
- consistency across Angular and React Native.

Negative consequences:

- developers must maintain interfaces and types,
- backend DTO changes must be reflected in frontend models.

---

# 15. Decision: JWT Authentication

## Decision

Authentication will use JWT access tokens and refresh tokens.

## Reason

The system has three client applications that communicate with User API or Admin API. JWT authentication works well for stateless API authorization and can be used by web and mobile clients.

Refresh tokens allow users to continue sessions without logging in repeatedly.

## Planned Endpoints

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

## Consequences

Positive consequences:

- suitable for REST API,
- works with web and mobile clients,
- supports role-based authorization,
- access token can contain user role claims.

Negative consequences:

- tokens must be stored carefully on clients,
- refresh tokens must be persisted and revoked correctly,
- token expiration and refresh logic must be implemented in each client.

---

# 16. Decision: Role-Based Authorization

## Decision

The system will use role-based authorization.

The main roles are:

```txt
Citizen
Admin
SuperAdmin
```

## Reason

The MVP has clear differences between citizen, admin and super admin permissions.

Role-based authorization is simple and sufficient for the MVP scope.

## Access Rules Summary

```txt
Citizen:
- can access own profile,
- can access own documents,
- can access own activity history,
- cannot access admin features.

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

## Consequences

Positive consequences:

- simple permission model,
- easy to implement in ASP.NET Core,
- easy to explain in documentation,
- matches MVP use cases.

Negative consequences:

- less flexible than permission-based access control,
- future complex permissions may require a more advanced model.

---

# 17. Decision: Swagger / OpenAPI Documentation

## Decision

The backend monolith will expose Swagger / OpenAPI documentation for User API and Admin API.

## Reason

Swagger helps developers test endpoints and understand request and response models. It is useful in a student/demo MVP because another developer or evaluator can quickly inspect and test the API.

## Consequences

Positive consequences:

- easier API testing,
- visible endpoint documentation,
- easier frontend-backend integration,
- useful for project presentation.

Negative consequences:

- API documentation must stay aligned with implementation,
- production deployments would require controlling Swagger exposure.

---

# 18. Decision: REST Endpoint Structure

## Decision

The backend monolith will expose two logical REST endpoint areas.

Planned endpoint groups:

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

## Reason

This structure matches the main modules and use cases of the system. It keeps citizen-facing endpoints separate from administration endpoints while both still use the same backend monolith and Application layer.

## Consequences

Positive consequences:

- clear endpoint grouping,
- simple integration for clients,
- good match with backend modules,
- easy Swagger navigation.

Negative consequences:

- endpoint naming must stay consistent,
- shared operations must be placed carefully to avoid confusion,
- authorization must be verified in handlers or policies, not only by route prefix.

---

# 19. Decision: Activity Logging

## Decision

The system will store important user and admin actions in activity logs.

## Reason

The application manages document-related data, so users and administrators should be able to see important actions. Activity logs are also part of the MVP scope and support audit-like functionality.

## Example Activity Types

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

## Consequences

Positive consequences:

- better visibility of system actions,
- citizen activity history can be displayed,
- admin audit preview can be implemented,
- useful for debugging and presentation.

Negative consequences:

- more database writes,
- activity log format must be consistent,
- sensitive data should not be stored in log descriptions.

---

# 20. Decision: QR Code Preview in MVP

## Decision

The MVP will include QR code preview for documents, but without real external verification.

## Reason

QR code preview is useful for demonstrating digital document functionality. However, integration with real verification systems is outside the MVP scope.

## Consequences

Positive consequences:

- improves demo value,
- gives citizens a document-related mobile feature,
- keeps scope manageable.

Negative consequences:

- QR code is not legally valid,
- QR verification is not implemented,
- future production-like verification would require additional design.

---

# 21. Decision: Docker for Local Development

## Decision

Docker will be used to support local development.

At minimum, Docker Compose will run PostgreSQL.

Optionally, Docker can also be used for:

```txt
Backend Monolith
CitizenWeb
AdminWeb
CitizenMobile development container
```

## Reason

Docker makes the local setup more repeatable and easier to run on different machines.

## Consequences

Positive consequences:

- easier onboarding,
- consistent local database,
- simple project startup,
- better presentation readiness.

Negative consequences:

- Docker must be installed,
- container networking can cause configuration issues,
- development setup must be documented clearly.

---

# 22. Decision: MVP Does Not Integrate with Real Government Systems

## Decision

The MVP will not integrate with real government systems, identity providers, trusted profile services or external document registries.

## Reason

The project is a student/demo application focused on architecture, functionality and presentation. Real government integrations would require legal, security and organizational requirements outside the MVP scope.

## Consequences

Positive consequences:

- realistic MVP scope,
- simpler implementation,
- no dependency on external public systems,
- easier local development and testing.

Negative consequences:

- documents are demo data only,
- QR codes are preview-only,
- system is not production-ready for real citizen data.

---

# 23. Decision: Seed Data for Demo

## Decision

The MVP will include seed data for local testing and presentation.

Required seed data:

```txt
Default roles
Default SuperAdmin account
Default Admin account
Default document types
Example citizen account
Example documents
```

Example users:

```txt
superadmin@mobywatel.local
admin@mobywatel.local
citizen@mobywatel.local
```

## Reason

Seed data allows the system to be tested immediately after setup. It also makes project presentation easier.

## Consequences

Positive consequences:

- easier manual testing,
- easier demo preparation,
- repeatable local environment.

Negative consequences:

- demo credentials must not be used in production,
- seed logic must be separated from real production data handling.

---

# 24. Decision: Testing Strategy

## Decision

The MVP should include backend tests first.

Planned test projects:

```txt
Mobywatel.Application.Tests
Mobywatel.Domain.Tests
Mobywatel.Infrastructure.Tests
Mobywatel.Api.Tests
```

## Reason

The backend contains the main business logic and authorization rules. Testing backend logic first gives the highest value for MVP stability.

## Consequences

Positive consequences:

- business rules can be verified,
- authorization behavior can be tested,
- safer future refactoring,
- better project quality.

Negative consequences:

- tests require additional time,
- test data and database setup must be maintained.

---

# 25. Initial Implementation Priority

The implementation should follow this order:

```txt
1. Backend solution structure
2. PostgreSQL Docker Compose setup
3. Domain entities
4. Entity Framework Core DbContext and migrations
5. Authentication and JWT
6. Role-based authorization
7. Citizen profile features
8. Document types and document statuses
9. Document management
10. QR code preview
11. Activity logs
12. Admin dashboard
13. Citizen mobile application
14. Citizen web application
15. Admin web application
16. Tests and documentation
```

This order is recommended because later modules depend on earlier backend foundations.

---

# 26. Summary of Accepted Decisions

| Area | Decision |
|---|---|
| Backend language | C# |
| Backend framework | .NET / ASP.NET Core Web API |
| Backend architecture | Backend monolith with internal Clean Architecture |
| API surfaces | User API and Admin API |
| Application patterns | CQRS + MediatR |
| ORM | Entity Framework Core |
| Database | PostgreSQL |
| Local database setup | Docker Compose |
| Web frontend | Angular |
| Mobile frontend | React Native |
| Frontend language | TypeScript |
| Authentication | JWT access token + refresh token |
| Authorization | Role-based access control |
| API documentation | Swagger / OpenAPI |
| API style | REST |
| MVP QR code | Preview only, no external verification |
| MVP integrations | No real government integrations |
| Initial testing focus | Backend tests |

---

# 27. Final Technical Direction

The project will use a .NET C# backend monolith with internal Clean Architecture, PostgreSQL database running in Docker Compose, Angular web applications and a React Native mobile application.

The backend will expose two logical REST API surfaces: User API for citizen-facing clients and Admin API for the administration panel. Authentication will use JWT tokens, access control will be based on three roles: Citizen, Admin and SuperAdmin, and application use cases will be implemented with CQRS + MediatR.

The MVP focuses on a clean modular structure, working document management, activity logging, local Docker-based setup and clear documentation.
