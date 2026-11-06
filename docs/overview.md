> This is the project's main description. To get the beta running, see the
> [quick start](../QUICKSTART.md). `pip`, `git` and Python commands here apply to the source code, which is
> not published yet; in the beta, type the same `climo` commands in the Command Prompt in the program's folder
> ([the command line](../QUICKSTART.md#the-command-line)).

# CLIMo — Canadian Lake Ice Model

[![Tests](https://github.com/h2o-geomatics/CLIMo-Beta/actions/workflows/tests.yml/badge.svg)](https://github.com/h2o-geomatics/CLIMo-Beta/actions/workflows/tests.yml)
[![Documentation](https://img.shields.io/badge/docs-site-blue)](https://h2o-geomatics.github.io/CLIMo/)

CLIMo is a one-dimensional thermodynamic model of lake ice. Driven by daily
air temperature, relative humidity, wind speed, cloud cover and snowfall, it
simulates freeze-up and break-up dates, ice thickness and composition (clear /
congelation ice and snow ice), snow depth, the temperature profile of the
snow and ice, and the components of the surface energy budget.

CLIMo was developed by Duguay et al. (2003) from the sea-ice model of Flato
and Brown (1996). This repository is a documented, source-available Python
version for research use. It also builds the model's input weather for any
lake from weather stations and ERA5, runs single points, whole lakes and many
lakes at once, and compares results with ERA5's own lake ice and with
observations.

**Status:** *CLIMo Classic* — a faithful port of the legacy Python CLIMo. It
reproduces the legacy model results bit for bit (see
`tests/test_classic_regression.py`), with bugs in the output bookkeeping
fixed ([docs/legacy_notes.md](legacy_notes.md)). Optional model
options, all off by default, correct the legacy surface-temperature solver
and replace single legacy equations ([docs/options.md](options.md));
more are planned.

## Start here

1. **Install** — [docs/install.md](install.md) takes you from no Python
   to a first run, step by step, on Windows, macOS or Linux. On Windows,
   type the commands in the Miniforge Prompt or the Command Prompt, not in
   PowerShell, which blocks the environment's activation script.
2. **Understand the steps** — [docs/how_it_runs.md](how_it_runs.md):
   what each command does, which options to choose, how long things take.
3. **Run** — build the forcing for a lake, then run the model on it:

   ```bash
   climo data --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
   climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth 10 --snow-percentage 0 50 100
   ```

   or do it all in a window: `climo gui`.

ERA5 needs a free Copernicus Climate Data Store key, set up once
([how](data.md#setting-up-era5-access-cds-key)). Station data and lake
search need no account, and the model itself runs offline.

## What it can do

| | |
|---|---|
| **Find a lake** | by name (OpenStreetMap), with its HydroLAKES mean depth: `climo lake` |
| **Weather stations** | NOAA ISD and GHCN-Daily worldwide, ECCC in Canada, only within a distance limit of the shore; list them first with `climo stations`, or name the ones to use ([data.md](data.md)) |
| **Comparing snow sources** | a station's measured snowfall and snow on the ground beside ERA5 and ERA5-Land, winter by winter, in the GUI or with `climo snow-compare` ([snow_comparison.md](snow_comparison.md)) |
| **ERA5** | fills what stations lack; or forcing from ERA5 alone |
| **Your own data** | your own forcing CSV, or your own snowfall series |
| **Whole lakes** | every ERA5 (0.25°) or ERA5-Land (0.1°) cell, with a mixing depth per cell you can fill from lake depth data and edit ([grid.md](grid.md)) |
| **Scenarios** | any combination of snow percentages and snow densities in one run |
| **Model options** | `--fixes on` corrects three numerical faults of the legacy solver (recommended for new work); albedo and snow-density options for sensitivity studies; all off by default, so a run without them is legacy CLIMo ([options.md](options.md)) |
| **Many lakes** | `climo batch` with a CSV of sites, or the Batch tab of `climo gui`; forcing only, or forcing and model ([workflow.md](workflow.md)) |
| **Outputs** | daily values, ice dates per winter, maxima per winter, a summary per scenario, and optionally an effective break-up ([outputs.md](outputs.md)) |
| **Comparison** | ERA5 FLake lake ice on the same grid (optional), and validation against ECCC observations at six lakes ([validation.md](validation.md)) |
| **Graphical interface** | `climo gui`: Data, Model, Results and Batch tabs ([gui.md](gui.md)) |

## Quick start in Python

```python
from climo import Parameters, read_forcing, run, run_scenarios, summarise

forcing = read_forcing("tests/data/forcing_YL_70.csv")
params = Parameters(latitude=62.4, mixing_depth=10.0, snow_percentage=100,
                    rho_snow_cold=250.0, rho_snow_warm=450.0)
daily, ice_dates = run(forcing, params, start=2015, end=2024)

# every combination of snow percentage and dry snow density, as long tables
daily, ice_dates = run_scenarios(forcing, latitude=62.4, mixing_depth=10.0, start=2015, end=2024,
                                 snow_percentages=[0, 50, 100], rho_snow_cold_values=[250, 330])
winters, summary = summarise(daily, ice_dates, latitude=62.4)
```

Command line (writes `results/yl70_daily.csv`, `_ice_dates.csv`,
`_winters.csv` and `_summary.csv`):

```bash
climo run --forcing tests/data/forcing_YL_70.csv --start 2015 --end 2024 --latitude 62.4 --mixing-depth 10 --snow-percentage 0 50 100 --rho-snow-cold 250 330 --out results/yl70
```

Because the `Snow` input is a snow depth, changing the dry snow density also
changes the snow mass that falls on the ice: a denser snowpack of the same
depth is heavier, insulates differently and floods the ice sooner.

## Inputs

**Forcing** — a CSV with one row per day:

| Column | Description | Units |
|---|---|---|
| `Date` | date | YYYY-MM-DD |
| `AirTemp` | mean air temperature | °C |
| `RelHum` | mean relative humidity | % |
| `WindSpd` | mean wind speed | m s⁻¹ |
| `Cloud` | mean cloud fraction | 0–1 |
| `Snow` | snowfall, as depth of fresh snow | m day⁻¹ |

Snowfall is given as the depth of fresh snow (as reported by weather
stations) and is converted to snow mass with the dry snow density of the
run. Legacy input files (with an unnamed index column) can be used as they
are; extra columns are ignored. To make your own, see
[docs/workflow.md](workflow.md#your-own-forcing). `climo data` builds this file for any lake
([data.md](data.md)); gridded forcing is a netCDF file with the same
variables ([grid.md](grid.md)).

**Parameters**

| Parameter | Description |
|---|---|
| `latitude` | lake latitude (decimal degrees); used for solar radiation |
| `mixing_depth` | effective mixed-layer depth (m); controls heat storage and freeze-up timing |
| `snow_percentage` | percentage of the input snowfall that accumulates on the ice (redistribution by wind) |
| `rho_snow_cold` | snow density while the snow is below 0 °C (kg m⁻³) |
| `rho_snow_warm` | snow density once the snow layer reaches 0 °C (kg m⁻³); rarely active, see [legacy notes](legacy_notes.md) |
| `breakup_threshold` | optional (m): also report the day the ice thins below this ([outputs.md](outputs.md#effective-break-up---breakup-threshold)) |
| `spinup_years` | copies of the first year run before it (default 1, at least 1); more for deep mixed layers ([how_it_runs.md](how_it_runs.md#spin-up-years)) |
| `fixes` | model option: `off` (default, legacy) or `on`, the three corrections of the legacy surface-temperature solver together (legacy notes 4, 5 and 11); changes no physics; see [options.md](options.md#fixes) |
| `thin_snow_albedo`, `thin_snow_albedo_depth` | model option (A1): albedo of thin snow, `legacy` (default) or `exponential` with an e-folding depth (m, default 0.03); see [options.md](options.md#a1-thin-snow-albedo) |
| `ice_type_albedo`, `snow_ice_albedo`, `snow_ice_albedo_melt`, `snow_ice_albedo_thickness` | model option (A2): albedo of snow-free ice by ice type, `legacy` (default), `surface_threshold` or `thickness_weighted`, with snow-ice albedos (default 0.75 cold, 0.55 melting) and threshold (m, default 0.05); see [options.md](options.md#a2-ice-type-albedo) |
| `melt_albedo_transition`, `melt_albedo_transition_ice`, `melt_albedo_transition_width`, `melt_albedo_transition_ice_width` | model option (A3): how the snow and the ice albedo go from cold to melting values, `step` (at 0 °C, default), `linear` or `logistic` over a width below the melting point (K, default 1); see [options.md](options.md#a3-melt-albedo-transition) |
| `snow_density_scheme`, `snow_density_fresh`, `snow_density_timescale`, `snow_density_transition_width` | model option (S1): density of the snow on the ice, `legacy` (default; `rho_snow_cold` or `rho_snow_warm` at each step) or `bulk_relaxation` (a state variable: new snow mixed in at a fresh density, default 330 kg m⁻³, relaxing with an e-folding time, default 100 h, towards `rho_snow_cold`, and towards `rho_snow_warm` within a width below melting, default 1 K; the snow mass is unchanged); see [options.md](options.md#s1-snow-density) |

The first simulated year is preceded by a spin-up year (a copy of the first
year; `spinup_years` copies) starting from 1 m of ice and 5 cm of snow; it is
not included in the output. A `SpinUpWarning` says when the starting ice does
not melt out during the spin-up.

## Model options

Every model option defaults to legacy CLIMo, so a run without options
reproduces it exactly. For new work, switch on the solver corrections:

```bash
climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth 10 --fixes on
```

or `Parameters(..., fixes="on")` in Python, or the Fixes box in the GUI's
Model tab. The albedo (A1-A3) and snow-density (S1) options, where each
helps and where it hurts at the six validation lakes, and the recommended
combinations are in [docs/options.md](options.md).

## Outputs

`daily` — one row per day; column names follow the legacy CLIMo outputs,
except the layer depths. `ice_dates` — freeze-up and break-up (day of year
and date) and ice duration for each ice season, e.g. `2015-2016`. `winters`
— maxima per ice season; `summary` — one row per scenario. Runs with several
scenarios add `snow_percentage` and `rho_snow_cold` columns. See
[docs/outputs.md](outputs.md).

## Documentation

The documentation is also published as a searchable site:
<https://h2o-geomatics.github.io/CLIMo/>.

- [docs/install.md](install.md) — installing Python and CLIMo, a first run, updating, troubleshooting
- [docs/how_it_runs.md](how_it_runs.md) — what each step does, choosing options, run times, where files go
- [docs/workflow.md](workflow.md) — the two steps, data folders and reuse, many sites with `climo batch`
- [docs/data.md](data.md) — forcing from weather stations and ERA5: the CDS key, stations and the distance limit, your own snowfall, method
- [docs/grid.md](grid.md) — whole lakes: ERA5 and ERA5-Land grids, mixing depth per cell, ERA5 FLake comparison
- [docs/gui.md](gui.md) — the graphical interface
- [docs/outputs.md](outputs.md) — output columns, ice seasons, effective break-up, winters and summary
- [docs/options.md](options.md) — the model options (`fixes`, albedo, snow density): what each does, where it helps and hurts, recommended combinations
- [docs/model.md](model.md) — equations of CLIMo Classic, with references
- [docs/formulations.md](formulations.md) — the science record of every model option: sources, equations, validation experiments
- [docs/validation.md](validation.md) — validation against ECCC ice thickness and lake-ice dates at six lakes, with ERA5 FLake
- [docs/legacy_notes.md](legacy_notes.md) — legacy behaviours reproduced by CLIMo Classic, and known issues
- [CHANGELOG.md](../CHANGELOG.md) — what changed in each release

## Testing

```bash
pip install -e ".[test]"
pytest
```

Tests that download data are skipped unless `CLIMO_NETWORK_TESTS=1` is set.

The regression test compares CLIMo Classic with outputs of the unmodified
legacy code (`legacy/CLIMo_legacy.py`), produced by
`scripts/make_legacy_reference.py`. It is exact to the last bit on Windows,
where those outputs were made. The legacy code itself rounds slightly
differently on other systems (details), so there it
allows 1e-6.

## Contributing

See CONTRIBUTING.md. Report security problems as described
in SECURITY.md.

## Licence

CLIMo is **source-available** under the [PolyForm Noncommercial License
1.0.0](../LICENSE):

- **Free** for research, teaching, personal study and any other
  non-commercial purpose, and for use by universities, public research
  organisations, government bodies and charities, whatever their funding.
  You may change the code and share it, with the licence and its notice.
- **Commercial use** — for example paid services or products built on CLIMo
  or its results — needs a separate licence from **H2O Geomatics**:
  info@h2ogeomatics.com.

Data shipped in this repository (observation extracts, test forcing) keep
their own terms; see [NOTICE.md](../NOTICE.md). Data CLIMo downloads keep the
terms of their providers, listed in [docs/data.md](data.md).

## How to cite

Please cite the model description:

> Duguay, C. R., Flato, G. M., Jeffries, M. O., Ménard, P., Morris, K., and Rouse, W. R. (2003).
> Ice-cover variability on shallow lakes at high latitudes: model simulations and observations.
> *Hydrological Processes*, 17(17), 3465–3483. https://doi.org/10.1002/hyp.1394

and this software (see `CITATION.cff`).

## References

- Duguay, C. R., et al. (2003). *Hydrological Processes*, 17, 3465–3483. https://doi.org/10.1002/hyp.1394
- Flato, G. M., and Brown, R. D. (1996). Variability and climate sensitivity of landfast Arctic sea ice. *Journal of Geophysical Research*, 101(C10), 25767–25777. https://doi.org/10.1029/96JC02431
- Ménard, P., Duguay, C. R., Flato, G. M., and Rouse, W. R. (2002). Simulation of ice phenology on Great Slave Lake, Northwest Territories, Canada. *Hydrological Processes*, 16, 3691–3706. https://doi.org/10.1002/hyp.1230
