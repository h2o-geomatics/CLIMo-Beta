> **In the beta, open the window by double-clicking `CLIMo-GUI.exe`.** Nothing needs installing.
> `climo gui` also works, typed in the Command Prompt in the program's folder
> ([the command line](../QUICKSTART.md#the-command-line)); `python -m climo.gui` and the data extras below are
> for the source code.

# Graphical interface

```bash
climo gui
```

or `python -m climo.gui`. The window replaces the single window of the
legacy program. It has four tabs (Data, Model, Results and
[Batch](#batch), for many lakes) and a log at the bottom, where long tasks
(downloads, batch and grid runs) report their progress while the window
stays usable. Problems are shown in a dialog: missing or invalid inputs, a
missing ERA5 key (with the setup steps from
[Setting up ERA5 access](data.md#setting-up-era5-access-cds-key)), no
station found, and so on.

The interface only calls the package (`climo.data`, `climo.run_scenarios`,
`climo.grid`, `climo.plots`, `climo.sites`) and writes the same files as the
command line, so anything done in it can be repeated with `climo data`,
`climo run` and `climo batch`.
Building forcing needs the data extras: `pip install -e ".[data]"`.

## Help

- **ERA5 key...** opens the steps for getting a free key, with links, and saves
  the key you paste into the file the command line reads too (`.cdsapirc`).
- **Documentation** opens the `docs` folder beside the program (or in the
  source code), which holds these pages as Markdown (`.md`) files. Any text
  editor opens them; Windows may ask which app to use. They are also shown
  formatted on [GitHub](https://github.com/h2o-geomatics/CLIMo-Beta/tree/main/docs),
  which it opens when there is no `docs` folder.
- **Copy report information** copies the version, system and where CLIMo keeps
  its files, to paste into a report. It says whether a key is set, never the key.
- **Open log folder** shows `climo-gui.log` in the cache folder (`~/.cache/climo/logs`,
  on Windows `C:\Users\<you>\.cache\climo\logs`).
  It holds what the window's log showed, and the details of errors. The key is
  removed from anything written there. A log over 5 MB starts afresh.
- **Report a problem** opens the form for reporting one.
- **About CLIMo** shows the version and build, the licence, and where the
  licence and notices files are.

## Data

Builds a forcing file for a lake, as `climo data` does (see [data.md](data.md)).
The tab has three steps, one page each, with **Find stations**, **Build
forcing** and **Load existing forcing...** below them on every page:

1. **Lake and settings**: the lake, and the settings below.
2. **Weather stations**: the stations near the lake (Find stations opens it).
3. **Forcing**: the report and map of the forcing (Build forcing and Load
   existing forcing open it).

You don't have to go through them in order: Build forcing finds and chooses
stations by itself, as `climo data` does.

- **Lake**, one of:
  - *By name*: type a name and press Search. Matches are listed with their
    area, region and HydroLAKES mean depth; pick one. Its location fills the
    latitude and longitude fields.
  - *Latitude / longitude*: a point (decimal degrees). With a name or an
    outline, these fields set the point CLIMo represents, e.g. one bay of a
    large lake; leave them empty for a point inside the lake.
  - *Outline file*: a GeoJSON, shapefile or other outline of the lake.
- **Settings**: start and end dates (YYYY-MM-DD); the source (`auto`: stations
  where they have enough data, otherwise ERA5; `station`; `era5`); **Snow from**;
  and the output prefix.
  - **Snow from**: `snowfall` (measured daily snowfall, the default) or
    `depth` (the daily rise of snow on the ground at stations, empty days
    after the snow has gone counting as 0, short station gaps keeping the
    station's change of depth and other days taking ERA5-Land snowfall at
    the conversion density, a different basis from the station's rises; point
    forcing only; see [data.md](data.md#snowfall-from-snow-depth)). After
    Find stations, changing it finds them again by itself (downloads are
    cached): the stations listed depend on it.

  The other settings are in sections that open with a click. **While closed,
  each says what its settings are**, so none is at work out of sight:

  - **Station search**: the **max station distance** (km): stations farther
    than this from the shore (or from the point, without an outline) are never
    used; empty for the default, 5 km from the shore of a lake or 50 km from a
    point. The **min coverage**: the fraction of days a station needs of a
    variable to be chosen automatically (default 0.9). **Candidates**: check only
    the nearest N stations of each kind; empty checks every station whose record
    could reach the min coverage.
  - **Snow conversion and your own snowfall**: the density used to convert ERA5
    snowfall to snow depth, and an optional **snowfall file**, your own daily
    snowfall (Date, Snow) in m or cm of fresh snow or mm of water equivalent, or
    snow on the ground in cm or m (`cm-depth`, `m-depth`), which replaces station
    or ERA5 snowfall (see [data.md](data.md#your-own-snowfall)).
  - **Whole lake (grid)**: forcing of every cell touching the lake (netCDF, needs
    a name or outline), on the `era5` (0.25°) or `era5-land` (0.1°) grid (see
    [grid.md](grid.md)). *Also ERA5 FLake lake ice*, grids only and **off by
    default**: ERA5's own lake ice for comparison. Ticking it shows how many CDS
    requests are needed and roughly how long they take (often an hour or more).
- **Where the forcing comes from**: at the top of **2. Weather stations**, one
  line per variable says where it would come from, before anything is built: the
  station, its network and distance, how much of the period it covers and about
  how many days ERA5 fills. [The next section](#choosing-where-the-forcing-comes-from)
  says how to change it.
- **Weather stations**: **Find stations** lists the stations within the
  distance limit, nearest the lake point first, as `climo stations`, with their
  distance from the point and the shore, one coverage column per variable
  (AirTemp, RelHum, WindSpd, Cloud, and for snow both **Snowfall** and **Snow
  depth**, with `*` on the one **Snow from** uses), network, kind, id and years of
  record, and shows them on a map under the table (drag the bar between them to
  resize). Snow depth is judged over
  [November to April](data.md#how-a-station-is-judged-on-snow-on-the-ground),
  the others over all days. Every station whose record could reach the min
  coverage is downloaded and checked, with progress in the log; the window
  stays usable meanwhile. The others say *not checked*. Records of one station under
  successive ids appear also spliced, with the ids joined by `+` (see
  [data.md](data.md#weather-stations-and-the-distance-limit)).
  - **Sites.** Records within 1 km of each other are one row, whatever their
    network or kind, since one place is often listed by ISD, ECCC and GHCN-Daily:
    click `+` to see them. The row of a site shows the name of its nearest record
    and each column's best coverage among its records, so open it to see which
    record that is. Grouping only tidies the table: the choice of stations does
    not use it. To use a station or check its coverage, select its record; Check
    selected also takes a whole site.
  - **Green** rows have enough for the automatic choice: temperature, humidity
    and wind for an hourly record, the measure **Snow from** uses for a daily one.
    A site is green if one of its records is, so a site can be green for hourly
    data while its snow is not. Grey rows are not checked.
  - **Sorting and filters.** Click a heading to sort by it (coverage starts from
    the best), and again to reverse. **Name or id** keeps the rows containing what
    you type, e.g. `hay`. **Show** limits the table to hourly or daily
    (snow) records, and **Hide stations without snow data** removes hourly records
    and stations checked and found empty for snow; those not checked stay, since
    their data are not known yet. The line under the table says how many rows the
    filters hide, and they never change which stations are used.
  - **Check selected** downloads and checks the selected rows that say *not
    checked*, and **Check all** does so for every one of them.
  - **Compare snow...** opens the [snow comparison](snow_comparison.md) of the
    selected daily (snow) station, or of the first one in a selected site: its
    measured snowfall and snow on the ground beside ERA5 and ERA5-Land, winter by
    winter, with checks for snowfall that looks incomplete. It is read from ARCO
    in the background, and **Use this station for Snow** chooses the station. The
    window says which reanalysis cell each column uses: snowfall at the lake point,
    snow depth at the land cell nearest the station.
  - **By year...** opens the *Station coverage by year* window (below): which
    station covers which years, and stations chosen for ranges of years.
  - **Use for...** uses the selected row for the variables you pick from its
    menu (temperature, humidity and wind together, or one variable). Select
    several rows of one kind to join them into one record, oldest first (the
    newer is used where they overlap). A station you choose is used whatever
    its coverage, ERA5 filling its gaps; the choices are listed under the
    buttons and in the table's *Used for* column. **Automatic choice for
    every variable**, at the bottom of the menu, clears them. Variables you
    do not choose are chosen automatically: the station nearest the lake
    point with enough coverage of that variable.
- **Build forcing** writes `<prefix>_forcing.csv`, `_metadata.json` and
  `_map.png` (or `_forcing.nc`, `_cells.png`, `_cells.csv` and, with FLake,
  `_flake.nc` for a grid), then shows the report of where each variable came
  from and the map. As with `climo data`, a forcing built before with the
  same settings is reused instead of downloaded again.
- **Load existing forcing...** uses a forcing file built earlier (CSV or
  `.nc`), or [your own](workflow.md#your-own-forcing), instead.

### Choosing where the forcing comes from

By default each variable takes the nearest station with enough data, **so
variables can come from different places**. On Great Slave Lake, for 2015 to 2024,
temperature comes from Yellowknife, 75 km from the lake point, and wind from Inner
Whalebacks, 5 km from it. The panel says so when that happens.

- **To use one place**, for example just Hay River: type `hay` in **Name or id**,
  select the site or any of its records, and press **Use only this site**. Every
  variable then comes from that site. If no one record has everything, each comes
  from the site's record that has it (at Hay River, cloud is only in an ISD record),
  and what the site doesn't report at all comes from ERA5, **never from another
  station**. Records not checked are not considered; **Check selected** adds them.
- **To change one variable**, use its list in the panel: *Automatic*, *ERA5* (*ERA5-Land*
  for Snow), or any checked station with data for it, nearest first, with its
  coverage. A station you choose is used whatever its coverage, with ERA5 filling
  its gaps.
- **To use only some networks**, untick the others under **Station search** in step 1
  and press **Find stations** again.
- **To use a station farther away**, raise **Max station distance** and press Find
  stations. It is a hard limit, so a station beyond it isn't listed. A larger limit
  also lets the automatic choice reach farther, so choose the station for every
  variable with **Use only this site** if that is all you want from it.

The panel can't be changed with a whole-lake grid, source `era5`, or (for Snow) your
own snowfall file, and says which applies. If you change a setting that decides which
stations are listed, it asks you to press Find stations again. On a short screen the
panel starts closed, and says the same in one line.

The choices are the same as `--station-for` on the command line, including
`Cloud=era5`, and the line at the bottom of the panel gives them in that form.
After a build, **3. Forcing** reports what was used, and `metadata.json` records it.
[data.md](data.md#choosing-from-one-place) shows how much the choice can matter.

### Station coverage by year

The window opened by **By year...** shows, for the variable picked at the
top (AirTemp, RelHum, WindSpd, Cloud or Snow), the stations of its kind
found by **Find stations** as rows, nearest the lake point first (id, name,
distance), and the years of the period as columns. Each year is shaded by
the share of its days with data: dark from 90 %, lighter from 50 %, palest
below, blank without data (for Snow with **Snow from** `depth`, of its days
from November to April). Hover over a year for the exact value (in the
line under the grid). Stations not checked say so; click them (Ctrl-click
for several) and press **Check selected**. The row at the top is what the
forcing would use for the variable: *Selected* (your choice, a colour per
station, paler where the year is not fully covered, grey where ERA5 fills),
or the *Automatic choice* when you have chosen none; the line below the
grid says it in words, e.g. *Selected for Snow: A 1990-2005, B 2006-2026;
ERA5-Land fills about 40 days*.

To use station A to 2005 and station B from 2006, say for Snow:

1. pick Snow at the top, click A's row (it is also selected in the station
   table); *From* and *To* show the years of the period it covers, here
   1990 and 2005; tick the variables it is for (Snow; for an hourly station,
   any of AirTemp, RelHum, WindSpd, Cloud) and press **Add**;
2. click B's row, set *From* to 2006 (B also covers 2005, but each year can
   come from one station only) and press **Add**.

The ranges are listed on the right (variable, station, years); **Remove
selected** deletes ranges, **Automatic for this variable** and **Automatic for
all** go back to the automatic choice. A range that overlaps one already
chosen for the variable is refused, with a message; gaps between ranges
are allowed and filled from ERA5. Clicking a spliced record adds each record
it joins for the days it supplies. The choice is the same as
`--station-for "Snow=A[1990-2005]+B[2006-2026]"` (shown above the station
table) and is used by **Build forcing**. The records of different stations
are not homogenised, so expect a possible step where one gives way to the
next (see [data.md](data.md#weather-stations-and-the-distance-limit)).

Either way the forcing file, its years and its latitude are passed to the
Model tab, and a found lake's HydroLAKES mean depth is shown there as a
guide for the mixing depth. The latitude of a CSV is known when its
`_metadata.json` is next to it; otherwise type it in the Model tab.

## Model

Runs CLIMo, as `climo run` does.

| Field | |
|---|---|
| Forcing file | CSV for one point, or `.nc` for a whole lake; built by CLIMo or [your own](workflow.md#your-own-forcing) |
| Start year, End year | years to simulate (spin-up years are added before the start) |
| Latitude | decimal degrees; not used for `.nc`, where each cell has its own |
| Mixing depth | m; for `.nc`, the depth of cells the per-cell table leaves empty |
| Per cell (netCDF) | a mixing-depth table (`cells.csv`); **Edit per cell...** opens it (see below) |
| Snow percentages | % of the snowfall that stays on the ice, comma-separated, e.g. `0, 50, 100` |
| Dry snow densities | kg m⁻³, comma-separated, e.g. `250, 330` |
| Break-up threshold | optional, m: also report the effective break-up, the day the ice thins below it ([outputs.md](outputs.md#effective-break-up---breakup-threshold)) |
| Wet snow density | kg m⁻³ |
| Spin-up years | copies of the start year run before it from 1 m of ice (default 1); use 3 or more for mixing depths of about 50 m or more ([how_it_runs.md](how_it_runs.md#spin-up-years)). A warning in the log says when the starting ice did not melt out during the spin-up |
| Workers | `.nc` runs: number of processes (empty: all cores) |
| Output prefix | results are written to `<prefix>_daily.csv` and `<prefix>_ice_dates.csv` (or `.nc`) |

**Model options.** Below the run settings, the *Model options* box has the
options of [options.md](options.md), all at their defaults, which reproduce
legacy CLIMo exactly:

| Control | Values | Parameters (enabled only when the chosen value uses them) |
|---|---|---|
| Fixes | box: off (legacy) or on: the corrections of the legacy surface-temperature solver; recommended for new work ([options.md](options.md#fixes)) | — |
| Thin-snow albedo | `legacy`, `exponential` (A1) | e-folding depth (m) |
| Ice-type albedo | `legacy`, `surface_threshold`, `thickness_weighted` (A2) | snow-ice albedo, cold and melting; snow ice at the top (m, `surface_threshold` only) |
| Snow melt-albedo transition | `step` (legacy), `linear`, `logistic` (A3, snow) | width (K below melting) |
| Ice melt-albedo transition | `step` (legacy), `linear`, `logistic` (A3, ice) | width (K below melting) |
| Snow density | `legacy`, `bulk_relaxation` (S1) | fresh-snow density (kg m⁻³), relaxation time (h), transition width (K) |

The list is built from `climo.model.FORMULATIONS`, so it always matches the
command line (`--fixes`, `--thin-snow-albedo`, ...). An empty parameter field
takes the default. The settings scroll when the window is short; **Run**
stays below them.

Every combination of snow percentage and dry snow density is run, with the
same model options. All fields, the model options included, are checked
before the run starts, and every problem found is listed. The log lists the
options in use, one line each, as `climo run` prints them, and the Results
tab shows them under the run's name; whole-lake results record them in the
netCDF attributes.

**Mixing depth per cell.** For whole-lake forcing, **Edit per cell...**
shows every lake cell with its latitude, longitude, lake fraction and mixing
depth. Double-click a row, or select rows, type a depth and press **Set
selected**; **Save** writes the table (by default the `cells.csv` written
next to the forcing) and fills the *Per cell* field. Empty cells take the
Mixing depth field. When a forcing with a `cells.csv` next to it is chosen,
the field is filled in automatically.

**Fill from bathymetry...** fills the table from lake depth data: GLOBathy or
your own GeoTIFF, with the cap, fraction and minimum you choose. The *From*
column says where each value came from. Cells you edited are kept unless you
answer Yes when asked, and nothing is written until **Save**. **Show map** draws
the depths in the table. GLOBathy's depths are modelled and perform poorly in
places, clearly so on Great Slave Lake: see
[grid.md](grid.md#filling-the-mixing-depths-from-lake-depth-data) for the
method and its weaknesses.

The run also writes the winters and summary tables
([outputs.md](outputs.md#winters-and-summary)), and for a grid with
`_flake.nc` next to the forcing, adds ERA5 FLake to the results.

## Results

Opens after a run, or with **Open results...** on a `<prefix>_daily.csv` or
`<prefix>_daily.nc` written earlier (the `_ice_dates` file is read with it).

- **Scenarios**: select one, or several (Ctrl- or Shift-click) to compare
  them on one figure; colour then shows the scenario and line style the
  variable.
- **Grid cell** (gridded results): one cell, or the mean over the cells
  with lake fraction ≥ 0.5 (every lake cell when none is that much lake).
- **Figure**: ice thickness and snow depth (with ERA5 FLake ice thickness,
  dotted, when the results have it); ice composition (snow ice and
  congelation ice); temperatures (air and surface, and the bottom of each
  layer); energy fluxes (radiation, turbulent and net); freeze-up, break-up
  (and the effective break-up, with a threshold) and ice duration, with ERA5
  FLake's as black squares when present. Gridded results hold only ice thickness, snow depth, ice
  composition, surface temperature and the ice dates, so the other panels
  are empty.
- **Observed snow on the ground (on land)**: adds a station's snow on the
  ground to *Ice thickness and snow depth*, as points, for one-point results.
  The stations of the `snow_depth.csv` next to the forcing or the results are
  listed, the one that gives `Snow` first; pick *none* to hide them, or **Open
  file...** for another. It is snow on land, not on the ice
  ([data.md](data.md#snow-on-the-ground-as-observed)).
- **Save figure...** writes PNG, PDF or SVG. The toolbar under the figure
  zooms and pans.

With the source code installed, the same figures can be made from Python,
without the window (the ready-made program does not include Python or the
test file used here):

```python
from climo import read_forcing, run_scenarios
from climo.plots import plot_ice_and_snow, plot_ice_dates

daily, ice_dates = run_scenarios(read_forcing("tests/data/forcing_YL_70.csv"), 62.4, 10.0, 2015, 2024,
                                 [0, 100], [330])
plot_ice_and_snow(daily).savefig("ice.png")
plot_ice_dates(ice_dates).savefig("ice_dates.png")
```

For gridded results, `climo.plots.grid_tables(daily, ice_dates, cell)` turns
the netCDF Datasets into the same tables for one cell, or the lake mean
(`cell=None`).

## Batch

Builds the forcing of many lakes, and if you ask runs the model on them, as
`climo batch` does ([workflow.md](workflow.md#many-lakes-or-points-climo-batch)).
The window stays usable during a batch, but as with every long task in this
window, no other can start until it ends.

**The table.** One row per site, with the columns of a
[sites CSV](workflow.md#the-sites-csv):

- **Open CSV...** shows a sites CSV. Other columns it has, such as a site's
  model options or your own notes, appear after the usual ones and are saved
  with it. Its problems, if it has any, are listed at once.
- **Add the Data tab's lake** adds the lake or point set on the Data tab: its
  name search and which match you picked, or its point or outline file, with
  the stations you chose (`stations` column), **Snow from** (`snow_method`), the
  whole-lake setting (`grid`) and your snowfall file. The site id comes from
  the lake's name; a number is added if the id is in use. The settings below
  apply to every site, so the line under the table says how they differ from the
  Data tab's, if they do; **Use the Data tab's settings** copies them.
- **Add row** and **Delete selected** do what they say, and **double-clicking a
  cell** edits it (Return or clicking elsewhere keeps the text, Escape does not).
  Editing a row clears its status. Empty cells take the settings below.
- **Save CSV...** writes the table as a sites CSV, which `climo batch` reads
  too. Saved to another folder, relative file paths become absolute, so they
  still lead to the same files. File paths are relative to the folder of the
  CSV opened or saved, or to the working folder before there is one.
- **Check sites** lists every problem with the table and the settings.

**The settings for every site**: start and end year (whole years); source;
**Snow from**, for sites with an empty `snow_method`; max station distance (empty
for the default, 5 km from the shore of a lake, 50 km from a point); snow conversion
density; the station networks; and the results folder. Closed sections say what
they hold: **More settings** (min coverage, candidates, the grid of grid sites,
ERA5 FLake for grid sites, off by default as it sends slow requests to the CDS, the
units of snowfall files, and rebuilding every site) and **Model run**, used only
with *Forcing and model* (a default mixing depth for sites with an empty
`mixing_depth`, snow percentages, dry and wet snow densities, break-up threshold,
spin-up years and workers). The model options are those of the Model tab, with a
site's own columns first; the section says which are in use.

**Running.** Choose **Forcing only** (no model, and no site needs a mixing depth)
or **Forcing and model**, and press **Run**. Everything is checked first, and every
problem is listed before anything is downloaded. Then each row says where its
site is, and turns yellow while it works, green when done and red if it failed:

| Status | |
|---|---|
| *waiting* | its turn has not come |
| *building* | its forcing is being built, or its settings compared with what was built before |
| *built*, *reused* | the forcing was built, or was there already with the same settings; *Forcing file or error* says where it is |
| *given* | the `forcing` column names an existing file; nothing is built |
| *running the model* | with Forcing and model |
| *done (forcing built)* | with Forcing and model: its results are in the tables of the results folder |
| *failed* | *Forcing file or error* has the error; select the row to read all of it under the table |

A site that fails does not stop the others. Fix the problem and press **Run**
again: what was built is reused, as the column then says. The table cannot be
edited during a run. To stop a batch, close the window; a site being built is
lost, the others are kept and reused the next time.

The results folder holds the same files as `climo batch`: a data folder per site,
`status.csv` and `batch.json`, and with Forcing and model also `daily.csv`,
`ice_dates.csv`, `winters.csv` and `summary.csv` ([workflow.md](workflow.md#folder-layout-and-outputs)).

**Load forcing into the Model tab** passes the forcing of the row selected to the
Model tab, with its years and latitude and the row's mixing depth, and sets the output
prefix to `climo` in the site's folder, so it can be run on its own with other
settings. It works for a site built earlier too, once its table is open again and
the results folder is the one it was built in. A site whose forcing was built but
whose model run failed can be loaded, to look into it there.
