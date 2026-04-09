# Visual Builder (Extension Platform) Service Specification

## Overview

The Visual Builder (Extension Platform) service provides a low-code environment for extending and customizing the Oracle Fusion Cloud Redwood UI. This aligns with Oracle Visual Builder (VB) and Visual Builder Studio (VBS).

## Data Model (SQLite)

- `extension_projects`: id (UUID), name, status (DRAFT/PUBLISHED/ARCHIVED), base_application (ERP/SCM/HCM/CX).
- `extension_pages`: id (UUID), project_id, name, path, components (JSON).
- `extension_service_connections`: id (UUID), project_id, name, base_url (link to a Service API), auth_type (OAUTH/BASIC).
- `extension_business_objects`: id (UUID), project_id, name, fields (JSON), source_type (NATIVE/EXTERNAL).
- `extension_build_jobs`: id (UUID), project_id, status (PENDING/BUILDING/SUCCESS/FAILED), start_time, artifact_url.
- `extension_deployments`: id (UUID), project_id, environment (DEV/TEST/PROD), version, deployed_at.

## API Endpoints (Axum/gRPC)

- `POST /visual-builder/projects`: Create a new UI extension project.
- `GET /visual-builder/projects/{id}/pages`: Retrieve all pages for an extension project.
- `POST /visual-builder/pages/save`: Save the UI layout and component configuration for a page.
- `POST /visual-builder/build/trigger`: Initiate the build and packaging of an extension.
- `GET /visual-builder/deployments/status`: View current deployment status across environments.
- `POST /visual-builder/service-connections`: Create a new connection to an external or internal service.

## Inter-service Interaction

- Queries **ALL Services** via `extension_service_connections` to display and modify data in the UI.
- Emits `Extension_Deployed` event to `Identity Service` to manage security roles.
- Queries `Identity Service` for developer permissions.
- Emits `Build_Failed` alert to project owners.

## Key Logic

- **Redwood Component Library:** Providing a pre-defined set of high-performance, accessible UI components (e.g., standard tables, forms, headers).
- **Visual Design Interface:** Allowing developers to drag-and-drop components onto a canvas and configure their properties.
- **REST Service Mapping:** Automatically generating business objects and UI bindings based on OpenAPI/Swagger definitions.
- **Declarative Business Logic:** Defining UI behavior (e.g., "On button click, call Service X") using a visual action chain editor.
- **CI/CD Automation:** Managing the full lifecycle from code commit to automated build, test, and production deployment.
- **Sandboxing:** Allowing developers to work on UI extensions in an isolated environment without affecting the live application.
