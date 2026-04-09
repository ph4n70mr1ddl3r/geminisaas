# HCM Journeys Service Specification

## Overview

The HCM Journeys service provides personalized, guided workflows for employees during key life and professional events (e.g., onboarding, role transfer, life events). This aligns with Oracle Fusion Cloud HCM Journeys and Oracle ME.

## Data Model (SQLite)

- `journey_templates`: id (UUID), name, status (ACTIVE/DRAFT), category (ONBOARDING/TRANSFER/LIFE_EVENT), estimated_duration_days.
- `journey_tasks`: id (UUID), template_id, task_name, task_type (FORM/DOCUMENT/VIDEO/URL/APPROVAL), sequence, owner_role.
- `employee_journeys`: id (UUID), template_id, employee_id, status (NOT_STARTED/IN_PROGRESS/COMPLETED), start_date, target_completion_date.
- `journey_task_instances`: id (UUID), journey_id, task_id, status (PENDING/COMPLETED), completion_date, assignee_id.
- `journey_comments`: id (UUID), journey_id, text, user_id, timestamp.
- `journey_notifications`: id (UUID), journey_id, task_id, notification_type (EMAIL/SMS/PUSH), status.

## API Endpoints (Axum/gRPC)

- `POST /journeys/assign`: Trigger a journey for an employee (e.g., after a life event).
- `GET /journeys/active/{employee_id}`: Retrieve all journeys currently in progress for an employee.
- `POST /journeys/task/complete`: Mark a specific task within a journey as finished.
- `GET /journeys/tasks/{journey_id}`: List all tasks and their current status for a specific journey.
- `POST /journeys/comments`: Add a comment or question to a journey.
- `GET /journeys/analytics/completion`: Retrieve completion rate metrics for a specific journey template.

## Inter-service Interaction

- Receives `Candidate_Hired` from `Recruiting` to automatically trigger an Onboarding journey.
- Queries `HCM Service` for employee role and department to personalize journey tasks.
- Emits `Journey_Completed` event to `HCM` to update employee records.
- Queries `Learning Service` to include training courses as tasks in a journey.
- Emits `Task_Overdue` alert to managers and administrators.

## Key Logic

- **Contextual Personalization:** Automatically showing or hiding tasks based on the employee's location, role, or country (e.g., specific tax forms for the US).
- **Conditional Task Flow:** Only enabling "Step 2" after "Step 1" has been completed and approved.
- **AI-Assisted Drafting:** Using AI to help managers draft personalized messages or feedback within a journey.
- **Social Connectivity:** Allowing employees to "check in" or "ask a question" directly to their onboarding buddy or manager within the journey.
- **Embedded Learning:** Providing training videos or documents directly within the task view.
- **Reminders & Escalations:** Automating follow-ups for tasks that are nearing their due dates.
