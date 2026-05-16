# Mobywatel

Mobywatel is a full-stack academic application inspired by the concept of a digital citizen wallet.

The system allows citizens to access their digital documents through a mobile application and a citizen web application, while administrators can manage users, documents, document types, statuses and activity logs through a dedicated administration panel.

The project is developed as part of two university courses:

- Business Application Programming
- Mobile Applications

The main goal of the project is to build one shared backend API and multiple independent client applications:

- Angular citizen web application
- Angular administration panel
- React Native mobile application

The backend is implemented in .NET using Clean Architecture principles and a modular monolith approach.

> This is an academic demo project. It is not connected with any official government service and does not process real public administration data.

---

## 1. Project Overview

Mobywatel is designed as a simple digital document management platform.

Citizens can log in, view their profile, browse assigned documents, check document statuses, display QR codes and view activity history.

Administrators can manage citizen accounts, documents, document statuses, document types and system activity logs.

The system is divided into three main application areas:

```txt
Mobywatel
├── Backend API        -> .NET / C# / ASP.NET Core / Clean Architecture
├── Web Frontend       -> Angular
│   ├── Citizen Web
│   └── Admin Web Panel
└── Mobile Frontend    -> React Native
```

All frontend applications communicate with one shared backend API.

```txt
Citizen Mobile App  ─┐
Citizen Web App     ├──>  Backend API  ───>  PostgreSQL Database
Admin Web Panel     ┘
```

---

## 2. Main Features

### Citizen Features

Citizens can:

- log in to the system,
- view their personal profile,
- view assigned digital documents,
- open document details,
- check document status,
- display a QR code connected with a document,
- view personal activity history.

Citizens cannot create, edit or assign documents.

### Admin Features

Admins can:

- log in to the admin panel,
- view dashboard statistics,
- view user lists and user details,
- create citizen accounts,
- update citizen data,
- view document lists,
- create and assign documents,
- update document statuses,
- view activity logs.

Admins cannot manage other administrators.

### SuperAdmin Features

SuperAdmins can:

- do everything an Admin can do,
- manage administrator accounts,
- manage document types,
- manage selected system dictionaries,
- view full activity logs.

The SuperAdmin role is mainly used to demonstrate extended role-based authorization.

---

## 3. Technology Stack

### Backend

- .NET
- C#
- ASP.NET Core Web API
- Entity Framework Core
- ASP.NET Core Identity
- JWT Authentication
- Swagger / OpenAPI
- Clean Architecture
- CQRS-style application layer
- MediatR-style request handling

### Web Applications

- Angular
- TypeScript
- HTML
- SCSS / CSS

The project contains two Angular web applications:

- `CitizenWeb`
- `AdminWeb`

### Mobile Application

- React Native
- TypeScript

The mobile application is:

- `CitizenMobile`

### Database

- PostgreSQL
- Entity Framework Core migrations
- ASP.NET Core Identity tables

### Local Environment

- Docker
- Docker Compose

All application parts should be started from Docker containers:

- backend API,
- PostgreSQL database,
- citizen web application,
- admin web application,
- mobile application development container.

---

## 4. Docker-Based Startup

The project uses Docker Compose as the main local development environment.

The Docker setup is split into two Compose files:

```txt
docker-compose-db.yml   -> PostgreSQL database only
docker-compose.yml      -> application services
```

This separation allows the database to be started independently from the application. It is useful when the backend is developed locally from an IDE, while PostgreSQL still runs in Docker.

### Database Startup

Start only the database:

```bash
docker compose -f docker-compose-db.yml up -d
```

Stop the database:

```bash
docker compose -f docker-compose-db.yml down
```

Stop the database and remove its volume:

```bash
docker compose -f docker-compose-db.yml down -v
```

### Application Startup

Start application services:

```bash
docker compose up --build
```

Stop application services:

```bash
docker compose down
```

The expected local setup should include containers for:

```txt
Backend API
PostgreSQL Database
Citizen Web Application
Admin Web Application
Citizen Mobile Application
```

A simplified container overview:

```txt
Mobywatel Docker Environment
│
├── backend-api          -> .NET ASP.NET Core Web API
├── postgres            -> PostgreSQL database
├── citizen-web         -> Angular citizen web application
├── admin-web           -> Angular admin web panel
└── citizen-mobile      -> React Native development container
```

The goal is that another developer can start the project without manually installing PostgreSQL, .NET runtime, Angular tooling or mobile dependencies on the host machine.

---

## 5. Architecture Overview

The project architecture can be described on several levels:

```txt
1. System / Deployment Architecture
2. Internal Backend Architecture
3. Application Patterns
4. Design Patterns
5. Integration and Communication Patterns
```

This separation makes the system easier to explain because deployment structure, backend code organization and communication rules are described independently.

---

## 6. System / Deployment Architecture

The system is deployed as a single backend application supported by separate frontend clients and a PostgreSQL database.

The backend is designed as a **single modular monolith**.

This means that the backend is deployed as one application, but internally it is divided into clear modules such as Authentication, Citizens, Documents, Document Types, Activity Logs and Dashboard.

```txt
Citizen Mobile App  ─┐
Citizen Web App     ├──>  Backend API Monolith  ───>  PostgreSQL Database
Admin Web Panel     ┘
```

### Deployment Units

The planned deployment units are:

```txt
backend-api       -> ASP.NET Core Web API monolith
citizen-web       -> Angular citizen web application
admin-web         -> Angular admin web panel
citizen-mobile    -> React Native development container
postgres          -> PostgreSQL database
```

### Why Single Monolith?

A single monolith is used because the MVP is a student/demo application with a limited scope. It is easier to develop, test, run and present than a distributed microservice system.

The project still keeps modular boundaries inside the codebase, so individual modules can be separated in the future if needed.

### Docker Compose Files

The project uses two Docker Compose files:

```txt
docker-compose-db.yml
```

Used for starting only PostgreSQL.

```txt
docker-compose.yml
```

Used for starting the main application containers.

The application containers should use the same Docker network as the database container.

---

## 7. Internal Backend Architecture

The backend follows **Clean Architecture**.

The planned backend layers are:

```txt
Mobywatel.Api
Mobywatel.Application
Mobywatel.Domain
Mobywatel.Infrastructure
Mobywatel.Shared
```

### Dependency Direction

Dependencies should point inward:

```txt
Mobywatel.Api  ─────────────┐
Mobywatel.Infrastructure ───┤
                            ▼
                    Mobywatel.Application
                            ▼
                      Mobywatel.Domain
```

The Domain layer should not depend on API, Infrastructure or external frameworks.

### Layer Responsibilities

#### Mobywatel.Api

Responsible for:

- controllers,
- HTTP request handling,
- authentication configuration,
- authorization configuration,
- middleware,
- filters,
- Swagger configuration.

#### Mobywatel.Application

Responsible for:

- use cases,
- commands,
- queries,
- DTOs,
- validation,
- application interfaces,
- application-level business logic.

#### Mobywatel.Domain

Responsible for:

- entities,
- enums,
- value objects,
- domain rules,
- base domain classes.

#### Mobywatel.Infrastructure

Responsible for:

- database access,
- Entity Framework Core DbContext,
- repositories,
- migrations,
- authentication services,
- technical services,
- external infrastructure implementations.

#### Mobywatel.Shared

Responsible for:

- shared constants,
- shared contracts,
- helper classes,
- common result models.

---

## 8. Application Patterns

### CQRS

The application layer should follow a CQRS-style structure.

CQRS means that write operations and read operations are separated into different use cases:

```txt
Commands -> change system state
Queries  -> read system state
```

Examples of commands:

```txt
LoginCommand
CreateCitizenCommand
CreateDocumentCommand
UpdateDocumentStatusCommand
CreateDocumentTypeCommand
CreateAdminCommand
```

Examples of queries:

```txt
GetCurrentUserQuery
GetCitizenProfileQuery
GetCitizenDocumentsQuery
GetDocumentDetailsQuery
GetAdminDashboardQuery
GetActivityLogsQuery
```

The project does not need separate read and write databases for the MVP. CQRS is used mainly as an application-layer organization pattern.

### Request / Response DTOs

The API should not expose domain entities directly.

Each endpoint should use request and response DTOs:

```txt
LoginRequestDto
LoginResponseDto
CreateDocumentRequestDto
DocumentDetailsDto
ActivityLogDto
```

### Validation Pipeline

Input validation should be handled before business logic is executed.

Typical validation examples:

- email is required,
- password is required,
- PESEL is unique,
- document number is unique,
- expiration date is later than issue date,
- document status is valid.

### Result Pattern

Application handlers can return a common result model instead of throwing exceptions for normal business validation errors.

Example response model:

```txt
ApiResponse<T>
PagedResponse<T>
ValidationErrorDto
```

---

## 9. Design Patterns

### Mediator Pattern

The project can use MediatR or a MediatR-style approach to dispatch commands and queries from controllers to application handlers.

Example flow:

```txt
Controller -> Mediator -> Command / Query Handler -> Repository / Service -> Database
```

This keeps controllers thin and moves business logic into the Application layer.

### Repository Pattern

Repositories can be used to hide database access details from the Application layer.

Example repositories:

```txt
UserRepository
DocumentRepository
ActivityLogRepository
```

The Application layer depends on repository interfaces, while the Infrastructure layer provides implementations.

### Dependency Injection

ASP.NET Core dependency injection is used to register services, repositories, validators and infrastructure implementations.

Example responsibilities:

```txt
Mobywatel.Application/DependencyInjection.cs
Mobywatel.Infrastructure/DependencyInjection.cs
```

### Unit of Work

Entity Framework Core `DbContext` can act as a Unit of Work.

Changes made by repositories and handlers are committed together using `SaveChangesAsync()`.

### Options Pattern

Configuration values such as JWT settings, database connection strings or token expiration settings can be mapped to strongly typed options classes.

Example:

```txt
JwtOptions
DatabaseOptions
```

### Factory / Service Pattern

Dedicated services can be used for technical or domain-related operations.

Examples:

```txt
JwtTokenService
QrCodeService
CurrentUserService
DateTimeService
ActivityLogService
```

---

## 10. Integration and Communication Patterns

The MVP does not require message brokers or asynchronous distributed communication. The system uses simple and explicit communication patterns suitable for a modular monolith.

### REST API Communication

All frontend clients communicate with the backend through REST API endpoints.

```txt
CitizenMobile  ─┐
CitizenWeb     ├──>  REST API  ───>  Application Layer
AdminWeb       ┘
```

Main endpoint groups:

```txt
/api/auth
/api/citizens
/api/admin
/api/super-admin
/api/documents
/api/document-types
/api/activity-logs
```

### Request / Response Communication

The system uses synchronous request / response communication.

A client sends an HTTP request, the backend processes it and returns a response immediately.

Example:

```txt
GET /api/citizens/me/documents
```

returns the current citizen's document list.

### Shared API Contract

The backend API is the integration contract between backend, web applications and mobile application.

Swagger / OpenAPI should be used to document:

- available endpoints,
- request models,
- response models,
- HTTP status codes,
- authorization requirements.

### JWT-Based Communication Security

Clients authenticate using JWT access tokens.

For protected endpoints, clients send:

```txt
Authorization: Bearer <access_token>
```

The backend validates the token, reads user claims and applies role-based and ownership-based authorization rules.

### Database Communication

Only the backend communicates directly with the database.

Frontend and mobile applications never connect to PostgreSQL directly.

```txt
Frontend / Mobile -> Backend API -> PostgreSQL
```

Database communication is handled through Entity Framework Core and repositories.

### Internal Module Communication

Backend modules communicate through the Application layer.

Controllers should not directly access Infrastructure or database code.

Example:

```txt
DocumentsController
    -> UpdateDocumentStatusCommand
        -> UpdateDocumentStatusHandler
            -> DocumentRepository
            -> ActivityLogService
```

### Activity Logging Pattern

Important system actions should create activity log entries.

Examples:

```txt
UserLoggedIn
DocumentViewed
DocumentCreated
DocumentStatusUpdated
UserCreated
DocumentTypeCreated
AdminCreated
```

For the MVP, activity logging can be synchronous and stored in the same PostgreSQL database.

### QR Code Communication Pattern

QR codes are not verified by an external system in the MVP.

The backend generates or returns a QR code value, and the frontend/mobile application displays it.

```txt
GET /api/documents/{id}/qr-code
```

The QR code is a preview feature only.

### Kubernetes as Optional Deployment Pattern

Kubernetes can be used as an optional deployment demonstration for container orchestration.

For the MVP, Docker Compose is the primary local development tool. Kubernetes can be added later for:

- backend API deployment,
- citizen web deployment,
- admin web deployment,
- PostgreSQL demo deployment,
- service discovery,
- scaling demonstration.

The React Native mobile app should normally stay outside Kubernetes because it is a mobile client and is usually developed through a Metro or Expo development server.

---

## 11. Main Modules

The system is divided into the following modules:

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

Each module has a clear responsibility and can contain:

- domain entities,
- commands,
- queries,
- DTOs,
- validators,
- repository interfaces,
- repository implementations,
- API endpoints,
- tests.

---

## 12. Authentication and Authorization

The system uses JWT authentication and role-based authorization.

Supported roles:

```txt
Citizen
Admin
SuperAdmin
```

The authentication flow is based on:

- email and password login,
- JWT access tokens,
- refresh tokens,
- logout with refresh token revocation,
- protected API endpoints.

Example authorization header:

```txt
Authorization: Bearer <access_token>
```

Authorization is based on:

- role-based access control,
- ownership checks for citizen data,
- backend-enforced permission rules.

A Citizen can only access their own profile, documents, QR codes and activity history.

Admins and SuperAdmins can access management features according to their roles.

---

## 13. Database

The project uses PostgreSQL as the relational database.

The database is managed through Entity Framework Core and ASP.NET Core Identity.

The database contains two main groups of tables:

```txt
Identity tables
Domain tables
```

### Identity Tables

Identity tables are responsible for users, roles, claims, logins and tokens.

Examples:

```txt
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetRoleClaims
AspNetUserLogins
AspNetUserTokens
```

### Domain Tables

Domain tables store business data.

Planned domain tables:

```txt
Citizens
Admins
Documents
DocumentTypes
ActivityLogs
RefreshTokens
```

Main relationships:

```txt
AspNetUsers 1 ─── 0..1 Citizens
AspNetUsers 1 ─── 0..1 Admins
AspNetUsers 1 ─── * RefreshTokens
AspNetUsers 1 ─── * ActivityLogs

Citizens 1 ─── * Documents
DocumentTypes 1 ─── * Documents
```

---

## 14. API Overview

The backend exposes a REST API shared by all client applications.

Base URL for local development:

```txt
https://localhost:5001/api
```

Alternative Docker/internal URL:

```txt
http://backend-api:8080/api
```

Main endpoint groups:

```txt
/api/auth
/api/citizens
/api/admin
/api/super-admin
/api/documents
/api/document-types
/api/activity-logs
```

Example response format:

```json
{
  "success": true,
  "data": {},
  "message": null,
  "errors": []
}
```

Example error response:

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

---

## 15. MVP Scope

The MVP focuses on delivering a working demo system.

The MVP includes:

- backend API,
- PostgreSQL database,
- citizen mobile application,
- citizen web application,
- admin web panel,
- authentication,
- authorization,
- basic document management,
- document status management,
- QR code preview,
- activity logging,
- Docker-based local development setup.

The MVP does not include:

- integration with real government systems,
- real identity verification,
- trusted profile integration,
- biometric authentication,
- payment services,
- legal document validation,
- production-level security certification,
- external document registry integration,
- push notifications,
- offline mode,
- advanced analytics,
- real personal data processing.

---

## 16. Project Structure

Planned high-level project structure:

```txt
Mobywatel/
│
├── src/
│   ├── Backend/
│   │   ├── Mobywatel.Api/
│   │   ├── Mobywatel.Application/
│   │   ├── Mobywatel.Domain/
│   │   ├── Mobywatel.Infrastructure/
│   │   └── Mobywatel.Shared/
│   │
│   ├── Web/
│   │   ├── CitizenWeb/
│   │   └── AdminWeb/
│   │
│   └── Mobile/
│       └── CitizenMobile/
│
├── docker/
│   ├── backend.Dockerfile
│   ├── citizen-web.Dockerfile
│   ├── admin-web.Dockerfile
│   └── mobile.Dockerfile
│
├── k8s/
│   ├── postgres/
│   ├── backend/
│   ├── citizen-web/
│   └── admin-web/
│
├── tests/
│   ├── Mobywatel.Application.Tests/
│   ├── Mobywatel.Domain.Tests/
│   ├── Mobywatel.Infrastructure.Tests/
│   └── Mobywatel.Api.Tests/
│
├── docs/
│   ├── architecture/
│   ├── database/
│   ├── api/
│   └── authorization/
│
├── docker-compose.yml
├── docker-compose-db.yml
└── README.md
```

---

## 17. Local Development

The project is intended to run locally using Docker Compose.

### Start Database Only

```bash
docker compose -f docker-compose-db.yml up -d
```

This starts only the PostgreSQL database. It is useful when the backend is run locally from an IDE.

### Start Application Services

```bash
docker compose up --build
```

This starts the application containers, such as backend API, web applications and mobile development container.

At minimum, the Docker Compose setup should include:

- PostgreSQL database container,
- backend API container,
- citizen web application container,
- admin web application container,
- mobile application development container.

### Example PostgreSQL Service

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
    networks:
      - mobywatel-network

volumes:
  postgres_data:

networks:
  mobywatel-network:
    name: mobywatel-network
    driver: bridge
```

### Example Service Naming Convention

```yaml
services:
  backend-api:
    # .NET backend API container

  postgres:
    # PostgreSQL database container

  citizen-web:
    # Angular citizen web application container

  admin-web:
    # Angular admin web panel container

  citizen-mobile:
    # React Native development container
```

When the backend runs inside Docker, the connection string should use the Compose service name:

```txt
Host=postgres;Port=5432;Database=mobywatel_db;Username=mobywatel_user;Password=mobywatel_password
```

When the backend runs directly on the host machine, the connection string should use `localhost`:

```txt
Host=localhost;Port=5432;Database=mobywatel_db;Username=mobywatel_user;Password=mobywatel_password
```

---

## 18. Planned Demo Data

The project should include seed data for local testing and presentation.

Required seed data:

```txt
Default roles
Default SuperAdmin account
Default Admin account
Default Citizen account
Default document types
Example documents
```

Example roles:

```txt
Citizen
Admin
SuperAdmin
```

Example users:

```txt
superadmin@mobywatel.local
admin@mobywatel.local
citizen@mobywatel.local
```

Example document types:

```txt
Identity Card
Driving License
Student Card
Passport
Residence Card
```

---

## 19. Testing

The MVP should include backend tests first.

Planned test projects:

```txt
Mobywatel.Application.Tests
Mobywatel.Domain.Tests
Mobywatel.Infrastructure.Tests
Mobywatel.Api.Tests
```

Important test areas:

- authentication,
- authorization,
- citizen ownership checks,
- document management,
- document status updates,
- API responses,
- validation rules.

---

## 20. Success Criteria

The project can be considered successful when:

- the backend API runs correctly,
- the database runs in Docker,
- Swagger documentation is available,
- Citizen can log in,
- Citizen can view own profile,
- Citizen can view own documents,
- Citizen can open document details,
- Citizen can display QR code,
- Citizen can view activity history,
- Admin can log in,
- Admin can view dashboard,
- Admin can manage users,
- Admin can manage documents,
- Admin can update document statuses,
- SuperAdmin can manage document types and admin accounts,
- activity logs are saved and displayed,
- mobile and web applications communicate with the shared API,
- all main application services can be started from Docker containers,
- Kubernetes manifests are available as an optional deployment example,
- the project can be started by another person using documentation.

---

## 21. Academic Disclaimer

This project is created for educational purposes.

Mobywatel is not an official government application.  
It does not connect to real public administration systems.  
It does not verify real identities or real documents.  
All data used in the project should be treated as demo data only.
