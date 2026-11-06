# Experiment E1_A1_h0.01

Candidate options: `thin_snow_albedo='exponential'`, `thin_snow_albedo_depth=0.01`  
Baseline: legacy CLIMo (Parameters defaults). Code: 7f6521c (with uncommitted changes). Run 2026-10-30 01:39 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Break-up improves while winter ice thickness degrades: the specification says not to accept a change for a better ice-off alone.
- Closure is worse in 4 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 5 / 12 / 19 | 3 / 6 / 3 | 0.006 |
| ice thickness (m) | mae | 1 / 6 / 29 | 1 / 2 / 9 | 0.000709 |
| maximum ice thickness (m) | bias | 0 / 11 / 25 | 0 / 7 / 5 | 0.004 |
| maximum ice thickness (m) | mae | 0 / 2 / 34 | 0 / 0 / 12 | 0.001 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 18 / 5 / 13 | 8 / 1 / 3 | 0.798 |
| break-up (days) | mae | 12 / 5 / 19 | 7 / 1 / 4 | -0.259 |
| effective break-up, ice < 0.1 m (days) | bias | 18 / 6 / 12 | 8 / 2 / 2 | 0.820 |
| effective break-up, ice < 0.1 m (days) | mae | 12 / 2 / 22 | 8 / 0 / 4 | -0.355 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.001 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000491 |
| ice thickness, mean seasonal maximum (m) | mean | changed 11 / unchanged 25 | changed 7 / unchanged 5 | 0.004 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -0.000355 |
| congelation ice, mean seasonal maximum (m) | mean | changed 11 / unchanged 25 | changed 7 / unchanged 5 | 0.004 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 5 / 0 / 31 | 3 / 0 / 9 | -0.005 |
| surface energy residual, largest daily absolute (W m-2) | closure | 7 / 4 / 25 | 3 / 1 / 8 | 0.139 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -5.18e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 21 / 0 / 15 | 10 / 0 / 2 | -0.044 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.52e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.49e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +2.8 | +0.7 -> +1.2 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +4.1 | +2.0 -> +2.8 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.17 | 0.22 -> 0.23 | -5.4 -> -5.4 | -5.6 -> -4.2 | -7.2 -> -5.6 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.19 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -5.4 | -8.1 -> -6.7 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.11 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -11.3 | -14.1 -> -12.7 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -10.7 | -13.3 -> -12.2 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -11.1 | -13.9 -> -12.6 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.13 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -8.6 | -11.7 -> -10.2 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.15 | +3.6 -> +3.6 | -4.5 -> -2.7 | -6.3 -> -4.2 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.16 | 0.19 -> 0.20 | +4.8 -> +4.8 | -1.6 -> -0.1 | -3.2 -> -1.8 | 0.07 -> 0.08 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.547 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.695 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.57e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 113.65 |
| FSWout | 57.71 | 59.64 |
| FLWin | 229.66 | 229.93 |
| FLWout | 263.45 | 263.51 |
| FSENS | -6.75 | -6.44 |
| FLAT | -6.47 | -6.32 |
| PenSW | 7.18 | 7.06 |
| conduction_top | -22.30 | -22.20 |
| conduction_bottom | -21.70 | -21.63 |
| water_flux | 3.93 | 3.82 |
| surface_melt_energy | 23.96 | 23.96 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
