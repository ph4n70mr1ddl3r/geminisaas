# Retail Demand Forecasting (RDF) Service Specification

## Overview

The Retail Demand Forecasting (RDF) service uses advanced statistical and machine learning models to predict future customer demand at the item/location/day level. This aligns with Oracle Retail Demand Forecasting.

## Data Model (SQLite)

- `forecast_parameters`: id (UUID), item_id, location_id, algorithm_type (ARIMA/ES/DL), seasonality_id.
- `demand_forecasts_retail`: id (UUID), item_id, location_id, date, forecast_qty, confidence_score, status (ACTIVE/ARCHIVED).
- `historical_sales_retail`: id (UUID), item_id, location_id, date, actual_qty, promo_flag (BOOL), stockout_flag (BOOL).
- `demand_influencing_factors` (DIF): id (UUID), type (PRICE/PROMO/WEATHER/EVENT), date, impact_multiplier.
- `forecast_accuracy_metrics`: id (UUID), forecast_id, actual_qty, mape (Mean Absolute Pct Error), bias.
- `customer_decision_trees`: id (UUID), category_id, tree_logic_json (JSON), importance_ranking (JSON).

## API Endpoints (Axum/gRPC)

- `POST /retail-forecasting/generate`: Trigger the AI engine to generate a new demand forecast for a set of items and locations.
- `GET /retail-forecasting/forecast`: Retrieve the predicted demand for an item/location intersection.
- `POST /retail-forecasting/dif/update`: Add a demand influencing factor (e.g., upcoming promotion) to the model.
- `GET /retail-forecasting/accuracy`: Retrieve performance reports for the forecasting engine.
- `POST /retail-forecasting/demand-transference`: Analyze the impact of a stockout on related items.

## Inter-service Interaction

- Queries `Retail Management` for real-time and historical sales transactions.
- Emits `Demand_Forecast_Updated` event to `Supply Chain Planning` and `MFP` for replenishment and financial planning.
- Receives `Inventory_Stockout` from `WMS` to flag historical sales as "Lost Demand" rather than "Zero Demand."
- Queries `Marketing` for upcoming promotion calendars.
- Emits `Potential_Stockout_Risk` to store managers.

## Key Logic

- **AI-Native Forecasting:** Using neural networks and deep learning to identify complex demand patterns across millions of intersections.
- **Seasonality & Holiday Handling:** Automatically adjusting for movable holidays (e.g., Easter, Diwali) and regional seasonal trends.
- **Lost Demand Estimation:** Correcting historical sales data to account for periods when an item was out of stock.
- **Demand Transference:** Predicting that if "Brand A Milk" is out of stock, 70% of customers will buy "Brand B Milk."
- **Promotional Lift Modeling:** Quantifying the incremental sales expected from a specific marketing event or price reduction.
- **Exception Management:** Proactively alerting planners to "Low Confidence" forecasts or large variances between forecast and actuals.
