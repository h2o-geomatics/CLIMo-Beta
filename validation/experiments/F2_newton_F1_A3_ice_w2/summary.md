# Experiment F2_newton_F1_A3_ice_w2

Candidate options: `newton_acceptance='fixed'`, `surface_solver='fixed'`, `melt_albedo_transition='linear'`, `melt_albedo_transition_ice='linear'`, `melt_albedo_transition_width=2`, `melt_albedo_transition_ice_width=2`  
Baseline: legacy CLIMo (Parameters defaults). Code: 0113baa (with uncommitted changes). Run 2026-10-30 13:08 UTC, 49 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 21 case-checks (short-wave residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 6 / 0 / 30 | 0 / 0 / 12 | -0.002 |
| ice thickness (m) | mae | 2 / 0 / 34 | 0 / 0 / 12 | -0.000299 |
| maximum ice thickness (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | -0.000885 |
| maximum ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.000913 |
| freeze-up (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000 |
| freeze-up (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000 |
| break-up (days) | bias | 4 / 13 / 19 | 2 / 2 / 8 | -0.645 |
| break-up (days) | mae | 3 / 9 / 24 | 1 / 1 / 10 | 0.312 |
| effective break-up, ice < 0.1 m (days) | bias | 3 / 10 / 23 | 2 / 1 / 9 | -0.624 |
| effective break-up, ice < 0.1 m (days) | mae | 2 / 8 / 26 | 1 / 0 / 11 | 0.298 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | -0.000402 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.000378 |
| ice thickness, mean seasonal maximum (m) | mean | changed 1 / unchanged 35 | changed 0 / unchanged 12 | -0.001 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -0.001 |
| congelation ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.000375 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -0.628 |
| surface energy residual, largest daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -17.806 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 1.17e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 0 / 21 / 15 | 0 / 8 / 4 | 0.042 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.09e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.36e-16 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +1.7 | +0.7 -> +0.5 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.1 | +2.0 -> +1.8 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.1 | +1.7 -> +1.1 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +1.9 | +1.8 -> +0.7 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.16 | 0.22 -> 0.22 | -5.4 -> -5.4 | -5.6 -> -6.1 | -7.2 -> -7.5 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -7.2 | -8.1 -> -8.6 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -13.0 | -14.1 -> -14.4 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -12.3 | -13.3 -> -13.7 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.8 | -13.9 -> -14.4 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -10.7 | -11.7 -> -12.1 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -5.0 | -6.3 -> -6.7 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> -1.9 | -3.2 -> -3.6 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 0.025 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 6.506 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.05e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.412 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.09e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.64e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 111.31 |
| FSWout | 57.71 | 55.96 |
| FLWin | 229.66 | 229.42 |
| FLWout | 263.45 | 263.40 |
| FSENS | -6.75 | -7.01 |
| FLAT | -6.47 | -6.65 |
| PenSW | 7.18 | 7.28 |
| conduction_top | -22.30 | -22.38 |
| conduction_bottom | -21.70 | -21.80 |
| water_flux | 3.93 | 3.99 |
| surface_melt_energy | 23.96 | 23.99 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
