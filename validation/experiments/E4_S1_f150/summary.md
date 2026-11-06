# Experiment E4_S1_f150

Candidate options: `snow_density_scheme='bulk_relaxation'`, `snow_density_fresh=150`  
Baseline: legacy CLIMo (Parameters defaults). Code: 717f7c4 (with uncommitted changes). Run 2026-10-30 14:10 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 46 case-checks (surface energy residual, largest daily absolute (W m-2), surface energy residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 13 / 10 / 13 | 6 / 4 / 2 | -0.051 |
| ice thickness (m) | mae | 14 / 9 / 13 | 6 / 3 / 3 | -0.007 |
| maximum ice thickness (m) | bias | 16 / 7 / 13 | 6 / 3 / 3 | -0.064 |
| maximum ice thickness (m) | mae | 16 / 7 / 13 | 6 / 3 / 3 | -0.020 |
| freeze-up (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000 |
| freeze-up (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000 |
| break-up (days) | bias | 6 / 18 / 12 | 2 / 8 / 2 | -1.594 |
| break-up (days) | mae | 6 / 18 / 12 | 2 / 8 / 2 | 0.616 |
| effective break-up, ice < 0.1 m (days) | bias | 4 / 18 / 14 | 1 / 8 / 3 | -1.609 |
| effective break-up, ice < 0.1 m (days) | mae | 5 / 18 / 13 | 1 / 8 / 3 | 0.808 |
| snow depth on the ice (m) | bias | 2 / 2 / 32 | 0 / 0 / 12 | -0.000766 |
| snow depth on the ice (m) | mae | 3 / 0 / 33 | 1 / 0 / 11 | -0.002 |
| ice thickness, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.064 |
| snow ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | 0.026 |
| congelation ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.071 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 0 / 24 / 12 | 0 / 10 / 2 | 0.097 |
| surface energy residual, largest daily absolute (W m-2) | closure | 2 / 22 / 12 | 2 / 8 / 2 | 4.259 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.34e-12 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 24 / 0 / 12 | 10 / 0 / 2 | -0.092 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -5.79e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.85e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.11 | 0.15 -> 0.13 | +4.9 -> +4.9 | +1.8 -> +0.2 | +0.7 -> -0.9 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.11 | 0.16 -> 0.13 | +2.2 -> +2.2 | +3.6 -> +1.6 | +2.0 -> +0.3 | 0.07 -> 0.06 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.13 | 0.22 -> 0.16 | -5.4 -> -5.4 | -5.6 -> -7.7 | -7.2 -> -9.1 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.15 | 0.26 -> 0.19 | -6.6 -> -6.6 | -6.7 -> -8.9 | -8.1 -> -10.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.13 | 0.10 -> 0.13 | +0.3 -> +0.3 | -12.4 -> -14.4 | -14.1 -> -15.9 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.12 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -13.9 | -13.3 -> -15.5 | 0.06 -> 0.05 |
| gimli | era5 | 50 % | 0.14 -> 0.16 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -14.1 | -13.9 -> -15.9 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.17 | 0.15 -> 0.16 | -1.9 -> -1.9 | -10.3 -> -12.4 | -11.7 -> -13.9 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.09 | 0.14 -> 0.09 | +3.6 -> +3.6 | -4.5 -> -7.2 | -6.3 -> -8.9 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.11 | 0.19 -> 0.12 | +4.8 -> +4.8 | -1.6 -> -4.4 | -3.2 -> -6.1 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.682 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 62.141 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.71e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 110.90 |
| FSWout | 57.71 | 57.40 |
| FLWin | 229.66 | 229.08 |
| FLWout | 263.45 | 262.65 |
| FSENS | -6.75 | -6.20 |
| FLAT | -6.47 | -6.37 |
| PenSW | 7.18 | 6.97 |
| conduction_top | -22.30 | -21.28 |
| conduction_bottom | -21.70 | -20.41 |
| water_flux | 3.93 | 3.91 |
| surface_melt_energy | 23.96 | 22.79 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
