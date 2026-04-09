# Financial Consolidation & Close (FCCS) Service Specification

## Overview

The Financial Consolidation and Close (FCCS) service automates the complex "Record-to-Report" process, including intercompany eliminations, currency translations, and the final financial close. This aligns with Oracle Fusion Cloud FCCS.

## Data Model (SQLite)

- `fccs_applications`: id (UUID), name, status (ACTIVE/ARCHIVED), base_currency, consolidation_method (FULL/EQUITY).
- `fccs_dimensions`: id (UUID), app_id, name (ACCOUNT/ENTITY/PERIOD/VIEW/CURRENCY/MOVEMENT/DATA_SOURCE/CONSOLIDATION).
- `fccs_consolidation_runs`: id (UUID), app_id, period_id, status (PENDING/IN_PROGRESS/COMPLETED/FAILED), start_time, end_time.
- `fccs_elimination_rules`: id (UUID), name, sequence, source_account_id, target_account_id, logic (AUTO_ELIMINATE/MANUAL).
- `fccs_intercompany_entries`: id (UUID), run_id, source_entity_id, target_entity_id, amount, currency, status.
- `fccs_close_checklist`: id (UUID), period_id, task_name, owner_id, status (PENDING/COMPLETED), due_date.
- `fccs_translation_rates`: id (UUID), from_currency, to_currency, period_id, average_rate, ending_rate.

## API Endpoints (Axum/gRPC)

- `POST /fccs/applications`: Create a new financial consolidation application.
- `POST /fccs/consolidate`: Trigger the consolidation process for a specific period and entity.
- `GET /fccs/intercompany/matching`: Retrieve a report of unmatched intercompany transactions.
- `POST /fccs/journal/create`: Create a manual consolidation or elimination journal entry.
- `GET /fccs/close-status`: Monitor the progress of the financial close checklist.
- `GET /fccs/financial-statements`: Generate consolidated Balance Sheet and Income Statement.

## Inter-service Interaction

- Queries `Financials Service` for trial balance data from subsidiary ledgers.
- Emits `Consolidation_Completed` event to `Narrative Reporting` for annual report generation.
- Queries `Enterprise Data Management (EDM)` for the latest legal entity and account hierarchies.
- Emits `Close_Task_Overdue` alert to the corporate controller.
- Queries `Tax Reporting` for global tax provisioning data.

## Key Logic

- **Automated Eliminations:** Identifying and canceling out transactions between legal entities within the same group (e.g., intercompany sales).
- **Multi-Currency Translation:** Converting subsidiary financials into the parent's reporting currency using historical, average, and ending rates.
- **Journal Workflow:** Requiring review and approval for all manual adjustments made at the consolidation level.
- **Data Status Tracking:** Monitoring whether data in a period has been modified since the last consolidation ("Impacted" status).
- **Equity Method Accounting:** Handling complex ownership structures and minority interest calculations.
- **Audit Lineage:** Providing a clear "Drill-Back" from consolidated numbers to the source ERP transactions.
