# Experiment E1_A1_h0.06

Candidate options: `thin_snow_albedo='exponential'`, `thin_snow_albedo_depth=0.06`  
Baseline: legacy CLIMo (Parameters defaults). Code: 7f6521c (with uncommitted changes). Run 2026-10-30 01:41 UTC, 46 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 32 case-checks (short-wave residual, mean of daily absolute (W m-2), surface energy residual, largest daily absolute (W m-2), surface energy residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 2 / 0 / 34 | 2 / 0 / 10 | -0.001 |
| ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.000475 |
| maximum ice thickness (m) | bias | 5 / 0 / 31 | 2 / 0 / 10 | -0.002 |
| maximum ice thickness (m) | mae | 2 / 0 / 34 | 0 / 0 / 12 | -0.000882 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 0 / 9 / 27 | 0 / 7 / 5 | -0.268 |
| break-up (days) | mae | 0 / 4 / 32 | 0 / 3 / 9 | 0.142 |
| effective break-up, ice < 0.1 m (days) | bias | 0 / 5 / 31 | 0 / 3 / 9 | -0.288 |
| effective break-up, ice < 0.1 m (days) | mae | 0 / 5 / 31 | 0 / 3 / 9 | 0.181 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | -0.000619 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.00039 |
| ice thickness, mean seasonal maximum (m) | mean | changed 4 / unchanged 32 | changed 2 / unchanged 10 | -0.002 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -0.000195 |
| congelation ice, mean seasonal maximum (m) | mean | changed 2 / unchanged 34 | changed 2 / unchanged 10 | -0.002 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 1 / 12 / 23 | 0 / 6 / 6 | 0.007 |
| surface energy residual, largest daily absolute (W m-2) | closure | 4 / 10 / 22 | 2 / 4 / 6 | 0.421 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.75e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 0 / 10 / 26 | 0 / 7 / 5 | 0.016 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 1.89e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.78e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +1.7 | +0.7 -> +0.4 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.1 | +2.0 -> +1.8 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.16 | 0.22 -> 0.22 | -5.4 -> -5.4 | -5.6 -> -6.1 | -7.2 -> -7.6 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.25 | -6.6 -> -6.6 | -6.7 -> -7.3 | -8.1 -> -8.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -13.0 | -14.1 -> -14.6 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -12.2 | -13.3 -> -13.7 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.9 | -13.9 -> -14.4 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -10.9 | -11.7 -> -12.5 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -5.2 | -6.3 -> -7.0 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.15 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> -2.5 | -3.2 -> -4.2 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.566 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.459 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.61e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 111.90 |
| FSWout | 57.71 | 56.90 |
| FLWin | 229.66 | 229.55 |
| FLWout | 263.45 | 263.46 |
| FSENS | -6.75 | -6.88 |
| FLAT | -6.47 | -6.53 |
| PenSW | 7.18 | 7.21 |
| conduction_top | -22.30 | -22.32 |
| conduction_bottom | -21.70 | -21.70 |
| water_flux | 3.93 | 3.95 |
| surface_melt_energy | 23.96 | 23.94 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
