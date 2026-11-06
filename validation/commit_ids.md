# Commit IDs before and after the history rewrite

On 2 November 2026 the commit author address was corrected across the whole
history, before the repository went public. Commit IDs are computed from the
commit details, so every commit got a new ID. The files in each commit did not
change, which was checked commit by commit.

The experiment records in `validation/experiments/` (`run.json` and
`summary.md`) were left as they were written, so they still name the code by
its old ID. To find the code behind a record, look up that ID here and use the
new one, for example `git show <new ID>`.

The list is in date order, oldest first.

| Old ID | New ID | Commit |
|---|---|---|
| `35a25e7` | `8450baa` | Initial version: CLIMo Classic port of the legacy Python CLIMo |
| `c6228eb` | `ef18810` | Fix output bookkeeping bugs in CLIMo Classic |
| `3fdc754` | `70d7d4c` | Add batch snow-scenario runs and consistent layer depth columns |
| `1b92151` | `a84b370` | Add data module: lake forcing from NOAA stations and ERA5 |
| `6440369` | `76a545b` | Read ERA5 from ECMWF's ARCO stores instead of Earthmover |
| `c406d77` | `10e812e` | Add lake search by name and document CDS key setup |
| `25b49fd` | `48ba512` | Add gridded runs over a whole lake (netCDF) |
| `944af24` | `305432c` | Report ice dates per ice season instead of per calendar year |
| `41ad890` | `851e33a` | Add a tabbed graphical interface (climo gui) |
| `c6dc842` | `7540ef6` | Add data folders with reuse, climo batch for many sites, and workflow docs |
| `db06a15` | `1446c94` | Merge season-based ice dates |
| `1803539` | `ef1a426` | Merge data workflow: data folders, climo batch, workflow docs |
| `f097a3a` | `a914468` | Merge tabbed GUI; align GUI and batch summaries with ice seasons |
| `a5fe031` | `99bc628` | Add validation cases: Yellowknife Back Bay and Baker Lake (roadmap step 7) |
| `74100b6` | `4aa833d` | Keep the validation results (tables and figures) in the repository |
| `02d0590` | `d5158e0` | Merge validation cases: Yellowknife Back Bay and Baker Lake |
| `f4dd8dc` | `058db51` | Mark the graphical interface (roadmap step 6) done |
| `0d95089` | `c4a5149` | License CLIMo under PolyForm Noncommercial 1.0.0 |
| `44dc249` | `79f7033` | Write per-winter and summary tables for every run |
| `072c30b` | `6a402dc` | Accept your own daily snowfall file in place of station/ERA5 snowfall |
| `881dd30` | `e81909c` | Add ECCC stations, a hard station distance limit and a station listing |
| `e78b097` | `3a1aa75` | Add ERA5-Land grids, a mixing depth per cell and ERA5 FLake comparison |
| `c399452` | `6975620` | Add the effective break-up diagnostic; fix small-lake means and FLake edges |
| `183dfa2` | `b32b621` | Write install and how-it-runs guides; document the new options; add validation sites |
| `2b14d60` | `27a884b` | Fix ECCC years without records; prefer the more complete record at a site |
| `cbbca25` | `0b57bfa` | Run cleanly with pandas 3 and shapely 2.2 |
| `e3c6e44` | `2184dcb` | Keep every Data tab control reachable on small screens |
| `d8763b8` | `c846ee1` | Tidy run_sites and a long line |
| `2d81679` | `0ad267e` | Add validation results for the four southern lakes and the effective break-up |
| `a4a315f` | `97eee6a` | Tell Windows users to install from the Command Prompt, not PowerShell |
| `3cd2bdb` | `3568023` | Check every station near the shore and choose one per variable |
| `18d1fb0` | `73c74b4` | Show station coverage by year and let stations fill chosen years |
| `553215d` | `5de6d4d` | Add snowfall from snow on the ground as an option |
| `56e9472` | `490b7f0` | Add a spin-up years setting and warn when the starting ice never melts |
| `7f6521c` | `066b757` | Add an experiment harness for comparing formulation options |
| `d128e99` | `feb5319` | Add the exponential thin-snow albedo option (A1) |
| `df274d2` | `1ee7a33` | Add the clear-ice versus snow-ice albedo option (A2) |
| `44e7bd7` | `49af167` | Add the smooth melt albedo transition option (A3) |
| `0113baa` | `ddcb194` | Add the surface-temperature solver fall-back fix (F1, legacy note 5) |
| `717f7c4` | `367e627` | Add the Newton-Raphson acceptance fix (F2, legacy note 4) |
| `30f75fb` | `817a765` | Add the bulk snow-density evolution option (S1) |
| `b7292c1` | `e4675fe` | Add the Newton step limit fix (F3, legacy note 11) |
| `f3aa213` | `aa632eb` | Combine the solver fixes into one option; add the options guide and GUI controls |
| `910f083` | `1dae337` | Release 1.0.0 |
