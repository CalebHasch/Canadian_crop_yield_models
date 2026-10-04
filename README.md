# Predicting Crop Yield and Revenue for Data-Driven Agricultural Planning in Canada

## Project Overview

Canadian agricultural producers must make planting and land-allocation decisions while facing uncertainty in crop yield, market prices, and weather conditions. This project develops a two-stage data science framework for estimating crop yield and inflation-adjusted farm price and combining those estimates to evaluate expected gross revenue per hectare.

The analysis integrates historical Canadian agricultural production data, climate observations, and Consumer Price Index (CPI) data. In addition to predictive performance, the project evaluates historical revenue variability so that expected revenue can be considered alongside risk.

The intended stakeholder is a Canadian agricultural producer or farm manager evaluating crop-selection and land-allocation decisions.

> **Note:** The financial outcome modeled in this project is **gross revenue rather than profit** because crop-specific production-cost data are not available.

---

## BLUF — Key Results

The final analysis uses a **1961–1984 modeling period**, selected after evaluating agricultural and climate-data coverage.

The primary findings are:

- The selected climate-enhanced **Ridge Regression yield model** achieved a final test **MAE of 585.85 kg/ha** and **R² of 0.977**.
- A simple three-year historical yield baseline remained extremely competitive and slightly outperformed Ridge on the untouched test period, with an **MAE of 565.61 kg/ha** and **R² of 0.978**.
- Growing-season climate improved Ridge validation MAE by approximately **4.8%** compared with the historical-only Ridge specification.
- Recent historical yield was the strongest source of predictive information in the final yield model, particularly the three-year historical average and previous-year yield.
- Adjusting historical prices for inflation removed most of the strong upward trends visible in nominal farm prices.
- For real farm-price forecasting, the **previous-year inflation-adjusted price baseline outperformed the tested machine-learning models**.
- The final price approach achieved a test **MAE of $29.69/tonne** and **R² of 0.673**.
- Combining yield and price estimates produced a gross-revenue test **MAE of $88.07/ha** and **R² of 0.718**.
- Historical normalized revenue risk ranged from approximately **16.34% for spring wheat** to **43.72% for sunflower seed**.
- Expected revenue and historical risk did not move together perfectly, demonstrating the value of considering both potential return and historical variability when comparing crops.

Overall, the analysis found that recent agricultural history provides exceptionally strong predictive information for crop yield, while growing-season climate provides additional value. Farm prices were substantially more difficult to predict consistently, making price uncertainty an important contributor to revenue uncertainty.

---

## Business Problem

Agricultural planning requires decisions before final crop yield and market price are known. Crop performance varies across geography and time, while weather and changing market conditions introduce additional uncertainty.

This project addresses the following questions:

1. How accurately can historical agricultural and climate information estimate crop yield?
2. How accurately can historical information forecast inflation-adjusted average farm price?
3. How do yield and price behave over time across different crops?
4. What historical relationship exists between yield and price?
5. Can separate yield and price estimates be combined into useful estimates of gross revenue per hectare?
6. How does expected gross revenue compare with historical revenue risk across crops?
7. Which variables contribute most strongly to yield predictions?

---

## Project Approach

The project follows a two-stage predictive framework.

### Stage 1 — Yield Estimation

Crop yield is modeled in kilograms per hectare using:

- Crop type
- Province
- Historical yield
- Historical rolling yield
- Time
- Growing-season temperature
- Growing-season precipitation
- Temperature and precipitation variability
- Warm and dry weather indicators

Multiple approaches were evaluated, including Ridge Regression, Random Forest, Gradient Boosting, historical baselines, trend-based models, crop-specific trends, and trend-residual modeling.

The final selected machine-learning approach is a **climate-enhanced Ridge Regression model**.

Because the climate predictors contain realized May–September weather from the prediction year, this model should be interpreted as an **in-season yield estimation model**, not a purely pre-plant forecast.

### Stage 2 — Real Farm-Price Forecasting

Historical farm prices were adjusted for inflation using Canada's all-items CPI and expressed in constant **1984 dollars**.

Ridge Regression, Random Forest, Gradient Boosting, and historical-price baselines were evaluated.

The strongest validation performance came from the simple **previous-year real farm-price baseline**, which was therefore retained as the final price forecasting strategy.

### Stage 3 — Expected Gross Revenue

Predicted yield is converted from kilograms per hectare to tonnes per hectare and combined with predicted real farm price to estimate expected gross revenue per hectare.

The resulting metric represents **gross revenue before production expenses**, not profit.

### Stage 4 — Historical Risk

Historical real gross revenue is calculated using observed yield and inflation-adjusted farm price.

Linear time trends are removed within crop-province histories before measuring variability. Risk is defined using the standard deviation of these detrended revenue residuals and is normalized relative to each crop's mean historical real revenue.

This approach preserves the historical joint behavior of yield and price instead of assuming the two variables are statistically independent.

---

## Data Sources

Three datasets are used in the project and are included in the `data/` directory.

### Farm Production Data

**File:** `farm_production_dataset.csv`

Contains historical Canadian agricultural information including:

- Year
- Province/geographic region
- Crop type
- Average farm price
- Average yield
- Production
- Seeded area
- Total farm value

The raw agricultural data cover a longer historical period, but the final modeling population is restricted based on agricultural and climate coverage.

### Canadian Climate History

**File:** `Canadian_climate_history.csv`

Contains daily historical climate observations for Canadian cities, including:

- Mean temperature
- Total precipitation

Daily observations are mapped from available weather stations to provinces and aggregated into May–September growing-season features.

### Canadian Consumer Price Index

**File:** `canadian_cpi.csv`

Contains monthly Canadian all-items Consumer Price Index observations.

Monthly CPI values are converted to annual averages and used to express historical farm prices in constant **1984 dollars**.

---

## Data Usage Guide

The notebook expects the following repository structure:

```text
project/
│
├── Canadian_agricultural_planning.ipynb
├── README.md
├── .gitignore
│
└── data/
    ├── farm_production_dataset.csv
    ├── Canadian_climate_history.csv
    └── canadian_cpi.csv
```

The notebook defines the data directory using a relative path:

```python
from pathlib import Path

DATA_DIR = Path("data")

FARM_DATA_PATH = DATA_DIR / "farm_production_dataset.csv"
CLIMATE_DATA_PATH = DATA_DIR / "Canadian_climate_history.csv"
CPI_DATA_PATH = DATA_DIR / "canadian_cpi.csv"
```

As long as the three datasets remain in the `data/` directory with these filenames, the notebook can load them without modifying local absolute paths.

---

## Data Preparation

Several preprocessing decisions were made after exploratory analysis.

### Historical Period

The final modeling period is **1961–1984**.

This period was selected after examining both agricultural and climate coverage. Earlier years contained substantial climate-data gaps for important stations, while coverage became substantially more complete beginning in 1961.

### Geographic
