# Customer Data Management (CDM) Service Specification

## Overview

The Customer Data Management (CDM) service provides a single source of truth for customer and contact information. It handles data deduplication, enrichment, and address verification. This aligns with Oracle Fusion Cloud Customer Data Management and Oracle Unity CDP.

## Data Model (SQLite)

- `master_accounts`: id (UUID), source_system_id, source_id, unified_name, tax_id, status.
- `master_contacts`: id (UUID), account_id, first_name, last_name, email, phone, status.
- `data_enrichment_logs`: id (UUID), master_id, attribute_name, old_value, new_value, source (D&B/ZOOMINFO/MANUAL).
- `duplicate_pairs`: id (UUID), record_1_id, record_2_id, match_score, status (POTENTIAL/MERGED/REJECTED).
- `address_verification`: id (UUID), record_id, address_json, status (VERIFIED/FAILED), last_checked.
- `customer_360_view`: id (UUID), master_id, lifetime_value (DECIMAL), last_order_date, sentiment_score.

## API Endpoints (Axum/gRPC)

- `POST /cdm/deduplicate`: Run the match-and-merge engine to identify duplicate accounts.
- `GET /cdm/golden-record/{master_id}`: Retrieve the unified, enriched profile of a customer.
- `POST /cdm/enrich`: Trigger external data enrichment for a specific record.
- `GET /cdm/duplicates`: List potential duplicate pairs for manual review.
- `POST /cdm/verify-address`: Validate and standardize a customer's address.

## Inter-service Interaction

- Receives streaming updates from `CRM`, `Service`, and `Marketing` when customer data changes.
- Emits `Record_Merged` event to all subscriber services to update their local IDs.
- Queries `External Providers` (e.g., Dun & Bradstreet) for company hierarchy and financial data.
- Emits `Unified_Customer_Score` to `CRM` to assist sales reps in prioritization.

## Key Logic

- **Match-and-Merge Rules:** Using fuzzy logic and deterministic rules (e.g., matching on Tax ID or normalized Email) to identify duplicate records.
- **Survivorship Rules:** Determining which field value "wins" during a merge (e.g., "always trust CRM over Marketing for the phone number").
- **Address Standardization:** Parsing and formatting addresses to meet local postal standards (e.g., USPS, Royal Mail).
- **Hierarchical Linking:** Connecting subsidiary accounts to parent organizations to provide a "Family Tree" view of large customers.
- **Data Governance:** Tracking the lineage and source of every attribute in the golden record.
