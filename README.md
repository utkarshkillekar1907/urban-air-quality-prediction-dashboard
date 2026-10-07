# Urban Air Quality Prediction Dashboard

## Project Progress — Data Preparation & EDA

This project focuses on building an end-to-end system for urban air-quality analysis and prediction using historical air-quality and weather data from Nagpur.

### Completed So Far

#### 1. Data Acquisition
- Collected hourly air-quality data for **Ambazari, Nagpur (MPCB/CPCB)** for 2024 and 2025.
- Required pollutants:
  - PM2.5
  - PM10
  - NO2
  - SO2
  - CO
  - O3
- Collected hourly historical weather data using **Open-Meteo**:
  - Temperature
  - Relative humidity
  - Wind speed
  - Wind direction
  - Weather condition/code
- Traffic data was not included due to the lack of a suitable hourly, location-aligned source and is documented as a project limitation.

#### 2. Data Understanding
- Inspected dataset structure, data types, date ranges and missing values.
- Checked duplicate timestamps.
- Analyzed missing-data patterns across pollutants and weather variables.
- Verified the hourly temporal structure of the datasets.

#### 3. Data Cleaning & Missing-Value Imputation
- Selected project-relevant variables from the raw CPCB data.
- Standardized column names and timestamp formats.
- Compared:
  - Mean imputation
  - Median imputation
  - KNN imputation
  - MICE (Iterative Imputation)
- Used masked validation with **MAE and RMSE** to compare methods.
- MICE performed best overall and was selected as the primary imputation method.
- Negative pollutant values were checked and invalid negative values were clipped to zero.
- Cleaned datasets were saved separately without modifying the raw data.

#### 4. Data Integration
- Combined 2024 and 2025 CPCB air-quality data.
- Integrated the data with hourly Open-Meteo weather data using timestamps.
- Open-Meteo weather variables were used as the primary weather features.
- Final integrated dataset contains **17,544 hourly observations** with no missing values or duplicate timestamps.

#### 5. Feature Engineering
Created features to capture temporal and historical pollution patterns:

- Hour, day, month, year and day-of-week
- Weekend indicator
- Season
- Lag features
- Rolling mean features
- Cyclic encoding for:
  - Hour
  - Month
  - Wind direction

The prediction task was defined as **1-hour-ahead prediction** for all six pollutants:

- PM2.5
- PM10
- NO2
- SO2
- CO
- O3

The feature-engineered dataset contains **17,519 model-ready observations**.

#### 6. Exploratory Data Analysis
Performed EDA covering:

- Pollutant distributions
- Outlier analysis
- Hourly pollution patterns
- Monthly pollution patterns
- Seasonal patterns
- Weather–pollution relationships
- Correlation analysis
- Feature-to-target correlation analysis

Major observations included strong temporal dependence in pollutant concentrations and strong relationships between current/lagged pollutant values and their corresponding 1-hour-ahead targets.

#### 7. Feature Selection
A reduced dataset was created using:

- All important current pollutant features
- Important weather features
- Cyclic time and wind-direction features
- Only lag and rolling features showing strong correlation with their **corresponding pollutant target**

A correlation threshold of **0.55** was used for selecting lag and rolling features.

### Current Dataset

The selected dataset contains:

- **17,519 rows**
- **49 columns**
- **0 missing values**
- **0 duplicate timestamps**

File:

`data/processed/Ambazari_Nagpur_Selected_Features_Dataset_2024_2025.csv`

### Project Status

| Phase | Status |
|---|---|
| Data Acquisition | ✅ Complete |
| Data Understanding | ✅ Complete |
| Data Cleaning | ✅ Complete |
| Missing-Value Analysis & Imputation | ✅ Complete |
| Data Integration | ✅ Complete |
| Feature Engineering | ✅ Complete |
| Exploratory Data Analysis | ✅ Complete |
| Feature Selection | ✅ Complete |
| Machine Learning | ⏳ Next |
| Model Evaluation | ⏳ Pending |
| Power BI Dashboard | ⏳ Pending |

## Next Phase

The next phase is **Machine Learning**, where multiple regression models will be trained to predict the 1-hour-ahead values of all six pollutants using a chronological train/test split.