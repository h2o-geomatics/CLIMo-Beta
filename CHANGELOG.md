# Changelog

All notable changes to CLIMo are listed here. Versions follow
[semantic versioning](https://semver.org/): with every model option at its
default, a run reproduces legacy CLIMo bit for bit, and a release that
changed that would be a new major version.

## 1.1.0b1 — 2026-11-04

The first beta: a Windows program for testers, built from the changes below.

### Tools

- Batch forcing. `climo batch --forcing-only` builds or reuses each site's forcing
  by the same rules as a batch with the model, runs no model, needs no mixing depth,
  and writes `status.csv` and `batch.json`; a site that fails does not stop the
  others, and a later batch with the model reuses the forcing (`climo.sites.build_sites`).
- New **Batch** tab in the graphical interface: a table of sites you can open from a
  sites CSV, add the Data tab's lake to (with its chosen stations), edit and save as
  CSV, the settings they share, the status of each site as it runs, forcing only or
  forcing and model, and a button to load a finished site's forcing into the Model tab.
  It uses the checks of `climo batch`, so every problem is listed before anything is
  downloaded ([docs/gui.md](docs/gui.md#batch)).
- From Python: `climo.sites.check_sites` checks a table of sites as `read_sites` does a
  CSV, `read_sites` has `need_depth`, and `run_sites` and `build_sites` have `on_status`.
  With a site left without a mixing depth, the message now reads `no mixing_depth: fill
  the mixing_depth column or give a default (--mixing-depth)`.
- Snow on the ground at stations is no longer missed. The station table and
  `<out>_stations.csv` show a station's coverage by both measured snowfall and
  snow on the ground, whichever is in use, and changing **Snow from** in the
  graphical interface finds the stations again by itself.
- With `--snow-method depth`, a station is judged on the days from November to
  April, so one that leaves the summer empty can be chosen automatically. This
  can change which station `Snow` comes from with that method. Forcing built with
  measured snowfall, the default, does not change.
- New `snow_depth.csv` next to point forcing: snow on the ground as observed at
  the stations. It is not part of the forcing. The Results tab can draw it on the
  ice and snow figure (`climo.plots.add_observed_depth`).
- It is now clear where a forcing would come from before it is built. A panel at
  the top of **2. Weather stations** says, per variable, which station or reanalysis
  is used, with its distance and coverage, and warns when variables come from
  different places. Each variable has a list to change it.
- New **Use only this site**: every variable from one site, and ERA5 for what the
  site doesn't report, never another station.
- `era5` can stand in place of a station in a choice (`--station-for Cloud=era5`,
  the `stations` column of a sites file, the lists above). Results with no such
  choice don't change, and the automatic choice is the same as before.
- The station table can be filtered by name or id, and the networks searched can be
  ticked under **Station search** (`--networks` on the command line).
- New snow comparison: a station's measured snowfall and snow on the ground beside
  ERA5 and ERA5-Land, winter by winter, with checks for snowfall that looks
  incomplete (**Compare snow...** in the graphical interface, `climo snow-compare`;
  [docs/snow_comparison.md](docs/snow_comparison.md)). It changes no forcing. ERA5
  snowfall is estimated, since the ARCO store of ERA5 holds none.
- The snow comparison at a point on a lake no longer shows ERA5-Land snow depth as
  a flat line at 0, as depth is 0 over a lake cell. Depth now comes from the land
  cell nearest the station, recognised from its snow since ARCO holds no land mask,
  and is left blank if there is none within 1°. Reanalysis snowfall and the checks
  on measured snowfall stay as they were, at the lake point. The figure, the window,
  the command line and the table of winters say which cell each column uses.
- Stations not downloaded now say *not checked* in the graphical interface, with
  a **Check all** button.
- The Data tab of the graphical interface has three steps: Lake and settings,
  Weather stations, and Forcing. Less used settings are in sections that say what
  they hold while closed. The station table has the width of the window, groups
  the records of one site, sorts by any column and can be filtered.

- Mixing depth per cell from lake depth data: `climo depths`, `climo data --grid
  --fill-depths`, `climo batch --fill-depths` (or a `fill_depths` column), and
  **Fill from bathymetry...** and **Show map** in the per-cell window. It fills
  `cells.csv` from GLOBathy or your own GeoTIFF: the mean over the cell's lake
  pixels of `min(fraction × depth, cap)`, with a cap of 40 m by default, and records
  where each value came from. Edits made by hand are kept unless you ask otherwise.
  GLOBathy's depths are modelled, and perform poorly in places, clearly so on Great
  Slave Lake, though the order from shallow to deep mostly holds, and that is what
  modelled freeze-up depends on most:
  [docs/grid.md](docs/grid.md#filling-the-mixing-depths-from-lake-depth-data)
  gives the science, the sources, the validation and the weaknesses.
- `cells.csv` written by `climo data` has two new columns, `depth_source` and
  `filled_depth_m`. Runs ignore them. `rasterio` joins the `data` extra.
- The model does not change: with depths set as before, results are the same.
- Fixed building forcing for a lake with no weather station within the
  distance limit, such as Prosperous Lake at the default 5 km, which failed with
  `KeyError: 'role'` instead of using ERA5. Model results do not change.
- The graphical interface has a Help menu: saving the ERA5 key, the
  documentation, report information, the log folder and reporting a problem. It
  shows the version in its title, and keeps a log file (`docs/gui.md`).
- `climo --version`.
- A Windows program for beta testers that needs no Python (`packaging/`).
- Documentation site, continuous integration, and contributing and security
  guides.
- The regression test is exact on Windows, where its reference outputs were
  made, and allows 1e-6 on other systems, where it failed on rounding of the
  math library alone. No change to the model.
- Documentation fixes: a working `#spin-up-years` link, the sources table in
  `docs/data.md`, and a stray bullet in `docs/formulations.md`.

See [Snowfall from snow depth](docs/data.md#snowfall-from-snow-depth). Model
results for a given forcing do not change.

## 1.0.1 — 2026-11-02

### Tools

- Fixed Find stations and Check selected in the graphical interface, which
  failed for every lake with "too many values to unpack (expected 2)" after
  the stations had been downloaded. The command line was not affected, and
  model results do not change.

## 1.0.0 — 2026-11-02

First release of CLIMo Classic, the Python port of the legacy CLIMo
(Duguay et al., 2003).

### Model

- CLIMo Classic reproduces the legacy model results bit for bit
  (`tests/test_classic_regression.py`), with bugs in the output bookkeeping
  fixed ([docs/legacy_notes.md](docs/legacy_notes.md)).
- Spin-up years setting (`--spinup-years`), with a warning when the
  starting ice does not melt out during the spin-up.
- Effective break-up diagnostic (`--breakup-threshold`).
- Model options, all off by default ([docs/options.md](docs/options.md);
  science and validation in [docs/formulations.md](docs/formulations.md)):
  - `fixes`: corrections of the legacy surface-temperature solver (legacy
    notes 4, 5 and 11), recommended for new work;
  - `thin_snow_albedo` (A1), `ice_type_albedo` (A2),
    `melt_albedo_transition` / `melt_albedo_transition_ice` (A3): albedo
    options;
  - `snow_density_scheme` (S1): snow density as a state variable.

### Forcing data

- Forcing for any lake from NOAA ISD, GHCN-Daily and ECCC stations and
  ERA5 / ERA5-Land, or your own forcing or snowfall files
  ([docs/data.md](docs/data.md)).
- Every station within 5 km of the shore is checked, a station is chosen
  per variable, successive station ids at one site are joined, and stations
  can be chosen by hand for ranges of years (station coverage by year).
- Snowfall from daily snow-on-the-ground differences as an option
  (`--snow-method depth`).
- Whole-lake runs on ERA5 / ERA5-Land grids with a mixing depth per cell,
  and comparison with ERA5 FLake lake ice ([docs/grid.md](docs/grid.md)).

### Tools

- `climo` command line (`lake`, `stations`, `data`, `run`, `batch`) and a
  graphical interface (`climo gui`) with Data, Model and Results tabs.
- Validation against ECCC ice thickness and ice dates at six Canadian lakes,
  and an experiment harness (`validation/compare.py`) that judges each model
  option against legacy CLIMo on frozen forcing, with mass and energy
  closure checks ([docs/validation.md](docs/validation.md)).
