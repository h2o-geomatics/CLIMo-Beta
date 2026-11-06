# Workflow: forcing first, then the model

CLIMo runs in two steps. **Step 1** prepares the daily meteorological forcing
for a lake or point; **step 2** runs the ice model on it. `climo batch` does
both steps, or only step 1, for many sites listed in a CSV file; the
[Batch tab](gui.md#batch) of the graphical interface does the same.

```
 step 1                          files passed on                 step 2
 climo data  ──────────────►  results/qinghai/forcing.csv  ──►  climo run
 (or your own CSV)            results/qinghai/metadata.json     results/qinghai/climo_daily.csv
                                                                results/qinghai/climo_ice_dates.csv
                                                                results/qinghai/climo_winters.csv
                                                                results/qinghai/climo_summary.csv
```

[how_it_runs.md](how_it_runs.md) explains what happens in each step and how
to choose the options; [install.md](install.md) sets CLIMo up.

## Before you start: the CDS key

The forcing is built from weather stations (no account needed) and the ERA5
reanalysis, which needs a free Copernicus Climate Data Store key in a
`.cdsapirc` file in your home folder. Set it up once by following
[Setting up ERA5 access](data.md#setting-up-era5-access-cds-key). Step 2 needs
no key, and neither does bringing your own forcing.

## Step 1: build (or provide) the forcing

```bash
climo data --lake-name "Qinghai Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/qinghai
climo data --lat 62.45 --lon -114.37 --start 2015-01-01 --end 2016-12-31 --out-dir results/yellowknife
climo data --lake my_lake.geojson --start 2015-01-01 --end 2016-12-31 --out-dir results/my_lake
```

To see which weather stations are near the lake before building anything,
run `climo stations` with the same location and period (see
[data.md](data.md#weather-stations-and-the-distance-limit)); choose the
ones you want per variable with `--station-for` (e.g. `--station-for
Snow=2204100+2204101`, or one station per range of years,
`--station-for "Snow=2204100[1990-2012]+2204101[2013-2024]"`).

Use one folder per lake or point. It holds plain files you can open:

| File | Contents |
|---|---|
| `forcing.csv` | daily forcing in CLIMo format (the input of step 2) |
| `metadata.json` | where each variable came from, stations, ERA5 cells, citations, and under `request` every setting that determines the forcing |
| `map.png` | the lake, the stations considered and used, the reanalysis cells |
| `snow_depth.csv` | snow on the ground as observed at the stations, if any reported it; not part of the forcing ([data.md](data.md#snow-on-the-ground-as-observed)) |

With `--grid` (a whole lake, see [grid.md](grid.md)) the folder holds
`forcing.nc` and `cells.png` instead of `forcing.csv` and `map.png`, plus
`cells.csv` (a mixing depth per cell, for you to edit) and, with `--flake`,
`flake.nc` (ERA5 FLake lake ice for comparison).

`--out results/qinghai` (a file prefix) still works and writes
`results/qinghai_forcing.csv`, `results/qinghai_metadata.json`, ... Reuse
(below) works the same way with either.

Build whole calendar years: the model simulates calendar years and runs the
first year twice (once as spin-up; more often with `--spinup-years`). See [data.md](data.md) for how the
forcing is made.

### Your own forcing

Instead of building the forcing, you can give CLIMo your own weather for one
point: a CSV file with one row per day and these columns, in any order (other
columns are ignored):

| Column | Daily value | Units |
|---|---|---|
| `Date` | the day | YYYY-MM-DD |
| `AirTemp` | mean air temperature | °C |
| `RelHum` | mean relative humidity | %, 0–100 |
| `WindSpd` | mean wind speed | m s⁻¹ |
| `Cloud` | mean cloud cover | fraction, 0–1 |
| `Snow` | snowfall, as the depth of fresh snow | m |

```text
Date,AirTemp,RelHum,WindSpd,Cloud,Snow
2015-01-01,-21.04,70.43,5.01,0.89,0
2015-01-02,-25.43,71.88,7.04,0.64,0
...
2015-01-05,-23.90,75.09,5.22,0.65,0.02
```

- **Every day, with no blanks.** A blank value stops the run with *forcing
  contains missing values*, so fill gaps first. A day left out altogether is
  not caught: it is simply missing from the run, without a warning. A full
  year has 365 or 366 rows.
- **Whole calendar years,** 1 January to 31 December. The model runs calendar
  years, and runs the start year once more before itself as spin-up, so the
  file must hold the start year from 1 January.
- **Snow is fresh snow, in metres per day,** not water equivalent. The run
  converts it to mass with its dry snow density. To give snowfall as water
  equivalent, or snow on the ground, build the forcing and add your own
  snowfall file instead ([data.md](data.md#your-own-snowfall)), which converts
  it.
- **The latitude is not in the file.** You give it with the run; it sets the
  sunshine.

To run it:

- **In the window,** choose the file as the **Forcing file** on the Model tab
  (or with **Load existing forcing...** on the Data tab). Its years are filled
  in; type the **Latitude**, which is filled only for forcing CLIMo built.
- **On the command line,** `climo run --forcing my_lake.csv --latitude 62.45
  --start 2015 --end 2016 --mixing-depth 10`.
- **In a batch,** name the file in the site's `forcing` column and give its
  `latitude` ([the sites CSV](#the-sites-csv)); nothing is built for that site.

This is the format of a single point. Whole-lake forcing is a netCDF file with
the same variables per cell, made by CLIMo ([grid.md](grid.md)).

### Reusing forcing built before

`climo data` records its settings in `metadata.json`. When it is run again
with the same output folder (or prefix), it compares the new request with the
recorded one:

- **Same settings** — the existing forcing is reused and nothing is
  downloaded; it prints `reusing results/qinghai/forcing.csv (built ... with
  the same settings)`.
- **A shorter period** inside the recorded one — the forcing is reused as it
  is, not cut, and the message says so. `climo run` selects its years from
  it. (Stations were then chosen by their coverage of the longer period.)
- **Anything else differs** — location (`--lat/--lon`), lake (`--lake-name`
  and `--pick`, or the outline in `--lake`, compared by its content, not its
  file name), a period not covered, `--source`, `--max-station-distance`,
  `--networks`, `--station`, `--station-for`, `--candidates`, `--min-coverage`,
  `--snow-density`, `--snow-method`, the snowfall file (`--snow-file`,
  compared by its content) and its units, point or `--grid` (and
  `--grid-source`), or the CLIMo version — the forcing is rebuilt, and the
  message lists what changed. `cells.csv` is never overwritten, and
  `flake.nc` is downloaded only if it is missing or covers a shorter period.
- **`--refresh`** always rebuilds.

The station, lake-search and HydroLAKES downloads behind a build are cached
separately in `~/.cache/climo` (see [data.md](data.md#sources)), so even a
rebuild repeats only part of the work. Delete a folder to forget a result.

## Step 2: run the model

```bash
climo run --forcing results/qinghai --start 2015 --end 2016 --mixing-depth 16.8 --snow-percentage 0 50 100 --rho-snow-cold 250 330
```

`--forcing` takes a data folder, a forcing CSV or a `.nc` from `--grid`.
Given a folder (or a CSV with its `metadata.json` next to it), the latitude
is read from the metadata, and the results are written in the folder:
`climo_daily.csv`, `climo_ice_dates.csv`, `climo_winters.csv` and
`climo_summary.csv` (for a grid, `.nc` for the first two and the lake mean
for the others). `--out` writes them elsewhere. For your own CSV give
`--latitude`. Options for grids: `--mixing-depth-file` (a depth per cell)
and `--flake` (see [grid.md](grid.md)); for any run, `--breakup-threshold`
adds the effective break-up ([outputs.md](outputs.md)), and `--spinup-years`
runs more spin-up years, for deep mixed layers
([how_it_runs.md](how_it_runs.md#spin-up-years)). Model options correct the
legacy solver (`--fixes on`, recommended for new work) or replace one legacy
equation, e.g. `--thin-snow-albedo exponential` (see
[options.md](options.md)); without them the run is legacy CLIMo.

The years must lie within the forcing; if they do not, `climo run` says which
`climo data` call builds the missing years. Output columns are described in
[outputs.md](outputs.md).

## Many lakes or points: `climo batch`

```bash
climo batch examples/lakes.csv --start 2015 --end 2016 --mixing-depth 10 --snow-percentage 0 100 --out-dir results/batch
```

For each site, `climo batch` builds or reuses the forcing in
`results/batch/<site>/` (as `climo data --out-dir` would, with the same
reuse rules), then runs every combination of `--snow-percentage` and
`--rho-snow-cold`. `--start` and `--end` are years; the forcing covers them
from 1 January to 31 December.

### Forcing only

```bash
climo batch examples/lakes.csv --start 2015 --end 2016 --forcing-only --out-dir results/batch
```

`--forcing-only` stops after step 1: each site's forcing is built or reused by
the same rules, no model runs, and no site needs a mixing depth (a `mixing_depth`
column is still checked if it has values). The options of the model are not
used. Only `status.csv` and `batch.json` are written, and `batch.json` says
`"forcing_only": true`. A site that names a `forcing` is only looked for, not
opened: `status.csv` says `given`.

The same forcing is what a batch with the model builds, so running the same
command later without `--forcing-only` reuses it. Tables such as `daily.csv` left by
an earlier batch in the same folder stay as they were, and the batch says so;
`batch.json` then describes the forcing-only batch, not those tables.

### The sites CSV

One row per site. [examples/lakes.csv](../examples/lakes.csv):

```
site,lake_name,lat,lon,mixing_depth,grid
qinghai,Qinghai Lake,,,16.8,
baker_lake,Baker Lake Nunavut,,,,
yellowknife,,62.45,-114.37,,
```

| Column | Meaning |
|---|---|
| `site` | **required**: a short id; also the name of the site's folder (letters, digits, `-`, `_`, `.`) |
| `lake_name` | a lake to search by name (as `climo lake`) |
| `outline` | a lake outline file (GeoJSON, shapefile, ...) |
| `lat`, `lon` | a point (decimal degrees); with `lake_name` or `outline`, a point on that lake |
| `forcing` | an existing forcing file (CSV or `.nc`) or data folder: no step 1 for this site |
| `pick` | which match of `lake_name` to use, when several lakes have the name |
| `mixing_depth` | mixed-layer depth (m); empty: `--mixing-depth`. Not needed with `--forcing-only` |
| `latitude` | latitude for the model; empty: the latitude of the forcing point |
| `grid` | `yes` for gridded forcing over the whole lake; empty: `--grid` (lakes only) |
| `fill_depths` | `yes` to fill a grid site's `cells.csv` from lake depth data and run with it ([grid.md](grid.md#filling-the-mixing-depths-from-lake-depth-data)); `mixing_depth` may then stay empty; empty: `--fill-depths`. With `--forcing-only` nothing is filled: the later batch with the model fills it |
| `snow_file` | your own daily snowfall CSV for this site (units: `--snow-units`; see [data.md](data.md#your-own-snowfall)) |
| `stations` | stations chosen per variable, as `climo data --station-for`, separated by `;`, e.g. `"Snow=2204100+2204101; AirTemp,RelHum,WindSpd=2204101"` (quoted, because of the commas), or per range of years, e.g. `Snow=2204100[1990-2012]+2204101[2013-2024]`, or `era5` for no station, e.g. `Cloud=era5`; point forcing only |
| `spinup_years` | spin-up years (1 or more; more for deep mixed layers); empty: `--spinup-years` |
| `snow_method` | `snowfall` or `depth`: Snow from measured snowfall or from the daily rise of snow on the ground (see [data.md](data.md#snowfall-from-snow-depth)); empty: `--snow-method`; point forcing only |
| `fixes` | `off` or `on`: the corrections of the legacy surface-temperature solver (F1, F2 and F3 together, legacy notes 4, 5 and 11; see [options.md](options.md#fixes)); empty: `--fixes` |
| `thin_snow_albedo` | `legacy` or `exponential` (A1, see [options.md](options.md#a1-thin-snow-albedo)); empty: `--thin-snow-albedo` |
| `thin_snow_albedo_depth` | e-folding snow depth (m) of `exponential`, above 0 and at most 1; empty: `--thin-snow-albedo-depth`. Needs `exponential`, from this site or from `--thin-snow-albedo` |
| `ice_type_albedo` | `legacy`, `surface_threshold` or `thickness_weighted` (A2, see [options.md](options.md#a2-ice-type-albedo)); empty: `--ice-type-albedo` |
| `snow_ice_albedo`, `snow_ice_albedo_melt` | albedo of cold and of melting snow ice, above 0 and at most 1; empty: `--snow-ice-albedo`, `--snow-ice-albedo-melt`. Need `surface_threshold` or `thickness_weighted`, from this site or the command line |
| `snow_ice_albedo_thickness` | snow ice at the top (m) above which `surface_threshold` uses the snow-ice albedo, above 0 and at most 1; empty: `--snow-ice-albedo-thickness`. Needs `surface_threshold` |
| `melt_albedo_transition`, `melt_albedo_transition_ice` | how the snow and the ice albedo go from their cold to their melting values: `step` (at the melting point, legacy), `linear` or `logistic` (A3, see [options.md](options.md#a3-melt-albedo-transition)); empty: `--melt-albedo-transition`, `--melt-albedo-transition-ice` |
| `melt_albedo_transition_width`, `melt_albedo_transition_ice_width` | kelvin below the melting point where each transition starts, above 0 and at most 10; empty: `--melt-albedo-transition-width`, `--melt-albedo-transition-ice-width`. Need `linear` or `logistic` for the same surface |
| `snow_density_scheme` | `legacy` or `bulk_relaxation`: the density of the snow on the ice, legacy's two values chosen at every step, or a state variable that mixes in new snow and relaxes towards an equilibrium density (S1, see [options.md](options.md#s1-snow-density)); the snow mass is the same; empty: `--snow-density-scheme` |
| `snow_density_fresh`, `snow_density_timescale`, `snow_density_transition_width` | density new snow takes in the pack (kg m⁻³, above 0 and at most 600), e-folding time of the relaxation (h, at most 8760) and width below melting of the transition to the warm density (K, at most 10); empty: `--snow-density-fresh`, `--snow-density-timescale`, `--snow-density-transition-width`. Need `bulk_relaxation`, from this site or the command line |

Each site needs one location: `lake_name`, `outline`, `lat` and `lon`, or
`forcing`. Empty cells take the command-line defaults (`--mixing-depth`,
`--spinup-years`, `--grid`, `--source`, `--max-station-distance`, `--networks`,
`--candidates`, `--min-coverage`, `--snow-density`, `--snow-method`, which
grid sites ignore, and the model options `--fixes`, `--thin-snow-albedo`,
`--thin-snow-albedo-depth`, `--ice-type-albedo`, `--snow-ice-albedo`,
`--snow-ice-albedo-melt`, `--snow-ice-albedo-thickness`,
`--melt-albedo-transition`, `--melt-albedo-transition-width`,
`--melt-albedo-transition-ice`, `--melt-albedo-transition-ice-width`,
`--snow-density-scheme`,
`--snow-density-fresh`, `--snow-density-timescale`,
`--snow-density-transition-width`).
`--grid-source`, `--flake`, `--breakup-threshold` and the depth options
(`--depth-cap`, `--depth-fraction`, `--depth-minimum`, `--depth-raster`) apply
to every site.
A site that sets a switch to `legacy` (`step` for the melt-albedo
transitions, `off` for `fixes`) runs that part as legacy CLIMo even when the command line
switches it on (e.g. `thin_snow_albedo` `legacy` with
`--thin-snow-albedo exponential`), and a site's switch value drops the
command-line parameters it does not use.
File paths are relative to the CSV's folder. The whole file is checked before
anything is downloaded, and every problem is listed with its line number.
Other columns (e.g. notes) are ignored with a warning.

Lake searches are spaced by `--pause` seconds (default 2), since the
OpenStreetMap search service allows about one request per second; a search
made in the last 30 days is answered from the cache.

### Folder layout and outputs

```
results/batch/
  batch.json          settings of the batch (years, scenarios, defaults, the sites read)
  status.csv          per site: ok or failed (with the error), forcing built / reused / given
  daily.csv           daily results of every site and scenario
  ice_dates.csv       ice dates per ice season of every point site and scenario
  winters.csv         per site, scenario and ice season: maximum ice thickness (and date),
                      snow depth, snow ice and congelation ice
  summary.csv         per site and scenario: season maximum ice thickness (mean, min, max),
                      mean maximum snow depth, mean freeze-up and break-up (day of year)
                      and mean ice duration
  qinghai/            the site's data folder: forcing.csv, metadata.json, map.png
  baker_lake/
  yellowknife/
```

All tables are long tables with `site`, `snow_percentage` and
`rho_snow_cold` columns, ready to filter or pivot. `winters.csv` uses the
model's ice seasons, as in `ice_dates.csv` (1 August to 31 July in the
Northern Hemisphere, 1 February to 31 January in the Southern; labelled e.g.
`2015-2016`, see [outputs.md](outputs.md)); only seasons simulated in full
are listed, so `--start 2015 --end 2016` gives one season. In `summary.csv`
freeze-up and break-up are averaged as dates within their season and given
as day of year, so a freeze-up on day 364 one year and day 5 the next
averages to day 2, not to midsummer.

**Grid sites** write their full netCDF results in their folder
(`climo_daily.nc`, `climo_ice_dates.nc`). `daily.csv`, `winters.csv` and
`summary.csv` hold the daily mean over the cells mostly covered by the lake
(`lake_fraction` ≥ 0.5, or all lake cells if none is); their ice dates are
only in the netCDF file. With `--flake`, grid sites also get ERA5 FLake ice
in their netCDF results (slow to download; see [grid.md](grid.md)).

**Failures.** A site that fails (lake not found, no CDS key, forcing file
missing, ...) does not stop the others: it is listed at the end and in
`status.csv`, and `climo batch` exits with status 1. Fix the problem and run
the same command again: the sites already built are reused.

From Python: `climo.sites.read_sites`, `climo.sites.run_sites` and, for the
forcing only, `climo.sites.build_sites` (with `read_sites(..., need_depth=False)`).
`on_status` reports each site as it moves on, which is how the Batch tab fills its
Status column.
