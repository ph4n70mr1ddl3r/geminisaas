# IoT Fleet Monitoring Service Specification

## Overview

The IoT Fleet Monitoring service provides real-time tracking, driver behavior analysis, and route monitoring for vehicle fleets using telematics data. This aligns with Oracle Fusion Cloud IoT Intelligent Applications (Fleet Monitoring).

## Data Model (SQLite)

- `iot_vehicles`: id (UUID), asset_id (link to ALM), license_plate, status (ON_TRIP/IDLE/MAINTENANCE), current_gps (JSON), battery_level.
- `fleet_trips`: id (UUID), vehicle_id, driver_id (link to HCM), start_time, end_time, origin, destination, distance_km, fuel_consumed.
- `driver_behavior_events`: id (UUID), trip_id, event_type (HARSH_BRAKING/OVERSPEEDING/IDLING), severity, timestamp.
- `iot_fleet_geofences`: id (UUID), vehicle_id, region_json, status.
- `fleet_incidents`: id (UUID), vehicle_id, description, location_gps, severity, status.
- `telematics_summary`: id (UUID), vehicle_id, period_id, avg_speed, idle_pct, safety_score.

## API Endpoints (Axum/gRPC)

- `GET /iot/fleet/tracking`: Retrieve the real-time location and status of all vehicles in the fleet.
- `POST /iot/fleet/trips/start`: Begin recording a new trip for a vehicle.
- `POST /iot/fleet/telematics/ingest`: Ingest real-time telematics data (GPS, speed, engine codes).
- `GET /iot/fleet/driver-scorecard/{driver_id}`: Retrieve a performance and safety scorecard for a driver.
- `POST /iot/fleet/geofences`: Create a virtual perimeter for fleet monitoring.
- `GET /iot/fleet/analytics/fuel-efficiency`: Analyze fuel consumption trends using sensor and trip data.

## Inter-service Interaction

- Queries `Asset Lifecycle Management (ALM)` for master vehicle definitions and engine specs.
- Emits `Shipment_Delayed` event to `Logistics` based on real-time traffic and speed data.
- Emits `Harsh_Driving_Detected` alert to fleet managers.
- Queries `HCM Service` for driver certifications and work schedules.
- Emits `Maintenance_Needed` to `Maintenance Service` based on engine fault codes or mileage.

## Key Logic

- **Real-Time GPS Tracking:** Providing sub-second updates on vehicle location and estimated time of arrival (ETA).
- **Driver Safety Scoring:** Using AI to rank drivers based on events like harsh braking, rapid acceleration, and overspeeding.
- **Geofence Alerts:** Automatically notifying dispatchers when a vehicle enters or leaves a customer site or warehouse.
- **Route Adherence:** Identifying when a driver deviates from the planned route provided by `Logistics`.
- **Engine Diagnostics Monitoring:** Capturing and decoding OBD-II or J1939 fault codes to identify mechanical issues early.
- **Fuel Consumption Analysis:** Correlating GPS data with engine telemetry to find opportunities for fuel savings (e.g., reducing idling).
