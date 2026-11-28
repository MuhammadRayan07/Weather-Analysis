# Weather Data Analysis and Prediction

Programming for AI, semester project. Instructor: Mahhek Tahir.
Team: 25L-0015, 25L-0032.

## Overview
This project collects ten years (2016 to 2025) of daily weather data for 20 cities from the Open-Meteo Historical Weather API, cleans and explores it, and builds two supervised models:
- **Regression:** predict the next day's maximum temperature.
- **Classification:** predict whether a day is rainy (precipitation above 0 mm) or non-rainy.

## Data source and attribution
Data: Open-Meteo Historical Weather API (https://open-meteo.com/en/docs/historical-weather-api), endpoint `https://archive-api.open-meteo.com/v1/archive`. The API is based on reanalysis datasets (ERA5, ERA5-Land and ECMWF IFS). We did not select a model, so the API's default selection was used. Values are model-based estimates for a grid cell, not weather-station measurements. Attribution: Open-Meteo. Licence and terms: https://open-meteo.com/en/features

## Repository contents
| Path | Description |
|---|---|
| `notebooks/01_data_collection.ipynb` | API collection, data collection log (roll numbers, times, requests, rows, SHA-256), data dictionary |
| `notebooks/02_analysis_modeling.ipynb` | Dataset understanding, cleaning, EDA, skewness, outliers, feature selection, models, evaluation, model training log |
| `data/weather_combined.csv` | Combined raw dataset: 73,060 rows (20 cities x 3,653 days), with a `collected_at` column |
| `data/city_reference.csv` | City, country, latitude, longitude |
| `data/raw_logged/` | Original API JSON responses per city and `collection_log.csv` |
| `data/weather_with_missing.csv` | Working copy with about 5% artificial missing values in input features |
| `data/weather_cleaned.csv` | Final cleaned dataset (imputed, outliers capped, skewed variables transformed) with a train/test column |

## Proof of own work
- Collection run: 2026-11-22, 14:39:50 to 15:07:13 UTC, 73,060 rows.
- SHA-256 of `weather_combined.csv`: `4cb30dc95f0b64192a1952589d81a3e468dd7d5b90192a5117a0faad3be0a1c8`
- SEED = 15 (last four digits of the lowest roll number), used for every random_state.

## How to run
1. Put the project in a Google Drive folder `weather-analysis` with subfolders `data/raw_logged` and `data/processed` (the notebooks use Colab and mount Drive).
2. Run `01_data_collection.ipynb` to collect the data (this takes about half an hour because of API rate limits), or copy `data/weather_combined.csv` into `data/processed`.
3. Run `02_analysis_modeling.ipynb` from the top. The feature selection cells are slow.

## Method summary
- Chronological 80:20 train/test split within each city; all fitted preprocessing uses training data only.
- Imputation (per-city mean, median or circular mean), one-hot encoding, scaling, outlier capping at the 99.9th percentile, log1p and sqrt transforms for skewed variables.
- Forward and backward feature selection with time-aware cross-validation.
- Three models per task, time-aware cross-validation, and hyperparameter tuning of Gradient Boosting.
- Advanced techniques: cyclical encoding (wind direction and day of year) and PCA (temperature variables).

## Main results (test set)
| Task | Best model | Result |
|---|---|---|
| Next-day maximum temperature | Gradient Boosting (tuned) | MAE 1.714 °C, RMSE 2.395, R² 0.946 |
| Rainy vs non-rainy day | Gradient Boosting | Accuracy 0.849, F1 0.851, AUC 0.932 |

Predicting that tomorrow equals today gives an MAE of 1.838 °C, so the regression models improve on it by about 0.1 °C. Cyclical encoding made almost no difference, and PCA made the regression models slightly worse.

## Limitations
Reanalysis data is not station data, the test period covers only 2024 to 2025, and performance varies by city (see Section 13.7 of the notebook).
