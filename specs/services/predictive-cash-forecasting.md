# Predictive Cash Forecasting Service Specification

## Overview

The Predictive Cash Forecasting (PCF) service uses AI and machine learning to generate highly accurate, short-to-medium term cash flow projections. It identifies patterns in historical AP/AR data to predict when payments will actually occur, rather than relying on due dates. This aligns with Oracle Fusion Cloud EPM/Treasury (Predictive Cash).

## Data Model (SQLite)

- `cash_forecast_models`: id (UUID), name, status (TRAINED/ACTIVE), target_currency, forecast_horizon_days.
- `predicted_cash_flows`: id (UUID), model_id, period_id, flow_type (INFLOW/OUTFLOW), predicted_amount, confidence_interval_low, confidence_interval_high.
- `payment_behavior_profiles`: id (UUID), customer_id (link to CDM), avg_days_late, seasonal_late_pattern (JSON), predicted_payment_date_offset.
- `external_cash_signals`: id (UUID), source (MARKET/FED/WEATHER), signal_type, value, timestamp.
- `forecast_variance_logs`: id (UUID), predicted_date, actual_amount, predicted_amount, error_pct.
- `liquidity_event_simulations`: id (UUID), event_name, impact_amount, status (DRAFT/EXECUTED).

## API Endpoints (Axum/gRPC)

- `POST /predictive-cash/models/train`: Train the AI model using historical GL, AP, and AR data.
- `GET /predictive-cash/forecast`: Retrieve the AI-generated cash forecast for a specific period.
- `GET /predictive-cash/customer-prediction/{id}`: Predict when a specific customer will pay an outstanding invoice.
- `POST /predictive-cash/simulate-liquidity`: Run a "what-if" simulation for a major liquidity event (e.g., debt repayment).
- `GET /predictive-cash/analytics/accuracy`: Retrieve performance metrics for the forecasting engine.
- `POST /predictive-cash/export`: Push the forecast to `Treasury` for cash positioning.

## Inter-service Interaction

- Queries `Financials` (AP/AR) for thousands of historical invoice and payment records.
- Emits `Cash_Shortfall_Predicted` event to `Treasury` and `Strategic Modeling`.
- Queries `CRM Service` for upcoming large opportunities that will impact future cash.
- Receives `FX_Rate_Update` from `Treasury` to value multi-currency predictions.
- Emits `Collection_Strategy_Recommended` to `Collections` based on predicted late payments.

## Key Logic

- **ML-Based Payment Timing:** Identifying that Customer X always pays on Tuesdays, or that Vendor Y is always paid 2 days early.
- **Seasonality Detection:** Automatically adjusting predictions for month-end, quarter-end, or holiday-related cash fluctuations.
- **External Signal Correlation:** Incorporating market indices or interest rate changes into the cash prediction model.
- **Confidence Intervals:** Providing a range (e.g., 95% confidence) for the cash position rather than a single static number.
- **Variance Driver Analysis:** Identifying the specific invoices or events that caused an actual cash flow to differ from the prediction.
- **Liquidity Threshold Monitoring:** Proactively alerting if the *predicted* cash balance (not just current) drops below a safety limit.
