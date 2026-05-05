# Project Structure

This document describes the planned folder and project structure for the Mobywatel application.

The system consists of:

- backend API,
- citizen web application,
- admin web application,
- citizen mobile application,
- tests,
- documentation,
- Docker configuration.

The main goal of this structure is to separate backend logic, frontend applications, mobile application, tests and documentation in a clean and maintainable way.

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
│   │   │   │   ├── AuthController.cs
│   │   │   │   ├── CitizensController.cs
│   │   │   │   ├── DocumentsController.cs
│   │   │   │   ├── DocumentTypesController.cs
│   │   │   │   ├── ActivityLogsController.cs
│   │   │   │   └── AdminController.cs
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
│   │   │   │   └── Security/
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