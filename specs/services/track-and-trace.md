# Intelligent Track & Trace (Blockchain) Service Specification

## Overview

The Intelligent Track and Trace service provides an immutable, decentralized ledger for tracking the end-to-end lifecycle of goods and assets across the entire supply chain. This aligns with Oracle Fusion Cloud Intelligent Track and Trace.

## Data Model (SQLite/Blockchain Proxy)

- `blockchain_ledgers`: id (UUID), network_id, status (ACTIVE/SYNCING), participants (JSON).
- `trace_events`: id (UUID), ledger_id, asset_id, event_type (PRODUCED/SHIPPED/RECEIVED/INSTALLED), timestamp, hash, signature, metadata (JSON).
- `tracked_assets`: id (UUID), asset_serial_number, lot_number, product_id, current_owner, genealogy_parent_id.
- `smart_contracts`: id (UUID), name, status (DRAFT/DEPLOYED), trigger_event, logic (JSON/Wasm), version.
- `blockchain_audit_trail`: id (UUID), asset_id, transaction_hash, block_number, timestamp.
- `track_exceptions`: id (UUID), asset_id, description, status (OPEN/RESOLVED), resolution_hash.

## API Endpoints (Axum/gRPC)

- `POST /track-trace/record-event`: Submit a new event to the blockchain ledger.
- `GET /track-trace/history/{asset_id}`: Retrieve the full immutable history and provenance of an asset.
- `GET /track-trace/lot-lineage/{lot_id}`: Trace the origin and distribution of a specific product lot.
- `POST /track-trace/contracts/deploy`: Deploy a new smart contract for automated settlement.
- `GET /track-trace/audit-log`: View the underlying blockchain transactions for a period.
- `POST /track-trace/verify-authenticity`: Validate that a digital record matches the blockchain hash.

## Inter-service Interaction

- Receives `Activity_Recorded` from `Manufacturing`, `Logistics`, and `Procurement` to update the ledger.
- Queries `Inventory Service` for serial number and lot ID details.
- Emits `Smart_Contract_Triggered` event for automated payments in `Financials`.
- Emits `Authenticity_Failed` alert to `Risk Management`.
- Queries `Maintenance Service` for asset history in the field.

## Key Logic

- **Immutable Proof of Origin:** Providing a cryptographically signed record of exactly when and where a product was manufactured.
- **Genealogy Tracking:** Maintaining a "Family Tree" of components (e.g., tracking which lot of batteries was used in which finished car).
- **Automated Settlement:** Executing smart contracts to pay a supplier automatically upon verified receipt of goods (Proof of Delivery).
- **Cold Chain Integrity:** Logging temperature and humidity sensor data from IoT devices directly into the blockchain.
- **Anti-Counterfeit:** Providing a public-facing portal to verify the authenticity of a high-value item based on its unique blockchain ID.
- **Targeted Recalls:** Instantly identifying all customers who received products containing a specific defective component lot.
