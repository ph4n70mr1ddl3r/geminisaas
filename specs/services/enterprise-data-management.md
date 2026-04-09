# Enterprise Data Management (EDM) Service Specification

## Overview

The Enterprise Data Management (EDM) service provides a centralized governance hub for managing master data across heterogeneous environments, including finance, product, and location domains. This aligns with Oracle Fusion Cloud Enterprise Data Management.

## Data Model (SQLite)

- `edm_applications`: id (UUID), name, type (FINANCE/PRODUCT/CUSTOMER/CUSTOM), status (ACTIVE/ARCHIVED), connection_details (JSON).
- `edm_data_chains`: id (UUID), application_id, dimension_id, node_type_id, hierarchy_set_id.
- `edm_nodes`: id (UUID), node_type_id, name, description, parent_node_id, status (ACTIVE/INACTIVE).
- `edm_requests`: id (UUID), user_id, status (DRAFT/SUBMITTED/APPROVED/COMMITTED), request_type (ADD/UPDATE/DELETE/MOVE), effective_date.
- `edm_request_items`: id (UUID), request_id, node_id, property_name, old_value, new_value.
- `edm_subscriptions`: id (UUID), source_dimension_id, target_dimension_id, auto_submit_status, transformation_rules (JSON).

## API Endpoints (Axum/gRPC)

- `POST /edm/requests`: Create a new data governance request for master data changes.
- `GET /edm/nodes/search`: Search for master data nodes across all applications and dimensions.
- `POST /edm/validate`: Run business logic validation on a pending request.
- `POST /edm/commit`: Finalize and push approved master data changes to downstream applications.
- `GET /edm/hierarchy-view/{dim_id}`: Retrieve a tree-based view of a specific hierarchy.
- `POST /edm/sync-subscription`: Manually trigger a data synchronization between dimensions.

## Inter-service Interaction

- Emits `Master_Data_Updated` event to `Financials`, `Procurement`, and `HCM` when a request is committed.
- Queries `Identity Service` for hierarchical approval workflows.
- Receives `Metadata_Import_Requested` from `EPM Service` to align charts of accounts.
- Queries `Product Hub` for product domain node definitions.

## Key Logic

- **Hierarchy Management:** Supporting complex, multi-parent, and versioned hierarchies for financial reporting and organizational structures.
- **Change Visualization:** Providing a side-by-side comparison of "Before" and "After" states for any data request.
- **Subscription Engine:** Automatically propagating changes made in a source dimension to all subscribed target dimensions (e.g., updating a cost center in ERP and EPM simultaneously).
- **Business Rules & Validations:** Enforcing data integrity (e.g., "Account code must be 6 digits") before a request can be submitted.
- **Audit Trail:** Maintaining a complete, immutable history of every change, who requested it, and who approved it.
