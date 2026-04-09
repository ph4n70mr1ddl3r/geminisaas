# Advanced Intercompany (AGIS) Service Specification

## Overview

The Advanced Intercompany (AGIS) service provides a centralized hub for managing, reconciling, and settling financial transactions between different legal entities within the same corporate group. This aligns with Oracle Fusion Cloud Advanced Global Intercompany System (AGIS).

## Data Model (SQLite)

- `intercompany_orgs`: id (UUID), name, legal_entity_id, status, primary_ledger_id.
- `intercompany_transactions`: id (UUID), initiator_org_id, recipient_org_id, amount, currency, status (DRAFT/SUBMITTED/APPROVED/REJECTED), description.
- `intercompany_lines`: id (UUID), transaction_id, gl_account_id, amount, debit_credit_flag, internal_tax_id.
- `intercompany_reconciliation_rules`: id (UUID), name, tolerance_amount, matching_logic (JSON).
- `intercompany_settlement_ledger`: id (UUID), transaction_id, settlement_method (AP_AR_INVOICE/JOURNAL_ONLY/NETTING), status.
- `intercompany_periods`: id (UUID), period_name, status (OPEN/CLOSED/PERMANENTLY_CLOSED).

## API Endpoints (Axum/gRPC)

- `POST /intercompany/transactions`: Submit a new intercompany transaction for approval by the recipient entity.
- `POST /intercompany/approve`: Record the recipient entity's acceptance of an intercompany charge.
- `GET /intercompany/reconciliation`: Retrieve a report of out-of-balance transactions between entities.
- `POST /intercompany/generate-invoices`: Automatically create AP and AR invoices from approved intercompany transactions.
- `POST /intercompany/netting`: Run the netting engine to consolidate multiple payables and receivables into a single settlement.
- `GET /intercompany/dashboard/close-status`: Monitor the intercompany close progress across all legal entities.

## Inter-service Interaction

- Emits `AR_Invoice_Required` and `AP_Invoice_Required` events to `Financials` (AP/AR).
- Emits `Intercompany_Journal_Entries` to `Financials` (GL).
- Queries `Enterprise Data Management (EDM)` for master legal entity and intercompany relationship hierarchies.
- Receives `SFO_Trade_Event` from `Supply Chain Financial Orchestration` to automate trade-related intercompany accounting.
- Emits `Unmatched_Alert` to entity controllers.

## Key Logic

- **Recipient Approval Workflow:** Forcing the receiving entity to review and approve any intercompany charge before it hits their ledger.
- **Intercompany Balancing:** Automatically generating balancing entries (Due To / Due From) to ensure the trial balance remains consistent.
- **Settlement Method Selection:** Choosing whether a transaction needs formal AP/AR invoices or can be handled via direct GL journals.
- **Intercompany Netting:** Reducing the number of physical payments by canceling out mutual payables and receivables between subsidiaries.
- **Currency Translation & Revaluation:** Handling transactions between entities with different functional currencies.
- **Internal Tax Calculation:** Identifying and applying the correct internal VAT or sales tax based on intercompany tax rules.
