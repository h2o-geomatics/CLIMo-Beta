# Experiment E4_S1_f150_t4800

Candidate options: `snow_density_scheme='bulk_relaxation'`, `snow_density_fresh=150`, `snow_density_timescale=4800`  
Baseline: legacy CLIMo (Parameters defaults). Code: 717f7c4 (with uncommitted changes). Run 2026-10-30 14:13 UTC, 48 s for 72 model runs.  
Forcing: frozen, 12 files checked against `validation/frozen_forcing.json`. Cases: 36 (6 sites x station/ERA5 forcing x 3 snow scenarios); "matching snow" is the scenario per site and forcing whose snow on the ice is closest to the measurements in the baseline.

**Warnings**

- Snow depth on the ice degrades in more cases than it improves.
- Closure is worse in 48 case-checks (surface energy residual, largest daily absolute (W m-2), surface energy residual, mean of daily absolute (W m-2)).

## Verdicts

Improved / degraded / unchanged cases per metric (MAE and closure: smaller is better; bias: smaller absolute value is better; unchanged within 0.005 m, 0.5 days, or 1% for closure, at least 0.01 W m-2 or 1e-06 kg m-2). Model amounts without observations show changed / unchanged. Mean difference is candidate minus baseline over all cases.

| Metric | Statistic | All cases: improved / degraded / unchanged | Matching snow | Mean difference |
|---|---|---|---|---|
| ice thickness (m) | bias | 5 / 19 / 12 | 1 / 9 / 2 | -0.225 |
| ice thickness (m) | mae | 6 / 18 / 12 | 2 / 8 / 2 | 0.101 |
| maximum ice thickness (m) | bias | 6 / 17 / 13 | 1 / 8 / 3 | -0.326 |
| maximum ice thickness (m) | mae | 7 / 16 / 13 | 2 / 7 / 3 | 0.130 |
| freeze-up (days) | bias | 1 / 0 / 35 | 0 / 0 / 12 | -0.010 |
| freeze-up (days) | mae | 0 / 1 / 35 | 0 / 0 / 12 | 0.021 |
| break-up (days) | bias | 1 / 22 / 13 | 0 / 10 / 2 | -8.494 |
| break-up (days) | mae | 2 / 22 / 12 | 0 / 10 / 2 | 6.417 |
| effective break-up, ice < 0.1 m (days) | bias | 1 / 23 / 12 | 0 / 10 / 2 | -8.696 |
| effective break-up, ice < 0.1 m (days) | mae | 1 / 23 / 12 | 0 / 10 / 2 | 7.090 |
| snow depth on the ice (m) | bias | 0 / 24 / 12 | 0 / 10 / 2 | 0.031 |
| snow depth on the ice (m) | mae | 0 / 23 / 13 | 0 / 9 / 3 | 0.019 |
| ice thickness, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.327 |
| snow ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | 0.082 |
| congelation ice, mean seasonal maximum (m) | mean | changed 24 / unchanged 12 | changed 10 / unchanged 2 | -0.372 |
| surface energy residual, mean of daily absolute (W m-2) | closure | 0 / 24 / 12 | 0 / 10 / 2 | 0.450 |
| surface energy residual, largest daily absolute (W m-2) | closure | 0 / 24 / 12 | 0 / 10 / 2 | 73.157 |
| bottom energy residual, largest daily absolute (W m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -1.07e-11 |
| short-wave residual, mean of daily absolute (W m-2) | closure | 24 / 0 / 12 | 10 / 0 / 2 | -0.505 |
| ice mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -2.41e-12 |
| snow mass residual, largest daily absolute (kg m-2) | closure | 0 / 0 / 36 | 0 / 0 / 12 | -4.18e-14 |

## Matching snow scenario, per site

Baseline -> candidate. Thickness and seasonal maximum: MAE (m); dates: bias (days, model minus observed); snow: MAE (m).

| Site | Forcing | Snow | Thickness | Seasonal max | Freeze-up | Break-up | Effective break-up | Snow on ice |
|---|---|---|---|---|---|---|---|---|
| yellowknife | era5 | 50 % | 0.13 -> 0.35 | 0.15 -> 0.51 | +4.9 -> +4.9 | +1.8 -> -12.7 | +0.7 -> -14.1 | 0.06 -> 0.07 |
| yellowknife | station | 50 % | 0.14 -> 0.34 | 0.16 -> 0.49 | +2.2 -> +2.2 | +3.6 -> -11.2 | +2.0 -> -12.9 | 0.07 -> 0.07 |
| baker_lake | era5 | 0 % | 0.09 -> 0.09 | 0.10 -> 0.10 | -8.3 -> -8.3 | +2.7 -> +2.7 | +1.7 -> +1.7 | 0.04 -> 0.04 |
| baker_lake | station | 0 % | 0.10 -> 0.10 | 0.12 -> 0.12 | -8.3 -> -8.3 | +3.1 -> +3.1 | +1.8 -> +1.8 | 0.04 -> 0.04 |
| island_lake | era5 | 50 % | 0.16 -> 0.23 | 0.22 -> 0.27 | -5.4 -> -5.4 | -5.6 -> -19.3 | -7.2 -> -21.2 | 0.06 -> 0.06 |
| island_lake | station | 50 % | 0.19 -> 0.21 | 0.26 -> 0.25 | -6.6 -> -6.6 | -6.7 -> -20.6 | -8.1 -> -22.5 | 0.06 -> 0.06 |
| big_trout_lake | era5 | 50 % | 0.12 -> 0.37 | 0.10 -> 0.52 | +0.3 -> +0.3 | -12.4 -> -25.8 | -14.1 -> -27.7 | 0.05 -> 0.06 |
| big_trout_lake | station | 50 % | 0.11 -> 0.35 | 0.11 -> 0.50 | -3.6 -> -3.6 | -12.1 -> -25.3 | -13.3 -> -27.1 | 0.06 -> 0.07 |
| gimli | era5 | 50 % | 0.14 -> 0.35 | 0.15 -> 0.40 | -4.3 -> -4.3 | -12.3 -> -24.3 | -13.9 -> -26.0 | 0.08 -> 0.10 |
| gimli | station | 50 % | 0.14 -> 0.33 | 0.15 -> 0.36 | -1.9 -> -1.9 | -10.3 -> -21.9 | -11.7 -> -24.2 | 0.08 -> 0.11 |
| lac_des_sables | era5 | 50 % | 0.13 -> 0.12 | 0.14 -> 0.14 | +3.6 -> +3.6 | -4.5 -> -12.0 | -6.3 -> -13.7 | 0.07 -> 0.09 |
| lac_des_sables | station | 50 % | 0.16 -> 0.11 | 0.19 -> 0.13 | +4.8 -> +4.8 | -1.6 -> -9.2 | -3.2 -> -11.1 | 0.07 -> 0.09 |

## Closure

Largest value over the cases (baseline / candidate). Residuals are explained in `climo/closure.py` and docs/validation.md.

| Check | Baseline | Candidate |
|---|---|---|
| surface energy residual, mean of daily absolute (W m-2) | 1.559 | 2.711 |
| surface energy residual, largest daily absolute (W m-2) | 59.787 | 2209.439 |
| bottom energy residual, largest daily absolute (W m-2) | 1.02e-10 | 1.02e-10 |
| short-wave residual, mean of daily absolute (W m-2) | 5.287 | 5.287 |
| ice mass residual, largest daily absolute (kg m-2) | 3.23e-11 | 3.23e-11 |
| snow mass residual, largest daily absolute (kg m-2) | 2.63e-12 | 2.69e-12 |

Energy terms, mean over the time steps with ice and over all cases (W m-2):

| Term | Baseline | Candidate |
|---|---|---|
| FSWin | 112.40 | 104.51 |
| FSWout | 57.71 | 56.05 |
| FLWin | 229.66 | 226.71 |
| FLWout | 263.45 | 258.34 |
| FSENS | -6.75 | -4.04 |
| FLAT | -6.47 | -5.99 |
| PenSW | 7.18 | 6.04 |
| conduction_top | -22.30 | -16.41 |
| conduction_bottom | -21.70 | -15.08 |
| water_flux | 3.93 | 3.72 |
| surface_melt_energy | 23.96 | 18.12 |

## Checks of the baseline

- Baseline against the committed validation results (validation/results/metrics.csv, 3 decimals): 432 of 432 values equal.
- Baseline against the stored E0 baseline: identical.

Files: `comparison.csv` (every case and metric), `summary.csv` (the counts above), `baseline_metrics.csv`, `candidate_metrics.csv`, `closure.csv`, `run.json`.
