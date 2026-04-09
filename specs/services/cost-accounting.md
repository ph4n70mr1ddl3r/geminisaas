# Cost Accounting Service Specification

## Overview

The Cost Accounting service provides a centralized engine for valuing inventory, calculating cost of goods sold (COGS), and analyzing manufacturing variances. It supports multiple costing methods (Standard, Perpetual Average, Actual, FIFO) and provides deep integration with the General Ledger. This aligns with Oracle Fusion Cloud Cost Accounting.

## Data Model (SQLite)

- `cost_orgs`: id (UUID), name, status, set_of_books_id.
- `cost_books`: id (UUID), cost_org_id, name, costing_method (STANDARD/AVG/FIFO), status.
- `inventory_item_costs`: id (UUID), cost_book_id, item_id, unit_cost, effective_date, status.
- `cost_elements`: id (UUID), name, type (MATERIAL/OVERHEAD/RESOURCE/PROFIT), description.
- `cost_distributions`: id (UUID), transaction_id (link to Inventory/Mfg), cost_book_id, gl_account_id, amount, debit_credit_flag.
- `manufacturing_variances`: id (UUID), work_order_id, type (USAGE/EFFICIENCY/RATE), amount, status (PENDING/POSTED).

## API Endpoints (Axum/gRPC)

- `POST /costing/calculate`: Execute the cost processor for a set of inventory or manufacturing transactions.
- `GET /costing/item-cost/{id}`: Retrieve the current unit cost for an item within a specific cost book.
- `POST /costing/create-distributions`: Generate accounting entries for costed transactions.
- `GET /costing/valuation-report`: Retrieve the total inventory value for a cost organization and period.
- `POST /costing/variance/analyze`: Identify and categorize manufacturing or purchase price variances.

## Inter-service Interaction

- Receives `Inventory_Transaction` from `Inventory Service` to trigger valuation.
- Receives `Work_Order_Closed` from `Manufacturing` to calculate final production costs and variances.
- Emits `Cost_Journal_Entries` to `Financials` (GL).
- Queries `Receipt Accounting` for landed cost adjustments and accruals.
- Emits `Item_Cost_Updated` to `Supply Chain Planning` for margin-based optimization.

## Key Logic

- **Multi-Book Costing:** Simultaneously maintaining different costs for the same item (e.g., one for Corporate GAAP and one for Local Tax).
- **Cost Component Granularity:** Breaking down an item's cost into Material, Labor, and Overhead for detailed margin analysis.
- **Overhead Absorption:** Automatically applying overhead costs to items or work orders based on absorption rules (e.g., % of labor cost).
- **Variance Identification:** Automatically flagging differences between actual production costs and "Standard" costs.
- **In-Transit Valuation:** Tracking the value of goods that have been shipped but not yet received by the target location.
- **Profit in Inventory (PII) Elimination:** Identifying and tracking internal profit added during intercompany transfers to ensure it is eliminated during consolidation.
