# B2B Service Specification

## Overview

The B2B Service (formerly Oracle Engagement Cloud Service) manages complex service requests, account-level SLAs, and collaboration for business-to-business customer service. This aligns with Oracle Fusion Cloud B2C Service but focuses on business accounts.

## Data Model (SQLite)

- `service_requests`: id (UUID), account_id (link to CDM), subject, description, severity (LOW/MED/HIGH), status (OPEN/ASSIGNED/SOLVED/CLOSED), contact_id, assigned_to_id.
- `service_request_threads`: id (UUID), service_request_id, sender_id, text, type (CUSTOMER/AGENT/SYSTEM), timestamp.
- `service_level_agreements` (SLA): id (UUID), account_id, type (PLATINUM/GOLD/SILVER), response_time_hours, resolution_time_hours, status.
- `service_milestones`: id (UUID), service_request_id, type (FIRST_RESPONSE/RESOLUTION), due_date, status (PENDING/COMPLETED/VIOLATED), completion_date.
- `service_queues`: id (UUID), name, members (JSON), routing_logic (JSON).
- `service_case_collaboration`: id (UUID), service_request_id, collaborator_id, role (EXPERT/REVIEWER), date_added.

## API Endpoints (Axum/gRPC)

- `POST /b2b-service/requests`: Create a new B2B service request for an account.
- `GET /b2b-service/requests/search`: Search for requests based on account, status, or assignee.
- `POST /b2b-service/requests/{id}/assign`: Route a request to a specific agent or queue.
- `GET /b2b-service/sla/check`: Retrieve SLA status and milestones for a request.
- `POST /b2b-service/collaborate`: Add a subject matter expert (SME) to a service case.
- `GET /b2b-service/analytics/sla-violation`: Identify and report on SLA breaches across accounts.

## Inter-service Interaction

- Queries `CDM Service` for a 360-degree view of the business account and contacts.
- Queries `Knowledge Management` for expert-authored articles and troubleshooting guides.
- Emits `Milestone_Violated` alert to service managers and account owners.
- Emits `Part_Order_Required` event to `Service Logistics` if a field repair is needed.
- Queries `Identity Service` for agent skills and queue memberships.

## Key Logic

- **Account-Based Routing:** Automatically routing requests to the dedicated account manager or a specialized service team.
- **SLA Milestone Tracking:** Real-time monitoring of response and resolution times against pre-negotiated customer contracts.
- **Milestone Escalation:** Automatically escalating a request if a milestone is nearing its due date without completion.
- **Internal Case Collaboration:** Enabling multiple departments (e.g., Engineering, Sales) to collaborate on a single complex service case.
- **Resolution Path Analysis:** Identifying common issues and the most successful resolution paths to improve agent efficiency.
- **Global Coverage:** Handling service requests across different time zones and languages based on the account's locations.
