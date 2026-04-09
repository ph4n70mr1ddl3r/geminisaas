# Community Development (Public Sector) Service Specification

## Overview

The Community Development service manages land use, planning, zoning, and code enforcement for local government agencies. It provides a platform for managing the entire lifecycle of community growth and safety. This aligns with Oracle Fusion Cloud Community Development (Public Sector).

## Data Model (SQLite)

- `planning_applications`: id (UUID), type (SUBDIVISION/REZONE/VARIANCE), location_gis (JSON), applicant_id, status (SUBMITTED/REVIEW/HEARING/DECIDED).
- `zoning_districts`: id (UUID), code (R1/C1/I1), name, allowable_uses (JSON), max_height, set_back_rules (JSON).
- `public_hearings`: id (UUID), application_id, hearing_date, location, status (SCHEDULED/COMPLETED), decision_type.
- `code_enforcement_cases`: id (UUID), violation_type (WEEDS/JUNK/STRUCTURE), address, status (OPEN/CITATION/REMEDIATED/CLOSED), reporter_id.
- `citation_ledger`: id (UUID), case_id, amount, due_date, status (PENDING/PAID/APPEALED).
- `property_development_history`: id (UUID), property_id (link to Permitting), event_type, date, summary.

## API Endpoints (Axum/gRPC)

- `POST /community-dev/planning/apply`: Submit a new land-use or zoning application.
- `GET /community-dev/zoning/lookup`: Retrieve allowable uses and rules for a specific location.
- `POST /community-dev/hearings/schedule`: Organize a public hearing for a planning application.
- `POST /community-dev/code-enforcement/report`: Log a potential municipal code violation.
- `GET /community-dev/cases/active-map`: View a spatial distribution of active code enforcement and planning cases.
- `POST /community-dev/citations/issue`: Generate a fine or citation for a code violation.

## Inter-service Interaction

- Receives `Property_Registered` from `Permitting & Licensing` to synchronize base property data.
- Emits `Zoning_Change_Approved` to `Permitting` to update building permit eligibility.
- Queries `Financials` for citation and fee collection.
- Emits `Safety_Hazard_Alert` to `Workforce Health & Safety` if a structure is deemed unsafe.
- Queries `Identity Service` for citizen accounts and professional licenses.

## Key Logic

- **GIS-Integrated Planning:** Visualizing the impact of a new subdivision or rezone directly on the municipal map.
- **Hearing Workflow:** Managing the notification process for neighbors and stakeholders within a specific radius of a proposed development.
- **Code Enforcement Escalation:** Automatically increasing fines for repeat violators or non-remediated issues.
- **Zoning Validation:** Automatically checking if a proposed land use matches the existing zoning ordinance.
- **Conditional Use Tracking:** Monitoring specific requirements (e.g., "Must plant 10 trees") that were mandated as a condition of approval.
- **Public Transparency:** Providing a portal for citizens to view proposed changes in their neighborhood and submit comments.
