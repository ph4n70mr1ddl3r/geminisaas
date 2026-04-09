# Procurement Service Specification

## Overview

The Procurement service manages the purchasing lifecycle, including requisitions, purchase orders, vendors, and receiving.

## Data Model (SQLite)

- `vendors`: id (UUID), name, tax_id, address.
- `requisitions`: id (UUID), requester_id (link to Identity), status (PENDING/APPROVED/REJECTED), description.
- `requisition_items`: id (UUID), requisition_id, item_id (link to Inventory), quantity, estimated_price.
- `purchase_orders`: id (UUID), vendor_id, status (DRAFT/SENT/CLOSED), total_amount.
- `po_items`: id (UUID), po_id, requisition_item_id, item_id, quantity, unit_price.
- `receipts`: id (UUID), po_id, date, received_quantity.

## API Endpoints (Axum/gRPC)

- `POST /procurement/requisitions`: Submit a new purchase requisition.
- `GET /procurement/requisitions`: List/View requisitions.
- `POST /procurement/pos`: Create/Approve a purchase order from a requisition.
- `POST /procurement/receipts`: Record the receipt of items.
- `GET /procurement/vendors`: Manage vendor profiles.

## Inter-service Interaction

- Queries `Inventory Service` for item availability/codes.
- Emits `PO_Created` and `Receipt_Recorded` events.
- Emits `Invoice_Required` event to `Financials` when a receipt is matched.
- Queries `Identity Service` for requisition approval workflows.

## Key Logic

- **Approval Workflow:** Requisitions above a certain amount require multiple approvals (configurable).
- **Three-Way Match:** Matching PO, Receipt, and Invoice before payment is authorized.
- **Vendor Rating:** Automated or manual rating based on delivery times and quality.
