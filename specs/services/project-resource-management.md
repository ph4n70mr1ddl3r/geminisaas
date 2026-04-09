# Project Resource Management Service Specification

## Overview

The Project Resource Management service handles the identification, allocation, and tracking of resources for project-based work. This aligns with Oracle Fusion Cloud Project Resource Management.

## Data Model (SQLite)

- `resource_profiles`: id (UUID), employee_id (link to HCM), skills (JSON), primary_role, home_office, resume_text.
- `resource_requests`: id (UUID), project_id (link to Project Mgmt), role_required, start_date, end_date, effort_pct, skills_required (JSON), status (NEW/SEARCHING/FILLED/CANCELLED).
- `resource_assignments`: id (UUID), request_id, resource_id, start_date, end_date, effort_pct, status (PROPOSED/COMMITTED).
- `resource_schedules`: id (UUID), resource_id, date, total_hours_committed, availability_pct.
- `resource_utilization`: id (UUID), resource_id, period_id, billable_hours, non_billable_hours, target_utilization_pct.
- `resource_pools`: id (UUID), name, manager_id, members (JSON).

## API Endpoints (Axum/gRPC)

- `POST /project-resources/requests`: Create a new request for a resource on a project.
- `GET /project-resources/search`: Find the best-fit resources based on skills, availability, and location.
- `POST /project-resources/assignments`: Confirm a resource assignment to a project.
- `GET /project-resources/utilization`: View real-time and projected utilization reports.
- `GET /project-resources/availability-heatmap`: Visualize resource gaps across the organization.
- `POST /project-resources/skills-update`: Refresh resource skills from their latest project experience.

## Inter-service Interaction

- Receives `Project_Created` from `Project Management` to begin resource planning.
- Queries `HCM Service` for employee core data, leave schedules, and reporting lines.
- Emits `Resource_Assigned` event to `Project Management` to update project costs/timelines.
- Emits `Assignment_Overdue` to resource managers.
- Queries `Learning Service` for recently acquired certifications.

## Key Logic

- **AI-Powered Matching:** Scoring and ranking resources for a request based on skill match (e.g., 80%) and availability (e.g., 100%).
- **Automated Replacement:** Efficiently finding a new resource if a committed resource becomes unavailable (e.g., due to leave).
- **Hard vs. Soft Booking:** Managing "proposed" assignments (soft) that don't yet consume availability, versus "committed" (hard) ones.
- **Inter-Regional Staffing:** Facilitating the borrowing and lending of resources between different business units or countries.
- **Utilization Forecasting:** Predicting future bench-time or over-allocation based on the project pipeline.
