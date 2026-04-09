# HCM Engagement (Touchpoints & Connections) Service Specification

## Overview

The HCM Engagement service manages employee continuous listening, sentiment analysis, and social networking within the organization. This aligns with Oracle Touchpoints and Oracle Connections (part of the Oracle ME suite).

## Data Model (SQLite)

- `employee_touchpoints`: id (UUID), employee_id, manager_id, status (ACTIVE/ARCHIVED), last_interaction_date.
- `engagement_checkins`: id (UUID), touchpoint_id, date, sentiment_score (1-5), comments (JSON), status (SCHEDULED/COMPLETED).
- `continuous_listening_surveys`: id (UUID), name, target_audience_id, status, results_summary (JSON).
- `social_connections_profiles`: id (UUID), employee_id, bio, skills_tags (JSON), interests_tags (JSON), endorsements_count.
- `connection_graph`: id (UUID), follower_id, following_id, interaction_type (FOLLOW/TEAM/COLLABORATE).
- `sentiment_trends_team`: id (UUID), manager_id, period_id, avg_sentiment, attrition_risk_signal.

## API Endpoints (Axum/gRPC)

- `POST /hcm-engagement/check-in`: Record a 1-on-1 check-in with sentiment data.
- `GET /hcm-engagement/pulse-survey`: Retrieve active engagement surveys for an employee.
- `GET /hcm-engagement/connections/search`: Search for colleagues based on skills or interests.
- `POST /hcm-engagement/endorse`: Give a skill endorsement to a colleague.
- `GET /hcm-engagement/team-health`: Retrieve aggregated sentiment and engagement metrics for a manager's team.
- `POST /hcm-engagement/profile/update`: Update an employee's social profile and "about me" content.

## Inter-service Interaction

- Queries `HCM Service` for core employee data and manager hierarchy.
- Emits `Sentiment_Alert` to HR if a team's engagement score drops significantly.
- Queries `Oracle Grow` for skills data to display on social profiles.
- Emits `Activity_Signal` to `Digital Assistant` to suggest timely check-ins.
- Receives `Recognition_Event` from `Oracle Celebrate` to update social profiles.

## Key Logic

- **Continuous Listening:** Automatically triggering pulse surveys during key journey milestones or periods of high stress.
- **Sentiment Analysis:** Using NLP to identify trends in check-in comments and survey responses (anonymized at scale).
- **Skill Discovery:** Helping employees find experts within the company through a tag-based social graph.
- **Engagement Scoring:** Calculating a real-time engagement index for each employee based on participation in social and feedback activities.
- **Networking Recommendations:** Suggesting "People you should know" based on common interests or project history.
- **Privacy Controls:** Ensuring employees can control what social information is shared with whom.
