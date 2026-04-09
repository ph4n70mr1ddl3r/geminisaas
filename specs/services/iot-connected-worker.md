# IoT Connected Worker Service Specification

## Overview

The IoT Connected Worker service provides real-time monitoring of worker safety, location, and productivity using wearable devices and environmental sensors. This aligns with Oracle Fusion Cloud IoT Intelligent Applications (Connected Worker).

## Data Model (SQLite)

- `worker_wearables`: id (UUID), worker_id (link to HCM), device_type (VEST/HELMET/WATCH), serial_number, status (ACTIVE/CHARGING/FAULT), last_ping.
- `biometric_telemetry`: id (UUID), device_id, metric_type (HEART_RATE/BODY_TEMP/STRESS_INDEX), value, timestamp.
- `safety_incidents_iot`: id (UUID), worker_id, event_type (FALL_DETECTED/HIGH_HEAT/NO_MOTION), location_gps (JSON), severity, status.
- `restricted_zone_violations`: id (UUID), worker_id, zone_id, entry_time, exit_time, status (ALERTED/RESOLVED).
- `worker_project_mapping`: id (UUID), worker_id, project_id, task_id, activity_status, duration_min.
- `environmental_sensors`: id (UUID), location_id, sensor_type (GAS/NOISE/TEMP), current_value, threshold_limit.

## API Endpoints (Axum/gRPC)

- `POST /iot/connected-worker/telemetry`: Ingest high-frequency biometric and location data from wearables.
- `GET /iot/connected-worker/real-time-map`: Retrieve coordinates and safety status for all active workers.
- `POST /iot/connected-worker/incident/acknowledge`: Record supervisor response to a safety alert.
- `GET /iot/connected-worker/biometric-history/{worker_id}`: View historical health trends for a specific employee.
- `POST /iot/connected-worker/hazard/mark`: Create a temporary digital hazard zone on the map.
- `GET /iot/connected-worker/productivity-analytics`: Correlate sensor-recorded movement with project task completion.

## Inter-service Interaction

- Queries `HCM Service` for worker certifications and emergency contact details.
- Emits `Safety_Incident_Reported` event to `Workforce Health & Safety` for formal investigation.
- Queries `Project Management` to link worker presence at a site to specific tasks.
- Emits `Emergency_Call_Triggered` to external safety protocols.
- Receives `Location_Config` from `Logistics/Manufacturing` to define safe vs. restricted zones.

## Key Logic

- **Fall Detection:** Using accelerometer and gyroscope data to automatically identify and alert on potential falls.
- **Predictive Heat Stress:** Analyzing ambient temperature and worker heart rate to predict exhaustion before it happens.
- **Credential Enforcement:** Alerting if a worker enters a zone (e.g., high-voltage area) without the required HCM-recorded certification.
- **Man-Down Alerts:** Triggering high-priority notifications if no movement is detected from an active device for a pre-set interval.
- **Automatic Muster Counting:** Real-time headcount of workers at a safe assembly point during an emergency evacuation.
- **Exposure Tracking:** Monitoring cumulative time spent in hazardous conditions (e.g., high noise or toxic gas levels).
