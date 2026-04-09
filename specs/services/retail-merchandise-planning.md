# Merchandise Financial Planning (Retail MFP) Service Specification

## Overview

The Merchandise Financial Planning (MFP) service provides strategic, high-level financial planning for retail organizations, covering sales, markdowns, receipts, and inventory. This aligns with Oracle Retail Merchandise Financial Planning.

## Data Model (SQLite)

- `merchandise_plans`: id (UUID), name, status (DRAFT/APPROVED), season (e.g., 'Spring 2026'), level (DEPT/CLASS), owner_id.
- `plan_versions`: id (UUID), plan_id, name (WORKING/APPROVED/LAST_YEAR), status.
- `mfp_metrics`: id (UUID), version_id, item_hierarchy_id, location_hierarchy_id, period_id, sales_amt, markdown_amt, receipt_amt, stock_amt, gross_margin_amt.
- `open_to_buy` (OTB): id (UUID), period_id, category_id, planned_receipts, committed_receipts, otb_remaining.
- `markdown_strategies`: id (UUID), name, threshold_rules (JSON), discount_pct, status.

## API Endpoints (Axum/gRPC)

- `POST /retail-planning/mfp/create`: Create a new seasonal merchandise financial plan.
- `GET /retail-planning/mfp/metrics`: Retrieve planned vs actual metrics for a specific product/location intersection.
- `POST /retail-planning/mfp/reconcile`: Sync high-level financial targets with bottom-up assortment plans.
- `GET /retail-planning/mfp/otb`: Calculate current Open-to-Buy availability for a merchant.
- `POST /retail-planning/mfp/markdown`: Run markdown simulation models to clear inventory.

## Inter-service Interaction

- Queries `Retail Management` for historical sales and inventory levels.
- Emits `Planned_Receipts_Finalized` event to `Procurement` to initiate bulk purchasing.
- Queries `EPM Service` (Planning) for top-down corporate financial targets.
- Emits `OTB_Updated` event to `Order Management` to constrain or enable new orders.
- Queries `Demand Forecasting` for AI-driven sales projections.

## Key Logic

- **Top-Down & Bottom-Up Reconciliation:** Aligning executive financial goals with the detailed plans of individual buyers.
- **In-Season Trend Adjustments:** Automatically re-forecasting the remainder of a season based on the first few weeks of actual sales.
- **Open-to-Buy (OTB) Control:** Dynamically calculating how much a buyer can spend based on current inventory, planned sales, and already-committed receipts.
- **Markdown Optimization:** Identifying slow-moving inventory early and suggesting price reductions that maximize gross margin.
- **WSSI (Weekly Stock Sales Intake):** Providing a weekly view of inventory health and intake requirements.
