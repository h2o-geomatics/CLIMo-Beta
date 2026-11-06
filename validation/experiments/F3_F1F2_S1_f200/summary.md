# Experiment F3_F1F2_S1_f200

Candidate options: `newton_step='limited'`, `surface_solver='fixed'`, `newton_acceptance='fixed'`, `snow_density_scheme='bulk_relaxation'`, `snow_density_fresh=200`  
Baseline: legacy CLIMo (Parameters defaults). Code: 30f75fb (with uncommitted changes). Run 2026-10-30 19:50 UTC, 56 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 7 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 14 / 10 / 12 | 6 / 4 / 2 | -0.030 |
| ice thickness (m) | mae | 14 / 9 / 13 | 6 / 3 / 3 | -0.006 |
| maximum ice thickness (m) | bias | 17 / 6 / 13 | 7 / 2 / 3 | -0.037 |
| maximum ice thickness (m) | mae | 17 / 5 / 14 | 7 / 1 / 4 | -0.014 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 6 / 18 / 12 | 2 / 8 / 2 | -0.899 |
| break-up (days) | mae | 5 / 15 / 16 | 1 / 8 / 3 | 0.265 |
| effective break-up, ice < 0.1 m (days) | bias | 5 / 18 / 13 | 1 / 8 / 3 | -0.911 |
| effective break-up, ice < 0.1 m (days) | mae | 3 / 17 / 16 | 0 / 8 / 4 | 0.398 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | -0.000744 |
| snow depth on the ice (m) | mae | 1 / 0 / 35 | 0 / 0 / 12 | -0.002 |
| ice thickness, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.037 |
| snow ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | 0.016 |
| congelation ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.041 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -0.622 |
| surface energy residual, largest daily absolute (W m-2) | closure | 25 / 7 / 4 | 9 / 3 / 0 | -7.254 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 1.02e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 24 / 0 / 12 | 10 / 0 / 2 | -0.052 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -6.37e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 2.22e-15 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.11 | 0.15 -> 0.13 | +4.9 -> +4.9 | +1.8 -> +0.9 | +0.7 -> -0.2 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.12 | 0.16 -> 0.14 | +2.2 -> +2.2 | +3.6 -> +2.5 | +2.0 -> +1.2 | 0.07 -> 0.06 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.14 | 0.22 -> 0.19 | -5.4 -> -5.4 | -5.6 -> -6.8 | -7.2 -> -8.3 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.16 | 0.26 -> 0.22 | -6.6 -> -6.6 | -6.7 -> -7.9 | -8.1 -> -9.4 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.11 | +0.3 -> +0.3 | -12.4 -> -13.4 | -14.1 -> -15.1 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.10 | -3.6 -> -3.6 | -12.1 -> -13.1 | -13.3 -> -14.4 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.15 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -13.3 | -13.9 -> -14.9 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.15 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -11.4 | -11.7 -> -12.8 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.10 | 0.14 -> 0.11 | +3.6 -> +3.6 | -4.5 -> -6.1 | -6.3 -> -7.9 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.13 | 0.19 -> 0.15 | +4.8 -> +4.8 | -1.6 -> -3.3 | -3.2 -> -5.0 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 0.038 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 25.674 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.03e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.283 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.02e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 3.08e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 111.56 |
| FSWout | 57.71 | 57.58 |
| FLWin | 229.66 | 229.33 |
| FLWout | 263.45 | 263.00 |
| FSENS | -6.75 | -6.42 |
| FLAT | -6.47 | -6.41 |
| PenSW | 7.18 | 7.06 |
| conduction_top | -22.30 | -21.71 |
| conduction_bottom | -21.70 | -20.94 |
| water_flux | 3.93 | 3.92 |
| surface_melt_energy | 23.96 | 23.26 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
