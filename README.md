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

### Geographic Filtering

National and aggregate geographic categories were removed so that the predictive analysis operates at the individual-province level.

Prince Edward Island remains available for agricultural descriptive and historical-risk analysis but is excluded from climate-enhanced modeling because the climate dataset does not provide a directly representative station.

### Inflation Adjustment

Strong upward trends were observed in nominal farm prices.

Canadian all-items CPI was therefore used to convert prices into constant 1984 dollars. After adjustment, most of the strong nominal price trends weakened substantially or disappeared.

### Historical Features

Lagged agricultural predictors were constructed using **exact calendar years** rather than simply taking the previous available observation.

Examples include:

- Previous-year yield
- Three-year historical yield average
- Previous-year real farm price
- Three-year historical real-price average

This prevents gaps in a crop-province time series from incorrectly turning an older observation into a one-year lag.

### Climate Features

Daily climate observations were aggregated over the May–September growing season.

Engineered predictors include:

- Growing-season mean temperature
- Temperature variability
- Total precipitation
- Precipitation variability
- Number of warm days
- Number of dry days
- Maximum consecutive dry days

Redundant climate variables were evaluated through correlation analysis before final feature selection.

---

## Leakage Prevention

Preventing future information from entering historical predictions was a major consideration throughout the project.

Key safeguards include:

- Chronological rather than random train/test splitting
- Exact calendar-year lag construction
- Historical rolling features based only on previous years
- Exclusion of realized production and total farm value as predictive inputs
- Separate validation and final test periods
- Hyperparameter tuning performed without accessing the final test period

The final test period was not evaluated until model-selection decisions had been completed.

---

## Validation Strategy

The feature-engineered observations were divided chronologically:

- **Training:** 1964–1978
- **Validation:** 1979–1981
- **Final Test:** 1982–1984

Model tuning used expanding-window chronological cross-validation within the training period.

This design more closely represents the intended forecasting problem than a random train/test split because models are always evaluated on observations occurring later in time.

---

## Yield Model Results

Ridge Regression, Random Forest, Gradient Boosting, historical baselines, and several alternative trend specifications were evaluated.

Ridge Regression regularization was tuned using expanding-window cross-validation, resulting in a selected alpha of **10**.

On the 1979–1981 validation period, the tuned climate-enhanced Ridge model achieved:

| Metric | Result |
|---|---:|
| MAE | 579.44 kg/ha |
| RMSE | 1,635.51 kg/ha |
| R² | 0.973 |

The model outperformed the three-year historical baseline across all three pooled validation metrics.

On the untouched 1982–1984 test period:

| Approach | MAE | RMSE | R² |
|---|---:|---:|---:|
| Climate-Enhanced Ridge | 585.85 kg/ha | 1,386.28 kg/ha | 0.977 |
| Three-Year Historical Baseline | 565.61 kg/ha | 1,376.33 kg/ha | 0.978 |

Although Ridge was correctly selected using validation data, the historical baseline slightly outperformed it on the final test period. The model was not changed after observing this result in order to preserve the integrity of the final test evaluation.

---

## Yield Model Interpretation

Permutation importance showed that recent historical yield was by far the strongest source of predictive information.

The most influential predictors were:

1. Three-year historical yield average
2. Previous-year yield
3. Growing-season temperature variability
4. Number of warm days
5. Province

Climate variables therefore provided supplementary predictive information, but recent agricultural history remained dominant.

Permutation importance was calculated only after final model selection and test evaluation and was used for post-hoc interpretation rather than additional feature selection or tuning.

---

## Real Farm-Price Results

The previous-year inflation-adjusted farm price outperformed the candidate machine-learning models during validation.

On the final 1982–1984 test period, this approach achieved:

| Metric | Result |
|---|---:|
| MAE | $29.69/tonne |
| RMSE | $54.72/tonne |
| R² | 0.673 |

Price performance varied substantially across crops, indicating that price uncertainty is a significant challenge in the combined revenue system.

Because the selected price approach is a single historical predictor rather than a multivariable machine-learning model, conventional feature-importance analysis is not applicable to the final price forecast.

---

## Gross Revenue Results

The final two-stage system combines the selected yield estimate with the previous-year real-price forecast.

On the common final test population, gross-revenue prediction achieved:

| Metric | Result |
|---|---:|
| MAE | $88.07/ha |
| RMSE | $153.65/ha |
| R² | 0.718 |

Prediction accuracy varied substantially by crop, demonstrating the importance of evaluating crop-level performance rather than relying exclusively on pooled metrics.

---

## Revenue and Risk Insights

Historical normalized revenue risk ranged from approximately:

- **16.34% — Spring wheat**
- **43.72% — Sunflower seed**

The analysis showed that expected gross revenue and historical risk do not necessarily move together.

For example:

- **Sugar beets** produced the highest predicted gross revenue at approximately **$1,472/ha**, but historical normalized risk was relatively high at **33.53%**.
- **Corn for silage** produced expected gross revenue of approximately **$1,004/ha** with normalized risk of **24.45%**.
- **Corn for grain** produced expected gross revenue of approximately **$860/ha** with normalized risk of **22.02%**.
- **Spring wheat** produced lower expected revenue of approximately **$438/ha**, but had the lowest normalized historical risk at **16.34%**.

These results illustrate why crop comparisons should consider expected return, historical variability, and forecast reliability together.

---

## Limitations

Several limitations should be considered when interpreting the results.

**Historical period:** The final analysis covers 1961–1984. Agricultural technology, crop varieties, climate conditions, government programs, and commodity markets have changed substantially since this period.

**Climate timing:** The yield model uses realized May–September weather and should therefore be interpreted as an in-season estimation model rather than a pre-plant forecast.

**Geographic resolution:** Available city-level weather stations were mapped to provinces, simplifying potentially substantial within-province climate variation.

**Price uncertainty:** Farm prices were less consistently predictable than crop yield. Additional economic variables could potentially improve price forecasting.

**Gross revenue rather than profit:** Production costs are not available. Higher expected gross revenue therefore does not necessarily indicate higher profitability.

**Crop sample sizes:** Some crops have relatively few observations during the final test period, making their crop-level error estimates less stable.

The results should therefore be interpreted as a historical decision-support analysis and modeling framework rather than current planting recommendations.

---

## Future Work

Potential extensions include:

- Incorporating more recent agricultural and climate observations
- Adding commodity futures and market-price information
- Incorporating exchange rates, inventories, and trade variables
- Adding crop-specific production costs to estimate profit
- Using more spatially detailed weather information
- Incorporating seasonal weather forecasts for true pre-plant prediction
- Developing probabilistic yield, price, and revenue forecasts
- Evaluating prediction intervals rather than point estimates alone

---

## Running the Project

1. Clone or download the repository.
2. Confirm that the three CSV files are located inside the `data/` directory.
3. Open `Canadian_agricultural_planning.ipynb` in Jupyter Notebook or JupyterLab.
4. Restart the kernel to ensure a clean environment.
5. Select **Run All** to execute the notebook sequentially.

The notebook is designed to use relative file paths, so no machine-specific path changes should be required when the repository structure is preserved.

---

## Project Files

```text
Canadian_agricultural_planning.ipynb
    Main analysis, preprocessing, modeling, evaluation, and conclusions.

data/farm_production_dataset.csv
    Historical Canadian agricultural data.

data/Canadian_climate_history.csv
    Historical Canadian climate data.

data/canadian_cpi.csv
    Canadian all-items CPI data.

README.md
    Project overview, methodology, key findings, and usage instructions.

.gitignore
    Excludes local development and temporary files from version control.
```

---

## Final Takeaway

The project demonstrates that model complexity does not automatically translate into better agricultural forecasts. Recent historical yield was exceptionally informative, and growing-season climate provided additional predictive value, but a simple historical yield baseline remained highly competitive. Similarly, previous-year real farm price outperformed more complex price models.

The strongest decision-support framework was therefore not the most complex possible system, but the one supported by chronological validation and honest comparison against strong baselines. Combining these predictions with historical revenue risk provides a more complete view of agricultural uncertainty than expected revenue alone.
