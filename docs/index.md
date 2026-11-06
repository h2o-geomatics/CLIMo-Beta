> **These pages describe the CLIMo source code, which is not published yet. The beta is a ready-made
> Windows program instead: start with the [quick start](../QUICKSTART.md).** Steps that use `git`, `pip` or
> Python, such as the quick start below, do not apply to it. The `climo` commands work the same when typed in the
> Command Prompt in the program's folder: see [the command line](../QUICKSTART.md#the-command-line).

# CLIMo: Canadian Lake Ice Model

CLIMo is a one-dimensional thermodynamic model of lake ice. Driven by daily
air temperature, relative humidity, wind speed, cloud cover and snowfall, it
simulates freeze-up and break-up dates, ice thickness and composition (clear /
congelation ice and snow ice), snow depth, the temperature profile of the snow
and ice, and the components of the surface energy budget.

CLIMo was developed by Duguay et al. (2003) from the sea-ice model of Flato and
Brown (1996). This is a documented, source-available Python version for
research use. With every option at its default it reproduces the legacy model
bit for bit. It also builds the input weather for any lake from weather stations
and ERA5, runs single points, whole lakes and many lakes at once, and compares
results with ERA5's own lake ice and with observations.

## Quick start: a first run in 5 minutes

This runs the model on a weather file that comes with the code, so it needs no
account and no downloads beyond the install. You need Python 3.10 or later and
git. Without them, follow [the full install guide](install.md), which starts
from a computer without Python.

**1. Get the code and install the model.**

On Windows, in the **Command Prompt, not PowerShell**, which blocks the script
that switches on the environment:

```bat
git clone https://github.com/h2o-geomatics/CLIMo-Beta
cd CLIMo
py -m venv .venv
.venv\Scripts\activate.bat
pip install -e .
```

On macOS and Linux:

```bash
git clone https://github.com/h2o-geomatics/CLIMo-Beta
cd CLIMo
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

**2. Run ten years at a site near Yellowknife, for three snow scenarios.**

```bash
climo run --forcing tests/data/forcing_YL_70.csv --start 2015 --end 2024 --latitude 62.4 --mixing-depth 10 --snow-percentage 0 50 100 --out results/yl70
```

It takes a few seconds and writes four tables to the `results` folder, which any
spreadsheet program can open:

| File | One row per | What it holds |
|---|---|---|
| `yl70_daily.csv` | day and scenario | ice thickness, snow depth, ice types, temperatures, energy fluxes |
| `yl70_ice_dates.csv` | ice season and scenario | freeze-up, break-up, ice duration |
| `yl70_winters.csv` | winter and scenario | maximum ice and snow |
| `yl70_summary.csv` | scenario | means over all winters |

**3. Check the result.** `yl70_summary.csv` should show these values (with some
columns left out here):

| `snow_percentage` | `mean_max_ice_thickness_m` | `mean_freeze_up_doy` | `mean_break_up_doy` |
|---|---|---|---|
| 0 | 1.599 | 315 | 154 |
| 50 | 1.182 | 315 | 150 |
| 100 | 1.231 | 315 | 152 |

Without snow the ice grows thickest: snow insulates it from the cold air.

## Your own lake

For a lake of your own, install the data module as well. ERA5 needs a free
Copernicus Climate Data Store key
([how to set it up](install.md#5-set-up-era5-access-once)); station data and
lake search need no account.

```bash
pip install -e ".[data]"
climo data --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth 10 --snow-percentage 0 50 100
```

Or do it all in a window with `climo gui` ([graphical interface](gui.md)). If
the data module fails to install, use [the conda route](install.md#3-create-an-environment-and-install-climo).

For new work, add `--fixes on`, which corrects three numerical faults of the
legacy solver ([model options](options.md#fixes)).

## Where to go next

| I want to... | Read |
|---|---|
| install everything, step by step, and fix install problems | [Install and first run](install.md) |
| understand each step, which options to choose, and how long it takes | [How CLIMo runs](how_it_runs.md) |
| manage data folders, or run many lakes with `climo batch` | [Workflow](workflow.md) |
| build forcing from stations and ERA5, or use my own data | [Forcing data](data.md) |
| model a whole lake on a grid | [Whole lakes](grid.md) |
| use the graphical interface | [Graphical interface](gui.md) |
| look up an output column | [Outputs](outputs.md) |
| switch on a model option, and see where it helps and hurts | [Model options](options.md) |
| read the equations | [Model description](model.md) |
| read the science behind each option | [Formulation record](formulations.md) |
| see how well CLIMo does at six Canadian lakes | [Validation](validation.md) |
| know which legacy behaviours are kept, and the known issues | [Legacy behaviours](legacy_notes.md) |

## Licence and citation

CLIMo is free for research, teaching, personal study and other non-commercial
use, and for universities, public research organisations, government bodies and
charities, under the [PolyForm Noncommercial License 1.0.0](../LICENSE).
Commercial use needs a separate licence from H2O Geomatics
(info@h2ogeomatics.com). Data shipped with the code keep their own terms
([NOTICE](../NOTICE.md)).

Please cite the model description and this software
([CITATION.cff](../CITATION.cff)):

> Duguay, C. R., Flato, G. M., Jeffries, M. O., Ménard, P., Morris, K., and Rouse, W. R. (2003).
> Ice-cover variability on shallow lakes at high latitudes: model simulations and observations.
> *Hydrological Processes*, 17(17), 3465–3483. https://doi.org/10.1002/hyp.1394
