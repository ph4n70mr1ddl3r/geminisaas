# Accounting Hub Service Specification

## Overview

The Accounting Hub service provides a high-performance engine for ingesting, transforming, and accounting for high-volume transactions from non-Oracle source systems. It enables organizations to centralize accounting rules and create a "Universal Journal" across all applications. This aligns with Oracle Fusion Cloud Accounting Hub (ERP).

## Data Model (SQLite)

- `source_systems_external`: id (UUID), name (e.g., 'Legacy Billing'), description, status (ACTIVE/INACTIVE).
- `accounting_event_types`: id (UUID), system_id, name (e.g., 'Loan Disbursement'), description.
- `accounting_rules_hub`: id (UUID), event_type_id, rule_name, debit_account_logic (JSON), credit_account_id_logic (JSON), amount_logic (JSON).
- `accounting_event_headers`: id (UUID), system_id, source_id, event_type_id, status (UNPROCESSED/PROCESSED/ERROR), event_date.
- `accounting_event_lines`: id (UUID), header_id, line_type, amount, currency, source_attributes (JSON).
- `hub_journal_entries`: id (UUID), header_id, gl_ledger_id, status (DRAFT/POSTED), total_debit, total_credit.

## API Endpoints (Axum/gRPC)

- `POST /accounting-hub/ingest`: Ingest a batch of transaction events from an external source system.
- `POST /accounting-hub/create-accounting`: Execute the rules engine to transform source events into double-entry accounting.
- `GET /accounting-hub/events/search`: Search for specific source events and their processing status.
- `GET /accounting-hub/drill-down/{journal_id}`: Retrieve the original source system attributes for a hub journal entry.
- `POST /accounting-hub/rules/update`: Define or modify the logic for mapping source attributes to GL accounts.
- `GET /accounting-hub/analytics/processing-volume`: View ingestion and accounting metrics by source system.

## Inter-service Interaction

- Emits `Journal_Entries_Finalized` event to `Financials` (GL) for master ledger recording.
- Queries `Financials` for master Chart of Accounts and Ledger definitions.
- Emits `Transformation_Error_Alert` to the accounting team if a rule fails.
- Queries `Identity Service` for data governance and audit permissions.
- Receives `Metadata_Update` from `Enterprise Data Management (EDM)` to align account mappings.

## Key Logic

- **Attribute Transformation:** Mapping raw source data (e.g., 'Branch 101') to target GL segments (e.g., 'Company 01, Dept 500').
- **Accounting Event Lifecycle:** Managing the flow from "Event Received" to "Accounted" to "Posted to GL."
- **High-Scale Ingestion:** Optimized for processing millions of rows from external files or APIs in a single batch.
- **Rules Customization:** Providing a declarative interface for accountants to define complex conditional logic (e.g., "If LoanType = Mortgage, use Account 1234").
- **Audit Traceability (Drill-Down):** Maintaining a permanent link between the final GL entry and every attribute of the original source system transaction.
- **Multicurrency Translation:** Automatically translating source amounts into the ledger's functional and reporting currencies.
