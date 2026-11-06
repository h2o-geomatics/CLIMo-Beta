# Output columns (CLIMo Classic)

`daily` keeps the column names of the legacy CLIMo outputs so that results
can be compared with earlier studies, except for the layer depths. Values are
those at the end of each day (the last 6-hour time step). Batch runs
(`run_scenarios`, or several values on the command line) stack the runs and
add two leading columns to both tables.

| Column | Description | Units |
|---|---|---|
| `snow_percentage`, `rho_snow_cold` | scenario (batch runs only) | %, kg m⁻³ |
| `Date`, `AirTemp`, `Cloud`, `RelHum`, `WindSpd`, `Snow` | input forcing | as input |
| `FLWins` | downwelling long-wave radiation | W m⁻² |
| `FLWouts` | emitted long-wave radiation | W m⁻² |
| `FSWins` | downwelling short-wave radiation | W m⁻² |
| `FSWouts` | reflected short-wave radiation | W m⁻² |
| `FLATs` | latent heat flux (positive downward) | W m⁻² |
| `FSENSS` | sensible heat flux (positive downward) | W m⁻² |
| `FLUX` | net surface flux after surface melt | W m⁻² |
| `PenSWs` | short-wave penetrating the surface | W m⁻² |
| `SnowIce` | snow ice grown this season; zero once growth stops (see legacy notes) | m |
| `CongelationIce` | congelation (clear) ice grown this season; zero once growth stops | m |
| `IceThickness` | total ice thickness | m |
| `SnowDepth` | snow depth, at the snow density in use (dry or wet) | m |
| `L1Depth`–`L5Depth` | depth of the bottom of layers 1–5 below the surface (see below) | m |
| `CTair` | air temperature at the last time step | °C |
| `SurTemp` | surface temperature (water temperature when ice-free) | °C |
| `L1Temp`–`L5Temp` | temperature at the bottom of layers 1–5 | °C |

The snow and ice slab has 5 layers. When there is snow (more than 1 mm),
layer 1 is snow and layers 2–5 split the ice equally, so `L1Depth` is the snow
depth and `L1Temp` the snow/ice interface temperature. Without snow, all 5
layers are ice. `L5Depth` is the total snow + ice thickness and `L5Temp` the
ice-bottom temperature (the freezing point). During open water, the depths
keep their last ice-covered values. The legacy columns map as follows:
`L1SnowDepth` = `L1Depth`; with snow, legacy `L{k}Thickness` = `L{k}Depth` −
`L1Depth`; at 0 % snow, legacy `L{k}Thickness` = `L{k}Depth`.

`ice_dates` has one row per ice season (winter):

| Column | Description |
|---|---|
| `Season` | ice season, e.g. `2015-2016` (see below) |
| `Freeze-up` | day of year on which the season's main ice cover forms |
| `Break-up` | day of year on which the main ice cover disappears |
| `Ice-Duration` | days from freeze-up to break-up, both included (legacy definition) |
| `Freeze-up-Date`, `Break-up-Date` | the same events as dates (YYYY-MM-DD) |
| `First-Ice-Date` | first ice formation of the season |
| `Last-Ice-Date` | break-up of the season's last ice cover |
| `Effective-Break-up`, `Effective-Break-up-Date` | with a break-up threshold only: the day the ice thins below it (see below) |

**Season window.** The season starting in year Y runs from 1 August of Y to
31 July of Y + 1 for lakes in the Northern Hemisphere (latitude ≥ 0), and six
months later, from 1 February of Y to 31 January of Y + 1, in the Southern
Hemisphere. It is labelled `Y-Y+1` in both. Days of year are calendar days of
year of the event, so a freeze-up on 5 January 2016 is `Freeze-up` 5 in
season `2015-2016`; the date columns remove any doubt.

**Which events.** Ice is present when its mass exceeds 0.01 kg m⁻²; a
freeze-up is a day on which ice appears and a break-up a day on which it
disappears. Each period of continuous ice cover belongs to the season in
which it forms. The season's main ice cover is its longest one (an ice cover
still present at the end of the run counts up to the last day), so a brief
early freeze or a few days of refreezing after break-up do not replace the
freeze-up and break-up of the winter ice cover. When they happen, they show
in `First-Ice-Date` and `Last-Ice-Date`; otherwise these equal the main ice
cover's dates. In the legacy calendar-year table, the freeze-up was the last
ice formation of a year and the break-up the first ice loss; for a winter
with a single ice cover the two definitions give the same days (see
[legacy notes](legacy_notes.md)).

**Rows and missing values.** There is one row for every season that
overlaps the simulated years, so a run of 2015–2024 in the Northern
Hemisphere gives seasons `2014-2015` to `2024-2025`. The first season began
in the (last) spin-up year, a copy of the start year, so its freeze-up, first
ice and duration are missing, but its break-up is a result. Nothing dated
before the start year is reported, however many spin-up years are run. Values are also
missing for seasons without ice and for events after the end of the run
(usually the last season's break-up). An ice cover lasting more than a year
is reported in the season in which it formed; the following season has no
freeze-up.

A season for `SnowIce` and `CongelationIce` ends at a pause in growth that
includes at least one ice-free day; it is not tied to the ice-date season
window.

## Effective break-up (`--breakup-threshold`)

Lake ice usually breaks up, moves and is blown ashore before it has melted
completely, so the break-up seen from the shore, by cameras or by satellites
is often earlier than CLIMo's `Break-up`, the day the modelled ice is gone.
With a break-up threshold (`--breakup-threshold 0.1` on the command line,
`breakup_threshold=0.1` in `Parameters` or `run_scenarios`, or the Model tab
field), the ice-date table also gives:

| Column | Description |
|---|---|
| `Effective-Break-up` | day of year of the first day after the season's maximum ice thickness on which `IceThickness` is below the threshold (m) |
| `Effective-Break-up-Date` | the same day as a date |

Only days after the season's maximum count, so thin ice in early winter is
not mistaken for break-up. The effective break-up is never later than
`Break-up`. It is a diagnostic only: the model's physics and its other
outputs are unchanged, and it is off by default, so results without it are
those of CLIMo Classic. It corresponds to the thickness-threshold diagnostic
(D1) of the conservative update specification;
the porosity-based variant described there is not implemented. The
validation cases report it at 0.1 m ([validation.md](validation.md)).

## Winters and summary

Every run also writes two smaller tables (from Python:
`climo.summarise(daily, ice_dates, latitude)`):

`<out>_winters.csv` — one row per scenario and ice season simulated in full
(the first season, which began in the last spin-up year, and the last, unfinished
one are left out):

| Column | Description |
|---|---|
| `snow_percentage`, `rho_snow_cold` | scenario (`site` too in a batch) |
| `Season` | ice season, as in `ice_dates` |
| `max_ice_thickness_m`, `max_ice_date` | the season's maximum ice thickness and its date |
| `max_snow_depth_m` | maximum snow depth on the ice |
| `max_snow_ice_m`, `max_congelation_ice_m` | maximum snow ice and congelation ice grown in the season |

`<out>_summary.csv` — one row per scenario:

| Column | Description |
|---|---|
| `winters` | number of seasons in `winters.csv` |
| `mean_max_ice_thickness_m`, `min_max_ice_thickness_m`, `max_max_ice_thickness_m` | mean, smallest and largest of the seasons' maximum ice |
| `mean_max_snow_depth_m` | mean of the seasons' maximum snow depth |
| `seasons_with_freeze_up` | seasons with a freeze-up in the run |
| `mean_freeze_up_doy`, `mean_break_up_doy` | mean freeze-up and break-up, as day of year |
| `mean_effective_break_up_doy` | with a break-up threshold only |
| `mean_ice_duration_days` | mean ice duration |

Freeze-up and break-up are averaged as dates within their season, so a
freeze-up on day 364 one year and day 5 the next averages to day 2, not to
midsummer. For gridded runs both tables describe the lake mean (see
[grid.md](grid.md)); `climo batch` writes the same tables for all its sites
together.

## Gridded runs

The netCDF results of whole-lake runs are described in
[grid.md](grid.md#step-2-runs-climo-run-on-a-nc-forcing), including the
ERA5 FLake variables added for comparison.
