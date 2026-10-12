# Urban Air Quality Prediction Dashboard

An end-to-end data science project for exploring historical air quality, studying its relationship with weather, predicting next-hour pollutant concentrations, and presenting air-quality patterns through an interactive Microsoft Power BI report for **Ambazari, Nagpur, Maharashtra, India**.

The project uses hourly CPCB/MPCB monitoring data for 2024–2025 and historical weather data from Open-Meteo. The current Power BI report is a **historical AQI and pollutant-analysis dashboard**; the machine-learning models are developed and evaluated separately in the notebooks.

## Project Objectives

- Build a reproducible workflow for understanding and cleaning air-quality and weather data.
- Compare missing-value imputation techniques using masked validation.
- Integrate the datasets on timestamps and engineer temporal and lag-based features.
- Explore pollutant distributions, seasonal patterns, correlations, and weather relationships.
- Train and evaluate models for one-hour-ahead prediction of six pollutants.
- Calculate an AQI-style index from pollutant sub-indices and communicate historical patterns in Power BI.

## Data Sources and Pollutants

### Air Quality

- **Location:** Ambazari, Nagpur monitoring site.
- **Period:** 2024 and 2025.
- **Frequency:** Hourly.
- **Source:** CPCB/MPCB monitoring data supplied as raw CSV files.

### Weather

Historical hourly weather data was obtained from **Open-Meteo** for the Nagpur location. The integrated and selected features include temperature, relative humidity, wind speed, wind direction, and weather code. CPCB measurements also include weather-related fields such as rainfall and solar radiation in the cleaned station data; the feature-engineered modelling dataset uses the selected weather fields.

### Pollutants

| Variable | Meaning | Unit used in the project |
|---|---|---|
| `pm25` | PM2.5 particulate matter | µg/m³ |
| `pm10` | PM10 particulate matter | µg/m³ |
| `no2` | Nitrogen dioxide | µg/m³ |
| `so2` | Sulphur dioxide | µg/m³ |
| `co` | Carbon monoxide | mg/m³ |
| `o3` | Ozone | µg/m³ |

**Traffic data is not included.** A suitable hourly traffic source aligned to the monitoring location was not available for this version of the project.

## Repository Workflow and Notebooks

Run the notebooks in numerical order. Paths in the notebooks are relative to the project layout, so open them from the repository and adjust paths only if your local folder structure differs.

| Notebook | Purpose |
|---|---|
| `notebooks/01_data_understanding.ipynb` | Inspect raw CPCB 2024/2025 and weather data: shapes, columns, data types, timestamp coverage, missingness, and temporal structure. |
| `notebooks/02_data_cleaning.ipynb` | Standardize names and timestamps, examine pollutant missingness, compare imputation methods with masked validation, impute missing pollutant values, and check negative concentrations. |
| `notebooks/03_data_integration.ipynb` | Combine the 2024 and 2025 station datasets and join the station and weather datasets by timestamp. |
| `notebooks/04_feature_engineering.ipynb` | Create time/season variables, cyclic encodings, pollutant lags and rolling statistics, and next-hour target columns. |
| `notebooks/05_eda.ipynb` | Explore distributions, outliers, temporal and seasonal patterns, correlations, and target-related feature relationships; select a reduced modelling feature set. |
| `notebooks/06_machine_learning.ipynb` | Compare persistence, Random Forest, Gradient Boosting, and XGBoost candidates using chronological validation; evaluate selected models on a held-out test period and save metrics, predictions, and models. |
| `notebooks/07_aqi_analysis.ipynb` | Calculate pollutant sub-indices and overall AQI-style values, assign AQI categories, validate results, and export the dataset used by Power BI. |

## Data Preparation and Feature Engineering

### Cleaning and Imputation

The raw CPCB files contain hourly records for 2024 (8,784 rows) and 2025 (8,760 rows); the Open-Meteo weather data contains 17,544 hourly records.

The cleaning notebook compares:

- Mean imputation
- Median imputation
- K-nearest-neighbours (KNN) imputation
- MICE using scikit-learn's `IterativeImputer`

For the comparison, 10% of values in complete 2024 pollutant rows were artificially masked, and MAE and RMSE were calculated on the hidden values. MICE was selected as the primary imputation approach because it achieved the lowest RMSE for all six pollutants and the lowest MAE for five of six pollutants in that experiment.

The notebook emits convergence warnings for some iterative-imputation runs; this should be considered when reproducing or further tuning the method. Negative pollutant concentrations are checked and clipped to zero.

### Integration

The 2024 and 2025 station records are concatenated, sorted by timestamp, and joined with weather data on `timestamp`. The integrated hourly table contains **17,544 rows** before feature engineering.

### Feature Engineering and Feature Selection

Features include hour, day, month, year and weekday information, weekend and season indicators, pollutant lag features, rolling means, and cyclic encodings for hour, month, and wind direction.

Targets are defined for **one hour ahead** for PM2.5, PM10, NO2, SO2, CO, and O3. Feature selection retains current pollutant and weather variables, temporal/cyclic variables, and lag/rolling features selected against their corresponding pollutant target using a correlation threshold of **0.55**.

The selected modelling dataset exported from the feature-selection workflow is:

`data/processed/Ambazari_Nagpur_Selected_Features_Dataset_2024_2025.csv`

The AQI notebook loads a 46-column version of this dataset with **17,519 rows**, covering `2024-01-02 00:00` through `2025-12-31 22:00`, with no missing values or duplicate timestamps in the loaded selected-feature table. The notebook then adds AQI and analysis columns and exports the 63-column dashboard dataset described below.

## Machine-Learning Approach and Results

The target is the next-hour concentration of each pollutant. The modelling notebook uses a chronological split of **70% training, 15% validation, and 15% test**; validation RMSE is used for model selection, and the test split is reserved for final evaluation. Original-scale and `log1p`-transformed target approaches are compared. Metrics reported are MAE, RMSE, and R².

### Final Held-Out Test Results

| Pollutant | Selected approach | Target transform | MAE | RMSE | R² |
|---|---|---|---:|---:|---:|
| CO | XGBoost | Log | 0.1429 | 0.2408 | 0.7198 |
| NO2 | XGBoost | Log | 4.3644 | 7.4594 | 0.7830 |
| O3 | Random Forest | Log | 4.6369 | 7.4780 | 0.9218 |
| PM10 | Random Forest | Log | 6.2323 | 10.2240 | 0.9427 |
| PM2.5 | Random Forest | Original | 5.3939 | 8.3499 | 0.9083 |
| SO2 | Gradient Boosting + persistence blend | Original | 4.5806 | 12.2117 | 0.6857 |

For SO2, the final prediction is a blend of **75% Gradient Boosting and 25% persistence**. The notebook saves validation results, final test metrics, predictions, tuning outputs, feature-importance outputs, model files, and model metadata in the configured output/model folders.

Metrics should be interpreted in the pollutant's concentration units; CO is measured in mg/m³ and the other listed pollutants in µg/m³.

## AQI Analysis

`notebooks/07_aqi_analysis.ipynb` creates rolling pollutant averages, calculates a sub-index for each pollutant, sets overall AQI to the maximum available pollutant sub-index, and assigns a category.

The AQI category thresholds used are:

| Category | AQI range |
|---|---:|
| Good | 0–50 |
| Satisfactory | 51–100 |
| Moderate | 101–200 |
| Poor | 201–300 |
| Very Poor | 301–400 |
| Severe | 401–500 |

The notebook uses concentration breakpoints and linear interpolation to estimate pollutant sub-indices. It uses 24-hour rolling averages for PM2.5, PM10, NO2, and SO2, and 8-hour rolling averages for CO and O3.

The strict full-window requirement means initial rolling-average values are unavailable; consequently, the exported dataset contains **17,519 rows, 17,496 valid overall AQI values, and 23 missing AQI values**. These missing AQI rows should not be interpreted as zero AQI.

**Methodology note:** This is a project implementation informed by the CPCB AQI reference, not a claim of official CPCB certification. The exact handling of breakpoint endpoints and interpolation is implemented in the notebook. Check the calculation against the latest official CPCB guidance before using the values for regulatory or public-health decisions.

### Recorded Findings from the Analysis

- Average AQI was **104.27 in 2024** and **89.19 in 2025**, a decrease of approximately **14.5%** between the two yearly averages.
- PM10 was identified as the dominant pollutant in **64.84%** of valid dominant-pollutant records; PM2.5 accounted for **30.24%**, O3 for **3.72%**, and CO for **1.21%**.
- Monthly analysis showed a higher average AQI in January and a lower average AQI in August in the analysed period.
- The calculated AQI ranged from **22 to 307** in the notebook output.

These are descriptive findings from the available two-year dataset and do not, by themselves, establish the causes of changes in air quality.

### AQI Dataset Exported for Power BI

`data/processed/Ambazari_Nagpur_AQI_Analysis_2024_2025.csv`

- **Rows:** 17,519
- **Columns:** 63
- Includes the original selected features, pollutant rolling averages and sub-indices, overall `AQI`, `AQI_Category`, and time/season analysis fields.

## Power BI Dashboard

The saved report is `POWER BI DASHBOARD.pbix`. It uses the AQI analysis CSV and contains six report pages:

1. **AQI Overview** — average, minimum, and maximum AQI cards; record count; historical AQI line chart; year and date-range slicers.
2. **Monthly & Category Analysis** — monthly AQI comparison by year, AQI category distribution, and additional monthly/year summaries.
3. **Pollutant Analysis — PM2.5 and PM10** — historical time-series charts for both particulate pollutants.
4. **Pollutant Analysis — NO2 and CO** — historical time-series charts for nitrogen dioxide and carbon monoxide.
5. **Pollutant Analysis — SO2 and O3** — historical time-series charts for sulphur dioxide and ozone.
6. **Insights & Findings** — key findings on yearly AQI change, dominant pollutant, and seasonal variation.

The report has a year slicer and a date-range slicer synced across the analysis pages. The report is for **historical analysis**; it does not currently display the machine-learning forecast outputs as a live prediction visual.

## Project Structure

```text
DHV MINI/
├── data/
│   ├── raw/                         # Original CPCB/MPCB and Open-Meteo files
│   └── processed/
│       ├── CPCB_Ambazari_2024_cleaned.csv
│       ├── CPCB_Ambazari_2025_cleaned.csv
│       ├── OpenMeteo_Nagpur_2024_2025_cleaned.csv
│       ├── Ambazari_Nagpur_Selected_Features_Dataset_2024_2025.csv
│       ├── Ambazari_Nagpur_AQI_Analysis_2024_2025.csv
│       └── model_outputs/           # Metrics, predictions, reports and feature importance
├── models/                          # Saved pollutant prediction models and metadata
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_data_integration.ipynb
│   ├── 04_feature_engineering.ipynb
│   ├── 05_eda.ipynb
│   ├── 06_machine_learning.ipynb
│   └── 07_aqi_analysis.ipynb
├── POWER BI DASHBOARD.pbix
└── README.md
```

Exact generated filenames may differ if notebook output paths have been changed locally. The notebooks are the source of truth for the data-processing and modelling steps.

## Tools and Libraries

- Python
- Jupyter Notebook
- pandas and NumPy
- Matplotlib and plotting libraries used in the notebooks
- scikit-learn (imputation, preprocessing, metrics, Random Forest, Gradient Boosting)
- XGBoost
- joblib (model persistence)
- Microsoft Power BI Desktop
- CPCB/MPCB air-quality data and Open-Meteo historical weather data

## Reproducing the Project

1. Place the raw hourly station CSVs and Open-Meteo weather CSV in the expected `data/raw/` location, preserving or updating the paths in the notebooks.
2. Install Python and Jupyter, then install the libraries imported by the notebooks. XGBoost is needed for the machine-learning notebook.
3. Run notebooks `01` through `07` in order, checking the output paths and generated files after each phase.
4. Open `data/processed/Ambazari_Nagpur_AQI_Analysis_2024_2025.csv` in Power BI Desktop using **Home → Get data → Text/CSV** if the report needs to be rebuilt or reconnected.
5. Save the report as a `.pbix` file. To share it with someone who does not have the source data in the same location, ensure the data source is included or reconnectable and test the report after reopening it.

Because the notebooks use local relative paths and may contain machine-specific output paths, update those paths if your repository layout differs. Re-running all cells may take time, especially during imputation and model tuning.

## Limitations and Future Improvements

- The analysis covers one monitoring location and two calendar years; results may not generalize to other sites or years.
- Traffic data is not included.
- MICE produced convergence warnings in the recorded notebook run and could be tuned or assessed further.
- AQI values are generated by the project's implemented breakpoint/interpolation method and should be checked against current official guidance before operational use.
- The Power BI report currently visualizes historical AQI and pollutant data rather than serving model predictions interactively.
- Future work could add more monitoring sites, suitable traffic features, robust model monitoring, prediction intervals, and a Power BI page for forecast-versus-actual evaluation.

## Project Status

| Phase | Status |
|---|---|
| Data acquisition and understanding | Complete |
| Data cleaning and imputation comparison | Complete |
| Data integration | Complete |
| Feature engineering and feature selection | Complete |
| Exploratory data analysis | Complete |
| Machine-learning training and held-out evaluation | Complete in notebook |
| AQI analysis and dataset export | Complete in notebook |
| Power BI historical dashboard | Built and saved |
| Live model predictions in Power BI | Not currently implemented |
