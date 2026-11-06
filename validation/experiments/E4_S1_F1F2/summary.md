# Experiment E4_S1_F1F2

Candidate options: `snow_density_scheme='bulk_relaxation'`, `surface_solver='fixed'`, `newton_acceptance='fixed'`  
Baseline: legacy CLIMo (Parameters defaults). Code: 717f7c4 (with uncommitted changes). Run 2026-10-30 14:28 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 6 case-checks (short-wave residual, mean of daily absolute (W m-2), surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.002 |
| ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000178 |
| maximum ice thickness (m) | bias | 2 / 1 / 33 | 0 / 1 / 11 | 0.003 |
| maximum ice thickness (m) | mae | 2 / 0 / 34 | 0 / 0 / 12 | 0.000534 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.057 |
| break-up (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.017 |
| effective break-up, ice < 0.1 m (days) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.076 |
| effective break-up, ice < 0.1 m (days) | mae | 0 / 0 / 36 | 0 / 0 / 12 | -0.043 |
| snow depth on the ice (m) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.000577 |
| snow depth on the ice (m) | mae | 1 / 0 / 35 | 0 / 0 / 12 | -0.000473 |
| ice thickness, mean seasonal maximum (m) | mean | changed 1 / unchanged 35 | changed 1 / unchanged 11 | 0.003 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -0.000129 |
| congelation ice, mean seasonal maximum (m) | mean | changed 3 / unchanged 33 | changed 1 / unchanged 11 | 0.003 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 36 / 0 / 0 | 12 / 0 / 0 | -0.622 |
| surface energy residual, largest daily absolute (W m-2) | closure | 24 / 2 / 10 | 7 / 1 / 4 | -7.888 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 7.14e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 0 / 4 / 32 | 0 / 1 / 11 | 0.009 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -2.28e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 1.27e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +1.9 | +0.7 -> +0.7 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.5 | +2.0 -> +1.9 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.17 | 0.22 -> 0.23 | -5.4 -> -5.4 | -5.6 -> -5.6 | -7.2 -> -7.1 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -6.7 | -8.1 -> -8.0 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.12 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -12.4 | -14.1 -> -13.9 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -12.0 | -13.3 -> -13.2 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.2 | -13.9 -> -13.8 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -10.3 | -11.7 -> -11.6 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.15 | +3.6 -> +3.6 | -4.5 -> -4.4 | -6.3 -> -6.1 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> -1.5 | -3.2 -> -3.1 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 0.041 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 26.670 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.03e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.283 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.02e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.63e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 112.46 |
| FSWout | 57.71 | 57.74 |
| FLWin | 229.66 | 229.68 |
| FLWout | 263.45 | 263.49 |
| FSENS | -6.75 | -6.78 |
| FLAT | -6.47 | -6.48 |
| PenSW | 7.18 | 7.20 |
| conduction_top | -22.30 | -22.37 |
| conduction_bottom | -21.70 | -21.75 |
| water_flux | 3.93 | 3.93 |
| surface_melt_energy | 23.96 | 23.98 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
