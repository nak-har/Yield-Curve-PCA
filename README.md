# Yield-Curve PCA: Level, Slope, and Curvature

A principal-component analysis of the U.S. Treasury yield curve, decomposing daily
yield changes into three interpretable factors — **level**, **slope**, and
**curvature** — and applying them to relative-value and risk-management questions.

The project runs end to end in a single notebook: raw yields → daily changes →
covariance → PCA → factor interpretation → factor scores → residual relative-value
test → key-rate-duration risk decomposition and hedging.

## Data

Daily Treasury par yield curve rates (11 tenors: 1M, 3M, 6M, 1Y, 2Y, 3Y, 5Y, 7Y,
10Y, 20Y, 30Y), covering **2020–2023** (~1,000 trading days across the COVID/ZIRP
period, the 2022 hiking cycle, and the 2023 disinversion).

Source: [U.S. Department of the Treasury, Daily Treasury Par Yield Curve Rates](https://home.treasury.gov/resource-center/data-chart-center/interest-rates/TextView?type=daily_treasury_yield_curve).
Downloaded as `par-yield-curve-rates-2020-2023.csv` (date column plus one column per
tenor, in percent).

## Method

Two deliberate modeling choices:

- **Changes, not levels.** Yield *levels* are near-unit-root (each day is roughly
  yesterday plus a small move), so PCA on levels would mostly encode the sample's
  trend. Differencing to daily changes strips the trend and leaves the stationary,
  repeatable curve dynamics.
- **Covariance, not correlation.** Intermediate tenors genuinely move more in basis
  points than the policy-anchored front end. Using the covariance preserves those
  real volatility differences, which is what produces the level/slope/curvature
  shapes; standardizing to correlation would distort them.

PCA is computed by symmetric eigendecomposition (`numpy.linalg.eigh`) of the 11×11
covariance of daily changes, with eigenvalues sorted descending.

## Results

**Factor structure.** The top 3 principal components explain ~92% of variance
(level ~69%, slope ~12%, curvature ~11%). Their loadings across the curve have
0 / 1 / 2 sign changes respectively — the signature of level, slope, and curvature.

**Relative value (residual).** Reconstructing each day's move from only the top 3
factors leaves a residual (the ~8% the factors miss). Testing that residual for
mean reversion, the lag-1 autocorrelation is ≈ 0 to −0.15 at every tenor — **no
tradeable daily signal**. The large residual at the short end is monetary-policy
driven, not a mean-reverting dislocation. This is a descriptive/diagnostic finding,
not a return claim; a real edge would require intraday data and a cost-aware
backtest.

**Risk decomposition and hedging.** A bond position's key-rate durations project
onto the three factors to give factor exposures and a variance split
(`Var = Σ λ_m (v_mᵀ·KRD)²`). A single 10Y bond is over 90% level risk. Hedging the
level exposure with a duration-weighted 2Y position drives the level share to ~0%
and cuts total variance substantially, leaving a cleaner curvature/slope position —
the standard "decompose into three factors, hedge the ones you have no view on"
workflow.

## Running it

```bash
pip install numpy pandas matplotlib
jupyter notebook yield_curve_pca.ipynb
```

Place `par-yield-curve-rates-2020-2023.csv` in the same directory (or update the
`FILE` variable in the second cell) and run top to bottom. All figures and tables
reproduce from a clean run.

## Files

| File | Description |
|------|-------------|
| `yield_curve_pca.ipynb` | Full analysis, top to bottom |
| `par-yield-curve-rates-2020-2023.csv` | Treasury par yields (user-supplied) |
| `README.md` | This file |

## Notes

All headline numbers are **descriptive** — variance explained, factor shapes,
residual autocorrelation, and risk shares — and reproduce by re-running the
notebook. Nothing here is a performance or return claim. The relative-value result
is deliberately reported as a negative: the framework is built and tested honestly,
and the data shows no daily signal in this sample.
