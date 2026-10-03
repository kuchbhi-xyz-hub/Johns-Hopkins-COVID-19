# Johns-Hopkins-COVID-19
Time-series analysis and forecasting of COVID-19 cases and deaths using Johns Hopkins data, with multi-model validation, ensemble forecasting, and health-planning insights.
# Forecasting US COVID-19 Cases and Deaths (Johns Hopkins Data)

Capstone project **PRCP-1023** (Healthcare). A single Jupyter notebook that analyses the Johns Hopkins CSSE COVID-19 time series, builds and compares 14-day forecasting models for daily US cases and deaths, and turns the final forecast into preparation suggestions for a health department.

## Project tasks

1. Prepare a complete data analysis report on the supplied data.
2. Fix a prediction period and forecast confirmed cases/deaths for a specific country from past values.
3. Suggest how the country's health department could prepare, based on the predictions.

The brief also asks for a model comparison report with a recommended production model, and a report on the data challenges faced and how they were handled. Everything is in one notebook.

## Data

The project zip contains three files from the [JHU CSSE COVID-19 repository](https://github.com/CSSEGISandData/COVID-19), covering 22 Jan to 21 Sep 2020:

| File | Rows | Content |
|---|---:|---|
| `time_series_covid19_confirmed_global.csv` | 266 | cumulative confirmed cases |
| `time_series_covid19_deaths_global.csv` | 266 | cumulative deaths |
| `time_series_covid19_recovered_global.csv` | 253 | cumulative recoveries (not used for modelling) |

Each file has four location columns followed by one column per date (244 dates). The data is cumulative, so daily values were derived by differencing after aggregating provinces to country level.

## Approach

**Data checks.** Inspected the files before assuming any structure, then checked date continuity, missing values, duplicates, inconsistent geography (provinces, cruise-ship rows) and reporting revisions (31 countries with falls in cumulative cases).

**Country selection.** All 186 countries were screened for revisions, reporting gaps, batch dumps and data volume. The US was chosen as the only shortlisted country with no anomalies in either cases or deaths over its active period, and with several outbreak phases that make validation meaningful.

**EDA findings that shaped the models**
- Two case waves (peaks around 10 Apr and 22 Jul) and an upturn at the end of the data
- A strong weekly reporting cycle: cases about 30% higher on Fridays than Mondays, deaths more than twice as high mid-week as on Sundays
- Deaths peaked 10 to 14 days after cases in both waves

**Forecasting setup**
- Targets: daily new confirmed cases and daily new deaths
- Horizon: 14 days (two full weekly cycles)
- Validation: rolling origin, 8 weekly origins (22 Jun to 10 Aug), expanding window
- Holdout: last 28 days (25 Aug to 21 Sep), evaluated from 2 origins and kept out of all model decisions
- Metrics: MAE, RMSE, MAPE, plus fitting time

**Models compared:** naive, seasonal naive, damped Holt-Winters (ETS), SARIMA, Ridge and Gradient Boosting (direct multi-horizon with leakage-safe features), and an equal-weight average of the top 3 models. The production model was selected in code from validation results before the holdout was run.

## Results

| Target | Model | Validation MAE | Holdout MAE |
|---|---|---:|---:|
| Daily cases | **Average of top 3 (production)** | 7,070 | 6,422 |
| | ETS damped, log scale | 7,102 | 7,614 |
| | SARIMA (1,1,1)(0,1,1,7) | 7,748 | 7,606 |
| | Ridge (direct) | 8,547 | 4,764 |
| | Seasonal naive | 10,688 | 5,254 |
| | Gradient Boosting (direct) | 12,365 | 5,161 |
| Daily deaths | **Average of top 3 (production)** | 167.9 | 124.6 |
| | Seasonal naive | 169.1 | 146.1 |
| | ETS damped, raw scale | 185.6 | 118.9 |
| | SARIMA (1,0,1)(0,1,1,7) | 195.6 | 124.7 |

The combination had the lowest and most consistent validation error for both targets and came second on the deaths holdout. On the cases holdout it came fourth: the trend models failed at a turning point just after Labor Day, where Ridge held up best. The model chosen before the holdout was kept, and this result is reported and discussed rather than used to switch models after the fact.

## Forecast (22 Sep to 5 Oct 2020)

- **Cases:** forecast to keep rising. Week 2 totals between about 319,000 and 424,000, depending on whether the unusually high last observation (21 Sep) is real growth or a reporting backlog; a sensitivity check shows the direction holds either way. Cumulative cases reach about 7.5 to 7.7 million by 5 Oct.
- **Deaths:** roughly flat at about 5,000 per week (cumulative about 210,000 by 5 Oct), with a possible rise just after the window if the case increase continues, given the 10 to 14 day lag.

The notebook turns these into cautious planning considerations (testing capacity, hospital readiness, reading data around the reporting cycle and holidays, monitoring), each with its limits stated.

## Repository structure

```
├── PRCP-1023___Johns_Hopkins_COVID-19.ipynb   # full analysis, models, forecast and reports
└── README.md
```

## Running the notebook

1. Download the project zip and unzip the three CSV files into a `dataset` folder.
2. Set `DATA_DIR` in the setup cell to that folder.
3. Run all cells in order.

Versions used: Python 3.13, numpy 2.5.3, pandas 3.0.2, matplotlib 3.11.1, seaborn 0.13.2, scikit-learn 1.9.1, statsmodels 0.15.0.

```bash
pip install numpy pandas matplotlib seaborn scikit-learn statsmodels
```

## Limitations

- National totals only; differences between states are hidden.
- Confirmed cases depend on testing volume, which is not in the data.
- Model comparison rests on 8 validation and 2 holdout origins from a single summer.
- No holiday or intervention information is modelled.
- The 80% forecast band is empirical and does not include uncertainty in the input data.
- All findings are associations over time, not causal effects.

## Author

Smita Sahu
