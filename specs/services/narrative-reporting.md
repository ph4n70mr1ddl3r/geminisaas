# Narrative Reporting Service Specification

## Overview

The Narrative Reporting service enables collaborative creation of internal and external financial and operational reports, combining data and narrative explanations. This aligns with Oracle Fusion Cloud Narrative Reporting.

## Data Model (SQLite)

- `report_packages`: id (UUID), name, status (DRAFT/IN_PROGRESS/FINAL), type (FINANCIAL/ANNUAL/MANAGEMENT), due_date.
- `report_doclets`: id (UUID), package_id, title, author_id, reviewer_id, status (NEW/EDITING/SUBMITTED/APPROVED), file_url, sequence.
- `report_variables`: id (UUID), name, current_value, data_source_query (JSON), last_refreshed.
- `report_approvals`: id (UUID), package_id, approver_id, status, comments, date.
- `report_versions`: id (UUID), doclet_id, version_number, file_url, timestamp, user_id.
- `report_comments`: id (UUID), doclet_id, user_id, text, type (GENERAL/FIXED), resolved_status.

## API Endpoints (Axum/gRPC)

- `POST /narrative/packages`: Create a new report package.
- `GET /narrative/packages/{id}/doclets`: Retrieve individual sections of a report.
- `POST /narrative/doclets/submit`: Author submitting a section for review.
- `GET /narrative/variables/refresh`: Update all dynamic data values from source systems.
- `POST /narrative/approve`: Final approval of the entire report package.
- `GET /narrative/export`: Generate a PDF or Word version of the report.

## Inter-service Interaction

- Queries `Financials Service` and `EPM Service` for real-time data values to populate report variables.
- Emits `Report_Finalized` event to stakeholder distribution lists.
- Queries `Identity Service` for author and reviewer roles.
- Emits `Comment_Added` notifications to doclet authors.

## Key Logic

- **Collaborative Authoring:** Allowing multiple users to work on different sections (doclets) of a single report package simultaneously with check-in/check-out controls.
- **Dynamic Variables:** Centralizing data points (e.g., "Total Revenue") so that a change in the source system automatically updates every instance of that value across the entire report.
- **Workflow Phases:** Managing distinct phases for a report: Authoring, Reviewing, and Approving.
- **Narrative Context:** Encouraging users to explain the "why" behind the numbers, not just the data itself.
- **Distribution Management:** Securely sharing finalized reports with internal management or external regulatory bodies.
