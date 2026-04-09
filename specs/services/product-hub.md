# Product Hub (Master Data Management) Service Specification

## Overview

The Product Hub service provides a centralized repository for managing product master data, ensuring consistency across manufacturing, sales, and supply chain. This aligns with Oracle Fusion Cloud Product Hub.

## Data Model (SQLite)

- `product_master`: id (UUID), item_number, name, description, status (ACTIVE/INACTIVE/OBSOLETE), primary_uom, item_class_id.
- `item_classes`: id (UUID), name, parent_class_id, status.
- `item_attributes`: id (UUID), item_id, attribute_group, attribute_name, attribute_value, effective_date.
- `item_relationships`: id (UUID), from_item_id, to_item_id, relationship_type (MAPPING/SUBSTITUTE/COMPLEMENT).
- `item_categories`: id (UUID), item_id, category_set_id, category_id.
- `item_attachments`: id (UUID), item_id, file_name, file_url, type (IMAGE/DOCUMENT/SPEC).

## API Endpoints (Axum/gRPC)

- `POST /product-hub/items`: Create or update a product in the master registry.
- `GET /product-hub/items/{id}`: Retrieve full product details including attributes and relationships.
- `GET /product-hub/search`: Search products using keywords, categories, or attributes.
- `POST /product-hub/bulk-upload`: Ingest product data from external files.
- `GET /product-hub/audit-trail/{id}`: View the change history of a product.

## Inter-service Interaction

- Emits `Item_Created/Updated` event to `Inventory`, `Manufacturing`, and `Commerce` for synchronization.
- Queries `Inventory Service` for stock availability context.
- Emits `Product_Structure_Changed` to `PLM (Product Lifecycle Management)`.
- Queries `Supplier Portal` for vendor-provided product data.

## Key Logic

- **Attribute Inheritance:** Automatically inheriting attributes from parent item classes.
- **Data Validation:** Ensuring product data meets quality standards (e.g., mandatory fields, value ranges).
- **Style/Sizing Matrix:** Managing products with variations (e.g., Color/Size) efficiently.
- **Global Data Sync (GDSN):** Synchronizing product data with international retailers and standards.
- **Relationship Mapping:** Tracking which items are components, replacements, or complementary products.
