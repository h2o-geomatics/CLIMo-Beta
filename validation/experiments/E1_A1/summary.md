# Experiment E1_A1

Candidate options: `thin_snow_albedo='exponential'`, `thin_snow_albedo_depth=0.03`  
Baseline: legacy CLIMo (Parameters defaults). Code: 7f6521c (with uncommitted changes). Run 2026-10-30 01:38 UTC, 46 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Break-up improves while winter ice thickness degrades: the specification says not to accept a change for a better ice-off alone.
- Closure is worse in 6 case-checks (surface energy residual, largest daily absolute (W m-2), surface energy residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 1 / 4 / 31 | 1 / 2 / 9 | 0.003 |
| ice thickness (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000281 |
| maximum ice thickness (m) | bias | 0 / 4 / 32 | 0 / 2 / 10 | 0.002 |
| maximum ice thickness (m) | mae | 0 / 2 / 34 | 0 / 0 / 12 | 0.00056 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 9 / 2 / 25 | 5 / 1 / 6 | 0.323 |
| break-up (days) | mae | 3 / 0 / 33 | 2 / 0 / 10 | -0.099 |
| effective break-up, ice < 0.1 m (days) | bias | 9 / 0 / 27 | 6 / 0 / 6 | 0.320 |
| effective break-up, ice < 0.1 m (days) | mae | 5 / 0 / 31 | 3 / 0 / 9 | -0.150 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000535 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000219 |
| ice thickness, mean seasonal maximum (m) | mean | changed 4 / unchanged 32 | changed 2 / unchanged 10 | 0.002 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -0.000197 |
| congelation ice, mean seasonal maximum (m) | mean | changed 2 / unchanged 34 | changed 2 / unchanged 10 | 0.002 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 5 / 2 / 29 | 4 / 0 / 8 | -0.001 |
| surface energy residual, largest daily absolute (W m-2) | closure | 2 / 4 / 30 | 1 / 1 / 10 | 0.621 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.64e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 12 / 0 / 24 | 6 / 0 / 6 | -0.020 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.32e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -2.8e-15 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +2.3 | +0.7 -> +1.0 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +3.8 | +2.0 -> +2.3 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.17 | 0.22 -> 0.23 | -5.4 -> -5.4 | -5.6 -> -5.1 | -7.2 -> -6.5 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -6.3 | -8.1 -> -7.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.11 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -12.2 | -14.1 -> -13.6 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -11.4 | -13.3 -> -13.0 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -12.0 | -13.9 -> -13.2 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -9.5 | -11.7 -> -11.0 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -3.6 | -6.3 -> -5.4 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> -1.0 | -3.2 -> -2.6 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.560 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.721 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.69e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 112.90 |
| FSWout | 57.71 | 58.51 |
| FLWin | 229.66 | 229.76 |
| FLWout | 263.45 | 263.46 |
| FSENS | -6.75 | -6.61 |
| FLAT | -6.47 | -6.40 |
| PenSW | 7.18 | 7.12 |
| conduction_top | -22.30 | -22.26 |
| conduction_bottom | -21.70 | -21.67 |
| water_flux | 3.93 | 3.88 |
| surface_melt_energy | 23.96 | 23.96 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
