# Promotion Optimization Service Specification

## Overview

The Promotion Optimization service manages the full lifecycle of marketing and price-based promotions. It uses AI to predict promotional "uplift," analyze cross-elasticity, and suggest targeted markdowns to maximize gross margin. This aligns with Oracle Retail Promotion Planning and Optimization.

## Data Model (SQLite)

- `promotion_offers`: id (UUID), name, status (DRAFT/ACTIVE/EXPIRED), type (BOGO/DISCOUNT/COUPON), start_date, end_date.
- `offer_components`: id (UUID), offer_id, item_id, discount_value, adjustment_type (PERCENT/AMOUNT).
- `uplift_forecasts`: id (UUID), offer_id, item_id, forecasted_lift_pct, confidence_score, date.
- `promotion_budgets`: id (UUID), name, total_allocated, total_spent, department_id.
- `cross_elasticity_links`: id (UUID), item_a_id, item_b_id, correlation_factor (e.g., 'If A is on sale, B sales drop by 10%').
- `promotion_performance`: id (UUID), offer_id, actual_sales_qty, incremental_revenue, incremental_margin.

## API Endpoints (Axum/gRPC)

- `POST /retail-promotion/offers`: Create a new promotional campaign.
- `POST /retail-promotion/simulate-uplift`: Trigger the AI engine to predict sales gain for a proposed offer.
- `GET /retail-promotion/recommend/markdowns`: Retrieve AI-suggested price reductions for slow-moving items.
- `POST /retail-promotion/budget/allocate`: Reserve funds for a marketing campaign.
- `GET /retail-promotion/analytics/roi`: Retrieve return-on-investment metrics for active or past promotions.
- `POST /retail-promotion/activate`: Push an approved promotion to Point-of-Sale (POS) and E-commerce systems.

## Inter-service Interaction

- Queries `Retail Demand Forecasting (RDF)` for baseline demand (non-promoted) data.
- Emits `Price_Update_Required` to `Retail Management` and `Commerce` to apply discounts.
- Queries `Marketing` for campaign creative and audience segment data.
- Emits `Stock_Alert_Promo` to `Supply Chain Planning` to ensure enough inventory is on hand for the expected uplift.
- Queries `Loyalty Service` for member-exclusive personalized offers.

## Key Logic

- **Uplift Modeling:** Distinguishing between "natural" demand and the additional demand created specifically by the promotion.
- **Cannibalization Analysis:** Identifying if a promotion on a new model is reducing sales of an older, higher-margin model.
- **Halo Effect Tracking:** Measuring if a promotion on "Pasta" increases sales of non-promoted "Pasta Sauce."
- **Markdown Timing Optimization:** Predicting the "sweet spot" for price reductions—low enough to clear stock but high enough to protect margin.
- **Budget Guardrails:** Automatically blocking promotions that would exceed the department's marketing budget.
- **Affinity Grouping:** Suggesting "Frequently Bought Together" bundles for promotional offers.
