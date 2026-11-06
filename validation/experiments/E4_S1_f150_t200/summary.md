# Experiment E4_S1_f150_t200

Candidate options: `snow_density_scheme='bulk_relaxation'`, `snow_density_fresh=150`, `snow_density_timescale=200`  
Baseline: legacy CLIMo (Parameters defaults). Code: 717f7c4 (with uncommitted changes). Run 2026-10-30 14:13 UTC, 47 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Closure is worse in 46 case-checks (surface energy residual, largest daily absolute (W m-2), surface energy residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 13 / 11 / 12 | 6 / 4 / 2 | -0.084 |
| ice thickness (m) | mae | 13 / 10 / 13 | 6 / 4 / 2 | 1.02e-05 |
| maximum ice thickness (m) | bias | 16 / 8 / 12 | 6 / 4 / 2 | -0.109 |
| maximum ice thickness (m) | mae | 16 / 8 / 12 | 6 / 4 / 2 | -0.017 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.015 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.015 |
| break-up (days) | bias | 6 / 18 / 12 | 2 / 8 / 2 | -2.698 |
| break-up (days) | mae | 5 / 18 / 13 | 1 / 8 / 3 | 1.256 |
| effective break-up, ice < 0.1 m (days) | bias | 4 / 20 / 12 | 1 / 9 / 2 | -2.761 |
| effective break-up, ice < 0.1 m (days) | mae | 3 / 19 / 14 | 1 / 9 / 2 | 1.620 |
| snow depth on the ice (m) | bias | 7 / 2 / 27 | 3 / 0 / 9 | 0.001 |
| snow depth on the ice (m) | mae | 7 / 1 / 28 | 3 / 0 / 9 | -0.002 |
| ice thickness, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.108 |
| snow ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | 0.036 |
| congelation ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.120 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 0 / 24 / 12 | 0 / 10 / 2 | 0.151 |
| surface energy residual, largest daily absolute (W m-2) | closure | 2 / 22 / 12 | 2 / 8 / 2 | 5.234 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.24e-12 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 24 / 0 / 12 | 10 / 0 / 2 | -0.162 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -5.48e-13 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -3.26e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.11 | 0.15 -> 0.14 | +4.9 -> +4.9 | +1.8 -> -1.0 | +0.7 -> -2.2 | 0.06 -> 0.06 |
| yellowknife | station | 50 % | 0.14 -> 0.11 | 0.16 -> 0.13 | +2.2 -> +2.2 | +3.6 -> +0.1 | +2.0 -> -0.9 | 0.07 -> 0.06 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.12 | 0.22 -> 0.13 | -5.4 -> -5.4 | -5.6 -> -9.2 | -7.2 -> -10.9 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.13 | 0.26 -> 0.15 | -6.6 -> -6.6 | -6.7 -> -10.6 | -8.1 -> -12.1 | 0.06 -> 0.05 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.17 | 0.10 -> 0.18 | +0.3 -> +0.3 | -12.4 -> -16.2 | -14.1 -> -17.4 | 0.05 -> 0.05 |
| big_trout_lake | station | 50 % | 0.11 -> 0.14 | 0.11 -> 0.16 | -3.6 -> -3.6 | -12.1 -> -15.6 | -13.3 -> -17.0 | 0.06 -> 0.05 |
| gimli | era5 | 50 % | 0.14 -> 0.19 | 0.15 -> 0.16 | -4.3 -> -4.3 | -12.3 -> -15.9 | -13.9 -> -17.5 | 0.08 -> 0.08 |
| gimli | station | 50 % | 0.14 -> 0.19 | 0.15 -> 0.16 | -1.9 -> -1.9 | -10.3 -> -13.8 | -11.7 -> -15.5 | 0.08 -> 0.08 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.09 | 0.14 -> 0.08 | +3.6 -> +3.6 | -4.5 -> -8.8 | -6.3 -> -10.7 | 0.07 -> 0.07 |
| lac_des_sables | station | 50 % | 0.16 -> 0.10 | 0.19 -> 0.11 | +4.8 -> +4.8 | -1.6 -> -5.8 | -3.2 -> -7.7 | 0.07 -> 0.07 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 1.795 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 67.074 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.71e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 109.86 |
| FSWout | 57.71 | 57.18 |
| FLWin | 229.66 | 228.70 |
| FLWout | 263.45 | 262.03 |
| FSENS | -6.75 | -5.80 |
| FLAT | -6.47 | -6.30 |
| PenSW | 7.18 | 6.82 |
| conduction_top | -22.30 | -20.50 |
| conduction_bottom | -21.70 | -19.51 |
| water_flux | 3.93 | 3.89 |
| surface_melt_energy | 23.96 | 22.00 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
