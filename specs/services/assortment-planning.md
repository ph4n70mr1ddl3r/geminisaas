# Assortment Planning Service Specification

## Overview

The Assortment Planning service enables retail buyers to curate the optimal mix of products for specific store clusters and periods. It uses AI to suggest attributes, option counts, and "fit" based on localized demand and store capacity. This aligns with Oracle Retail Assortment Planning.

## Data Model (SQLite)

- `assortment_plans`: id (UUID), name, season_id, status (DRAFT/BASELINED), owner_id.
- `store_clusters`: id (UUID), name, criteria_json (JSON - e.g., 'High Income', 'Urban'), member_store_ids (JSON).
- `assortment_wedges`: id (UUID), plan_id, cluster_id, department_id, status.
- `assortment_items`: id (UUID), wedge_id, item_id, status (MANDATORY/OPTIONAL), planned_sales_qty, planned_receipt_qty.
- `attribute_recommendations`: id (UUID), item_id, cluster_id, recommended_color, recommended_size, confidence_score.
- `option_count_targets`: id (UUID), wedge_id, category_id, target_options_count, actual_options_count.

## API Endpoints (Axum/gRPC)

- `POST /retail-assortment/plans`: Create a new seasonal assortment plan.
- `POST /retail-assortment/clusters`: Define a new group of stores based on performance or demographic attributes.
- `GET /retail-assortment/recommend/attributes`: Retrieve AI-driven suggestions for item variations (e.g., "In NYC, stock more black than navy").
- `POST /retail-assortment/items/add`: Assign a product to a specific store cluster wedge.
- `GET /retail-assortment/fit-analysis`: Analyze how well the current assortment matches the cluster's demand profile.
- `POST /retail-assortment/baseline`: Lock the assortment plan and push it to execution systems.

## Inter-service Interaction

- Queries `Merchandise Financial Planning (MFP)` for top-down budget and sales targets.
- Emits `Assortment_Finalized` event to `Order Management` and `Procurement` to trigger purchasing.
- Queries `Retail Demand Forecasting (RDF)` for cluster-level demand signals.
- Queries `Inventory Planning Optimization (IPO)` for replenishment constraints.
- Emits `Assortment_Data` to `Visual Builder` for buyer-facing dashboards.

## Key Logic

- **Store Clustering:** Automatically grouping stores with similar selling patterns to simplify the planning process.
- **Option Count Optimization:** Balancing the breadth of the assortment (number of different items) against the depth (inventory per item).
- **Attribute Recommendations:** Using ML to identify which product features (e.g., "Sustainable Fabric") are trending in specific regions.
- **Wedge Management:** Providing a visual workspace for buyers to "fill" their assortment based on style, price point, and brand.
- **Capacity Constraint Check:** Ensuring the planned assortment actually fits within the physical shelf space or "fixture" limits of the target stores.
- **Reconciliation:** Ensuring the sum of the detailed assortment plans matches the high-level financial plan (MFP).
