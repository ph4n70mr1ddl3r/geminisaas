# HR Help Desk Service Specification

## Overview

The HR Help Desk service provides a dedicated case management system for employee HR inquiries, ensuring rapid resolution and consistent service delivery. This aligns with Oracle Fusion Cloud HR Help Desk.

## Data Model (SQLite)

- `hr_cases`: id (UUID), employee_id (link to HCM), subject, description, category (BENEFITS/PAYROLL/RELOCATION), status (NEW/ASSIGNED/SOLVED), priority (LOW/MED/HIGH), assignee_id.
- `hr_case_threads`: id (UUID), case_id, sender_id, text, type (EMPLOYEE/AGENT/SYSTEM), timestamp.
- `hr_knowledge_links`: id (UUID), case_id, article_id (link to Knowledge Management).
- `hr_service_queues`: id (UUID), name, specialization (e.g., 'Payroll Experts'), members (JSON).
- `hr_case_slas`: id (UUID), case_id, response_due, resolution_due, status (MET/BREACHED).
- `hr_action_logs`: id (UUID), case_id, action_taken, user_id, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /hr-helpdesk/cases`: Create a new HR inquiry for an employee.
- `GET /hr-helpdesk/cases/my`: Retrieve inquiries submitted by the current user.
- `POST /hr-helpdesk/cases/{id}/assign`: Route a case to a specific HR agent or queue.
- `GET /hr-helpdesk/agent/inbox`: Retrieve pending cases for an HR representative.
- `POST /hr-helpdesk/cases/{id}/resolve`: Mark an inquiry as solved and record the solution.
- `GET /hr-helpdesk/analytics/volume`: View case volume trends by category.

## Inter-service Interaction

- Queries `HCM Service` for employee data, role, and manager to provide context to the agent.
- Queries `Knowledge Management` to suggest relevant HR policy articles.
- Emits `Payroll_Inquiry_Escalated` event to `Global Payroll` if a pay discrepancy is identified.
- Emits `SLA_Breach_Alert` to HR managers.
- Queries `Identity Service` for HR agent permissions and queue memberships.

## Key Logic

- **Privacy & Sensitivity:** Ensuring that only authorized HR agents can see sensitive cases (e.g., disciplinary actions).
- **Automated Routing:** Using employee location and issue category to route cases to the correct regional HR team.
- **Sentiment Analysis:** Automatically flagging cases where an employee expresses high frustration for urgent review.
- **Standard Response Templates:** Providing pre-approved templates for common HR questions.
- **SLA Management:** Tracking and reporting on response times to ensure quality of service.
- **Redwood Employee Portal:** Providing a card-based interface for employees to track their inquiries on mobile.
