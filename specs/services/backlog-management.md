# Backlog Management Service Specification

## Overview

The Backlog Management service provides advanced capabilities for prioritizing, rescheduling, and re-promising open orders when supply or demand changes. It enables organizations to maximize fulfillment efficiency and revenue while meeting customer commitments. This aligns with Oracle Fusion Cloud Backlog Management (SCM).

## Data Model (SQLite)

- `backlog_orders`: id (UUID), order_line_id (link to Order Mgmt), current_promised_date, requested_date, customer_priority, revenue_value.
- `backlog_plans`: id (UUID), name, status (DRAFT/EXECUTING/RELEASED), total_orders_included, improvement_pct.
- `priority_rules`: id (UUID), name, criteria (JSON - e.g., 'Customer Tier = Gold', 'Margin > 20%'), weight_pct.
- `rescheduling_results`: id (UUID), plan_id, order_line_id, old_promised_date, new_promised_date, fulfillment_status.
- `backlog_constraints`: id (UUID), type (SUPPLY/CAPACITY/LOGISTICS), description, impact_on_delay_days.

## API Endpoints (Axum/gRPC)

- `POST /backlog-management/plans`: Create a new backlog management plan for a set of orders and supply constraints.
- `POST /backlog-management/prioritize`: Run the ranking engine to score orders based on business rules.
- `POST /backlog-management/solve`: Execute the rescheduling algorithm to find the optimal set of new promised dates.
- `GET /backlog-management/analytics/impact`: Compare the new plan against the current backlog status (e.g., revenue gain, delay reduction).
- `POST /backlog-management/release`: Finalize the plan and update the promised dates in the order management system.

## Inter-service Interaction

- Queries `Order Management` for open sales orders and their current status.
- Queries `GOP (Global Order Promising)` for real-time availability and transit times.
- Emits `Backlog_Updated` event to `Order Management` to communicate changes to customers.
- Queries `Supply Chain Planning` for future supply availability (PO/Work Orders).
- Emits `Expedite_Request` to `Procurement` or `Manufacturing` for critical delayed orders.

## Key Logic

- **Business-Priority Ranking:** Assigning scores to orders based on strategic importance (e.g., "Always prioritize medical orders").
- **Supply-Demand Matching:** Automatically re-allocating limited supply from low-priority to high-priority orders.
- **Revenue Maximization:** Identifying rescheduling options that allow for earlier revenue recognition in the current period.
- **Fairness Rules:** Ensuring that orders aren't delayed indefinitely (e.g., "Don't delay an order more than twice").
- **Mass Re-Promising:** Updating thousands of order lines in a single automated run after a major supply disruption (e.g., port strike).
- **Attribute-Based Selection:** Filtering the backlog by product category, customer, or geographical region for focused rescheduling.
