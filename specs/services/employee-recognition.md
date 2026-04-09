# Employee Recognition (Oracle Celebrate) Service Specification

## Overview

The Employee Recognition service provides a platform for peers and managers to recognize and reward employees for their contributions and behaviors. This aligns with Oracle Celebrate, part of the Oracle ME employee experience suite.

## Data Model (SQLite)

- `recognition_events`: id (UUID), sender_id, receiver_id, message, recognition_type (PEER/MANAGER/SOCIAL), status (DRAFT/SENT), points_awarded (optional).
- `recognition_awards`: id (UUID), event_id, award_value, award_currency (POINTS/CASH), status (PENDING/PAID).
- `recognition_badges`: id (UUID), name, icon_url, criteria_description, status.
- `employee_badges`: id (UUID), employee_id, badge_id, date_received, event_id.
- `recognition_programs`: id (UUID), name, budget_total, budget_remaining, start_date, end_date, active (BOOL).
- `social_recognition_feed`: id (UUID), event_id, visibility (TEAM/COMPANY/PUBLIC), likes_count, comments_count.

## API Endpoints (Axum/gRPC)

- `POST /celebrate/recognize`: Send a recognition message and optional award to a colleague.
- `GET /celebrate/my-recognition`: Retrieve recognition received and sent by the current user.
- `GET /celebrate/social-feed`: Retrieve a live feed of public recognition events.
- `POST /celebrate/award/redeem`: Transition a point-based award into a physical or financial reward.
- `GET /celebrate/programs`: List active recognition programs and remaining budgets.
- `POST /celebrate/like-comment`: Interact with a recognition event on the social feed.

## Inter-service Interaction

- Queries `HCM Service` for employee data, team relationships, and department hierarchy.
- Emits `Award_Paid` event to `Global Payroll` for cash-based recognition.
- Queries `Identity Service` for user profile photos and titles.
- Emits `High_Performance_Signal` to `Goal & Performance Management`.
- Queries `Financials` for department budget availability for recognition programs.

## Key Logic

- **GenAI Messaging Assistant:** Helping employees draft impactful recognition notes based on the context of the achievement.
- **Budget Pooling:** Allowing managers to manage a recognition budget for their team, ensuring fair distribution.
- **Point-to-Cash Conversion:** Automatically calculating the tax impact and payroll entries for financial rewards.
- **Milestone Automation:** Automatically triggering recognition for work anniversaries, birthdays, or project completions.
- **Social Engagement:** Creating a culture of appreciation through likes, comments, and visibility controls.
- **Program Governance:** Enforcing rules on how often an employee can be recognized or how many points can be sent.
