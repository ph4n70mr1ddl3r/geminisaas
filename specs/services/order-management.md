# Order Management Service Specification

## Overview

The Order Management service (part of SCM) handles sales orders, pricing, credit checks, and coordination of fulfillment through Inventory and Shipping. This aligns with Oracle Fusion Cloud Order Management.

## Data Model (SQLite)

- `sales_orders`: id (UUID), customer_id (link to CRM), order_date, total_amount, status (DRAFT/OPEN/FULFILLED/CANCELLED).
- `order_lines`: id (UUID), order_id, item_id (link to Inventory), quantity, unit_price, discount, status.
- `price_lists`: id (UUID), name, currency, effective_start, effective_end.
- `price_list_items`: id (UUID), price_list_id, item_id, price.
- `shipments`: id (UUID), order_id, status (PENDING/SHIPPED/DELIVERED), carrier, tracking_number.
- `customers`: id (UUID), name, account_number, credit_limit (synced with CRM/Financials).

## API Endpoints (Axum/gRPC)

- `POST /orders`: Submit a new sales order.
- `GET /orders/{id}`: View order status and details.
- `POST /orders/{id}/calculate-price`: Invoke pricing engine based on customer and quantity.
- `GET /customers/{id}/credit-check`: Verify customer credit before order approval.
- `POST /shipments`: Record a shipment for an order.

## Inter-service Interaction

- Queries `CRM Service` for customer master data.
- Queries `Inventory Service` for item availability (ATP - Available to Promise).
- Emits `Order_Placed` to reserve stock in `Inventory Service`.
- Emits `Order_Fulfilled` to `Financials` to trigger AR invoice creation.
- Emits `Shipping_Notice` to the customer.

## Key Logic

- **Pricing Engine:** Multi-tiered pricing based on customer segments, volume discounts, and promotions.
- **Credit Check:** Integrating with AR in `Financials` to prevent order fulfillment for delinquent accounts.
- **ATP (Available to Promise):** Real-time calculation of when an item can be delivered based on current stock and incoming receipts.
- **Orchestration:** Coordinating multiple fulfillment steps (e.g., ship, bill, install).
