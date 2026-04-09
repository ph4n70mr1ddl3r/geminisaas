# Product Lifecycle Management Service Specification

## Overview

The Product Lifecycle Management (PLM) service, often referred to as Product Hub, serves as the master repository for all product data. It manages item definitions, revisions, and product development lifecycles. This aligns with Oracle Fusion Cloud Product Hub and PLM.

## Data Model (SQLite)

- `items`: id (UUID), name, description, category_id, status (PRE-RELEASE/ACTIVE/OBSOLETE), version (e.g., 'A.1').
- `item_classes`: id (UUID), name, parent_id, attribute_definitions (JSON).
- `item_revisions`: id (UUID), item_id, revision_code, effective_date, change_order_id.
- `change_orders` (ECO): id (UUID), title, priority, status (OPEN/REVIEW/APPROVED/IMPLEMENTED), requester_id.
- `product_structures`: id (UUID), parent_item_id, component_item_id, quantity, optional (BOOL).

## API Endpoints (Axum/gRPC)

- `POST /plm/items`: Create a new master item with version control.
- `POST /plm/items/{id}/revise`: Create a new revision of an existing item.
- `POST /plm/change-orders`: Initiate an Engineering Change Order (ECO).
- `GET /plm/items/{id}/bom-explosion`: View the complete product structure (multi-level).
- `POST /plm/items/categorize`: Assign items to hierarchical product categories.

## Inter-service Interaction

- Acts as the source of truth for `Inventory`, `Manufacturing`, and `Order Management` for item data.
- Receives `Defect_Reported` events from `Quality Management` which may trigger an ECO.
- Emits `Item_Updated` events to sync master data across all consuming services.

## Key Logic

- **Version Control:** Ensuring that production only uses 'Active' and 'Approved' revisions of an item.
- **Inheritance:** Item classes can define attributes that are inherited by all child items (e.g., all 'Electronics' have a 'Voltage' attribute).
- **Impact Analysis:** Automatically identifying which assemblies are affected when a low-level component is changed.
- **Workflow Approvals:** Multi-step approval process for change orders, involving Engineering, Manufacturing, and Quality.
