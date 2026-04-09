# IoT Asset Monitoring Service Specification

## Overview

The IoT Asset Monitoring service provides real-time visibility into the health, location, and utilization of physical assets using IoT sensor data. This aligns with Oracle Fusion Cloud IoT Intelligent Applications (Asset Monitoring).

## Data Model (SQLite)

- `iot_devices`: id (UUID), device_type, serial_number, status (ONLINE/OFFLINE/ERROR), firmware_version, last_ping.
- `asset_telemetry_stream`: id (UUID), asset_id (link to ALM), device_id, reading_type (TEMP/VIBRATION/PRESSURE), value, timestamp.
- `iot_asset_geofences`: id (UUID), asset_id, geofence_json, alert_on (ENTER/EXIT), status.
- `iot_asset_anomalies`: id (UUID), asset_id, description, severity (LOW/MED/HIGH), status (OPEN/RESOLVED), timestamp.
- `iot_maintenance_triggers`: id (UUID), asset_id, criteria_json (e.g., 'Temp > 100C'), status.

## API Endpoints (Axum/gRPC)

- `POST /iot/devices/register`: Onboard a new IoT sensor or gateway.
- `POST /iot/telemetry/ingest`: High-scale ingestion of sensor data.
- `GET /iot/assets/real-time/{id}`: Retrieve the latest sensor readings and health score for an asset.
- `POST /iot/geofences`: Define a virtual boundary for an asset.
- `GET /iot/analytics/utilization`: Analyze asset run-time vs. idle-time using sensor data.
- `POST /iot/anomalies/resolve`: Mark a detected anomaly as handled.

## Inter-service Interaction

- Queries `Asset Lifecycle Management (ALM)` for master asset definitions and service history.
- Emits `Predictive_Maintenance_Required` event to `Maintenance Service` when an anomaly is detected.
- Queries `Identity Service` for device authentication and security certificates.
- Emits `Geofence_Violation` alert to `Logistics` or `Security`.
- Receives `Work_Order_Completed` from `Maintenance` to reset health monitoring thresholds.

## Key Logic

- **Digital Twin Sync:** Maintaining a digital representation of the physical asset that mirrors real-time sensor states.
- **Anomaly Detection:** Using AI/ML to identify patterns in telemetry data that deviate from the asset's "normal" operating signature.
- **Rule-Based Actions:** Automatically triggering events (e.g., closing a valve) when specific sensor thresholds are exceeded.
- **Geofencing:** Real-time alerting when an expensive or sensitive asset leaves a pre-defined work zone.
- **Health Score Calculation:** Aggregating multiple sensor inputs into a single "Health Index" (0-100) for the asset.
- **High-Scale Data Handling:** Optimized for processing thousands of telemetry messages per second with minimal latency.
