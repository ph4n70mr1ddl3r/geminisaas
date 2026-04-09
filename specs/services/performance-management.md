# Goal & Performance Management Service Specification

## Overview

The Goal & Performance Management service handles the lifecycle of employee performance tracking, including goal setting, continuous feedback, and annual reviews. This aligns with Oracle Fusion Cloud Performance Management.

## Data Model (SQLite)

- `performance_goals`: id (UUID), employee_id, description, category (BUSINESS/DEVELOPMENT), target_date, weight_pct, status (NOT_STARTED/IN_PROGRESS/COMPLETED), priority.
- `performance_reviews`: id (UUID), employee_id, manager_id, period_id, status (DRAFT/SUBMITTED/APPROVED/FINAL), overall_rating (1-5), comments.
- `continuous_feedback`: id (UUID), employee_id, sender_id, text, visibility (PUBLIC/PRIVATE), date.
- `performance_check_ins`: id (UUID), employee_id, manager_id, date, discussion_points (JSON), status.
- `performance_rating_models`: id (UUID), name, levels (JSON - e.g., 'Exceeds Expectations'), description.
- `competency_profiles`: id (UUID), employee_id, competency_name, level (1-5), last_assessed.

## API Endpoints (Axum/gRPC)

- `POST /performance/goals`: Create or update an employee goal.
- `GET /performance/goals/my`: Retrieve current goals for the logged-in employee.
- `POST /performance/reviews/submit`: Submit an annual or periodic performance evaluation.
- `POST /performance/feedback`: Send real-time feedback to a colleague or direct report.
- `POST /performance/check-in`: Record a 1-on-1 performance conversation.
- `GET /performance/analytics/rating-distribution`: View performance trends across a department.

## Inter-service Interaction

- Queries `HCM Service` for employee core data and reporting lines.
- Emits `Performance_Rating_Finalized` event to `Workforce Compensation` to inform merit increases.
- Emits `Skill_Gap_Identified` to `Learning Service` to suggest relevant training.
- Queries `Project Management` for project-based feedback or KPIs.
- Emits `Succession_Candidate_Flagged` to `Succession Planning`.

## Key Logic

- **Cascading Goals:** Allowing managers to share their goals with direct reports to ensure organizational alignment.
- **AI-Assisted Writing:** Using GenAI to help managers draft constructive feedback and review comments.
- **Normalization (Calibration):** Providing tools for senior leadership to ensure rating consistency across different managers.
- **360-Degree Feedback:** Collecting anonymous or named feedback from peers, subordinates, and external partners.
- **Development Plans:** Automatically linking performance review outcomes to personalized learning and development paths.
- **Continuous Performance:** Moving beyond annual reviews to support ongoing check-ins and real-time feedback loops.
