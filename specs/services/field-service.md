# Field Service Management Service Specification

## Overview

The Field Service service manages the dispatching of technicians, scheduling of field activities, and tracking of service-related assets. This aligns with Oracle Fusion Cloud Field Service.

## Data Model (SQLite)

- `field_technicians`: id (UUID), employee_id (link to HCM), skills (JSON), home_location, current_location, status (AVAILABLE/ASSIGNED/OFF_DUTY).
- `service_activities`: id (UUID), service_request_id (link to CRM), location, type (REPAIR/INSTALLATION/MAINTENANCE), duration_est, scheduled_start, scheduled_end, technician_id.
- `technician_schedules`: id (UUID), technician_id, date, start_time, end_time, capacity_pct.
- `service_parts`: id (UUID), service_activity_id, item_id (link to Inventory), quantity, status (ORDERED/PICKED/INSTALLED).
- `field_asset_tracking`: id (UUID), asset_id (link to Fixed Assets), customer_id, location, installation_date, last_service_date.

## API Endpoints (Axum/gRPC)

- `GET /field-service/technicians`: View/Search available technicians based on skills and location.
- `POST /field-service/activities`: Create and schedule a field service task.
- `POST /field-service/optimize`: Run the dispatch optimization engine.
- `GET /field-service/technician-app/{tech_id}`: Retrieve current tasks for a specific technician.
- `POST /field-service/activities/{id}/complete`: Log task completion and parts used.

## Inter-service Interaction

- Receives `Service_Request_Escalated` from `CRM` to initiate field work.
- Queries `HCM Service` for technician skills and availability.
- Emits `Parts_Used` event to `Inventory` for stock reduction.
- Emits `Service_Billable_Hours` to `Project Management/Financials` for billing.
- Queries `Logistics` for part delivery status to a field location.

## Key Logic

- **Routing & Optimization:** Automatically assigning the closest technician with the right skills while minimizing travel time.
- **Service SLAs:** Prioritizing high-priority service activities to meet customer response time agreements.
- **Inventory Reservation:** Reserving parts from the nearest warehouse or technician's vehicle stock.
- **Work Order Completion:** Enforcing mandatory steps (e.g., photo proof, customer signature) before a task can be marked complete.
- **Predictive Maintenance:** Analyzing asset history to proactively schedule service before a failure occurs.
