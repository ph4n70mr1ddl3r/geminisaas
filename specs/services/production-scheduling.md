# Production Scheduling Service Specification

## Overview

The Production Scheduling service provides high-granularity, constraint-based shop floor scheduling. It optimizes the sequence of work orders across resources (machines and labor) to meet delivery dates while minimizing setup times and bottlenecks. This aligns with Oracle Fusion Cloud Production Scheduling (SCM).

## Data Model (SQLite)

- `scheduling_resources`: id (UUID), name, type (MACHINE/LABOR/TOOL), capacity_units, status (AVAIL/DOWN).
- `resource_shifts`: id (UUID), resource_id, start_time, end_time, efficiency_pct.
- `scheduled_operations`: id (UUID), work_order_id (link to Manufacturing), resource_id, scheduled_start, scheduled_end, sequence, setup_time_min.
- `scheduling_constraints`: id (UUID), type (MATERIAL/CAPACITY/CALENDAR), description, severity (HARD/SOFT).
- `setup_matrix`: id (UUID), from_item_category, to_item_category, setup_time_impact_min.
- `production_schedule_runs`: id (UUID), run_date, status (DRAFT/PUBLISHED), total_duration_hours, bottleneck_resource_id.

## API Endpoints (Axum/gRPC)

- `POST /production-scheduling/optimize`: Execute the constraint-based scheduling engine for a factory or work center.
- `GET /production-scheduling/gantt`: Retrieve the detailed operation sequence for visualization.
- `POST /production-scheduling/publish`: Finalize a draft schedule and update work order operation dates.
- `GET /production-scheduling/bottlenecks`: Identify resources that are limiting total production throughput.
- `POST /production-scheduling/resource/status`: Update real-time machine status (e.g., unplanned breakdown).

## Inter-service Interaction

- Receives `Work_Order_Released` from `Manufacturing` to begin scheduling.
- Queries `Inventory Service` for component availability before scheduling an operation.
- Emits `Scheduled_Dates_Updated` event to `Manufacturing` and `Supply Chain Planning`.
- Queries `Maintenance Service` for planned machine downtime (PM).
- Emits `Material_Shortage_Alert` to `Procurement` if a scheduled job lacks components.

## Key Logic

- **Constraint-Based Optimization:** Simultaneously considering material availability, labor skills, and machine capacity.
- **Setup Time Minimization:** Grouping similar products together to reduce costly machine changeovers.
- **Back-to-Front Scheduling:** Working backward from the customer due date to find the latest possible start time.
- **What-If Simulations:** Creating multiple draft schedules to see the impact of adding a new shift or a rush order.
- **Real-Time Rescheduling:** Automatically adjusting the sequence when a machine breaks down or a material shipment is delayed.
- **Resource Leveling:** Smoothing out workloads to prevent over-utilization of specific machines or crews.
