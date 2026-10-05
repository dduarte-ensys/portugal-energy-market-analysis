# Portugal Electricity Market Analysis: Renewable Generation, Price Dynamics, and Storage Value (2023–2025)

## Overview

This project birthed from a desire to self-learn data analytics and i took advantage of my educational background to kick start this journey. This project analyzes three years (2023–2025) of hourly electricity market data for Portugal, sourced from the ENTSO-E Transparency Platform, to answer:

1. How does renewable generation share relate to day-ahead electricity prices?
2. Is the observed price behavior (including near-zero/negative pricing) structural, or a one-off anomaly?
3. What is the theoretical arbitrage value of grid-scale battery storage under these price dynamics, and how has it changed over time?

## Data Sources

- **Day-ahead prices** (EUR/MWh) — ENTSO-E Transparency Platform, RESTful API
- **Actual generation per production type** (MW) — ENTSO-E Transparency Platform, RESTful API
- **Actual total load** (MW) — ENTSO-E Transparency Platform, RESTful API

All data covers the Portuguese bidding zone (`BZN|PT`), January 1, 2023 to December 31, 2025, pulled via the `entsoe-py` Python client.

## Methodology

### Data Cleaning
- Parsed and standardized timestamps across all three data sources (mixed hourly/15-minute resolution in the price series was resampled to hourly)
- Handled DST transitions: dropped duplicate timestamps from the autumn clock change, dropped non-existent timestamps from the spring clock change 
- Verified data integrity by cross-checking suspicious repeated values against known market mechanisms; confirmed as genuine market behavior rather than a data artifact
- Final merged dataset: 26,301 hourly rows × 12 columns (price, 10 generation types, load)

### Renewable Share Definition
Renewable generation = Biomass + Hydro (run-of-river + reservoir) + Solar + Wind (onshore + offshore). The data sets iclude Hydro Pumped Storage, however this is excluded because is was classified as storage/consumption, not primary generation.

## Key Findings

### 1. Renewable Share Is Strongly, Negatively Correlated With Price
Across the full 3-year dataset, renewable share and day-ahead price show a correlation of **-0.667**. This is a non-linear relationship: price variance increases sharply as renewable share rises above ~50%, consistent with a "funnel" pattern where high-renewable hours produce both very low (near-zero or negative) and occasionally still-high prices, depending on demand.

### 2. Zero/Negative Pricing Is a Structural, Growing Phenomenon
Hours with day-ahead prices at or below €0/MWh:
- Occur almost exclusively above ~65% renewable share
- Concentrate heavily in spring months (March–April) and midday hours (9:00–18:00) — consistent with simultaneous hydro, wind, and solar output overlapping with moderate demand
- **Increased sharply year over year**: negligible in 2023, peaking in April 2024, remaining elevated in 2025

This indicates the pattern is driven by structural renewable capacity growth, not one-off weather events, though weather amplifies it in specific years. This can be a consequence of climate change, as seasons get more intense each year.

### 3. Predictive Modeling
Two models were built to predict day-ahead price from renewable share, load, hour, month, and year:

| Model | R² | MAE (EUR/MWh) |
|---|---|---|
| Linear Regression | 0.513 | 26.07 |
| Random Forest (100 trees) | **0.774** | **15.88** |

The Random Forest's substantial improvement confirms that price relationships with time-of-day and season are non-linear. Feature importance confirms renewable share as the dominant predictor (55.9% of total importance), ahead of month (15.4%), load (12.0%), year (10.5%), and hour (6.1%).

### 4. Battery Storage Arbitrage Value Is Rising
A theoretical 1 MW / 4 MWh battery (90% round-trip efficiency, one charge/discharge cycle per day, charging during the 4 cheapest hours and discharging during the 4 priciest hours of each day) was modeled across the full period:

| Year | Avg. Daily Profit | Annual Total |
|---|---|---|
| 2023 | €189.83 | €69,288 |
| 2024 | €196.33 | €71,858 |
| 2025 | €279.67 | €102,080 |

Arbitrage value rose **~47% from 2023 to 2025**, tracking the same renewable-driven volatility increase identified in Finding 2. This suggests that as Portugal's grid adds renewable capacity, both curtailment risk and storage investment value are increasing together — a signal relevant to grid-scale battery investment decisions.

## Limitations

- The storage model assumes perfect foresight (optimal hour selection with no forecasting error) and does not enforce charge-before-discharge sequencing, so results represent a theoretical upper bound rather than an achievable real-world strategy
- Regression models use only within-dataset features; external drivers of price (gas prices, interconnector flows with Spain, plant outages) are not included
- Analysis covers only the Portuguese bidding zone; MIBEL is a shared Iberian market with Spain, which was not incorporated

## Tools Used

Python (pandas, numpy, matplotlib, scikit-learn), `entsoe-py` (ENTSO-E API client)

## Possible Extensions

- Gradient boosting models (XGBoost) and hyperparameter tuning
- Charge-before-discharge sequencing constraint in the arbitrage model
- Cross-border comparison with the Spanish market
- Interactive dashboard (Power BI) for non-technical stakeholder exploration