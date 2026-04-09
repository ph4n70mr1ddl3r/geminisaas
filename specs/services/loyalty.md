# Customer Loyalty & Engagement Service Specification

## Overview

The Customer Loyalty & Engagement service manages points, rewards, membership tiers, and personalized engagement activities to drive customer retention. This aligns with Oracle Fusion Cloud CrowdTwist and CX Loyalty.

## Data Model (SQLite)

- `loyalty_programs`: id (UUID), name, status (ACTIVE/DRAFT), currency_name (e.g., Points/Miles), spend_to_point_ratio.
- `loyalty_tiers`: id (UUID), program_id, name (GOLD/SILVER/BRONZE), min_points, max_points, benefits (JSON).
- `loyalty_members`: id (UUID), customer_id (link to CDM), program_id, current_tier_id, total_points, lifetime_points, status.
- `loyalty_transactions`: id (UUID), member_id, type (EARN/REDEEM/EXPIRED), amount, description, source_order_id.
- `loyalty_rewards`: id (UUID), program_id, name, points_cost, quantity_available, type (COUPON/PHYSICAL/EXPERIENCE).
- `engagement_activities`: id (UUID), member_id, type (SURVEY/REVIEW/SOCIAL_SHARE), points_awarded, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /loyalty/enroll`: Register a customer into a loyalty program.
- `GET /loyalty/member/{customer_id}`: Retrieve a member's points balance and tier status.
- `POST /loyalty/earn`: Record a point-earning activity (e.g., after a purchase).
- `POST /loyalty/redeem`: Record a point-redemption activity for a reward.
- `GET /loyalty/rewards/available`: List available rewards for a specific member.
- `POST /loyalty/activities/record`: Record a non-transactional engagement activity (e.g., survey).

## Inter-service Interaction

- Receives `Order_Fulfilled` from `Commerce/Order Management` to trigger point earning.
- Emits `Tier_Changed` event to `Marketing` to update campaign segmentation.
- Queries `CDM Service` for customer profile data for personalized rewards.
- Emits `Reward_Redeemed` to `Order Management` for physical reward fulfillment.
- Queries `Financials` for tax implications of high-value rewards.

## Key Logic

- **Tier Progression:** Automatically promoting or demoting members between tiers based on their 12-month rolling point total.
- **Points Expiration:** Automatically expiring points after a certain period of inactivity (e.g., 24 months).
- **Fraud Detection:** Identifying and flagging unusual point-earning patterns (e.g., too many transactions in a short time).
- **Gamification:** Awarding badges or bonus points for completing "challenges" (e.g., "Review 3 products").
- **Individualized Rewards:** Using AI and CDP data to suggest rewards that a specific member is most likely to find valuable.
