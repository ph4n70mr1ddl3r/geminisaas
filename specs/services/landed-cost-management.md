# Landed Cost Management (LCM) Service Specification

## Overview

The Landed Cost Management (LCM) service provides visibility into the true acquisition cost of an item by capturing and allocating charges such as freight, insurance, duties, and taxes. This aligns with Oracle Fusion Cloud Landed Cost Management.

## Data Model (SQLite)

- `trade_operations`: id (UUID), name, status (DRAFT/OPEN/CLOSED), start_date, end_date, total_estimated_cost.
- `landed_cost_charges`: id (UUID), trade_operation_id, charge_type (FREIGHT/INSURANCE/DUTY/TAX), estimated_amount, actual_amount, allocation_basis (VALUE/WEIGHT/VOLUME/QUANTITY).
- `charge_allocations`: id (UUID), charge_id, item_id, po_line_id, amount_allocated.
- `landed_cost_variances`: id (UUID), charge_id, estimated_amount, actual_amount, variance_amount, status (PENDING/POSTED).
- `trade_operation_items`: id (UUID), trade_operation_id, item_id, source_po_id, received_qty.

## API Endpoints (Axum/gRPC)

- `POST /landed-cost/trade-operations`: Create a new trade operation to track acquisition costs for a shipment.
- `POST /landed-cost/charges`: Add an estimated or actual charge to a trade operation.
- `POST /landed-cost/allocate`: Trigger the allocation engine to distribute charges across items.
- `GET /landed-cost/item-valuation/{item_id}`: Retrieve the full landed cost breakdown for a specific item.
- `POST /landed-cost/reconcile`: Match actual invoices from carriers/customs to estimated charges.
- `GET /landed-cost/analytics/variance`: Analyze the gap between estimated and actual acquisition costs.

## Inter-service Interaction

- Receives `Receipt_Confirmed` from `Procurement/Inventory` to trigger charge allocation.
- Queries `Financials` (AP) for actual invoices from third-party service providers (carriers, brokers).
- Emits `Item_Cost_Updated` event to `Inventory` and `Financials` (Cost Accounting) to update valuation.
- Queries `Logistics` for shipment weights and volumes used in allocation rules.
- Emits `Landed_Cost_Variance_Posted` to `Financials` (GL).

## Key Logic

- **Estimated vs. Actual Costing:** Allowing organizations to record item value based on estimates immediately upon receipt, then adjusting when actual bills arrive.
- **Allocation Engine:** Automatically distributing global charges (e.g., $5000 for a container) to individual items based on their relative value, weight, or volume.
- **Trade Operation Lifecycle:** Managing the period of time during which charges can be accumulated for a specific set of receipts.
- **Multi-Currency Support:** Handling charges in different currencies and translating them to the functional currency for allocation.
- **AI-Driven Estimation:** Using historical data to suggest more accurate estimated charges for future shipments.
- **Audit Traceability:** Providing a line-by-line breakdown of how every cent of landed cost was calculated and assigned.
