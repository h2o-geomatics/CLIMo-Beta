# Experiment E4_S1_f150_F1F2

Candidate options: `snow_density_scheme='bulk_relaxation'`, `surface_solver='fixed'`, `newton_acceptance='fixed'`, `snow_density_fresh=150`  
Baseline: legacy CLIMo (Parameters defaults). Code: 717f7c4 (with uncommitted changes). Run 2026-10-30 14:17 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 8 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 14 / 10 / 12 | 6 / 4 / 2 | -0.049 |
| ice thickness (m) | mae | 14 / 9 / 13 | 6 / 3 / 3 | -0.007 |
| maximum ice thickness (m) | bias | 16 / 7 / 13 | 6 / 3 / 3 | -0.062 |
| maximum ice thickness (m) | mae | 16 / 7 / 13 | 6 / 3 / 3 | -0.019 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 6 / 18 / 12 | 2 / 8 / 2 | -1.524 |
| break-up (days) | mae | 5 / 18 / 13 | 1 / 8 / 3 | 0.561 |
| effective break-up, ice < 0.1 m (days) | bias | 4 / 18 / 14 | 1 / 8 / 3 | -1.530 |
| effective break-up, ice < 0.1 m (days) | mae | 5 / 18 / 13 | 1 / 8 / 3 | 0.752 |
| snow depth on the ice (m) | bias | 1 / 2 / 33 | 0 / 0 / 12 | -0.000395 |
| snow depth on the ice (m) | mae | 2 / 0 / 34 | 1 / 0 / 11 | -0.002 |
| ice thickness, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.062 |
| snow ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | 0.026 |
| congelation ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.069 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -0.621 |
| surface energy residual, largest daily absolute (W m-2) | closure | 24 / 8 / 4 | 8 / 4 / 0 | -6.437 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.05e-12 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 24 / 0 / 12 | 10 / 0 / 2 | -0.092 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -7.13e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.84e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.11 | 0.15 -> 0.13 | +4.9 -> +4.9 | +1.8 -> +0.2 | +0.7 -> -0.9 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.11 | 0.16 -> 0.13 | +2.2 -> +2.2 | +3.6 -> +1.7 | +2.0 -> +0.4 | 0.07 -> 0.06 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.13 | 0.22 -> 0.17 | -5.4 -> -5.4 | -5.6 -> -7.5 | -7.2 -> -8.9 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.15 | 0.26 -> 0.19 | -6.6 -> -6.6 | -6.7 -> -8.8 | -8.1 -> -10.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.13 | 0.10 -> 0.13 | +0.3 -> +0.3 | -12.4 -> -14.3 | -14.1 -> -15.8 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.12 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -14.0 | -13.3 -> -15.4 | 0.06 -> 0.05 |
| gimli | era5 | 50 % | 0.14 -> 0.16 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -13.9 | -13.9 -> -15.7 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.16 | 0.15 -> 0.16 | -1.9 -> -1.9 | -10.3 -> -12.2 | -11.7 -> -13.8 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.09 | 0.14 -> 0.09 | +3.6 -> +3.6 | -4.5 -> -7.2 | -6.3 -> -8.9 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.11 | 0.19 -> 0.12 | +4.8 -> +4.8 | -1.6 -> -4.3 | -3.2 -> -5.9 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 0.054 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 51.278 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.03e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.283 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.02e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.64e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 111.00 |
| FSWout | 57.71 | 57.49 |
| FLWin | 229.66 | 229.11 |
| FLWout | 263.45 | 262.68 |
| FSENS | -6.75 | -6.21 |
| FLAT | -6.47 | -6.37 |
| PenSW | 7.18 | 6.97 |
| conduction_top | -22.30 | -21.31 |
| conduction_bottom | -21.70 | -20.44 |
| water_flux | 3.93 | 3.90 |
| surface_melt_energy | 23.96 | 22.82 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
