# Data module: forcing from weather stations and ERA5

`climo data` builds a CLIMo forcing file for a lake from the weather
stations near it (NOAA worldwide, ECCC in Canada), using ERA5 for whatever
the stations cannot provide. `climo lake` finds a lake by name, and
`climo stations` shows the stations near it before anything is built.

```bash
pip install -e ".[data]"
climo lake "Great Slave Lake"
climo stations --lake-name "Qinghai Lake" --start 2015-01-01 --end 2024-12-31 --out results/qinghai
climo data --lake-name "Qinghai Lake" --start 2015-01-01 --end 2024-12-31 --out-dir results/qinghai
climo data --lat 62.45 --lon -114.37 --start 2015-01-01 --end 2024-12-31 --out-dir results/yellowknife
climo data --lake my_lake.geojson --start 2015-01-01 --end 2024-12-31 --out-dir results/my_lake
```

This is step 1 of the [workflow](workflow.md); `climo run --forcing
results/qinghai ...` is step 2.

Station data and lake search need no account. **ERA5 needs a CDS key — set
it up once as below before the first run.** For forcing over a whole lake
(every ERA5 cell, as netCDF) see [grid.md](grid.md).

## Setting up ERA5 access (CDS key)

ERA5 is read from ECMWF's analysis-ready (ARCO) stores, which use the same
credentials as the Copernicus Climate Data Store (CDS) and its `cdsapi`
client. If you already download ERA5 with `cdsapi`, you are set up.

1. **Register** (free) at https://cds.climate.copernicus.eu and log in.
2. **Copy your personal access token** (API token) from your user profile
   page, https://cds.climate.copernicus.eu/profile.
3. **Accept the licences** of the two datasets used: on the Download tab of
   [ERA5 hourly data on single levels](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=download)
   and of [ERA5-Land hourly data](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-land?tab=download),
   accept the licence under "Terms of use". The CDS refuses data from a
   dataset whose licence you have not accepted.
4. **Create the file `.cdsapirc`** in your home folder:

   | System | File |
   |---|---|
   | Windows | `C:\Users\<your user name>\.cdsapirc` |
   | macOS | `/Users/<your user name>/.cdsapirc` |
   | Linux | `/home/<your user name>/.cdsapirc` |

   with these two lines (use your own token):

   ```
   url: https://cds.climate.copernicus.eu/api
   key: <your personal access token>
   ```

   On Windows, make sure the file is not saved as `.cdsapirc.txt` (in
   Notepad, choose "All files" when saving), or create it from the Command
   Prompt or Miniforge Prompt: `notepad %USERPROFILE%\.cdsapirc`.

**Keeping the file somewhere else.** Set the environment variable
`CDSAPI_RC` to the file's full path (the `cdsapi` client reads the same
variable), or pass it for one run:

```bash
climo data --cds-rc D:/keys/cdsapirc --lake-name "Baker Lake Nunavut" --start 2015-01-01 --end 2015-12-31
```

On shared or cloud machines you can instead put the token itself in the
`CDSAPI_KEY` environment variable. In Python, set
`os.environ["CDSAPI_RC"]` before calling `build_forcing`.

CLIMo looks for the key in this order: `CDSAPI_KEY`, then the file named by
`CDSAPI_RC`, then `.cdsapirc` in your home folder. If none is found, the
error message says which file it looked for. Keep the token private: do not
commit `.cdsapirc` to a repository.

## Finding a lake by name

```bash
climo lake "Great Slave Lake"
climo lake "Long Lake" --pick 2 --save long_lake.geojson
```

Names and outlines come from OpenStreetMap through its Nominatim search
service. When several lakes match, they are listed with their area,
location and region; choose one with `--pick N`, or add the region or
country to the name (e.g. "Baker Lake Nunavut"). The chosen lake is matched
to its HydroLAKES record (Messager et al., 2016), which gives its HydroLAKES
id, area, **mean depth** (a guide for CLIMo's mixing depth) and elevation.
HydroLAKES itself names only about 3,000 of its 1.4 million lakes, so it is
not used for the search. Its points file (79 MB) is downloaded on first use.

`climo data --lake-name NAME` does the search and builds the forcing in one
step; `--save` keeps the outline for later runs with `--lake FILE`.

For a large lake, CLIMo represents one location: by default a point inside
the lake, chosen by the outline. Give the location you want with
`--lat/--lon` together with `--lake-name` or `--lake` (for example Back Bay
on Great Slave Lake: `--lat 62.45 --lon -114.37`). For a whole lake, one
column per grid cell, see [grid.md](grid.md).

The search service allows about one request per second and limits heavy
use, so results are cached for 30 days and a repeated search does not
contact it again. If it reports being busy, wait a few minutes. Setting
`CLIMO_CONTACT` to your e-mail address includes it with requests, as the
service's usage policy suggests.

## Weather stations and the distance limit

Stations are searched in three networks:

| Network | Kind | Variables | Coverage |
|---|---|---|---|
| NOAA ISD-Lite (`isd`) | hourly | air temperature, dewpoint, wind speed, total cloud cover | about 35,000 stations worldwide |
| ECCC (`eccc`) | hourly | air temperature, dewpoint, wind speed (no cloud) | about 1,700 Canadian stations |
| ECCC (`eccc`) | daily | snowfall (or snow on the ground, with `--snow-method depth`) | about 8,000 Canadian stations |
| NOAA GHCN-Daily (`ghcn`) | daily | snowfall (or snow depth, with `--snow-method depth`) | worldwide, including many Canadian stations |

Many Canadian stations are in both ECCC and NOAA's archives, but ECCC has
every station, including many NOAA lacks (e.g. hourly records at Big Trout
Lake, Ontario, 1953–1992, where ISD has only 1973–1977), and is updated
daily.

**Why ECCC hourly and daily records are both read.** ECCC's daily records
have daily temperature, snowfall and snow on the ground, but no mean
humidity, mean wind or cloud; its hourly records have humidity (from the
dewpoint) and wind, but no snowfall. So temperature, humidity and wind come
from hourly records (as daily means), and snowfall from daily ones. A
station usually has both, under the same climate id; they are listed as an
`hourly` and a `snowfall` row.

**The limit.** `--max-station-distance` (km) is a hard limit: a station
farther than this is never used, whatever its record, and the variables it
would have given come from ERA5 instead. With a lake outline
(`--lake-name` or `--lake`) the distance is measured **from the shore**
(0 for a station on the lake or on an island in it) and the default is
**5 km**: stations by the water share the lake's weather, wherever they are
along a large lake. Without an outline it is measured from the point and
the default is **50 km** (as before, so point runs and the validation cases
are unchanged). Keep the limit small enough that a station shares the lake's
weather: less than 50 km from a point in mountains or along coasts. The
limit is drawn on the map and recorded in `metadata.json`. `--radius-km` is
the old name of the same option and still works.

**Every candidate is checked.** All stations within the limit whose record
could cover `--min-coverage` of the period are downloaded and their
coverage measured, nearest the lake point first. Whether a record could is
judged from the first and last dates in the station lists, so a station
that started or closed during the period is not downloaded when it cannot
reach the threshold. `--candidates N` checks only the nearest N of each kind
(the old behaviour, with N = 5); `all` (the default) checks every one.
Downloads are cached (see [Sources](#sources)), so checking again, or
building after `climo stations`, is quick.

**Stations that changed id.** ECCC gives a station a new climate id when its
instruments or operator change, e.g. Yellowknife A was 2204100 (daily
records 1942 to 16 January 2013) and is 2204101 since (from 15 January
2013); neither alone covers a period across 2013. Records of one network
and kind are **spliced** into one when they are clearly one site: the same
name (ignoring case and punctuation), within 2 km of each other, and
following each other in time (each starts and ends after the one before and
overlaps it by at most 31 days). Where they overlap, the newer id is used,
and the older fills its gaps. The spliced record is listed as a candidate
with the ids joined by `+`, oldest first (`2204100+2204101`), next to the
single records, and its coverage is that of the joined series. The rule is
deliberately strict: records of the same name running side by side for
years (e.g. a manned and an automatic station) or a renamed station
(Yellowknife Airport, 2204108) are not joined; join them yourself (below)
if you know they belong together. The metadata records, for each variable,
which ids gave which days.

**Seeing the stations first.** `climo stations` lists every station within
the limit that has records in the period, nearest the lake point first,
with the network, kind (hourly or snowfall), distance from the point and
from the shore, the dates its record spans and the share of days it covers
for each variable (or why it was not checked):

```bash
climo stations --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out results/prosperous
```

It writes `<out>_stations.csv`, `<out>_coverage_by_year.csv` and
`<out>_stations.png` (the stations, the limit and the lake) and downloads no
ERA5. `<out>_coverage_by_year.csv` has one row per checked station and
variable, and one column per year of the period: the fraction of that year's
days with data (for the first and last years, of their days in the period;
for a spliced record, of the spliced series). It shows which station covers
which years, e.g. that 2204100 ends early in 2013 and 2204101 starts then.
Whichever snow method is in use, a checked snowfall station's coverage by both
is listed (`coverage_snowfall`, `coverage_depth` in `<out>_stations.csv`), so a
station that reports only snow on the ground no longer looks empty
([why they differ](#how-a-station-is-judged-on-snow-on-the-ground)). With
`--snow-method depth`, `<out>_coverage_by_year.csv` also has a `Snow_season` row
per station, of the days from November to April.
The GUI's Data tab shows the same table with **Find stations**, and the
years in its **By year...** window ([gui.md](gui.md#data)).

**Choosing stations yourself.** By default CLIMo chooses for each variable
(see [Method](#method)). To use a particular station for some variables,
give `--station-for VARIABLES=ID`, once per station:

```bash
climo data --lake-name "Great Slave Lake" --start 2010-01-01 --end 2016-12-31 --out-dir results/great_slave \
           --station-for Snow=2204100+2204101 --station-for AirTemp,RelHum,WindSpd=2204100+2204101
```

Variables are `AirTemp`, `RelHum`, `WindSpd`, `Cloud` (hourly stations) and
`Snow` (snowfall stations), separated by commas. `+` joins several ids into
one record in the order given, the last being the newest (it wins where
they overlap). A station you choose is used **whatever its coverage**: its
coverage is still measured and reported, and ERA5 fills the days it misses
(listed in `Filled`). Each id must be within the limit. Variables you do not
name are chosen automatically. In a batch sites CSV the same choices go in
a `stations` column, separated by `;` (e.g.
`"Snow=2204100+2204101; AirTemp,RelHum,WindSpd=2204101"`, in quotes because
of the commas; a spreadsheet adds them); in the GUI, select rows in the
station table and pick the variables under **Use for...**, or use the
[lists in the panel above the table](gui.md#choosing-where-the-forcing-comes-from).
The choices are part of the recorded request, so changing them rebuilds the forcing.

**ERA5 for a variable.** `era5` in place of a station takes the variable from ERA5
(Snow: ERA5-Land) with no station, while the others go on as before:
`--station-for Cloud=era5`. It can't be joined with stations, since ERA5 already fills
their gaps. It contradicts `--source station` for temperature, humidity or wind, which
is refused. It is recorded as `"chosen": "by you"` under the variable in `metadata.json`.

#### Choosing from one place

Because each variable takes its own nearest good station, a forcing can mix places,
and which ones depends on the period. The automatic choice on Great Slave Lake
(lake point 61.89° N, 113.66° W) gives, for 2015 to 2024, wind from Inner Whalebacks
and the rest from Yellowknife; for 2015 to 2017, snow from Yellowknife-Henderson and the
rest from Yellowknife. **Use only this site** in the GUI gives one place (on the command
line, name the stations with `--station-for`). Forcing built each way for 2015 to 2017:

| Forcing | Temperature, humidity, wind, cloud | Snow | Mean air temperature (°C) | Mean wind (m/s) | Mean cloud | Snow, 3 years (m) |
|---|---|---|---|---|---|---|
| Automatic | Yellowknife (ISD) | Yellowknife-Henderson (GHCN) | −2.61 | 3.99 | 0.53 | 3.36 |
| Automatic, `Snow=era5` | Yellowknife (ISD) | ERA5-Land | −2.61 | 3.99 | 0.53 | 3.85 |
| Only Yellowknife | Yellowknife (ISD) | Yellowknife A (ECCC) | −2.61 | 3.99 | 0.53 | 4.79 |
| Only Hay River | Hay River Merlyn Carter (ISD) | Hay River A (GHCN) | −0.91 | 3.46 | 0.59 | 5.05 |

Two stations about 3 km apart differ here by over 40 % in snow, so it is worth looking
at where Snow comes from, with the [snow comparison](snow_comparison.md), before trusting
any one of these. This table shows that the choice matters, not which is right: none of
them was compared with ice on the lake.

**Stations for ranges of years.** When one station covers the early years
and another the later ones, give each its years in brackets after its id,
joined with `+`. For example, if station A has records to 2005 and station
B from 2005 to 2026:

```bash
climo data ... --start 1990-01-01 --end 2026-10-25 --station-for "Snow=A[1990-2005]+B[2006-2026]"
```

`[YYYY-YYYY]` is a range of whole years, inclusive (`[1990-2005]` is 1
January 1990 to 31 December 2005); `[YYYY-MM-DD:YYYY-MM-DD]` is a range of
days, e.g. `2204100[2008-01-01:2013-01-16]+2204101[2013-01-17:2016-12-31]`.
The brackets keep ranges apart from ids that contain `-`, such as ISD's
`719360-99999[2001-2010]`. Each station is used **only within its range**,
whatever it has outside it. Rules:

- every station of the choice has a range, or none has (without ranges, `+`
  splices records as above);
- the ranges must not overlap (`[1990-2005]` and `[2005-2026]` overlap in
  2005 and are refused): each day comes from one station;
- they may leave gaps: days in no range, and days a station misses within
  its range, come from ERA5 (ERA5-Land for snowfall) and are listed in
  `Filled`;
- all stations of a variable must be of its kind (hourly for `AirTemp`,
  `RelHum`, `WindSpd`, `Cloud`; snowfall for `Snow`), but may be of
  different networks.

The order you write them in does not matter; they are recorded oldest first,
whole years as `YYYY-YYYY`. The same syntax works in the sites CSV's
`stations` column, and the GUI builds it in its **By year...** window. The
report and `metadata.json` list, for each variable, every range: its station,
its years, the first and last days it supplied and how many, and the days
filled from ERA5.

Records of different stations are **not homogenised**: an instrument,
exposure or site difference between A and B shows up as a step on the day
one gives way to the other (e.g. a change in snowfall totals from 2006).
Look at the series around each join, and prefer one long record of one
station where there is one.

`--station ID ...` (the older way) still works: a named station is used for
every variable it covers on at least `--min-coverage` of the days, the rest
of its kind coming from ERA5 rather than another station. `--networks isd
ghcn` leaves ECCC out (or any other combination).

## Your own snowfall

If you have better snowfall for the lake than the stations or ERA5-Land
(e.g. a gauge on the lake), give it as a CSV with a `Date` column
(YYYY-MM-DD) and a `Snow` column:

```bash
climo data --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous \
           --snow-file my_snowfall.csv --snow-units cm
```

| `--snow-units` | Meaning |
|---|---|
| `m` (default) | depth of fresh snow, in metres (CLIMo's own unit) |
| `cm` | depth of fresh snow, in centimetres (as most station records) |
| `mm-swe` | water equivalent, in millimetres; converted to fresh-snow depth with `--snow-density`, as ERA5 is |
| `cm-depth`, `m-depth` | snow **on the ground**, in centimetres or metres; snowfall is its daily rise (see [Snowfall from snow depth](#snowfall-from-snow-depth)); empty days where the snow had gone count as 0, and the first day of the file and the day after each other gap come from ERA5-Land |

The file replaces the `Snow` column; days it does not cover (or leaves
empty) come from ERA5-Land and are listed in `Filled`. No snowfall station is
searched. Only the input changes: the model still builds and melts the snow
on the ice itself, from this snowfall and the snow percentage of each
scenario. For `--grid`, every lake cell gets the same series. The file is
checked before anything is downloaded, and editing it rebuilds the forcing
on the next `climo data`. In a batch CSV, give it in a `snow_file` column; in
the GUI, in the Data tab.

## Outputs

In the folder given by `--out-dir` (or named `<out>_forcing.csv` etc. with
`--out`):

- `forcing.csv` — daily forcing in CLIMo format (`Date, AirTemp, RelHum,
  WindSpd, Cloud, Snow`), plus `SnowSWE` (ERA5 snowfall water equivalent, m;
  empty when snowfall comes from a station or from a snow depth file) and `Filled`
  (station variables taken from ERA5 on that day; `Snow(gap)` for a short
  gap in station snow depth whose total is the station's, see
  [Snowfall from snow depth](#snowfall-from-snow-depth)). CLIMo ignores the
  extra columns.
- `metadata.json` — the lake (with HydroLAKES attributes when found by
  name), sources of each variable (station ids, with their ranges of years
  when you gave some, and the days each supplied),
  stations considered (with distances from the lake point and shore, and
  coverage), the ERA5 and ERA5-Land cells,
  citations, and under `request` every setting that determines the result.
- `map.png` — lake, stations considered and used, and the reanalysis cells.
- `snow_depth.csv` — [snow on the ground as observed](#snow-on-the-ground-as-observed)
  at the stations, if any reported it. Not part of the forcing.

Running `climo data` again with the same folder and settings reuses the
forcing instead of downloading it again; `--refresh` rebuilds it. See
[Reusing forcing built before](workflow.md#reusing-forcing-built-before).

From Python: `climo.data.build.build_forcing(lat, lon, start, end, ...)`.

## Sources

| Source | Variables | Access |
|---|---|---|
| NOAA ISD-Lite (Smith et al., 2011), hourly, ~35,000 stations worldwide | air temperature, dewpoint, wind speed, total cloud cover | AWS Open Data `noaa-isd-pds` |
| NOAA GHCN-Daily (Menne et al., 2012), daily | snowfall | AWS Open Data `noaa-ghcn-pds` |
| ECCC historical climate data, hourly and daily (Open Government Licence – Canada) | hourly: air temperature, dewpoint, wind speed; daily: snowfall | MSC GeoMet API, `api.weather.gc.ca` (`climate-hourly`, `climate-daily`) |
| ERA5 hourly single levels (Hersbach et al., 2020), 0.25° | 2 m temperature and dewpoint, 10 m wind, total cloud cover | ECMWF ARCO store `reanalysis_era5_single_levels/sfc/geoChunked.zarr` |
| ERA5-Land hourly (Muñoz-Sabater et al., 2021), 0.1° | snowfall | ECMWF ARCO store `reanalysis_era5_land/sfc-snow/geoChunked.zarr` |
| OpenStreetMap via Nominatim (© OpenStreetMap contributors, ODbL) | lake names and outlines | nominatim.openstreetmap.org |
| HydroLAKES v1.0 (Messager et al., 2016) | lake id, area, mean depth, elevation | data.hydrosheds.org |

Downloads are cached in `~/.cache/climo` (change it with the `CLIMO_CACHE`
environment variable).

**ECMWF's ARCO stores** (`arco.datastores.ecmwf.int`) are geo-chunked: each
chunk holds about 8 years of hourly values for a few grid cells, so a 10-year
point series of all variables reads in a few seconds. (Google's ARCO-ERA5
store, by contrast, holds one global field per hour per chunk and is suited
to maps rather than time series.) ECMWF notes that the structure of these
stores may change; the URLs are set in `climo/data/era5.py`.

**Snowfall from ERA5-Land.** The ERA5 single-level store has precipitation
but not snowfall, so snowfall comes from ERA5-Land. ERA5-Land covers land
and lakes but has no values over the sea; for a lake by the sea the nearest
cell with data is used (searched within 1° of the lake), since snow falls on
lake and shore alike. Its hourly values
are snowfall in the hour ending at the time stamp.

## Method

1. **Stations, per variable.** Every station within the distance limit
   whose record could cover `--min-coverage` (default 90 %) of the period
   is downloaded (spliced records of stations that changed id included).
   For each variable — temperature, humidity, wind and cloud from hourly
   stations (ISD and ECCC), snowfall from snowfall stations (GHCN-Daily and
   ECCC daily) — the station **nearest the lake point** whose daily series
   covers at least `--min-coverage` of the days for that variable is used.
   So temperature and wind may come from different stations, when the
   nearest station's wind record is patchy. The distance to the point ranks
   the stations, not the distance to the shore: on a large lake every
   station within the limit is near the water, and the one nearest the
   point CLIMo represents has the most similar weather. Stations less than
   1 km farther than the nearest good one count as the same site (usually
   one station in both ISD and ECCC, or a single and a spliced record); of
   those, the record covering the variable on the most days is used, and
   on a tie the one covering more variables (so ISD, which reports cloud,
   is used for its cloud). Stations you choose with `--station-for` are
   used instead, whatever their coverage; stations named with `--station`
   as described above.
2. **ERA5** at the grid cell nearest the lake (and ERA5-Land for snowfall)
   supplies every variable without a suitable station, and fills the days a
   chosen station is missing (listed in `Filled`).
3. **Daily values** are means over the local solar day (UTC shifted by
   longitude / 15, rounded to whole hours). A station day needs at least 4
   observations. ERA5 snowfall is summed over the day. With `--snow-method
   depth`, Snow is the rise of snow on the ground from one day to the next
   (see [Snowfall from snow depth](#snowfall-from-snow-depth)).
4. **Relative humidity** is computed from temperature and dewpoint with the
   Tetens formula and the coefficients of Buck (1981) over water (as in the
   ECMWF IFS), for both stations and ERA5.
5. **Cloud cover** from stations is the ISD total-coverage code in oktas
   divided by 8; codes for obscured sky and descriptive cover are treated as
   missing. Few stations report cloud, so it usually comes from ERA5.
6. **Distances** are measured from the shore when the outline is known,
   otherwise from the lake point; both are reported. ECCC wind speeds (km/h)
   are converted to m s⁻¹, and ECCC daily snowfall flagged as missing,
   accumulated over several days or of uncertain amount is not used.

With `--source era5` no stations are used; with `--source station` the run
fails unless stations within the limit meet the coverage threshold for
temperature, humidity and wind.

`metadata.json` records, for each variable taken from a station, the
station id (ids joined by `+` for a spliced record), its network, name and
coverage of the variable, whether it was chosen automatically, named or
chosen by you, the days filled from ERA5 and, under `records`, the first
and last day and the number of days each id supplied. The report printed by
`climo data` (and shown in the GUI) gives the same.

## Snowfall

CLIMo's `Snow` input is daily snowfall as a depth of fresh snow (m) — the
quantity weather stations report and that earlier CLIMo studies used. The
model adds `rho_snow_cold × Snow` to the snow mass on the ice.

ERA5-Land gives snowfall as water equivalent. It is converted to fresh-snow
depth with `--snow-density` (default 100 kg m⁻³, the conventional 10:1
ratio), so that station and reanalysis snowfall are the same quantity and
can fill each other's gaps. At Yellowknife (2015–2016), measured snowfall was
1.18 m per winter and ERA5-Land snowfall 0.96 m at 100 kg m⁻³; converting at
330 kg m⁻³ would give only 0.29 m. Rafat et al. (2025) used 150 kg m⁻³ for
this conversion.

Because the model then applies `rho_snow_cold`, the snow mass reaching the
ice is `rho_snow_cold / snow_density` times the snowfall water equivalent
(e.g. 3.3 at 330 and 100 kg m⁻³). This is the long-standing CLIMo
convention; the snow percentage scenarios account for it together with wind
redistribution.

To see how a station's snowfall and snow on the ground compare with ERA5 and ERA5-Land
before choosing, see [Comparing snow sources](snow_comparison.md).

### Snowfall from snow depth

```bash
climo data --lat 62.45 --lon -114.37 --start 2022-10-01 --end 2024-06-30 --out-dir results/yk_depth --snow-method depth
```

`--snow-method depth` (GUI: **Snow from** `depth`; sites CSV: a
`snow_method` column, or `climo batch --snow-method depth`) derives the daily
`Snow` from **snow on the ground** instead of measured snowfall: the rise of
the snow depth from one day to the next. It is for stations that report only
the depth (e.g. Yellowknife Airport, ECCC 2204108, since 2022; GHCN-Daily
CAN02300501 at Baker Lake, snow depth 2004–2026 but snowfall only
2005–2007), and for comparing the two. **Measured snowfall
(`--snow-method snowfall`) stays the default**, and forcing built before is
unchanged.

What is read: ECCC daily `SNOW_ON_GROUND` (cm; values flagged missing,
accumulated or uncertain are dropped, as for snowfall) and GHCN-Daily `SNWD`
(mm; values failing a quality check are dropped). A **trace** (ECCC flag `T`,
GHCN-Daily `M_FLAG` `T`; both report it as 0) counts as 0. GHCN-Daily
stations are listed by the years of their snow depth records, so stations
reporting only depth are candidates. The coverage of `Snow` (in the station
list, the coverage by year and the choice) is that of the depth-derived
series, after the snow-free days below, over the days
[from November to April](#how-a-station-is-judged-on-snow-on-the-ground).

**Days without a depth after the snow has gone count as 0.** Many stations
leave snow on the ground empty when there is none rather than reporting 0
(e.g. Yellowknife A, ECCC 2204101, from about May to mid-October, without a
missing flag), so on its own such a station covers only about half of a
year's days. A gap (one or more days without a depth) is set to 0 only if
the snow had already gone (`climo.data.snowfall.snow_free_as_zero`):

- **between two depths**: both are 0 or trace, and no day of the gap is in
  December, January or February. So a gap in June and July between two
  zeros is snow-free, but a gap in February or March between 30 and 32 cm
  stays missing, and so does a gap in April between 5 cm and 0 (the snow
  may have gone on any of its days) or one in October between 0 and 5 cm.
  The winter months are excluded so that a station closed over a winter (0
  before and after) is not taken as snow-free;
- **before the first or after the last depth** (a record that starts or
  ends within the period): only if that depth is 0 or trace and every day of
  the gap is in May to September.

This applies to ECCC and GHCN-Daily stations alike, judged over the whole
years read (ECCC) or the whole record (GHCN-Daily), and to your own snow
depth file (`cm-depth`, `m-depth`). It applies to missing-flagged days too.
At Yellowknife A (July 2014 to June 2016) it raises the coverage of `Snow`
from 51 % to 99 %; the depth-derived totals do not change (96 and 98 cm per
winter), since the days set to 0 are snow-free. Automatic stations that
report 0 (e.g. 2204108) are not affected.

How the depth becomes snowfall (`climo.data.snowfall.snowfall_from_depth`):

- **a rise** of at least **1 cm** is the day's snowfall;
- **a smaller rise, or a drop**, is no snowfall. Depth is reported in whole
  centimetres and automatic sensors are noisy, so rises below 1 cm are not
  counted; drops (melt, settling, drifting) are clipped to zero. Clipping
  matters: over a winter the drops add up to about as much as the rises;
- **a day without a depth, or whose day before has none** (after the
  snow-free days above), has no value: it is filled (listed in `Filled`)
  rather than taken as no snowfall. The day before the period is read, so
  the first day has a value.

**Filling a station's gaps** (`climo.data.snowfall.fill_depth_gaps`):

- **Short gaps between two depths** (at most 14 days without a depth,
  `GAP_DAYS`): the station's own total is kept. The snowfall over the gap's
  days and the day after (the days the gap leaves without a value) adds up
  to the depth after the gap minus the depth before it, or 0 if that is
  less than 1 cm (a drop included). The total is spread over those days in
  proportion to **ERA5-Land snowfall** on them, or put on the day after the
  gap if ERA5-Land has none. These days are listed as `Snow(gap)` in
  `Filled`.
- **Other days** (longer gaps, a gap at either end, or years of the period
  the chosen stations do not cover) take **ERA5-Land snowfall converted to
  fresh snow at `--snow-density`** (100 kg m⁻³ by default): exactly the
  values the default method fills with, with no factor fitted to the
  station. These days are listed as `Snow` in `Filled`.

Without a station for `Snow` at all, `Snow` is ERA5-Land snowfall, the same
series as with `--snow-method snowfall` (its `SnowSWE` included), and the
report says so.

**The days filled from ERA5-Land are on a different basis from the station's
days.** ERA5-Land snowfall at 100 kg m⁻³ stands for measured snowfall, not
for the rise of snow on the ground, which is net of settling and melt. At
the validation sites (GHCN-Daily snow depth of the station the validation
forcing takes snowfall from, against ERA5-Land snowfall at the cell of that
forcing, on the days the station has a value, per snow season July to June
with at least 180 such days) the station's rises are about 0.4–0.8 of
ERA5-Land snowfall, while measured snowfall at the same stations is about
0.6–1.3 of it:

| Site (station) | Seasons | Rises / ERA5-Land: median (range) | Measured / ERA5-Land: median (range) |
|---|---|---|---|
| Yellowknife (CAN02204100), 1980–2013 | 34 | 0.81 (0.51–1.13) | 1.28 (0.98–1.78) |
| Baker Lake (CAN02300500), 1980–2020 | 37 | 0.42 (0.25–1.04) | 0.62 (0.44–1.00) |
| Island Lake (CAN05061376), 1973–1998 | 27 | 0.58 (0.25–0.75) | 1.07 (0.76–1.40) |
| Gimli (CAN05031038), 1973–1991 | 19 | 0.61 (0.41–0.90) | 1.16 (0.88–1.60) |
| Big Trout Lake (CAN06010738), 1973–1992 | 20 | 0.63 (0.43–0.99) | 0.93 (0.74–1.20) |
| Lac des Sables (CAN07036762), 1976–1992 | 16 | 0.64 (0.51–0.77) | 1.15 (0.85–1.28) |

So a day filled from ERA5-Land typically carries about 1.2–2.4 times the
snow the station's rises would have shown (most at Baker Lake, least at
Yellowknife). That matters little when the station has few long gaps, which
is the usual case with the snow-free days counted as 0 (at Yellowknife A,
July 2014 to June 2016, only 3 short gaps, 6 days, are left to fill); for a
period with long gaps, or years without the station, prefer a station with
more depth records, or measured snowfall.

**Comparison by season, for information.** For each snow season (July to
June) with at least 180 days with a station value (`MIN_OVERLAP_DAYS`) and
5 cm of ERA5-Land snowfall on them (`MIN_ERA5_TOTAL`), the station's rises
and ERA5-Land snowfall on those days and their ratio are given in the report
and in `metadata.json` under `snow.gap_filling.seasons`
(`climo.data.snowfall.compare_seasons`). It changes nothing: it shows how far
the filled days are from the station's basis.

#### How a station is judged on snow on the ground

With `--snow-method depth`, a station's coverage of `Snow` is the share of the
days **from November to April** with a value (`SNOW_SEASON_MONTHS`,
`climo.data.search.coverage`), not the share of all days. Only this choice of
station changes: what the forcing holds on each day, and how gaps are filled,
do not.

**Why.** Observers report snow on the ground while there is snow, and many
leave the rest of the year empty. The rule above sets a summer gap to 0 only
if the readings on either side are 0, because otherwise the day the snow went
is unknown. A station whose last spring reading is above 0, as at Hay River
Climate (ECCC 2202402) in 7 of the 10 years 2015–2024, keeps its whole summer
gap. Counted over
all days it then covers about 50 % even with readings on 91–100 % of the days
from November to March, and it can never reach the default minimum coverage
of 90 %.

**Why November to April.** Candidate windows, on 31 ECCC stations with records
over 2015–2024 (up to eight from each band of latitude from 48° N to 82.5° N;
the depth-derived series, after the snow-free days):

| Days judged | Stations reaching 90 % |
|---|---|
| All days (before this change) | 2 |
| October to May | 4 |
| November to April | 7 |

October and May still count against a station, since the empty days before
the first snow and after the last are mostly not gaps in the record. The five
stations gained are Peawanuck, La Ronge, Uranium City, Iqaluit Climate and
Inuvik Climate. Most stations still fall short (24 of the 31): many automatic
stations have real gaps in winter, so choose those yourself if you want them.

**What it leaves out.** Snow also falls outside the window, and those days
come from the station only if it reported a value, otherwise from ERA5-Land
snowfall (listed in `Filled`). The share of the daily rises that falls from
November to April, median by latitude:

| Latitude | Stations | Rises from November to April |
|---|---|---|
| 48–55° N | 7 | 98 % |
| 55–60° N | 5 | 88 % |
| 60–65° N | 6 | 84 % |
| 65–70° N | 5 | 84 % |
| Above 70° N | 8 | 71 % |

Measured snowfall tells the same story where it is available: 92–100 % at two
stations south of 55° N, 86 % at Fort Smith A and 77 % at Rankin Inlet A. So a
station that passes can still miss autumn snow, which matters most north of
70° N (56 % at Alert). The months are for the Northern Hemisphere, as are the
winter months of the rule above.

**Check against measured snowfall.** Hay River Climate reports only snow on the
ground (95 % of the days from November to April over 2015–2024; 50 % of all
days), and Hay River A, less than 1 km away, also measures snowfall. On the
days from November to April when both have a value:

| Season | Hay River Climate rises (cm) | Hay River A measured (cm) | Ratio | Days with snowfall at Hay River A |
|---|---|---|---|---|
| 2015–16 | 91 | 139 | 0.65 | 62 |
| 2016–17 | 110 | 212 | 0.52 | 70 |
| 2017–18 | 80 | 128 | 0.63 | 69 |
| 2018–19 | 55 | 93 | 0.59 | 52 |
| 2019–20 | 128 | 97 | 1.32 | 63 |
| 2020–21 | 88 | 76 | 1.16 | 51 |
| 2021–22 | 105 | 111 | 0.95 | 65 |
| 2022–23 | 74 | 24 | 3.08 | 24 |
| 2023–24 | 51 | 32 | 1.59 | 12 |

The first four seasons agree with the 0.55–0.75 below. The later ones do not,
and the measured values look like the reason: rises larger than measured
snowfall are not expected, and Hay River A has far fewer days with snowfall in
the last two seasons, while it still reports a value on most days. The cause is
not known, and this comparison cannot separate the two records from 2019–20 on.
The rises at the two stations agree better: Hay River Climate's are 1.14 times
Hay River A's overall, on the days all three series have a value.

At Great Slave Lake from its default point, the automatic choice is Yellowknife
A with either method, because it is nearer. Fort Reliance (ECCC 2201903) reaches
only 79 %. At Hay River itself, for October 2020 to June 2022, Hay River Climate
is now chosen automatically, at 99 % of the days from November to April.

#### Snow on the ground as observed

Each point forcing with stations also writes `snow_depth.csv`
(`<prefix>_snow_depth.csv`), **whatever the snow method**: one column per
station (in m, by date, the station that gives `Snow` first, then the nearest).

- It holds the values **as reported**, read by
  `climo.data.search.observed_depths`. No snow-free days are filled in, so
  empty means the station reported nothing. Values flagged missing, accumulated
  or uncertain (or failing a GHCN-Daily quality check) are left out, and a trace
  is 0, as for the forcing.
- Every single record of a snowfall station within the distance limit is read,
  or the nearest N with `--candidates`, and those that reported nothing are left
  out. There is no file with `--source era5`, with your own snowfall file (no
  snowfall station is searched), or for a grid.
- A station that cannot be read is left out with a message: the observations
  are not part of the forcing and never stop a build.
- `metadata.json` describes it under `snow.observed_depth`: each station's name,
  network, distances, days reported and whether it gives `Snow`. A forcing built
  by an earlier version gets the file when it is rebuilt (a new CLIMo version
  does this).

**This is snow on land, not on the ice.** Wind strips snow from lake ice, and
the CLIMo snow percentage stands for that, so it is a guide to the timing and
size of the snowpack, not a direct measure of `SnowDepth`. The Results tab can
draw it on the *Ice thickness and snow depth* figure ([gui.md](gui.md#results)),
and `climo.plots.add_observed_depth` does the same in Python.

#### Reporting, and why measured snowfall is the default

The number of short gaps and the days filled each way are in the report and
under `snow.gap_filling` too. The constants are in `climo/data/snowfall.py`.
The report and `metadata.json` (`settings.snow_method`, `snow.method`)
record the method, which is part of the request: switching it rebuilds the
forcing. Measured snowfall (`--snow-method snowfall`) is filled from
ERA5-Land snowfall as before, and so are the gaps of your own snowfall file.

Why measured snowfall is the default: the rise of the snow depth is snowfall
**net of the same day's settling, melt and drifting**, of snow on land. Fresh
snow settles within hours, a day with snowfall and melt can show no rise,
and wind moves snow between days. At Yellowknife stations the sum of the
daily rises is about **0.55–0.75 of measured snowfall**: at Yellowknife
airport about 94 cm against 148 cm per winter in the GHCN-Daily record, and
at Yellowknife-Henderson (GHCN-Daily CAN02204110, which reports both) 89 and
98 cm against 124 and 128 cm in the winters 2014–15 and 2015–16. The model
treats `Snow` as fresh snow and adds `rho_snow_cold × snow percentage ×
Snow` to the snow on the ice, with no compaction, so depth-derived snowfall
puts a quarter to a third less snow on the ice for the same snow percentage
(at Yellowknife-Henderson in 2015–16, with 100 % and a 5 m mixing depth,
1.31 m of ice at most instead of 1.37 m, and ice-off 2 days earlier). The
snow percentage scenarios, which stand for the share of snowfall that stays
on the ice (wind redistribution), were set up for measured snowfall;
with depth-derived snowfall a higher percentage gives the same snow load.
Snow on the ground is also measured on land, in a sheltered site, while
snow on lake ice is usually thinner and wind-packed.

`--snow-method depth` is for point forcing; whole-lake grids
(`--grid`) use ERA5-Land snowfall.

## Checks

At Yellowknife (2015–2016), ERA5 and the airport station agree closely for
daily air temperature (bias −0.07 °C, r = 0.998), relative humidity (−1 %,
r = 0.96) and wind speed (−0.6 m s⁻¹, r = 0.92); ERA5 cloud cover is higher
(+0.13, r = 0.69). CLIMo driven by either forcing gives maximum ice of
0.94–1.02 m in 2016 and break-up within 2 days.

## Limitations

- A point run uses one ERA5 cell (the nearest) for what stations do not
  give; for every cell of a lake see [grid.md](grid.md).
- None of ECMWF's ARCO stores has the lake variables (ERA5 FLake ice), so
  `climo data --grid --flake` and the validation cases fetch
  `lake_ice_depth` from the CDS API instead, which is slow (see
  [grid.md](grid.md#era5-flake-for-comparison)).
- ECCC hourly records have no cloud cover; near stations that report only to
  ECCC, cloud comes from ERA5.
- Splicing of stations that changed id is deliberately strict: when a third
  record of the same name runs alongside (e.g. Fort Resolution A and Hay
  River A, hourly, around 2014–2016), none of them is joined. Join the ones
  you want with `--station-for ...=OLD+NEW`.
- Each variable is chosen on its own, so cloud may come from a station
  farther away than the one giving temperature (any station within the
  limit whose cloud record is complete enough), and wind from another
  station than temperature when the nearest station's wind record is
  patchy. Choose stations yourself to keep them together.
- Lake search depends on the OpenStreetMap name and tagging; small or
  unnamed lakes may not be found (use coordinates or an outline file).
- Snowfall from snow depth is net of settling and melt and is smaller than
  measured snowfall. With `--snow-method depth`, days without a station
  value (beyond short gaps) take ERA5-Land snowfall at 100 kg m⁻³, which is
  about 1.2–2.4 times the station rises at the validation sites; the report
  compares the two by season (see
  [Snowfall from snow depth](#snowfall-from-snow-depth)).

## References

- Buck, A. L. (1981). New equations for computing vapor pressure and enhancement factor. *Journal of Applied Meteorology*, 20, 1527–1532.
- ECMWF (2016). IFS Documentation CY41R2, Part IV: Physical Processes, Eq. 7.5.
- Environment and Climate Change Canada. Historical climate data (hourly and daily), MSC GeoMet API. https://api.weather.gc.ca. Contains information licensed under the Open Government Licence – Canada.
- Hersbach, H., et al. (2020). The ERA5 global reanalysis. *Quarterly Journal of the Royal Meteorological Society*, 146, 1999–2049. https://doi.org/10.1002/qj.3803
- Messager, M. L., Lehner, B., Grill, G., Nedeva, I., and Schmitt, O. (2016). Estimating the volume and age of water stored in global lakes using a geo-statistical approach. *Nature Communications*, 7, 13603. https://doi.org/10.1038/ncomms13603
- Muñoz-Sabater, J., et al. (2021). ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. *Earth System Science Data*, 13, 4349–4383. https://doi.org/10.5194/essd-13-4349-2021
- Menne, M. J., Durre, I., Vose, R. S., Gleason, B. E., and Houston, T. G. (2012). An overview of the Global Historical Climatology Network-Daily database. *Journal of Atmospheric and Oceanic Technology*, 29, 897–910. https://doi.org/10.1175/JTECH-D-11-00103.1
- Rafat, A., Cheng, B., and Kheyrollah Pour, H. (2025). *Water Resources Research*, 61(5), e2024WR038615. https://doi.org/10.1029/2024WR038615
- Smith, A., Lott, N., and Vose, R. (2011). The Integrated Surface Database: recent developments and partnerships. *Bulletin of the American Meteorological Society*, 92, 704–708. https://doi.org/10.1175/2011BAMS3015.1

ERA5 and ERA5-Land data: generated using Copernicus Climate Change Service information.
Neither the European Commission nor ECMWF is responsible for any use that may
be made of the Copernicus information or data it contains.
