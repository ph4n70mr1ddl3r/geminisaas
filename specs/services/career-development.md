# Career Development (Oracle Grow) Service Specification

## Overview

The Career Development service provides an AI-driven platform for employees to discover career paths, close skills gaps, and pursue growth opportunities. This aligns with Oracle Grow, part of the Oracle ME employee experience suite.

## Data Model (SQLite)

- `growth_profiles`: id (UUID), employee_id, current_skills (JSON), target_roles (JSON), growth_interest_areas (JSON).
- `career_paths`: id (UUID), from_role_id, to_role_id, average_tenure_years, common_skills_acquired.
- `skill_gap_analysis_grow`: id (UUID), employee_id, target_role_id, skills_missing (JSON), recommended_actions (JSON).
- `growth_activities`: id (UUID), employee_id, activity_type (LEARNING/GIG/MENTORSHIP), status (SUGGESTED/ACTIVE/COMPLETED), source_id.
- `career_conversations`: id (UUID), employee_id, manager_id, date, summary, status (DRAFT/FINAL).
- `role_aspirations`: id (UUID), employee_id, role_id, target_date, status.

## API Endpoints (Axum/gRPC)

- `GET /grow/my-path`: Retrieve AI-generated career path recommendations for the current user.
- `GET /grow/skill-gap/{role_id}`: Analyze the delta between current skills and a target role.
- `POST /grow/interests/update`: Update an employee's professional interests and growth goals.
- `GET /grow/recommendations`: Retrieve personalized learning and gig suggestions from the growth engine.
- `POST /grow/career-conversation`: Record a formal career discussion between an employee and manager.
- `GET /grow/analytics/skill-readiness`: View aggregated workforce readiness for future critical roles.

## Inter-service Interaction

- Queries `HCM Service` for core employee data, current job, and competency profile.
- Queries `Learning Service` for training courses that can close identified skill gaps.
- Queries `Opportunity Marketplace` for short-term gigs that match an employee's growth goals.
- Emits `Skill_Acquired` event to `HCM` talent profile.
- Queries `Succession Planning` to identify high-potential candidates for critical roles.

## Key Logic

- **AI-Powered Recommendations:** Using ML to suggest the "Next Best Move" based on what successful peers in similar roles have done.
- **Skill Centricity:** Moving beyond traditional job titles to focus on the underlying skills required for career progression.
- **Career Path Visualization:** Providing an interactive map showing different routes an employee can take to reach their long-term goals.
- **Aspiration Tracking:** Allowing HR to see which employees are interested in specific future roles to inform workforce planning.
- **Mentorship Matching:** Automatically suggesting potential mentors who possess the skills an employee is looking to develop.
- **Unified Growth Workspace:** Consolidating learning, gigs, and career planning into a single "Redwood" user experience.
