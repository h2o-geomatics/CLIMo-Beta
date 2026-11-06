# Experiment F2_newton

Candidate options: `newton_acceptance='fixed'`  
Baseline: legacy CLIMo (Parameters defaults). Code: 0113baa (with uncommitted changes). Run 2026-10-30 13:05 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 8 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.001 |
| ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000257 |
| maximum ice thickness (m) | bias | 0 / 1 / 35 | 0 / 1 / 11 | 0.002 |
| maximum ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000508 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.059 |
| break-up (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.029 |
| effective break-up, ice < 0.1 m (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.093 |
| effective break-up, ice < 0.1 m (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.046 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000326 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000219 |
| ice thickness, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.002 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 9.72e-06 |
| congelation ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.002 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -0.609 |
| surface energy residual, largest daily absolute (W m-2) | closure | 19 / 8 / 9 | 5 / 2 / 5 | -1.851 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 2.45e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -0.001 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -7.1e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 9.94e-15 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +1.8 | +0.7 -> +0.7 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.6 | +2.0 -> +2.2 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.16 | 0.22 -> 0.23 | -5.4 -> -5.4 | -5.6 -> -5.6 | -7.2 -> -7.2 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -6.6 | -8.1 -> -8.0 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -12.4 | -14.1 -> -14.0 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -12.0 | -13.3 -> -13.2 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.2 | -13.9 -> -13.8 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -10.2 | -11.7 -> -11.6 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -4.4 | -6.3 -> -6.1 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> -1.5 | -3.2 -> -3.1 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 0.128 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.810 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.03e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.277 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.02e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.66e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 112.48 |
| FSWout | 57.71 | 57.77 |
| FLWin | 229.66 | 229.68 |
| FLWout | 263.45 | 263.48 |
| FSENS | -6.75 | -6.76 |
| FLAT | -6.47 | -6.47 |
| PenSW | 7.18 | 7.17 |
| conduction_top | -22.30 | -22.32 |
| conduction_bottom | -21.70 | -21.73 |
| water_flux | 3.93 | 3.93 |
| surface_melt_energy | 23.96 | 23.99 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
