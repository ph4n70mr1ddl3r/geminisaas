# Banking Core Financials (FTP & ALM) Service Specification

## Overview

The Banking Core Financials service manages Funds Transfer Pricing (FTP) and Asset Liability Management (ALM) for financial institutions. It calculates the internal cost of funds and analyzes interest rate and liquidity risk across the balance sheet. This aligns with Oracle Financial Services Analytical Applications (OFSAA).

## Data Model (SQLite)

- `banking_instrument_data`: id (UUID), account_id (link to Banking ERP), instrument_type (LOAN/DEPOSIT/SEC), principal, interest_rate, maturity_date.
- `ftp_rules`: id (UUID), name, method (MATCHED_MATURITY/LIQUIDITY_PREMIUM), currency, status.
- `ftp_results_banking`: id (UUID), account_id, period_id, transfer_rate, margin_amount, net_interest_income.
- `alm_stochastic_scenarios`: id (UUID), scenario_name, interest_rate_path (JSON), probability_weight.
- `alm_liquidity_gaps`: id (UUID), bucket_name (e.g., '1-3 Months'), gap_amount, cumulative_gap, period_id.
- `banking_capital_adequacy`: id (UUID), period_id, tier_1_capital, risk_weighted_assets, car_ratio.

## API Endpoints (Axum/gRPC)

- `POST /banking-analytics/ftp/calculate`: Run the Funds Transfer Pricing engine for a specific period and portfolio.
- `GET /banking-analytics/alm/liquidity-report`: Retrieve the dynamic liquidity gap analysis.
- `POST /banking-analytics/scenarios/run`: Execute interest rate stress tests based on stochastic modeling.
- `GET /banking-analytics/margins/branch`: View net interest margin (NIM) by branch or product line.
- `POST /banking-analytics/capital/check`: Analyze capital adequacy against Basel III/IV requirements.

## Inter-service Interaction

- Queries `Banking ERP` for real-time account balances and transaction history.
- Emits `NIM_Warning` alert to the Treasury team if margins drop below a threshold.
- Queries `Treasury` for market interest rate curves used in FTP calculations.
- Emits `Liquidity_Risk_Signal` to `Risk Management`.
- Queries `Financial Consolidation (FCCS)` for legal entity hierarchy.

## Key Logic

- **Matched Maturity FTP:** Assigning a unique transfer rate to each transaction based on its specific maturity and the market rate at the time of origination.
- **Dynamic Liquidity Analysis:** Forecasting future cash flows from loans and deposits to identify potential liquidity shortages in specific time buckets.
- **Interest Rate Risk (IRRBB):** Measuring the sensitivity of the bank's earnings and economic value to changes in interest rates.
- **Behavioral Modeling:** Predicting when "non-maturity" deposits (e.g., savings accounts) will actually be withdrawn.
- **Prepayment Modeling:** Estimating the rate at which borrowers will pay off loans early, which impacts long-term cash flow.
- **Net Interest Margin (NIM) Attribution:** Breaking down bank profit into "Funding Center" profit and "Business Line" profit.
