# Project Structure

This document describes the planned folder and project structure for the Mobywatel application.

The system consists of:

- backend monolith,
- User API,
- Admin API,
- citizen web application,
- admin web application,
- citizen mobile application,
- tests,
- documentation,
- Docker configuration.

The main goal of this structure is to keep the backend as one deployable monolith while separating internal business logic, API surfaces, frontend applications, mobile application, tests and documentation in a clean and maintainable way.

The backend architecture is:

```txt
Single deployable backend monolith
    ├── User API
    ├── Admin API
    └── Internal Clean Architecture
            ├── Api
            ├── Application
            ├── Domain
            ├── Infrastructure
            └── Shared
```

The Application layer uses CQRS with MediatR. Controllers do not contain business logic. They only validate HTTP concerns, authorize access and send commands or queries through MediatR.

---

# Main Structure

```txt
Mobywatel/
│
├── src/
│   │
│   ├── Backend/
│   │   ├── Mobywatel.Api/
│   │   │   ├── Controllers/
│   │   │   │   ├── UserApi/
│   │   │   │   │   ├── AuthController.cs
│   │   │   │   │   ├── ProfileController.cs
│   │   │   │   │   ├── DocumentsController.cs
│   │   │   │   │   └── ActivityLogsController.cs
│   │   │   │   │
│   │   │   │   └── AdminApi/
│   │   │   │       ├── AuthController.cs
│   │   │   │       ├── DashboardController.cs
│   │   │   │       ├── UsersController.cs
│   │   │   │       ├── DocumentsController.cs
│   │   │   │       ├── DocumentTypesController.cs
│   │   │   │       ├── ActivityLogsController.cs
│   │   │   │       └── AdminsController.cs
│   │   │   │
│   │   │   ├── Middlewares/
│   │   │   ├── Filters/
│   │   │   ├── Extensions/
│   │   │   ├── Program.cs
│   │   │   └── appsettings.json
│   │   │
│   │   ├── Mobywatel.Application/
│   │   │   ├── Common/
│   │   │   │   ├── Interfaces/
│   │   │   │   ├── Exceptions/
│   │   │   │   ├── Models/
│   │   │   │   ├── Security/
│   │   │   │   └── Behaviors/
│   │   │   │
│   │   │   ├── Auth/
│   │   │   │   ├── Commands/
│   │   │   │   │   └── Login/
│   │   │   │   ├── Queries/
│   │   │   │   └── DTOs/
│   │   │   │
│   │   │   ├── Citizens/
│   │   │   │   ├── Commands/
│   │   │   │   ├── Queries/
│   │   │   │   └── DTOs/
│   │   │   │
│   │   │   ├── Documents/
│   │   │   │   ├── Commands/
│   │   │   │   │   ├── CreateDocument/
│   │   │   │   │   ├── AssignDocumentToCitizen/
│   │   │   │   │   └── UpdateDocumentStatus/
│   │   │   │   ├── Queries/
│   │   │   │   │   ├── GetCitizenDocuments/
│   │   │   │   │   ├── GetDocumentDetails/
│   │   │   │   │   └── GetAllDocuments/
│   │   │   │   └── DTOs/
│   │   │   │
│   │   │   ├── DocumentTypes/
│   │   │   │   ├── Commands/
│   │   │   │   ├── Queries/
│   │   │   │   └── DTOs/
│   │   │   │
│   │   │   ├── ActivityLogs/
│   │   │   │   ├── Commands/
│   │   │   │   ├── Queries/
│   │   │   │   └── DTOs/
│   │   │   │
│   │   │   └── DependencyInjection.cs
│   │   │
│   │   ├── Mobywatel.Domain/
│   │   │   ├── Entities/
│   │   │   │   ├── User.cs
│   │   │   │   ├── Citizen.cs
│   │   │   │   ├── Admin.cs
│   │   │   │   ├── Document.cs
│   │   │   │   ├── DocumentType.cs
│   │   │   │   ├── DocumentStatus.cs
│   │   │   │   ├── ActivityLog.cs
│   │   │   │   └── RefreshToken.cs
│   │   │   │
│   │   │   ├── Enums/
│   │   │   │   ├── UserRole.cs
│   │   │   │   └── DocumentStatusType.cs
│   │   │   │
│   │   │   ├── ValueObjects/
│   │   │   │   ├── Pesel.cs
│   │   │   │   ├── DocumentNumber.cs
│   │   │   │   └── Address.cs
│   │   │   │
│   │   │   ├── Events/
│   │   │   └── Common/
│   │   │       ├── BaseEntity.cs
│   │   │       └── AuditableEntity.cs
│   │   │
│   │   ├── Mobywatel.Infrastructure/
│   │   │   ├── Persistence/
│   │   │   │   ├── MobywatelDbContext.cs
│   │   │   │   ├── Configurations/
│   │   │   │   │   ├── UserConfiguration.cs
│   │   │   │   │   ├── DocumentConfiguration.cs
│   │   │   │   │   ├── DocumentTypeConfiguration.cs
│   │   │   │   │   └── ActivityLogConfiguration.cs
│   │   │   │   ├── Migrations/
│   │   │   │   └── Seed/
│   │   │   │       ├── RoleSeeder.cs
│   │   │   │       ├── AdminSeeder.cs
│   │   │   │       └── DocumentTypeSeeder.cs
│   │   │   │
│   │   │   ├── Repositories/
│   │   │   │   ├── UserRepository.cs
│   │   │   │   ├── DocumentRepository.cs
│   │   │   │   └── ActivityLogRepository.cs
│   │   │   │
│   │   │   ├── Authentication/
│   │   │   │   ├── JwtTokenService.cs
│   │   │   │   └── PasswordHasher.cs
│   │   │   │
│   │   │   ├── Services/
│   │   │   │   ├── CurrentUserService.cs
│   │   │   │   ├── DateTimeService.cs
│   │   │   │   └── QrCodeService.cs
│   │   │   │
│   │   │   └── DependencyInjection.cs
│   │   │
│   │   └── Mobywatel.Shared/
│   │       ├── Constants/
│   │       ├── Contracts/
│   │       ├── Helpers/
│   │       └── Result.cs
│   │
│   ├── Web/
│   │   ├── CitizenWeb/
│   │   │   ├── src/
│   │   │   │   ├── app/
│   │   │   │   │   ├── core/
│   │   │   │   │   │   ├── guards/
│   │   │   │   │   │   ├── interceptors/
│   │   │   │   │   │   ├── services/
│   │   │   │   │   │   └── models/
│   │   │   │   │   │
│   │   │   │   │   ├── features/
│   │   │   │   │   │   ├── auth/
│   │   │   │   │   │   ├── dashboard/
│   │   │   │   │   │   ├── profile/
│   │   │   │   │   │   ├── documents/
│   │   │   │   │   │   └── activity-history/
│   │   │   │   │   │
│   │   │   │   │   ├── shared/
│   │   │   │   │   │   ├── components/
│   │   │   │   │   │   ├── pipes/
│   │   │   │   │   │   └── directives/
│   │   │   │   │   │
│   │   │   │   │   ├── app.routes.ts
│   │   │   │   │   └── app.config.ts
│   │   │   │   │
│   │   │   │   ├── assets/
│   │   │   │   └── environments/
│   │   │   │
│   │   │   ├── angular.json
│   │   │   └── package.json
│   │   │
│   │   └── AdminWeb/
│   │       ├── src/
│   │       │   ├── app/
│   │       │   │   ├── core/
│   │       │   │   │   ├── guards/
│   │       │   │   │   ├── interceptors/
│   │       │   │   │   ├── services/
│   │       │   │   │   └── models/
│   │       │   │   │
│   │       │   │   ├── features/
│   │       │   │   │   ├── auth/
│   │       │   │   │   ├── dashboard/
│   │       │   │   │   ├── users/
│   │       │   │   │   ├── documents/
│   │       │   │   │   ├── document-types/
│   │       │   │   │   ├── activity-logs/
│   │       │   │   │   └── system-settings/
│   │       │   │   │
│   │       │   │   ├── shared/
│   │       │   │   │   ├── components/
│   │       │   │   │   ├── tables/
│   │       │   │   │   ├── forms/
│   │       │   │   │   └── dialogs/
│   │       │   │   │
│   │       │   │   ├── layout/
│   │       │   │   │   ├── admin-layout/
│   │       │   │   │   ├── sidebar/
│   │       │   │   │   └── topbar/
│   │       │   │   │
│   │       │   │   ├── app.routes.ts
│   │       │   │   └── app.config.ts
│   │       │   │
│   │       │   ├── assets/
│   │       │   └── environments/
│   │       │
│   │       ├── angular.json
│   │       └── package.json
│   │
│   └── Mobile/
│       └── CitizenMobile/
│           ├── src/
│           │   ├── api/
│           │   │   ├── apiClient.ts
│           │   │   ├── authApi.ts
│           │   │   ├── documentsApi.ts
│           │   │   └── activityApi.ts
│           │   │
│           │   ├── navigation/
│           │   │   ├── AppNavigator.tsx
│           │   │   ├── AuthNavigator.tsx
│           │   │   └── CitizenNavigator.tsx
│           │   │
│           │   ├── screens/
│           │   │   ├── LoginScreen.tsx
│           │   │   ├── DashboardScreen.tsx
│           │   │   ├── ProfileScreen.tsx
│           │   │   ├── DocumentsScreen.tsx
│           │   │   ├── DocumentDetailsScreen.tsx
│           │   │   ├── QrCodeScreen.tsx
│           │   │   └── ActivityHistoryScreen.tsx
│           │   │
│           │   ├── components/
│           │   │   ├── DocumentCard.tsx
│           │   │   ├── StatusBadge.tsx
│           │   │   ├── QrCodePreview.tsx
│           │   │   └── AppButton.tsx
│           │   │
│           │   ├── store/
│           │   │   ├── authStore.ts
│           │   │   ├── userStore.ts
│           │   │   └── documentStore.ts
│           │   │
│           │   ├── types/
│           │   │   ├── auth.types.ts
│           │   │   ├── user.types.ts
│           │   │   └── document.types.ts
│           │   │
│           │   ├── utils/
│           │   └── constants/
│           │
│           ├── app.json
│           ├── package.json
│           └── tsconfig.json
│
├── tests/
│   ├── Mobywatel.Application.Tests/
│   ├── Mobywatel.Domain.Tests/
│   ├── Mobywatel.Infrastructure.Tests/
│   └── Mobywatel.Api.Tests/
│
├── docs/
│   ├── architecture/
│   │   ├── clean-architecture.md
│   │   ├── backend-layers.md
│   │   └── frontend-structure.md
│   │
│   ├── database/
│   │   ├── database-schema.md
│   │   └── seed-data.md
│   │
│   ├── api/
│   │   ├── auth-endpoints.md
│   │   ├── citizen-endpoints.md
│   │   ├── document-endpoints.md
│   │   └── admin-endpoints.md
│   │
│   └── product-vision.md
│
├── docker/
│   ├── backend.Dockerfile
│   ├── citizen-web.Dockerfile
│   ├── admin-web.Dockerfile
│   └── mobile.Dockerfile
│
├── docker-compose.yml
├── README.md
├── .gitignore
└── Mobywatel.sln
```

---

# Backend Architecture Rules

The backend is a monolith. It is deployed and run as one ASP.NET Core application, but it exposes two logical APIs:

```txt
User API   - citizen-facing endpoints used by CitizenMobile and CitizenWeb
Admin API  - administration endpoints used by AdminWeb
```

Both APIs use the same Application, Domain, Infrastructure and database. They must not duplicate business logic.

## API Surface Split

The API project should keep controllers separated by responsibility:

```txt
Mobywatel.Api/Controllers/UserApi/
Mobywatel.Api/Controllers/AdminApi/
```

Recommended route prefixes:

```txt
/api/user/*
/api/admin/*
```

Authentication endpoints should be exposed under `/api/user/auth` and `/api/admin/auth`. The preferred MVP approach is to reuse the same authentication use cases internally and apply role checks after login.

## Internal Clean Architecture

The backend follows internal Clean Architecture dependency rules:

```txt
Api -> Application -> Domain
Infrastructure -> Application -> Domain
Shared can be used for simple cross-cutting contracts and result types
```

Rules:

- Domain contains entities, value objects, enums and domain rules.
- Application contains use cases expressed as commands and queries.
- Infrastructure contains database, Identity, external services and technical implementations.
- Api contains controllers, filters, middleware, authentication setup and Swagger setup.
- Controllers call Application through MediatR and do not access Infrastructure directly.

## Application Patterns

The Application layer uses CQRS with MediatR:

```txt
Command  - changes state
Query    - reads state
Handler  - executes one command or query
Behavior - cross-cutting MediatR pipeline logic
```

Examples:

```txt
CreateDocumentCommand
CreateDocumentCommandHandler
GetCitizenDocumentsQuery
GetCitizenDocumentsQueryHandler
ValidationBehavior
AuthorizationBehavior
UnhandledExceptionBehavior
```
