# Gridded runs over a whole lake (netCDF)

A single CLIMo run represents one location. To cover a whole lake, CLIMo can
run every reanalysis grid cell that touches the lake outline, each as its own
column with its own forcing, latitude and mixing depth.

```bash
# 1. forcing for every ERA5 cell touching the lake (needs a CDS key, see data.md)
climo data --lake-name "Qinghai Lake" --grid --start 2015-01-01 --end 2024-12-31 --out-dir results/qinghai

# 2. run every cell for every snow scenario
climo run --forcing results/qinghai --start 2015 --end 2024 --mixing-depth 16.8 \
          --snow-percentage 0 50 100 --rho-snow-cold 250 330
```

`--lake my_lake.geojson` can be used instead of `--lake-name`, and `--out
results/qinghai` (a file prefix) instead of `--out-dir`. Use whole years for
the forcing period: runs simulate calendar years and use the first year as
spin-up (`--spinup-years` times), as for a single point.

## Step 1: forcing (`climo data --grid`)

### Two grids

| `--grid-source` | Cells | Temperature, humidity, wind | Cloud | Snowfall |
|---|---|---|---|---|
| `era5` (default) | ERA5, 0.25° (about 28 km) | ERA5 single levels at the cell | ERA5 | ERA5-Land, nearest cell with data |
| `era5-land` | ERA5-Land, 0.1° (about 11 km) | ERA5-Land at the cell; ERA5 for cells over the sea | ERA5 (nearest 0.25° cell) | ERA5-Land at the cell |

ERA5-Land resolves smaller lakes, bays and shores, and its 2 m temperature
over a lake comes from its own lake model. It covers land and lakes, even the
Great Lakes, but has no values over the sea; cells it leaves empty (lakes by
the coast) take temperature, humidity and wind from the nearest ERA5 cell,
and the variable `atmosphere_source` records which cells did. ERA5-Land has
no cloud cover, so cloud comes from ERA5 on both grids.

```bash
climo data --lake-name "Prosperous Lake" --grid --grid-source era5-land --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
```

Weather stations are not used for gridded forcing. Your own snowfall
(`--snow-file`, see [data.md](data.md#your-own-snowfall)) is given to every
lake cell.

### Files

In the data folder (or named `<out>_forcing.nc`, ... with `--out`):

- `forcing.nc` — the forcing on the grid around the lake:

  | Variable | Dimensions | Description |
  |---|---|---|
  | `AirTemp`, `RelHum`, `WindSpd`, `Cloud` | time, latitude, longitude | daily means at the cell (°C, %, m s⁻¹, 0–1) |
  | `Snow`, `SnowSWE` | time, latitude, longitude | daily snowfall as fresh-snow depth and water equivalent (m) |
  | `lake_fraction` | latitude, longitude | fraction of the cell covered by the lake; 0 for cells not touching it |
  | `snow_cell_latitude`, `snow_cell_longitude` | latitude, longitude | the ERA5-Land cell giving each cell's snowfall |
  | `atmosphere_source` | latitude, longitude | `era5-land` grids only: 0 from ERA5-Land, 1 from ERA5 (over the sea) |

  Cells not touching the lake are empty (NaN). The attributes record the
  grid (`grid_source`, `resolution_deg`), the sources, the day definition
  (one local solar day for the whole lake), the snow conversion density,
  citations, and the lake's name and HydroLAKES attributes when it was found
  by name.
- `cells.png` — a map of the cells, shaded by the fraction covered by the lake.
- `cells.csv` — the mixing depth of each lake cell, for you to edit or to
  [fill from lake depth data](#filling-the-mixing-depths-from-lake-depth-data) (below).
- `flake.nc` — with `--flake` only: ERA5 FLake lake ice (below).
- `metadata.json` — the settings, for reuse (see [workflow.md](workflow.md)).

### Mixing depth per cell

The mixing depth sets how much heat the open lake stores before it freezes,
so deep basins freeze later than shallow bays. `cells.csv` lists every lake
cell:

```
latitude,longitude,lake_fraction,mixing_depth_m
62.6,-114.2,0.4894,24.4
62.6,-114.1,0.0485,24.4
...
```

It starts with `--mixing-depth` given to `climo data`, or else the
HydroLAKES mean depth of the lake, or else empty. **Fill it from lake depth
data** ([below](#filling-the-mixing-depths-from-lake-depth-data)), **or edit
the `mixing_depth_m` column** (in a spreadsheet or text editor), e.g. from a
bathymetry map, and give the file to the run:

```bash
climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth-file results/prosperous/cells.csv --mixing-depth 10
```

Cells the file leaves empty, or does not list, take `--mixing-depth`; without
it, every lake cell needs a value. Rows must be lake cells of the forcing
(matched to the nearest cell centre). `climo data` never overwrites an
existing `cells.csv`, so your edits are kept when the forcing is rebuilt or
reused. In the GUI, **Edit per cell...** on the Model tab opens the same
table.

### Filling the mixing depths from lake depth data

For a lake with hundreds of cells, a depth per cell cannot be typed by hand.
`climo depths` fills `cells.csv` from gridded lake depth, and the table stays
yours to edit:

```bash
climo depths --forcing results/great_bear
climo run --forcing results/great_bear --start 2015 --end 2016 --mixing-depth-file results/great_bear/cells.csv
```

`climo data --grid --fill-depths` does it when the forcing is built, and
`climo batch --fill-depths` (or a `fill_depths` column, `yes` or `no`) does it
for grid sites and runs them with the table; without it, a grid site still
runs with its one `mixing_depth`. In the GUI, **Edit per cell...** has **Fill
from bathymetry...** and **Show map**; nothing is written until **Save**.

> **GLOBathy performs poorly as a map of lake depth.** It is modelled from the
> distance to the shore, not surveyed. It overstates the mean depth of Great
> Bear Lake by half and of Great Slave Lake about threefold, and before the cap
> its depth per cell is worse than the lake's single mean depth on three of the
> seven lakes tested. After the 40 m cap it beats one value on five of the seven
> and loses on two. On Great Slave Lake it puts the deepest water in the wrong
> basin and makes the west about 13 m too deep, which delays modelled freeze-up
> there by about 18 days. What it mostly gets right is the order from shallow to
> deep, and that is [what the ice depends on most](#what-the-pattern-is-worth).
> Use it as a starting point, and replace it with real bathymetry wherever you
> have some. See [where it is weak](#where-it-is-weak).

| Option | Default | Meaning |
|---|---|---|
| `--depth-cap M` | 40 | mixing depth where the lake is deeper |
| `--depth-fraction F` | 1 | mixing depth as a fraction of lake depth, below the cap |
| `--depth-minimum M` | 2 | smallest value of a cell |
| `--depth-raster FILE` | GLOBathy | your own GeoTIFF of lake depth instead (positive, in m; anything else counts as not lake). Only pixels on the lake outline are used |
| `--hylak-id N`, `--lake FILE` | from the data folder | which lake, if the folder does not say (`climo depths` only) |
| `--default M` | none | value of cells nothing else gives one to (`climo depths` only) |
| `--replace-edits` | keep them | replace cells edited by hand too (`climo depths` only); the previous table is kept as `cells.backup-<time>.csv` |
| `--dry-run` | | say what would happen, write nothing (`climo depths` only) |

Files written next to the forcing: `cells.csv` (the table, with new columns),
`cells_fill.json` (the settings, the rasters, the citation) and
`cells_depth.png` (a map; **Show map** in the GUI draws the same one).

| Column of `cells.csv` | Meaning |
|---|---|
| `mixing_depth_m` | the value runs use |
| `depth_source` | where it came from: `globathy:<HydroLAKES id>`, `raster:<file>`, `nearest_cell`, `hydrolakes_mean`, `default`, `given`, `manual`, or empty |
| `lake_depth_m` | mean lake depth of the cell's lake pixels, before the cap; empty without depth data |
| `filled_depth_m` | the value the fill or `climo data` wrote |

**Your edits are kept.** A cell whose `mixing_depth_m` differs from
`filled_depth_m` was edited, whatever edited it, and a new fill keeps it and
marks it `manual`. Tables from before this column existed cannot show where
their values came from, so their values count as edited unless they are the
HydroLAKES mean depth that `climo data` wrote. Runs read the table as before;
they ignore the new columns.

Downloads are cached in `~/.cache/climo/globathy/`: the first use reads the
list of GLOBathy's rasters (197 MB, kept as a 10 MB index), then each lake's
raster once (Great Bear 311 MB, most lakes far smaller; 33 s for both, here).
No CDS or ARCO request is made. It needs `rasterio`, part of the `data` extra.

#### What is done, and why

1. **Depth data: GLOBathy** (Khazaei et al., 2022; data CC0, paper CC BY 4.0):
   a 1 arc-second raster for each of the 1.4 million HydroLAKES waterbodies,
   used as published. Rasters are read out of its 16.7 GB archive by HTTP range
   requests. Each raster holds one waterbody, so a pond sharing a cell with the lake is not mixed in.
2. **From depth to mixing depth**, per lake pixel: `H = min(fraction × depth, cap)`.
3. **To one value per cell**: the mean of `H` over the *lake pixels in the
   cell*, so a cell partly on land gets the depth of its water, and
   `lake_fraction` does not weight it.
4. **Cells without depth data** take the nearest filled cell (`nearest_cell`).
   With no raster at all, every cell takes `min(fraction × HydroLAKES mean
   depth, cap)` (`hydrolakes_mean`; Messager et al., 2016), then `--default`
   (`default`), then stays empty, and the run names the cells without a value.
   On Great Bear, 2 of 829 cells at 0.1° take the nearest cell, each with a lake
   fraction of at most 0.0009.

**Not from the literature.** These are choices made for this tool, and no
published study we found supports them. Change them or ignore the tool's
values if you disagree:

- *The cap comes before the mean.* A column stores heat per unit area, so a cell
  half 5 m bay and half 200 m basin stores what a 22.5 m column stores, not 40 m.
  On Great Bear (0.25°) cells average 26.3 m this way and 29.9 m by capping each
  cell's mean depth.
- *The 2 m minimum.* Shore cells otherwise reach 0.3 m from a few pixels at the
  water's edge. Brown and Duguay (2011, Table 3) went as low as 2 m (Farnworth
  Lake). Set `--depth-minimum 0` to turn it off.
- *The nearest-cell fallback*, and *which waterbodies count as the lake*: its own,
  and others with their outlet inside the outline and at least half of their pixels
  too, because HydroLAKES can split a lake. A neighbour draining into the lake has
  its outlet on the shore too, and fails the second test (Hylak 330 at Great Slave
  Lake does).

We tried scaling GLOBathy to the HydroLAKES mean depth, which lowered the errors
below on six of the seven lakes. It is not in the literature, so CLIMo does not do
it, and the depths stay as GLOBathy published them.

#### Lake depth is not mixing depth

CLIMo's mixed layer is a slab of fixed depth. When there is no ice its
temperature follows the surface energy budget, so it stands for the heat the
lake stores and must lose before freezing (Brown and Duguay, 2011, p. 871;
Duguay et al., 2003, which we read only through those citing it). That
sets the relation:

- **Shallow water**: lakes are usually well mixed top to bottom in the
  ice-free period, so lake depth is a good stand-in (Brown and Duguay, 2011).
  Robinson (2024, p. 113 of the thesis) uses mean lake depth as the mixing
  depth for this reason: for ice-on, the autumn turnover is what matters. Hence
  `fraction = 1`.
- **Deep water**: the deep water does not take part in the seasonal exchange,
  so lake depth stops being the right number. Published CLIMo runs on these very
  lakes used 10 m at Back Bay and Hay River and 40 m at Main Basin-GSL and
  Deline-GBL (Kheyrollah Pour et al., 2012, Section 3.3, following Ménard et al.,
  2002). Brown and Duguay (2011) used 30 m for large deep lakes. FLake sets a
  false bottom at 50 m for deep lakes, on the argument that abyssal water changes
  little (Mironov et al., 2010, p. 221), and 40–60 m is typical (Kheyrollah Pour
  et al., 2012, Section 3.4). Hence `cap = 40`.

**How firm this is.** The 40 m is a modelling convention carried from paper to
paper, not a measurement: we found no measured autumn mixing depth for these
lakes. Those studies gave one value to a site; applying the cap cell by cell over
a whole lake is our use of it. Kheyrollah Pour et al. also note that
one-dimensional models miss the mixing mechanisms of deep, large lakes, and that
CLIMo's fixed layer holds more heat than FLake's, giving longer autumn cooling.
Formulas for the thermocline from fetch (e.g. Hanna, 1990) describe summer
stratification, not the autumn heat store, so they are not used; we did not read
them in the original. We could not open Ménard et al. (2002), Duguay et al. (2003)
or Carmack et al. (2021, on deep convection in Great Slave Lake): the publishers
refuse automated access.

#### What validation showed

Run with `python validation/depth_fill.py`; the numbers are in
`validation/experiments/depth_fill/`. Every method is compared on the same cells,
at 0.25°. *One value* is the HydroLAKES mean depth, capped in the capped columns.
The references: NOAA NCEI soundings for the Great Lakes, which are measured; and
for the northern lakes, chart rasters (Kheyrollah Pour and Attiah, 2022, digitised
from Johnson's and Schertzer et al.'s figures), a weaker test: about 1 km, in depth
classes (we take the middle of each class on Great Bear), and on Great Slave only
west of 112.74°W, without the deep East Arm. The chart rasters' licence (CC BY-NC-SA
4.0) is why they are used to validate only, and are not offered as a source.

| Lake (reference) | Cells | Mixing depth with the 40 m cap: RMSE, one value (m) | RMSE, GLOBathy (m) | r | Bias (m) | Lake depth before the cap: RMSE, one value (m) | RMSE, GLOBathy (m) |
|---|---|---|---|---|---|---|---|
| Superior (soundings) | 196 | **8.4** | 8.9 | 0.79 | −5.5 | 71 | **68** |
| Michigan (soundings) | 132 | 13.9 | **6.1** | 0.88 | −2.3 | 57 | **46** |
| Ontario (soundings) | 49 | 13.0 | **3.1** | 0.96 | −0.8 | 51 | **24** |
| Huron (soundings) | 148 | 17.7 | **9.4** | 0.83 | −5.2 | 40 | **32** |
| Erie (soundings) | 61 | **8.3** | 9.1 | 0.61 | +0.1 | **8.9** | 11.5 |
| Great Bear (chart) | 144 | 16.5 | **6.9** | 0.81 | +2.9 | **42** | 58 |
| Great Slave, west (chart) | 67 | 23.5 | **16.5** | 0.62 | +13.4 | **40** | 173 |

Results at 0.1° are alike. Lake means (m):

| Lake | Published | HydroLAKES | Reference raster | GLOBathy |
|---|---|---|---|---|
| Great Bear | 71.7 | 72.2 | 69.0 | 108.6 |
| Great Slave | 41 | 59.1 | 28.1 (west only) | 133.3 |
| Superior | | 146.7 | 148.3 | 109.0 |
| Michigan | | 84.2 | 84.3 | 82.6 |
| Ontario | | 84.8 | 88.5 | 87.9 |
| Huron | | 59.8 | 58.9 | 51.7 |
| Erie | | 19.4 | 17.6 | 20.8 |

**The real Great Bear run**, through the window (ERA5, 0.25°, 2015–2016): 174
cells, filled in 33 s with the downloads and about 5 s after, from 2 to 40 m with a
mean of 26 m. Map:
[great_bear_fill_0.25.png](../validation/experiments/depth_fill/great_bear_fill_0.25.png).
Deep through the central basin, shallower toward the shores and along the arms,
as on the chart raster, though 4 to 5 m too deep in three of the arms:

| Arm (boxes drawn by eye) | Cells | Fill (m) | Chart, capped (m) |
|---|---|---|---|
| Keith | 14 | 25.1 | 25.8 |
| McVicar | 25 | 24.7 | 20.4 |
| McTavish | 35 | 26.1 | 26.3 |
| Dease | 21 | 21.3 | 16.8 |
| Smith | 23 | 24.0 | 19.0 |
| central | 40 | 35.2 | 34.6 |

The depths matter to the ice. In the 2015–2016 season, with 100 % snow, cells at
least half lake freeze on average about 4 days earlier with the fill than with 40 m
everywhere (day 334.5 against 338.5); freeze-up follows the cell's depth (r = 0.94 over all cells). With the
72.2 m the table used to start with, some cells do not freeze until the next
January: another reason for a cap. Break-up barely moves (day 158 to 159). We did
not compare these dates with observed freeze-up.

#### What the pattern is worth

Errors in metres do not say what a depth does to the ice, so
`python validation/depth_impact.py` runs the model (2015–2016, 100 % snow, 330 kg/m³,
3 spin-up years; tables in `validation/experiments/depth_fill/`).

**Depth mostly moves freeze-up.** With no ice, the slab's temperature is
`T = Tf + H / (ρc·d)`, so `d` sets how much heat must be lost before freezing. Once
there is ice the slab stays at freezing, and that store no longer plays a part. One
column in the centre of Great Bear:

| Mixing depth (m) | Freeze-up | Break-up | Max ice (m) | Peak surface (°C) |
|---|---|---|---|---|
| 2 | 23 Oct | 8 Jun | 1.37 | 16.8 |
| 10 | 3 Nov | 8 Jun | 1.36 | 14.5 |
| 20 | 18 Nov | 7 Jun | 1.33 | 12.3 |
| 40 | 7 Dec | 6 Jun | 1.29 | 9.2 |
| 100 | 26 Dec | 9 Jun | 1.37 | 4.6 |
| 200 | 9 Jan | 5 Jun | 1.25 | 2.6 |

In the whole-lake run on the chart depths, freeze-up follows the cell's depth at
1.26 days per metre on Great Bear (r = 0.93) and 1.43 on Great Slave's west
(r = 0.99). Its spread between cells is 15.5 and 18.0 days (one standard deviation);
that of break-up is 4.9 and 2.3 days, and of maximum ice 0.07 and 0.05 m.

**So the pattern matters, on these two lakes.** Each row below is a whole-lake run
compared cell by cell with the run on the chart depths. *Shuffled* uses GLOBathy's
values on the wrong cells: same values, no pattern.

| Depths | Great Bear (167 cells): freeze-up error (days) | r | Bias (days) | Great Slave, west (76 cells): freeze-up error (days) | r | Bias (days) |
|---|---|---|---|---|---|---|
| GLOBathy | **6.7** | **0.85** | +2.1 | 20.1 | **0.46** | +17.9 |
| One value, the chart's mean | 12.3 | 0.45 | +2.3 | **17.2** | −0.23 | +1.5 |
| GLOBathy, shuffled | 16.9 | 0.14 | +2.7 | 26.5 | −0.17 | +18.5 |
| One value, 40 m | 16.4 | 0.43 | +16.4 | 30.0 | −0.42 | +30.0 |

Error here is the mean absolute difference. Across all these runs the break-up error
is at most 1.9 days and the maximum ice error at most 0.07 m.

On Great Bear, GLOBathy beats even one value at the correct mean, which a user would
not normally know. On Great Slave's west, one value at the correct mean still has
the smaller error, because GLOBathy is about 12 m too deep there on average, but
only GLOBathy's dates follow the chart run's from cell to cell at all (r = 0.46).

**GLOBathy mostly gets the order right.** Cells grouped into quarters by GLOBathy's
depth, shallowest first, with the 40 m cap (0.25°):

| Lake | Cells | Rank correlation | Reference by quarter (m) | GLOBathy by quarter (m) | Reference cells under 15 m | Found by GLOBathy | Called under 15 m wrongly |
|---|---|---|---|---|---|---|---|
| Superior | 196 | 0.92 | 27, 36, 40, 40 | 14, 28, 39, 40 | 6 | 5 | 23 |
| Michigan | 132 | 0.91 | 18, 27, 37, 40 | 11, 25, 37, 40 | 12 | 9 | 15 |
| Ontario | 49 | 0.97 | 19, 28, 37, 40 | 16, 29, 36, 40 | 4 | 4 | 2 |
| Huron | 148 | 0.86 | 14, 23, 33, 40 | 5, 14, 30, 40 | 30 | 27 | 34 |
| Erie | 61 | 0.73 | 8, 15, 20, 22 | 4, 10, 17, 34 | 28 | 27 | 7 |
| Great Bear (chart) | 144 | 0.88 | 15, 24, 32, 39 | 17, 29, 34, 40 | 30 | 12 | 1 |
| Great Slave, west (chart) | 67 | 0.73 | 11, 15, 18, 35 | 23, 32, 38, 40 | 33 | 0 | 0 |

The reference rises from quarter to quarter on all seven, so shallow cells are mostly
shallow and deep cells mostly deep. Being in the right order does not make the values
right: GLOBathy is too shallow near the shores of the larger Great Lakes and too deep
on the two northern lakes, and it finds none of Great Slave's shallow western cells.

**How far this goes.** The chart run is the reference, so this compares the model with
itself and says nothing about observed dates. It is one season, on two very large
lakes, with chart rasters that are digitised figures. On Great Slave it covers only
the west, not the East Arm. We have not shown that a better depth pattern gives a
better match to observed freeze-up.

#### Where it is weak

- **GLOBathy's pattern is geometry, not bathymetry.** Depth is `Dmax × distance
  to shore / largest distance to shore` (Khazaei et al., 2022, Eq. 1). Two parts of a
  lake equally far from shore get the same depth whatever is really there. Its
  maximum depth is right on these lakes because it is an input, so agreeing with
  the published maximum says nothing about the pattern.
- **Its mean depth can be far off**, since the formula fixes the maximum and not the
  volume: Great Bear 108.6 m against the published 71.7 m, Great Slave 133.3 m against
  41 m, Superior 109.0 m against 148.3 m from soundings.
- **The cap hides much of this, and it can also beat GLOBathy.** Before the cap, one
  mean depth beats GLOBathy on Great Bear, Great Slave and Erie. After it, one value
  beats GLOBathy on Superior and Erie, at both 0.25° and 0.1°.
- **Great Slave Lake is the worst case.** GLOBathy puts its 614 m in the West
  Basin (61.69°N, 114.12°W), the part farthest from shore, where the chart raster
  never goes below 100 m and averages about 20 m after the cap. The fill gives the
  West Basin's cells that are at least half lake a mean of 37 m, and it is 13 m too
  deep on average in the west. Within the west the order from shallow to deep still
  holds (see [above](#what-the-pattern-is-worth)), but the values do not, and modelled
  freeze-up comes about 18 days late. Between the basins the order is wrong too: the
  614 m must lie in the narrow East Arm, which the chart raster does not reach, and
  there the fill gives only 4 to 31 m
  ([map](../validation/experiments/depth_fill/great_slave_fill_0.25.png)). We expect
  that is too shallow, but cannot say by how much. For this lake, use your own
  raster, or edit both parts.
- **Great Bear's deepest point is misplaced too**: GLOBathy has it in the central
  basin (66.05°N, 120.80°W), while the record of the 1963 Canadian survey has it off
  the east end of McTavish Arm. The cap hides most of this, since both are far over
  40 m.
- **Capped depths are low on the larger Great Lakes** (bias −5.5 to −0.8 m on all but
  Erie): a straight ramp from the shore is gentler than real shores.
- **The tested lakes are not typical.** They are seven of the largest lakes in the
  world. In GLOBathy's table, 99.9 % of lakes use a random-forest estimate of the
  maximum depth, and these seven use a different value, presumably observed; the
  paper gives NSE 0.86 and NRMSE 0.29 out of sample for that estimate, on 1,503
  waterbodies. With the estimate in place of the value used, the capped error here
  falls on six lakes (by up to 2.2 m) and rises on Erie (to 10.9 m), while the depths
  before the cap get worse on all five Great Lakes. These large lakes may be among
  the estimate's training data, so this says little about an ordinary lake. No small
  lake has been tested.
- **The reference for the northern lakes is thin**: chart rasters from published
  figures, not surveys, and their class middles are our assumption.
- **The cap**: see [above](#lake-depth-is-not-mixing-depth). Results depend on it.
  Try `--depth-cap 30` or `50` to see how much.
- **Outlines differ**: GLOBathy follows HydroLAKES, cells follow the OpenStreetMap
  outline. That leaves few cells without data on Great Bear and Great Slave; other
  lakes may differ.

**Sources considered and not used.** GLOBathy is the only source built in, because
it covers lakes worldwide, its terms allow it, and its order from shallow to deep
holds up on the lakes tested. The others:

- *Canada1Water terrain model* (CC BY 4.0 on Zenodo; depth is its DEM band minus its
  DTM band, in whole metres; lakes of at least 10 km²). In a one-off comparison,
  not part of the validation scripts, its depths equal GLOBathy's, rounded, on most
  of 13 NWT lakes tested, and follow NOAA's soundings on Lake Michigan (capped RMSE
  0.6 m). On Great Bear and Great Slave's west they follow the chart rasters (capped
  RMSE 7.1 and 4.0 m; GLOBathy on the same cells 6.7 and 15.8 m), but on Great
  Slave's East Arm they reach at most 11 m, which is almost certainly wrong. So it
  adds little on an ordinary lake. We could not read its paper (publisher refused
  automated access), so its sources are inferred, and if it does use the chart
  rasters, its licence needs checking against theirs.
- *3D-LAKES* (Zenodo curves CC BY, article CC BY-NC-ND; 510,530 lakes): depth from
  how often a pixel is seen as water, so only within the observed range of water
  levels, which would not reach the bed of a deep lake. Its rasters are on Earth
  Engine.
- *CHS NONNA-100* (measured, Open Government Licence plus its own licence): its index
  has no tile on Great Bear and two on the west end of Great Slave.
- *GEBCO 2020*: both lake points tried, one on each northern lake, read a flat 156 m,
  the lake surface, so there is no depth there. *GLDB* could not be checked (its
  server refused the connection). *ERA5's lake depth* needs a CDS request.
- *Measured, but regional*: NOAA's Great Lakes grids are better than GLOBathy there.
  Alberta's Geological Survey (169 lakes), Minnesota DNR, and Ontario's historic maps
  also publish lake beds; we did not check their licences. Schertzer (2000) describes
  a digital bathymetry of Great Slave Lake, which we did not read. Any of these can
  be converted to positive depth and given with `--depth-raster`, under their own
  terms.

### ERA5 FLake for comparison

ERA5 simulates lakes with its own lake model, FLake (Mironov et al., 2010),
and stores its ice thickness as `lake_ice_depth`. It is a useful second
opinion beside CLIMo, but it is **off by default** because it is slow to
download: ECMWF's ARCO stores do not hold the lake variables, so they are
requested from the Copernicus Climate Data Store queue.

```bash
climo data --lake-name "Prosperous Lake" --grid --flake --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
```

Before downloading, CLIMo prints how many requests are needed (one per
decade, for the lake's box at 12 UTC each day) and a rough time: each
request typically waits 5 to 60 minutes in the CDS queue, longer at busy
times. Progress is printed per request. Finished requests are cached in
`~/.cache/climo/cds/flake/`, so an interrupted download resumes and later
builds over the same box reuse them. The result is `flake.nc`
(`lake_ice_depth` and ERA5's `lake_cover` on the 0.25° grid). In the GUI,
the FLake box on the Data tab shows the estimate when ticked.

`climo run` adds the FLake file next to the forcing automatically (or the one
given with `--flake FILE`); see below.

## Step 2: runs (`climo run` on a `.nc` forcing)

Every cell with `lake_fraction > 0` is run for every combination of snow
percentage and dry snow density, using the cell's latitude and mixing depth.
Runs are spread over all CPU cores (`--workers N` to change). Each cell gives
exactly the result of a single-point run with the same forcing and depth
(tested).

Outputs (in the data folder, or `--out`):

- `climo_daily.nc` — `IceThickness`, `SnowDepth`, `SnowIce`,
  `CongelationIce` (m) and `SurTemp` (°C), with dimensions
  (snow_percentage, rho_snow_cold, time, latitude, longitude); `lake_fraction`
  and `mixing_depth` (the depth used for each cell).
- `climo_ice_dates.nc` — `freeze_up`, `break_up`, `first_ice`, `last_ice`
  (day of year) and `ice_duration` (days), with dimensions
  (snow_percentage, rho_snow_cold, season, latitude, longitude);
  `effective_break_up` too with `--breakup-threshold`. The `season`
  coordinate is the year in which each ice season starts, with labels such
  as `2015-2016` in `season_label`; the file attribute `ice_season` gives the
  season window and event definitions ([outputs.md](outputs.md)). Days of
  year are calendar days of the event: a freeze-up on 5 January 2016 is
  `freeze_up` = 5 in season 2015.
- Both files record the run's settings as attributes (`mixing_depth_m`,
  `rho_snow_warm_kg_m3`, `spinup_years`, and `breakup_threshold_m` when
  given). If the starting ice of some cells does not melt out during the
  spin-up, one warning names how many cells, with an example.
- `climo_winters.csv` and `climo_summary.csv` — the maxima per winter and
  the summary per scenario of the **lake mean**: the cells with
  `lake_fraction` ≥ 0.5, or every lake cell when none is (a small lake on a
  coarse grid).

With FLake, `climo_daily.nc` also holds `flake_ice_thickness` and
`flake_lake_cover`, and `climo_ice_dates.nc` holds `flake_freeze_up`,
`flake_break_up` and `flake_ice_duration`, defined as CLIMo's (main ice
cover of the season; missing at the edges of the data). Each CLIMo cell takes
the 0.25° FLake cell containing its centre. FLake values do not depend on the
snow scenario. The GUI plots them beside CLIMo's.

In Python:

```python
import xarray as xr

daily = xr.open_dataset("results/qinghai/climo_daily.nc")
dates = xr.open_dataset("results/qinghai/climo_ice_dates.nc")

scenario = dict(snow_percentage=50, rho_snow_cold=330)
max_ice = daily["IceThickness"].sel(**scenario).groupby("time.year").max()   # map per year
lake_mean = max_ice.where(daily["lake_fraction"] >= 0.5).mean(["latitude", "longitude"])
break_up = dates["break_up"].sel(**scenario).sel(season=2015)   # winter 2015-2016
max_ice.sel(year=2016).plot()
```

`climo.plots.grid_tables(daily, dates, cell)` turns one cell (or the lake
mean) into the same tables as a point run.

## Speed and size

| Lake | Grid | Cells | Forcing (2015–2016) | Runs | Output |
|---|---|---|---|---|---|
| Qinghai Lake | ERA5 | 15 | 35 s | 15 × 6 scenarios in 8 s | < 1 MB |
| Great Slave Lake | ERA5 | 132 | 2 min 50 s | 132 × 4 scenarios in 32 s (16 cores) | 5 MB |
| Prosperous Lake | ERA5-Land | 7 | 1 min | 7 × 2 scenarios in 10 s | < 1 MB |

Reading the forcing is dominated by the number of storage chunks, which hold
several years each, so longer periods add relatively little time. A 10-year
run takes about 2 s per cell and scenario on one core. ERA5-Land grids have
about six times as many cells as ERA5 grids for the same lake. FLake adds the
CDS queue time (above).

## Limitations

- Cells on the shore are partly land, and their values are partly land
  values; filter with `lake_fraction`.
- ERA5 cells are about 28 km wide (ERA5-Land 11 km). Lakes smaller than a
  cell are better run as a single point (`climo data` without `--grid`),
  which can also use weather stations.
- The mixing depth is the same for every cell unless you fill or edit
  `cells.csv`; filled depths are modelled and
  [perform poorly in places](#where-it-is-weak).
- Each CLIMo column is independent: no exchange of heat or ice between
  cells.

## References

- Brown, L.C., and Duguay, C.R. (2011). The fate of lake ice in the North American Arctic. *The Cryosphere*, 5, 869–892. https://doi.org/10.5194/tc-5-869-2011
- Canada1Water terrain model. Zenodo. https://zenodo.org/records/17807459; paper in *Data in Brief*, https://doi.org/10.1016/j.dib.2026.112878 (not read; considered, not used)
- Carmack, E.C., et al. (2021). Thermobaric processes both drive and constrain seasonal ventilation in deep Great Slave Lake, Canada. *JGR Earth Surface*. https://doi.org/10.1029/2021JF006288 (not read)
- Duguay, C.R., et al. (2003). Ice-cover variability on shallow lakes at high latitudes: model simulations and observations. *Hydrological Processes*, 17, 3465–3483. https://doi.org/10.1002/hyp.1394 (read only through those citing it)
- Hanna, M. (1990). Evaluation of models predicting mixing depth. *Canadian Journal of Fisheries and Aquatic Sciences*, 47, 940–947. https://doi.org/10.1139/f90-108 (not read)
- Hersbach, H., et al. (2020). The ERA5 global reanalysis. *Quarterly Journal of the Royal Meteorological Society*, 146, 1999–2049. https://doi.org/10.1002/qj.3803
- Huang, C.H., et al. (2025). 3D-LAKES: Three-Dimensional Global Lake and Reservoir Bathymetry from ICESat-2 Altimetry and Landsat Imagery. *Scientific Data*, 12. https://doi.org/10.1038/s41597-025-05911-y (considered, not used)
- Khazaei, B., et al. (2022). GLOBathy, the global lakes bathymetry dataset. *Scientific Data*, 9, 36. https://doi.org/10.1038/s41597-022-01132-9
- Kheyrollah Pour, H., and Attiah, G. (2022). Great Bear Lake and Great Slave Lake Bathymetry. Borealis. https://doi.org/10.5683/SP3/BZWZQL (validation only)
- Kheyrollah Pour, H., et al. (2012). Simulation of surface temperature and ice cover of large northern lakes with 1-D models. *Tellus A*, 64, 17614. https://doi.org/10.3402/tellusa.v64i0.17614
- Ménard, P., et al. (2002). Simulation of ice phenology on Great Slave Lake, Northwest Territories, Canada. *Hydrological Processes*, 16, 3691–3706. https://doi.org/10.1002/hyp.1230 (read only through those citing it)
- Messager, M.L., et al. (2016). Estimating the volume and age of water stored in global lakes using a geo-statistical approach. *Nature Communications*, 7, 13603. https://doi.org/10.1038/ncomms13603
- Mironov, D., et al. (2010). Implementation of the lake parameterisation scheme FLake into the numerical weather prediction model COSMO. *Boreal Environment Research*, 15, 218–230.
- Muñoz-Sabater, J., et al. (2021). ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. *Earth System Science Data*, 13, 4349–4383. https://doi.org/10.5194/essd-13-4349-2021
- Robinson, A.L. (2024). *Lake Ice in Canada: Improving one-dimensional lake ice modelling and investigating changing ice cover and trends*. PhD thesis, University of Toronto. https://utoronto.scholaris.ca/items/b7c117dd-0f9a-4de7-a898-561f80128875
- Schertzer, W.M. (2000). *Digital bathymetry of Great Slave Lake*. NWRI Contribution 00-257, Environment Canada. https://publications.gc.ca/site/eng/9.871553/publication.html (not read)
- Regional lake beds, not used: NOAA NCEI Great Lakes, https://www.ncei.noaa.gov/products/great-lakes-bathymetry; Alberta Geological Survey, https://ags.aer.ca/interactive-data-and-tools/lake-bathymetry-data; Minnesota DNR, https://gisdata.mn.gov/dataset/water-lake-bathymetry; Ontario MNRF, https://geohub.lio.gov.on.ca/datasets/mnrf::historic-bathymetry-maps-1/about
