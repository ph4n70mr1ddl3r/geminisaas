# Supply Chain Command Center Service Specification

## Overview

The Supply Chain Command Center service provides a unified, AI-driven workspace for supply chain leaders to monitor global operations, identify disruptions in real-time, and execute remediation strategies. This aligns with the Oracle Supply Chain Command Center (2026-2027 roadmap).

## Data Model (SQLite)

- `supply_chain_nodes`: id (UUID), name, type (FACTORY/WH/SUPPLIER/CARRIER), location_gis (JSON), current_risk_score.
- `global_disruption_events`: id (UUID), type (PORT_STRIKE/WEATHER/SUPPLIER_FAIL), severity, impact_scope (JSON), status (ACTIVE/RESOLVED).
- `inventory_visibility_global`: id (UUID), item_id, total_on_hand, total_in_transit, total_backordered, period_id.
- `command_center_kpis`: id (UUID), metric_name (OTIF/LEAD_TIME_VAR/COST_VAR), current_value, target_value, trend.
- `remediation_playbooks`: id (UUID), trigger_event_type, action_steps (JSON - e.g., 'Re-route via Air', 'Increase Safety Stock').
- `digital_supply_chain_twin`: id (UUID), version, snapshot_time, simulation_results (JSON).

## API Endpoints (Axum/gRPC)

- `GET /scm-command-center/map`: Retrieve a global view of nodes, routes, and active disruptions.
- `GET /scm-command-center/kpis`: Retrieve real-time supply chain health metrics.
- `POST /scm-command-center/simulate-disruption`: Run a "what-if" scenario to see the impact of a potential disruption (e.g., "What if the Suez Canal is blocked?").
- `POST /scm-command-center/remediate`: Trigger an automated playbook to resolve an identified supply gap.
- `GET /scm-command-center/inventory/aging`: Identify stock at risk of expiry or obsolescence across the entire network.
- `POST /scm-command-center/collaborate`: Initiate a real-time war-room session with key stakeholders.

## Inter-service Interaction

- Queries `Logistics / OTM / GTM` for real-time shipment locations and trade compliance status.
- Queries `Inventory / WMS / Manufacturing` for current stock and production output.
- Emits `Expedite_Order_Required` event to `Procurement` and `Transportation`.
- Receives `External_Risk_Signals` (e.g., Resilinc, weather data) to flag node risks.
- Emits `Backlog_Prioritization_Update` to `Backlog Management`.

## Key Logic

- **End-to-End Visibility:** Transcending individual modules to show the flow of goods from Tier 2 suppliers to final customers.
- **Risk Scoring:** Automatically calculating a risk index for every node based on historical reliability, geopolitical factors, and real-time events.
- **Automated Root Cause:** Using AI to trace a customer service failure back to a specific machine breakdown or supplier delay.
- **Playbook Orchestration:** Suggesting the most successful remediation path based on how similar disruptions were handled in the past.
- **Inventory Rebalancing:** Identifying opportunities to move stock from a high-inventory region to a region with a forecasted shortage.
- **Sustainability Impact:** Visualizing the carbon footprint impact of different remediation options (e.g., Air vs. Ocean).
