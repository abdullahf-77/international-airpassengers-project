# AirPassengers forecasting recommendation

**Recommendation: ship AutoARIMA for the 12-month planning horizon because its
eight-origin MASE is 0.647, versus 1.313 for the seasonal-naive floor—a 51%
reduction in scaled point error.**

## What the data says

The data contains 144 uninterrupted monthly observations from January 1949 to
December 1960. Demand rises from roughly 100 thousand passengers per month at
the start to more than 400 thousand at the end. It also follows a pronounced
yearly cycle: summer is consistently strongest and winter weakest, while the
size of those swings grows with the overall market.

STL decomposition confirms that this is not a flat series with random noise.
Trend strength is 0.996 and seasonal strength is 0.975. The trend panel carries
the sustained rise, the seasonal panel carries the repeating within-year
pattern, and the remainder is much smaller, though more variable late in the
sample. The raw autocorrelation is 0.760 at 12 months and 0.532 at 24 months,
so the same month in prior years remains informative. A useful model therefore
needs both a changing level and an annual seasonal mechanism.

## A fair comparison against the floor

Seasonal naive—the operational floor—predicts each month with the observation
from the same month one year earlier. A plain last-observation naive model was
also included as a second benchmark. The floor removes the timing of the annual
cycle, but its residuals stay mostly positive in runs: lag-1 residual
autocorrelation is 0.746, while lag-12 is only -0.044. In business terms, last
year's corresponding month captures *when* the peaks occur but systematically
falls behind a market whose level is still rising.

AutoGluon is used as a model-zoo framework to fit and rank Naive,
SeasonalNaive, AutoETS, AutoARIMA, and Theta candidates on a held-out year. Its
`score_val` is negative MASE—closer to zero is better—and its internal ranking
uses one validation window. That ranking is treated only as a shortlist, not as
the shipping decision.

The final evidence comes from one leakage-safe harness shared by every model:
eight rolling origins, a 12-month forecast horizon, and a 12-month step. Each
fold's model fit and scaling denominator use only data available at that
origin. Across the 96 held-out month forecasts, the results are:

| Model | MASE | Scaled CRPS | 80% coverage | Mean 80% width | Fold MASE range |
|---|---:|---:|---:|---:|---:|
| AutoARIMA | **0.647** | **0.0339** | 69.8% | 44.1 | 0.330–1.594 |
| AutoTheta | 0.898 | 0.0460 | 59.4% | 57.6 | 0.451–1.405 |
| AutoETS | 0.952 | 0.0499 | 49.0% | 48.7 | 0.565–1.554 |
| Seasonal naive | 1.313 | 0.0616 | 51.0% | 77.8 | 0.412–1.959 |
| Naive | 2.119 | 0.1103 | 68.8% | 142.9 | 1.422–3.196 |

AutoARIMA wins on both halves of the forecast: the point forecast (MASE) and
the full probability distribution (scaled CRPS). Its advantage is not uniform
every year—the worst fold MASE is 1.594—but the mean result remains materially
ahead of the floor. AutoTheta and AutoETS improve point accuracy too, but give
up more distributional accuracy and interval coverage.

## Intervals: useful, but not yet honest enough

AutoARIMA's nominal 80% interval is 44.1 thousand passengers wide on average.
It covers 69.8% of the held-out observations rather than the promised 80%, so
the band is too narrow and should not be presented as fully calibrated. The
seasonal-naive floor is wider at 77.8 thousand yet covers only 51.0%, which
shows that width alone does not make an interval honest. Plain naive reaches
68.8% coverage only with an average width of 142.9 thousand and very poor point
and distribution scores.

The correct operational message is therefore two-part: AutoARIMA is the best
central forecast, and its uncertainty band needs calibration before it is used
for capacity commitments. A simple first release can widen the 80% interval
using the empirical rolling-origin errors until historical coverage is close
to 80%, while continuing to report the unadjusted model band for monitoring.

## Residuals and what remains

After a 12-month warm-up, AutoARIMA's fitted residuals are centered close to
zero (mean 0.24 thousand). Their autocorrelation is approximately -0.001 at lag
1 and -0.109 at lag 12, with no strong repeatable pattern through 24 months.
That is a major improvement over the benchmark: the model has absorbed the
leftover trend and short-run dependence.

The larger misses still occur near the highest peaks and deepest troughs, and
their scale grows somewhat with passenger volume. The model therefore appears
to be missing changing volatility more than a recurring calendar pattern. This
matches the under-coverage result: the center is good, but the distribution is
overconfident during extreme months.

## One next change

The next change should be **rolling empirical interval calibration by horizon**.
For each of the 12 lead months, keep the absolute errors from recent rolling
origins and use their empirical quantiles to adjust the lower and upper bounds.
This does not leak future information: each calibration set contains only
errors already observed at that forecast origin. I expect this change to move
80% coverage from 69.8% toward the nominal 80% with a controlled increase in
band width, while leaving the strong MASE of the AutoARIMA central forecast
unchanged.

The model and calibration should be re-evaluated as each new year arrives. If
coverage remains weak, the next modeling experiment would be a log-scale or
Box-Cox AutoARIMA, because the original chart and residual spread both indicate
that variability increases with the level.

---

All performance numbers in this report come from the notebook's common
eight-origin rolling cross-validation harness. The one-window framework
leaderboard is used only for shortlisting; it is not reported as final evidence.

