# Notices

CLIMo is licensed under the PolyForm Noncommercial License 1.0.0 (see
[LICENSE](LICENSE)). Use for research, teaching, personal study, and by
non-profit, educational, public research and government organisations is
free. Commercial use needs a separate licence from H2O Geomatics
(info@h2ogeomatics.com).

Required Notice: Copyright (c) 2026 H2O Geomatics. Commercial licences: info@h2ogeomatics.com

## Model

CLIMo was developed by Duguay et al. (2003) from the sea-ice model of Flato
and Brown (1996). CLIMo Classic (`climo/`) is a port of the previous Python
version of CLIMo, kept unmodified in `legacy/` for reference and
for the regression test. Please cite the model description as given in the
[README](docs/overview.md#how-to-cite).

## Third-party data in this repository

These files are not covered by the CLIMo licence; they keep their own terms.

| Files | Source | Terms |
|---|---|---|
| `validation/observations/*_thickness.csv` | Environment and Climate Change Canada, Canadian Ice Thickness Program (collections 1947–2002 and 2002–) | Open Government Licence – Canada (https://open.canada.ca/en/open-government-licence-canada). Contains information licensed under the Open Government Licence – Canada. |
| `validation/observations/*_ice_dates.csv` | Canadian Lake Ice Database records (Lenormand et al., 2002) in the Global Lake and River Ice Phenology Database, NSIDC G01377 (Benson et al., 2000) | Free to use; cite the dataset (see [docs/validation.md](docs/validation.md)) |
| `validation/results/`, `validation/experiments/` | Derived from the data above and from ERA5/ERA5-Land and NOAA station data | Contains modified Copernicus Climate Change Service information; NOAA data are in the public domain |
| `validation/experiments/depth_fill/` | Summary numbers and maps from GLOBathy (Khazaei et al., 2022), HydroLAKES (Messager et al., 2016) and ERA5/ERA5-Land, with statistics of comparison against NOAA NCEI soundings and chart rasters (Kheyrollah Pour and Attiah, 2022). No depths from the chart rasters are included | GLOBathy: CC0. HydroLAKES: CC BY 4.0. Chart rasters: CC BY-NC-SA 4.0. NOAA data are in the public domain. Contains modified Copernicus Climate Change Service information |

Data that CLIMo downloads when you run it (ERA5, ERA5-Land, ERA5 lake
variables, NOAA ISD and GHCN-Daily, ECCC climate data, OpenStreetMap
outlines, HydroLAKES attributes, GLOBathy lake depths) are not distributed with CLIMo; their terms
are listed in [docs/data.md](docs/data.md). Files built from them carry the
attribution in their metadata.
