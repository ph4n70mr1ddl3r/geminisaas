# Analytics Service Specification

## Overview

The Analytics service provides cross-module reporting, KPI dashboards, and AI-driven predictive insights. It aggregates data from all services into a unified analytical layer.

## Data Model (SQLite/In-Memory)

- `kpi_definitions`: id (UUID), name, formula, target_value, frequency.
- `analytical_snapshots`: id (UUID), service_source, metric_name, value, timestamp.
- `report_templates`: id (UUID), name, query_definition, layout_json, owner_id.
- `ai_predictions`: id (UUID), target_metric, predicted_value, confidence_score, date.
- `dashboards`: id (UUID), name, owner_id, layout_config (JSON).

## API Endpoints (Axum/gRPC)

- `GET /analytics/kpis`: Retrieve real-time key performance indicators.
- `GET /analytics/reports`: Generate and download custom reports.
- `POST /analytics/predictions`: Request AI-based forecasting (e.g., cash flow, demand).
- `GET /analytics/dashboards`: Fetch configured dashboard layouts and data.
- `POST /analytics/snapshots`: Manually trigger data aggregation for point-in-time analysis.

## Inter-service Interaction

- Queries all other services (Financials, Procurement, CRM, etc.) for raw transactional data.
- Receives events from all services to update real-time KPI counters.
- Emits `Threshold_Breach` events to `Identity/Notification Service` when KPIs go outside targets.
- Integrates with `EPM Service` for strategic planning and variance reporting.

## Key Logic

- **Data Aggregation:** Periodically snapshot transactional data to avoid performance impact on live services.
- **Predictive ML:** Use historical data (stored in snapshots) to train simple linear/polynomial regression models for forecasting.
- **Role-Based Reporting:** Filter data based on user permissions defined in `Identity Service`.
- **Drill-Down:** Allow users to navigate from high-level KPI summaries to individual source transactions.
