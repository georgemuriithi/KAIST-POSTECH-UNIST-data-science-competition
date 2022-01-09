# Shale Gas Wells Productions Prediction
https://www.datascience-contest.com

<p align="left">
    <a href="https://github.com/georgemuriithi/KAIST-POSTECH-UNIST-data-science-contest/blob/main/LICENSE">
        <img alt="License" src="https://img.shields.io/github/license/georgemuriithi/KAIST-POSTECH-UNIST-data-science-contest.svg?color=blue&cachedrop">
    </a>
</p>

The Korea National Oil Corporation is interested in purchasing shale gas wells from the United States of America, and needs to predict their productions in order to select the wells that maximize profit.

A combination of **LightGBM Regressor** and **Exponential smoothing** is used to predict the productions and Integer programming using **Gurobi** is used for optimization to maximize profit. Evaluation is based on **sMAPE (symmetric mean absolute percentage error).** Our team ranked among the best with a percentage error of **25.54%.** The best team had a percentage error of **19.49%.**

## Problem description
### Data

*Unfortunately, the train and exam datasets are confidential and therefore, will not be provided in this repository.*

- **trainSet.csv** - Data of 280 shale gas wells for training models
- **examSet.csv** - Data of 44 shale gas wells for prediction

### Predicting gas production
Predict the monthly average gas productions of 44 shale gas wells given in **examSet.csv** for the next 6 months.

Evaluation will be based on **sMAPE (symmetric mean absolute percentage error):**

<p align="center">
  <img src="https://user-images.githubusercontent.com/21691211/148675936-b3f0def1-44fa-4d76-a9b4-05bc79049fca.png">
</p>

- F<sub>i</sub> - predicted monthly average gas production of the i<sup>th</sup> gas well over the next 6 months
- A<sub>i</sub> - actual monthly average gas production of the i<sup>th</sup> gas well over the next 6 months
- n - number of gas wells (44 in this problem)

### Investment decision
Suppose that a budget of $15,000,000 is given. Purchase gas wells among the 44 candidates given in **examSet.csv** to maximize profit.

<p align="center">
  <img src="https://user-images.githubusercontent.com/21691211/148675948-b08621d8-68cf-4fa3-82a5-467c3b973347.png">
</p>

- A<sub>i</sub> - actual monthly average gas production of the i<sup>th</sup> gas well over the next 6 months
- P<sub>i</sub> - price of the i<sup>th</sup> gas well
- P<sub>s</sub> - shale gas price ($5 per 1 Mcf)
- C<sub>i</sub> - monthly operation cost of the i<sup>th</sup> gas well
- X<sub>i</sub> - decision variable to purchase the i<sup>th</sup> gas well (If purchasing the i<sup>th</sup> gas well: X<sub>i</sub> = 1, if not: X<sub>i</sub> = 0)

## Solution approach
The wells are divided into **new wells** and **old wells**. New wells do not have data on gas production per month, non-gas production per month and hours operated per month. This data is available in old wells.

Therefore, **regression** is used to predit the monthly average productions of **new wells for the first 6 months** and **exponential smoothing** is used to predict the monthly average productions of **old wells for the last 6 months.**

### New wells
For regression, the following **advanced decision tree-based models** are tested:

- `BaggingRegressor`
  - `n_estimators=50`
- `RandomForestRegressor`
  - `n_estimators=50`
- `XGBRegressor`
  - `max_depth=5`
  - `objective='reg:squarederror'`
- `LGBMRegressor`
- `VotingRegressor`
  - `estimators=[bagging, random_forest, xgb, lgbm]`

`LGBMRegressor` turns out as the best performing, with the minimum **sMAPE**.

### Old wells
The following **exponential smoothing models** are used:

- `SimpleExpSmoothing`
  - `smoothing_level=0.2`
  - `smoothing_level=0.6`
  - optimized smoothing level
- `Holt`
  - Additive model
  - Multiplicative model
  - Damped additive model
  - Damped multiplicative model
- `ExponentialSmoothing`
  - `use_boxcox=True`
    - Additive model
    - Damped additive model

Depending on the model with the minimum **SSE (sum of squared error)** for each well, different models are used to forecast different wells.
