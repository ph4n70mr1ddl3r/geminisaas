# Enterprise Contracts Service Specification

## Overview

The Enterprise Contracts service provides a centralized repository for managing all types of legal agreements, including procurement contracts, sales contracts, and project contracts. This aligns with Oracle Fusion Cloud Enterprise Contracts.

## Data Model (SQLite)

- `contracts`: id (UUID), name, number, version, type (PURCHASE/SALES/PROJECT), status (DRAFT/ACTIVE/EXPIRED/TERMINATED), start_date, end_date, currency, total_value, party_id (link to Vendor or Customer).
- `contract_terms`: id (UUID), contract_id, clause_id, content_override (optional), sequence.
- `clause_library`: id (UUID), title, text, version, type (STANDARD/NON_STANDARD), status (APPROVED/DRAFT).
- `contract_approvals`: id (UUID), contract_id, approver_id, status (PENDING/APPROVED/REJECTED), comments, timestamp.
- `contract_amendments`: id (UUID), contract_id, description, change_date, previous_version_id.
- `contract_deliverables`: id (UUID), contract_id, name, due_date, status (PENDING/COMPLETED), responsibility (INTERNAL/EXTERNAL).

## API Endpoints (Axum/gRPC)

- `POST /contracts`: Create a new contract from a template.
- `GET /contracts/{id}/terms`: View the clauses and legal language.
- `POST /contracts/{id}/amend`: Initiate a contract amendment.
- `POST /contracts/{id}/approve`: Record an approval step.
- `GET /contracts/clause-library`: Search for standard legal clauses.
- `GET /contracts/expiring`: List contracts nearing their end date for renewal.

## Inter-service Interaction

- Receives `PO_Created` from `Procurement` to link to a master purchase agreement.
- Emits `Contract_Activated` to `Project Management` to enable billing against a project contract.
- Queries `Identity Service` for legal counsel and signing authority approvals.
- Emits `Contract_Renewal_Alert` to `CRM` and `Procurement` for upcoming expirations.

## Key Logic

- **Clause Library:** Maintaining a version-controlled repository of approved legal language.
- **Contract Templates:** Pre-defining common contract structures (e.g., MSA, SOW, NDA).
- **Deviation Analysis:** Flagging when a contract uses non-standard clauses that differ from the library.
- **Auto-Renewal:** Logic for identifying and processing contracts that automatically extend unless cancelled.
- **Digital Signature Integration:** Orchestrating the workflow for electronic signatures (e.g., DocuSign/Adobe Sign).
