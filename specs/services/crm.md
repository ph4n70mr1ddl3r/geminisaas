# CRM Service Specification

## Overview

The CRM (Customer Relationship Management) service manages customers, leads, opportunities, sales orders, and customer service requests.

## Data Model (SQLite)

- `customers`: id (UUID), name, tax_id, address, contact_email.
- `leads`: id (UUID), name, status (NEW/QUALIFIED/CONVERTED), description.
- `opportunities`: id (UUID), customer_id, value (DECIMAL), close_date, status.
- `sales_orders`: id (UUID), customer_id, date, status (DRAFT/PLACED/FULFILLED/CANCELLED), total_amount.
- `order_items`: id (UUID), sales_order_id, item_id (link to Inventory), quantity, price.
- `service_requests`: id (UUID), customer_id, issue_description, priority, status.

## API Endpoints (Axum/gRPC)

- `POST /crm/customers`: Create/View customer profile.
- `POST /crm/leads`: Record/Track potential sales.
- `POST /crm/orders`: Create/Process a sales order.
- `GET /crm/orders`: List/View sales orders.
- `POST /crm/service`: Submit a customer support request.

## Inter-service Interaction

- Queries `Inventory Service` for stock availability before order placement.
- Emits `Order_Placed` event to `Inventory` for stock reservation.
- Emits `Order_Fulfilled` event to `Financials` for invoice generation (Accounts Receivable).
- Queries `Financials` for customer credit limits before approving large orders.

## Key Logic

- **Credit Check:** Automate credit check for orders based on past payment behavior.
- **Conversion Workflow:** Lead -> Opportunity -> Quote -> Order.
- **Support SLAs:** Track response/resolution time for service requests.
