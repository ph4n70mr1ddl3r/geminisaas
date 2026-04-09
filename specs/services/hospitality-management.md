# Hospitality Management (Opera) Service Specification

## Overview

The Hospitality Management service provides core property management (PMS) capabilities for hotels and resorts, including reservations, room management, and guest profiles. This aligns with Oracle Opera.

## Data Model (SQLite)

- `hotel_properties`: id (UUID), name, brand, location, total_rooms, status (ACTIVE/RENOVATION).
- `room_inventory`: id (UUID), property_id, room_number, room_type (DELUXE/SUITE/STANDARD), status (VACANT/OCCUPIED/CLEANING/MAINTENANCE).
- `hotel_reservations`: id (UUID), guest_id (link to CDM), property_id, arrival_date, departure_date, status (CONFIRMED/CHECKED_IN/CHECKED_OUT/CANCELLED), rate_amount.
- `guest_profiles_hotel`: id (UUID), customer_id (link to CDM), preferences (JSON - e.g., 'High Floor', 'Extra Pillows'), loyalty_tier (link to Loyalty).
- `folio_transactions`: id (UUID), reservation_id, item_type (ROOM/FOOD/SPA/TAX), amount, status (UNPAID/PAID).
- `housekeeping_tasks`: id (UUID), room_id, task_type (CLEAN/STAY_OVER), assigned_staff_id, status, time_completed.

## API Endpoints (Axum/gRPC)

- `POST /hospitality/reservations`: Create a new guest reservation.
- `POST /hospitality/check-in`: Record guest arrival and assign a room.
- `POST /hospitality/check-out`: Finalize guest charges and release room.
- `GET /hospitality/inventory/available`: Retrieve real-time room availability for a date range.
- `POST /hospitality/folio/charge`: Post a new charge (e.g., room service) to a guest's bill.
- `GET /hospitality/housekeeping/schedule`: Retrieve the daily room cleaning list.

## Inter-service Interaction

- Queries `CDM Service` for guest master profiles and history.
- Emits `Payment_Received` event to `Financials / Banking ERP` for billing settlement.
- Queries `Loyalty Service` to apply member benefits or award points.
- Emits `Maintenance_Requested` to `Maintenance Service` for room repairs.
- Queries `Food & Beverage Management` for restaurant charges posted to rooms.

## Key Logic

- **Yield Management:** Automatically adjusting room rates based on occupancy levels and seasonal demand.
- **Room Assignment Logic:** Finding the best available room that matches guest preferences (e.g., away from elevators).
- **Folio Management:** Maintaining a real-time, accurate list of all guest spending during their stay.
- **Group Booking:** Handling complex multi-room reservations for events or conferences with shared billing.
- **Late Check-out / Early Check-in:** Managing room availability and extra fees for non-standard arrival/departure times.
- **Guest Recognition:** Proactively notifying staff of high-value loyalty members during check-in.
