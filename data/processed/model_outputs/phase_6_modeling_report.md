# Phase 6 — Machine Learning Report

## Objective

Predict next-hour concentrations of PM2.5, PM10, NO2, SO2, CO, and O3
using hourly air-quality and weather data from Ambazari, Nagpur.

## Evaluation approach

- Chronological split: 70% training, 15% validation, 15% test.
- Model selection based on validation RMSE.
- Test set reserved for final evaluation.
- Original and log1p-transformed target approaches compared.
- Metrics: MAE, RMSE, and R-squared.
- SO2 final prediction uses a 75% Gradient Boosting + 25% persistence blend.

## Final test results

| Pollutant | Model | Transform | MAE | RMSE | R² |
|---|---|---|---:|---:|---:|
| CO | XGBoost | Log | 0.1429 | 0.2408 | 0.7198 |
| NO2 | XGBoost | Log | 4.3644 | 7.4594 | 0.7830 |
| O3 | Random Forest | Log | 4.6369 | 7.4780 | 0.9218 |
| PM10 | Random Forest | Log | 6.2323 | 10.2240 | 0.9427 |
| PM25 | Random Forest | Original | 5.3939 | 8.3499 | 0.9083 |
| SO2 | Gradient Boosting + Persistence | Original | 4.5806 | 12.2117 | 0.6857 |