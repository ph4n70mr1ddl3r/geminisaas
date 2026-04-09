# Construction & Engineering Service Specification

## Overview

The Construction & Engineering service provides specialized tools for managing large-scale capital projects, including scheduling, risk management, and field collaboration. This aligns with Oracle Construction and Engineering Global Business Unit (CEGBU) products like Primavera and Aconex.

## Data Model (SQLite)

- `capital_projects`: id (UUID), name, type (INFRASTRUCTURE/BUILDING/ENERGY), status, total_budget, prime_contractor_id.
- `project_schedules`: id (UUID), project_id, version, baseline_date, critical_path (JSON).
- `construction_tasks`: id (UUID), schedule_id, name, duration, start_date, end_date, resources_assigned (JSON).
- `field_inspections`: id (UUID), task_id, inspector_id, date, status (PASSED/FAILED), safety_concerns (JSON).
- `rfis_construction`: id (UUID), project_id, title, sender_id, receiver_id, status (OPEN/RESPONDED/CLOSED), due_date.
- `bim_models`: id (UUID), project_id, version, file_url, elements_linked (JSON).

## API Endpoints (Axum/gRPC)

- `POST /construction/projects`: Create a new capital construction project.
- `GET /construction/schedule/critical-path`: Identify the sequence of tasks that determine the project end date.
- `POST /construction/field/inspection`: Record a safety or quality inspection from the job site.
- `POST /construction/rfi`: Submit a Request for Information (RFI) to the architect or engineer.
- `GET /construction/safety/predictive-score`: Retrieve an AI-driven safety risk score for a job site.
- `POST /construction/bim/link`: Associate a physical task with a 3D model element.

## Inter-service Interaction

- Queries `Project Management` (Financials) for budget and actual cost alignment.
- Emits `Safety_Risk_Alert` to project owners based on predictive analytics.
- Receives `Employee_Certification` from `Learning` to verify on-site worker skills.
- Queries `Procurement` for large material orders (e.g., steel, concrete).
- Emits `Fixed_Asset_Created` to `Fixed Assets` when a building or plant is commissioned.

## Key Logic

- **Critical Path Method (CPM):** Managing thousands of dependencies to calculate the most efficient construction timeline.
- **Safety Analytics:** Using historical data from thousands of projects to predict and prevent on-site incidents.
- **BIM Integration:** Syncing 3D design models with the project schedule (4D) and cost data (5D).
- **Subcontractor Management:** Managing the workflow for RFIs, submittals, and payment applications from multiple vendors.
- **Construction-in-Progress (CIP):** Automatically tracking project costs that will eventually be capitalized as Fixed Assets.
- **Visual Progress:** Supporting photo and video documentation from the field to verify task completion.
