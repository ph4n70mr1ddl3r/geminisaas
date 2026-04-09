# Project Execution Management Service Specification

## Overview

The Project Execution Management (PEM) service handles the day-to-day execution of project tasks, issue tracking, and team collaboration. This aligns with Oracle Fusion Cloud Project Execution Management, distinct from Project Financials.

## Data Model (SQLite)

- `project_plans`: id (UUID), project_id (link to Project Mgmt), status (DRAFT/ACTIVE/ARCHIVED), version.
- `project_tasks_execution`: id (UUID), project_id, name, start_date, end_date, duration_days, assigned_to_id, status (NOT_STARTED/IN_PROGRESS/COMPLETED/BLOCKED), pct_complete, parent_task_id, sequence.
- `project_issues`: id (UUID), project_id, title, description, severity (LOW/MED/HIGH), reporter_id, assignee_id, status (OPEN/IN_PROGRESS/CLOSED).
- `project_deliverables`: id (UUID), project_id, name, due_date, status (PENDING/DELIVERED), file_url (optional).
- `project_task_dependencies`: id (UUID), from_task_id, to_task_id, dependency_type (FS/SS/FF/SF), lag_days.
- `project_collaboration_messages`: id (UUID), project_id, task_id (optional), user_id, message, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /project-execution/tasks`: Create or update tasks in a project plan.
- `GET /project-execution/tasks/my-tasks`: Retrieve tasks assigned to the current user.
- `POST /project-execution/tasks/{id}/status`: Update the progress or status of a specific task.
- `POST /project-execution/issues`: Create a new project issue.
- `GET /project-execution/gantt/{project_id}`: Retrieve the full project timeline for Gantt chart visualization.
- `POST /project-execution/deliverables/submit`: Record the delivery of a project output.

## Inter-service Interaction

- Queries `Project Management Service` (Financials) for project metadata and core budget constraints.
- Emits `Task_Completed` event to `Project Management` to update percentage-of-completion revenue recognition.
- Queries `Resource Management` for resource availability and skill profiles.
- Emits `Issue_Escalated` alert to project managers and sponsors.
- Queries `Identity Service` for project team roles and permissions.

## Key Logic

- **Critical Path Method (CPM):** Automatically calculating the project end date based on task durations and dependencies.
- **Dependency Handling:** Managing Finish-to-Start (FS), Start-to-Start (SS), and other complex relationship types.
- **Progress Rollevup:** Automatically calculating the percentage completion of a project based on individual task progress.
- **Issue Resolution Workflow:** Routing issues through a lifecycle from identification to remediation.
- **Resource Leveling:** Identifying and flagging over-allocated resources across multiple tasks.
- **Interactive Gantt Logic:** Providing the backend data structure for drag-and-drop timeline adjustments.
