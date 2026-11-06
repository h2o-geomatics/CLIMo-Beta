# Experiment E2_A2_a0.84

Candidate options: `ice_type_albedo='surface_threshold'`, `snow_ice_albedo=0.84`, `snow_ice_albedo_melt=0.55`, `snow_ice_albedo_thickness=0.05`  
Baseline: legacy CLIMo (Parameters defaults). Code: d128e99 (with uncommitted changes). Run 2026-10-30 02:09 UTC, 48 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Break-up improves while winter ice thickness degrades: the specification says not to accept a change for a better ice-off alone.
- Closure is worse in 10 case-checks (surface energy residual, largest daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 3 / 14 / 19 | 1 / 6 / 5 | 0.006 |
| ice thickness (m) | mae | 1 / 8 / 27 | 0 / 4 / 8 | 0.002 |
| maximum ice thickness (m) | bias | 0 / 3 / 33 | 0 / 1 / 11 | 0.001 |
| maximum ice thickness (m) | mae | 0 / 2 / 34 | 0 / 0 / 12 | 0.000722 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 14 / 8 / 14 | 7 / 2 / 3 | 2.843 |
| break-up (days) | mae | 13 / 8 / 15 | 6 / 2 / 4 | 0.080 |
| effective break-up, ice < 0.1 m (days) | bias | 15 / 7 / 14 | 7 / 2 / 3 | 2.886 |
| effective break-up, ice < 0.1 m (days) | mae | 14 / 8 / 14 | 7 / 2 / 3 | -0.196 |
| snow depth on the ice (m) | bias | 0 / 0 / 36 | 0 / 0 / 12 | 0.000379 |
| snow depth on the ice (m) | mae | 0 / 0 / 36 | 0 / 0 / 12 | 0.000144 |
| ice thickness, mean seasonal maximum (m) | mean | changed 3 / unchanged 33 | changed 1 / unchanged 11 | 0.001 |
| snow ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | -1.4e-06 |
| congelation ice, mean seasonal maximum (m) | mean | changed 0 / unchanged 36 | changed 0 / unchanged 12 | 0.001 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 11 / 0 / 25 | 5 / 0 / 7 | -0.007 |
| surface energy residual, largest daily absolute (W m-2) | closure | 3 / 10 / 23 | 1 / 3 / 8 | 0.600 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 3.29e-13 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 21 / 0 / 15 | 8 / 0 / 4 | -0.064 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | 1.11e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.85e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.13 | 0.15 -> 0.15 | +4.9 -> +4.9 | +1.8 -> +2.2 | +0.7 -> +1.0 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.14 | 0.16 -> 0.16 | +2.2 -> +2.2 | +3.6 -> +5.5 | +2.0 -> +4.0 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.17 | 0.22 -> 0.22 | -5.4 -> -5.4 | -5.6 -> -2.7 | -7.2 -> -4.2 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.20 | 0.26 -> 0.26 | -6.6 -> -6.6 | -6.7 -> -2.9 | -8.1 -> -4.6 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.11 | 0.10 -> 0.10 | +0.3 -> +0.3 | -12.4 -> -9.7 | -14.1 -> -11.2 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.11 | 0.11 -> 0.11 | -3.6 -> -3.6 | -12.1 -> -8.8 | -13.3 -> -10.1 | 0.06 -> 0.06 |
| gimli | era5 | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -4.3 -> -4.3 | -12.3 -> -11.1 | -13.9 -> -12.5 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.14 | 0.15 -> 0.15 | -1.9 -> -1.9 | -10.3 -> -8.1 | -11.7 -> -9.6 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.13 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> +1.8 | -6.3 -> +0.3 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.17 | 0.19 -> 0.19 | +4.8 -> +4.8 | -1.6 -> +5.6 | -3.2 -> +4.2 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.557 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 59.787 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.5e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 116.27 |
| FSWout | 57.71 | 62.75 |
| FLWin | 229.66 | 230.77 |
| FLWout | 263.45 | 264.10 |
| FSENS | -6.75 | -6.41 |
| FLAT | -6.47 | -6.26 |
| PenSW | 7.18 | 7.02 |
| conduction_top | -22.30 | -22.02 |
| conduction_bottom | -21.70 | -21.49 |
| water_flux | 3.93 | 3.80 |
| surface_melt_energy | 23.96 | 23.68 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
