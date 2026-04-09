# Account Reconciliation Service Specification

## Overview

The Account Reconciliation service automates the validation of balance sheet and sub-ledger accounts, ensuring accuracy during the financial close process. This aligns with Oracle Fusion Cloud Account Reconciliation.

## Data Model (SQLite)

- `reconciliation_profiles`: id (UUID), name, account_id (link to Financials), frequency (MONTHLY/QUARTERLY), preparer_id, reviewer_id, status (ACTIVE/ARCHIVED).
- `reconciliations`: id (UUID), profile_id, period_id, status (PENDING/PREPARED/REVIEWED/CLOSED), balance_gl, balance_subledger, variance.
- `reconciliation_items`: id (UUID), reconciliation_id, source_id, type (SOURCE/SYSTEM), amount, description, status (MATCHED/UNMATCHED).
- `reconciliation_transactions`: id (UUID), recon_id, source_system, date, reference, amount.
- `matching_rules`: id (UUID), name, description, match_type (ONE_TO_ONE/ONE_TO_MANY/MANY_TO_MANY), rule_logic (JSON).
- `reconciliation_adjustments`: id (UUID), recon_id, amount, explanation, status (PENDING/POSTED).

## API Endpoints (Axum/gRPC)

- `GET /reconciliation/profiles`: View and manage the reconciliation calendar.
- `POST /reconciliation/prepare`: Submit a completed reconciliation for review.
- `POST /reconciliation/match`: Run the automated transaction matching engine.
- `GET /reconciliation/{id}/items`: Retrieve individual transaction items for a reconciliation.
- `POST /reconciliation/adjust`: Create a correcting entry for a balance variance.
- `GET /reconciliation/dashboard`: View overall completion progress and high-variance accounts.

## Inter-service Interaction

- Queries `Financials Service` for real-time GL and sub-ledger balances.
- Emits `Adjustment_Approved` to `Financials` to create a journal entry.
- Receives `Period_Closed` from `Financials` to lock all reconciliations.
- Queries `Identity Service` for preparer and reviewer workflow steps.

## Key Logic

- **Auto-Matching Engine:** Using rule-based logic to link thousands of transactions between systems (e.g., bank vs. GL) based on date, reference, and amount.
- **Risk-Based Reconciliation:** Prioritizing high-value or high-variance accounts for more frequent review.
- **Workflow Orchestration:** Ensuring that the preparer and reviewer are different individuals (Separation of Duties).
- **Variance Tolerance:** Automatically marking a reconciliation as "In Balance" if the variance is within a pre-defined amount (e.g., <$5.00).
- **Audit Documentation:** Storing supporting documents and comments for each reconciliation to meet compliance standards (e.g., SOX).
