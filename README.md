# International Air Passengers Forecasting

Submitted by: Abdullah Faisal Alsehli — academy: @SDAIAAcademy

This capstone applies the complete time-series forecasting pipeline to the
monthly Box & Jenkins AirPassengers series. The dataset contains 144 monthly
observations from January 1949 through December 1960 with no missing periods.

## Recommendation

Ship **AutoARIMA** for the 12-month planning horizon. Across eight rolling
origins, it achieved a MASE of **0.647**, compared with **1.313** for the
seasonal-naive benchmark—a reduction of approximately 51% in scaled point
error. Its 80% prediction interval covered 69.8% of held-out observations, so
the central forecast is recommended while the interval should be calibrated
before use in capacity commitments.

## What the project includes

- Exploration of trend, seasonality, and autocorrelation.
- STL decomposition and textbook trend/seasonal strength measures.
- Naive and seasonal-naive benchmark forecasts with prediction intervals.
- Residual plots and ACF diagnostics for the benchmark and selected model.
- An AutoGluon model zoo with Naive, SeasonalNaive, AutoETS, AutoARIMA, and
  Theta candidates.
- Leakage-safe cross-validation using eight rolling origins, a 12-month
  horizon, and a 12-month step.
- Evaluation with MASE, RMSSE, scaled CRPS, interval width, and empirical 80%
  coverage.
- A manager-ready recommendation in `report.md`.

## Main results

| Model | MASE | Scaled CRPS | 80% coverage |
|---|---:|---:|---:|
| AutoARIMA | **0.647** | **0.0339** | 69.8% |
| AutoTheta | 0.898 | 0.0460 | 59.4% |
| AutoETS | 0.952 | 0.0499 | 49.0% |
| Seasonal naive | 1.313 | 0.0616 | 51.0% |
| Naive | 2.119 | 0.1103 | 68.8% |

## Run in Google Colab

[Open `project.ipynb` in Google Colab](https://colab.research.google.com/github/abdullahf-77/international-airpassengers-project/blob/main/project.ipynb)

Choose **Runtime → Run all**. The setup cell clones this fork and installs the
project together with AutoGluon. The notebook then runs the analysis from data
loading through the final forecast.

## Repository files

- `project.ipynb` — complete executable analysis and forecasting workflow.
- `report.md` — manager-ready report and final recommendation.
- `air_passengers.csv` — cached copy of the canonical AirPassengers data.
- `brief.md` — original project brief.
- `rubric.md` — original assessment rubric.
- `coursekit/` — plotting and scoring utilities supplied with the project.

## Verification

The notebook was run from top to bottom in Google Colab with AutoGluon enabled.
All 18 code cells completed without errors, and the rolling-origin harness
produced 96 held-out forecasts across the eight evaluation windows.

