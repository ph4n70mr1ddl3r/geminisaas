# IoT Production Monitoring Service Specification

## Overview

The IoT Production Monitoring service enables real-time tracking of manufacturing performance, machine health, and overall equipment effectiveness (OEE) using factory-floor IoT data. This aligns with Oracle Fusion Cloud IoT Intelligent Applications (Production Monitoring).

## Data Model (SQLite)

- `iot_factory_nodes`: id (UUID), factory_id (link to Manufacturing), machine_id, device_id, status (RUNNING/IDLE/DOWN/ALARM).
- `machine_telemetry`: id (UUID), machine_id, metric_type (OEE/SPEED/AVAILABILITY/QUALITY), value, timestamp.
- `production_anomalies`: id (UUID), machine_id, work_order_id, description, severity, status.
- `digital_twin_configs`: id (UUID), machine_id, schema_json (mapping sensor tags to production metrics).
- `oee_logs`: id (UUID), machine_id, period_id, availability_pct, performance_pct, quality_pct, oee_score.
- `iot_sensor_events`: id (UUID), machine_id, event_type (CYCLE_START/CYCLE_END/PART_REJECTED), timestamp.

## API Endpoints (Axum/gRPC)

- `GET /iot/production/oee/{machine_id}`: Retrieve real-time OEE metrics for a specific production line or machine.
- `POST /iot/production/event`: Record a machine-level event (e.g., machine cycle completion).
- `GET /iot/production/machine-health`: Retrieve a dashboard view of all machines in a factory.
- `POST /iot/production/alarm/acknowledge`: Record operator acknowledgment of a machine alarm.
- `GET /iot/production/bottleneck-analysis`: Identify production bottlenecks using AI-driven telemetry analysis.

## Inter-service Interaction

- Queries `Manufacturing Service` for active work orders and production schedules.
- Emits `Production_Delay_Detected` event to `Order Management` and `Planning` when OEE drops.
- Emits `Machine_Down_Alert` to `Maintenance Service` to automatically create an emergency work order.
- Receives `Work_Center_Config` from `Manufacturing` to align digital twin nodes.
- Emits `Scrap_Recorded` event to `Inventory` when the IoT sensor detects a rejected part.

## Key Logic

- **OEE Calculation Engine:** Real-time computation of Availability x Performance x Quality based on machine sensor data.
- **Machine Connectivity:** Supporting industry standards like OPC-UA and MQTT to ingest data from diverse PLC and CNC systems.
- **Cycle Time Analysis:** Automatically identifying deviations between the theoretical cycle time and actual sensor-recorded times.
- **Planned vs. Unplanned Downtime:** Using operator input or machine state logic to categorize reasons for production stops.
- **Bottleneck Identification:** Correlating telemetry across a production line to find the slowest node impacting total output.
- **Digital Factory Map:** Providing data for a spatial visualization of the factory floor with real-time status overlays.
