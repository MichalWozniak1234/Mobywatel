# Product Vision

## 1. Product Name

**Mobywatel**

---

## 2. Product Summary

Mobywatel is a digital citizen application that allows users to access their personal documents and basic public-service-related information from one place.

The system consists of:

- a mobile application for citizens,
- a web administration panel,
- a backend API,
- a relational database.

The mobile application is designed for citizens who want quick access to their digital documents.  
The web application is designed for administrators who manage users, documents, document types, statuses and system data.

The main goal of the project is to create one shared backend API that can be used by two different frontend applications: Angular web application and React Native mobile application.

---

## 3. Problem Statement

Citizens often need fast access to important personal documents and document-related information. In a traditional model, documents are stored physically and must be carried manually. This can be inconvenient, especially when a user needs to quickly check document details, confirm document status or present basic document information.

From the administration side, managing user documents manually can be time-consuming and error-prone. Administrators need a structured system for managing citizens, documents, document types, statuses and activity logs.

The problem solved by Mobywatel is the lack of one simple digital place where:

- citizens can view their documents,
- citizens can check document details and statuses,
- administrators can manage users and documents,
- system activity can be tracked,
- data can be accessed through a consistent API.

---

## 4. Target Users

### 4.1 Citizen

A citizen is the main user of the mobile application.

The citizen can:

- log into the mobile application,
- view their profile,
- view a list of assigned documents,
- open document details,
- check document status,
- display a QR code connected with a document,
- view activity history.

The mobile app should be simple, readable and optimized for quick access.

---

### 4.2 Admin

An admin is a user of the web administration panel.

The admin can:

- log into the web panel,
- view dashboard data,
- manage users,
- manage citizen documents,
- assign documents to users,
- update document statuses,
- view activity logs.

The admin panel should provide convenient tools for managing application data.

---

### 4.3 Super Admin

A super admin has the highest level of access in the system.

The super admin can:

- manage administrators,
- manage system dictionaries,
- manage document types,
- access audit logs,
- configure selected system data.

This role is intended for advanced system management.

---

## 5. Product Goals

The main goals of the product are:

1. Provide citizens with easy access to their digital documents.
2. Provide administrators with a web panel for managing users and documents.
3. Build one backend API shared by mobile and web applications.
4. Use a clean and maintainable backend architecture.
5. Store data in a relational database.
6. Use Docker to simplify local development and project startup.
7. Separate user-facing mobile functionality from administrative web functionality.
8. Prepare the project for future extension with additional modules.

---

## 6. Business Value

Mobywatel provides value by centralizing citizen document management in one system.

For citizens, the value is:

- faster access to documents,
- clear document status information,
- convenient mobile interface,
- simple document preview,
- access to activity history.

For administrators, the value is:

- easier user management,
- easier document management,
- structured data access,
- status control,
- activity monitoring.

For the development team, the value is:

- one shared backend API,
- reusable business logic,
- clean separation between backend and frontend,
- clear project structure,
- easier testing and maintenance.

---

## 7. Core Features

The first version of the system should include the following core features:

### Mobile Application

- user login,
- user profile preview,
- list of user documents,
- document details,
- document status,
- QR code preview,
- activity history.

### Web Application for citizens
- user login,
- user profile preview,
- list of user documents,
- document details,
- document status,
- activity history.
- additional features

### Web Application for admins

- admin login,
- dashboard,
- user list,
- user details,
- document list,
- document creation,
- document status update,
- activity log preview.




### Backend API

- authentication,
- authorization,
- user management,
- document management,
- document type management,
- activity logging,
- API documentation,
- database integration.

---

## 8. Product Boundaries

The application focuses on document management and basic citizen-service functionality.

The product does not focus on:

- integration with real government systems,
- real identity verification,
- real public administration data,
- payment services,
- legal document validation,
- production-level security certification,
- external trusted profile integration,
- biometric identity verification.

These elements may be considered in a future version only as conceptual extensions.

---

## 9. Success Criteria

The project can be considered successful when:

- the backend API works and exposes documented endpoints,
- the database runs in Docker,
- the Angular web panel communicates with the API,
- the React Native mobile app communicates with the API,
- a citizen can log in and view documents,
- an admin can manage users and documents,
- document statuses can be changed,
- activity logs are saved and displayed,
- the project can be started by another person using the documentation,
- the code follows the planned architecture.

---

## 10. Product Vision Statement

For citizens who need quick access to their personal documents, Mobywatel is a digital document management application that allows users to view documents, check statuses and access basic document-related information from a mobile device.

For administrators, Mobywatel provides a web panel for managing users, documents and system data.

Unlike a single frontend application, Mobywatel uses one shared backend API and two dedicated frontend applications, which makes the system easier to extend, maintain and present across different platforms.

---

## 11. Long-Term Vision

In the future, Mobywatel could be extended with:

- notifications,
- document verification flow,
- advanced QR code verification,
- push notifications,
- audit dashboard,
- document expiration reminders,
- support for more document types,
- offline mobile access,
- integration with external services,
- improved analytics for administrators.

The long-term vision is to create a modular system that can grow from a simple document management application into a broader citizen-service platform.