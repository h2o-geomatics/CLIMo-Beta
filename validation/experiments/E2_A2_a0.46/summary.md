# Experiment E2_A2_a0.46

Candidate options: `ice_type_albedo='surface_threshold'`, `snow_ice_albedo=0.46`, `snow_ice_albedo_melt=0.36`, `snow_ice_albedo_thickness=0.05`  
Baseline: legacy CLIMo (Parameters defaults). Code: d128e99 (with uncommitted changes). Run 2026-10-30 02:09 UTC, 46 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 1 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 0 / 3 / 33 | 0 / 2 / 10 | 0.001 |
| ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000503 |
| maximum ice thickness (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000195 |
| maximum ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000134 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 12 / 4 / 20 | 6 / 0 / 6 | 0.654 |
| break-up (days) | mae | 9 / 4 / 23 | 5 / 0 / 7 | -0.048 |
| effective break-up, ice < 0.1 m (days) | bias | 14 / 3 / 19 | 8 / 0 / 4 | 0.695 |
| effective break-up, ice < 0.1 m (days) | mae | 11 / 3 / 22 | 7 / 0 / 5 | -0.151 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 8.71e-05 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 3.23e-05 |
| ice thickness, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.000168 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -7.04e-05 |
| congelation ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.000171 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 2 / 0 / 34 | 1 / 0 / 11 | -0.003 |
| surface energy residual, largest daily absolute (W m-2) | closure | 5 / 1 / 30 | 1 / 1 / 10 | -0.140 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -6.12e-14 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 8 / 0 / 28 | 2 / 0 / 10 | -0.013 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 5.72e-14 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -9.5e-15 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +2.0 | +0.7 -> +0.7 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.7 | +2.0 -> +2.3 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.16 | 0.22 -> 0.22 | -5.4 -> -5.4 | -5.6 -> -4.9 | -7.2 -> -6.6 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -6.1 | -8.1 -> -7.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -12.1 | -14.1 -> -13.4 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -11.3 | -13.3 -> -12.6 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.1 | -13.9 -> -13.4 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -9.6 | -11.7 -> -11.1 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -2.6 | -6.3 -> -4.2 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> +0.7 | -3.2 -> -0.9 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.559 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.787 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.66e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 113.28 |
| FSWout | 57.71 | 58.83 |
| FLWin | 229.66 | 229.92 |
| FLWout | 263.45 | 263.61 |
| FSENS | -6.75 | -6.68 |
| FLAT | -6.47 | -6.43 |
| PenSW | 7.18 | 7.15 |
| conduction_top | -22.30 | -22.22 |
| conduction_bottom | -21.70 | -21.65 |
| water_flux | 3.93 | 3.90 |
| surface_melt_energy | 23.96 | 23.87 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
