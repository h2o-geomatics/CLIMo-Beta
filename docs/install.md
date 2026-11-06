> **During the beta, use the [quick start](../QUICKSTART.md) instead of this page.** The beta is a
> ready-made Windows program that needs no Python. This page describes installing from the source code,
> which is not published yet, so its `git clone` and `pip install` steps do not work for now.

# Installing CLIMo and running it for the first time

This guide takes you from a computer without Python to a first CLIMo run.
It takes about 20 minutes, most of it downloads. If you already use Python
3.10 or later, skip to [step 2](#2-get-the-code).

What you need:

- Windows 10/11, macOS or Linux, about 3 GB of free disk space, and an
  internet connection.
- For ERA5 forcing (most runs), a free account at the Copernicus Climate
  Data Store; [step 5](#5-set-up-era5-access-once) sets it up. Station data
  and lake search need no account, and neither does running the model on
  forcing you already have.

> **On Windows, type every command in the Miniforge Prompt or the Command
> Prompt, never in PowerShell.**
>
> PowerShell blocks the scripts that switch on a Python environment. The
> error says *"…Activate.ps1 cannot be loaded because running scripts is
> disabled on this system"*. Some commands below (`set`, `%USERPROFILE%`) are
> also Command Prompt commands that PowerShell does not understand.
>
> You can tell which window you are in from the line where you type:
>
> | Window | The line starts with |
> |---|---|
> | PowerShell (wrong) | `PS C:\Users\you>` |
> | Command Prompt (right) | `C:\Users\you>` |
> | Miniforge Prompt (right) | `(base) C:\Users\you>` |
>
> Windows Terminal and the terminal in VS Code open PowerShell by default.
> In Windows Terminal, open a Command Prompt tab from the **˅** menu next to
> the tabs. In VS Code, pick **Command Prompt** from the **˅** next to the
> **+** in the terminal panel.

CLIMo is free for research, teaching and other non-commercial use under the
PolyForm Noncommercial License 1.0.0 ([LICENSE](../LICENSE)). Commercial use
needs a licence from H2O Geomatics (info@h2ogeomatics.com).

## 1. Install Python (Miniforge)

CLIMo's data module uses packages with compiled parts (for maps, lake
outlines and netCDF files). The simplest reliable way to get them on every
system is **Miniforge**, a small Python distribution that installs packages
from the community *conda-forge* channel.

**Windows**

1. Download `Miniforge3-Windows-x86_64.exe` from
   https://github.com/conda-forge/miniforge/releases/latest.
2. Run it. Choose "Just Me" and keep the suggested options.
3. From the Start menu, open **Miniforge Prompt** (type "Miniforge" after
   pressing the Windows key). It is a Command Prompt with conda already set
   up, and its line starts with `(base)`. **Use this window for every command
   in this guide, and again every time you use CLIMo.** Do not use
   PowerShell; the box above explains why.

**macOS and Linux**

Open a terminal and run:

```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Accept the licence, keep the default location, and answer `yes` when asked
whether to initialise conda. Close the terminal and open a new one.

Check that it worked: `conda --version` prints a version number.

## 2. Get the code

**With git** (recommended: updating is then one command):

```bash
git clone https://github.com/h2o-geomatics/CLIMo-Beta
cd CLIMo
```

If `git` is not installed, install it with `conda install -c conda-forge git`.

**Or as a ZIP file:** on the repository page, choose **Code → Download
ZIP**, unpack it somewhere without spaces or special characters in the path
(e.g. `C:\Users\<you>\CLIMo` or `~/CLIMo`), and go into that folder:

```bat
cd /d C:\Users\<you>\CLIMo
```

On Windows, keep the `/d`: without it, `cd` does not change to a folder on
another drive (such as `D:\Projects\CLIMo`). On macOS and Linux, use
`cd ~/CLIMo`.

All later commands are run from this folder, the one that contains
`README.md` and `pyproject.toml`. To check, run `dir pyproject.toml`
(Windows) or `ls pyproject.toml` (macOS/Linux): it should list the file, not
say "File Not Found".

## 3. Create an environment and install CLIMo

An *environment* is a separate set of packages for one project, so CLIMo
cannot disturb (or be disturbed by) other Python software on the computer.

```bash
conda create -n climo -c conda-forge python=3.12 geopandas netcdf4 xarray zarr matplotlib
conda activate climo
pip install -e ".[data,test]"
```

- The first command creates the environment `climo` with the compiled
  packages from conda-forge (answer `y` when asked).
- `conda activate climo` switches to it. The line now starts with `(climo)`
  instead of `(base)`. **Do this each time you open a new Miniforge Prompt**,
  before any `climo` command.
- `pip install -e ".[data,test]"` installs CLIMo from this folder with the
  data module (`data`) and the test tools (`test`). `-e` ("editable") means
  the installed CLIMo is this folder: after updating the code you do not
  need to reinstall, unless the dependencies change.

Only want the model itself (bringing your own forcing)? `pip install -e .`
needs just numpy and pandas. The validation scripts need
`pip install -e ".[validation]"`.

**Without conda.** If you already have Python 3.10 or later from
python.org, a virtual environment usually works instead. Run these from the
CLIMo folder.

*Windows: in the Command Prompt, not PowerShell* (open it by pressing the
Windows key, typing `cmd` and pressing Enter):

```bat
py -m venv .venv
.venv\Scripts\activate.bat
pip install -e ".[data,test]"
```

*macOS and Linux:*

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[data,test]"
```

- The first command creates the environment in a `.venv` folder inside the
  CLIMo folder.
- The second switches to it. The line then starts with `(.venv)`. **Each
  time you open a new window, go to the CLIMo folder and run the second
  command again** before any `climo` command.
- If `pip` fails to build `geopandas`, `pyproj` or `netCDF4`, use the conda
  route above.

## 4. Check the installation

```bash
climo --help
pytest
```

- `climo --help` lists the commands.
- `pytest` runs the tests in about a minute. It should end with a line such
  as `165 passed, 6 skipped`. The numbers can differ slightly; what matters
  is that the line says neither `failed` nor `error`.
- The skipped tests download data and run only when asked. To run them in
  the Command Prompt, use `set CLIMO_NETWORK_TESTS=1` and then `pytest`. On
  macOS and Linux, use `CLIMO_NETWORK_TESTS=1 pytest`.

## 5. Set up ERA5 access (once)

ERA5 (and ERA5-Land) fill the variables no weather station near the lake
provides, and drive gridded runs. Access needs a free Copernicus Climate Data
Store (CDS) key:

1. Register at https://cds.climate.copernicus.eu and log in.
2. Copy your **personal access token** from your profile page
   (https://cds.climate.copernicus.eu/profile).
3. **Accept the licences** on the Download tab of
   [ERA5 hourly data on single levels](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=download)
   and [ERA5-Land hourly data](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-land?tab=download).
   The ERA5 single-levels licence also covers the ERA5 FLake lake ice used
   for comparisons.
4. Create the file `.cdsapirc` in your home folder
   (`C:\Users\<you>\.cdsapirc` on Windows, `~/.cdsapirc` elsewhere) with
   two lines:

   ```
   url: https://cds.climate.copernicus.eu/api
   key: <your personal access token>
   ```

   On Windows, create it with `notepad %USERPROFILE%\.cdsapirc` and make
   sure it is not saved as `.cdsapirc.txt`.

More options (another location, an environment variable) are in
[data.md](data.md#setting-up-era5-access-cds-key).

## 6. A first run

This example models Prosperous Lake near Yellowknife for two winters. Each
step prints what it does; the whole example takes a few minutes.

**Find the lake** (OpenStreetMap, with its HydroLAKES depth):

```bash
climo lake "Prosperous Lake"
```

**See the weather stations near it** (optional). Stations farther than
`--max-station-distance` (default 5 km) from the shore are never used:

```bash
climo stations --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out results/prosperous
```

This prints each station within the limit with its distance and how much of
the period it covers, and writes `results/prosperous_stations.csv` and a map.

**Build the forcing** (step 1): stations where good enough, ERA5 for the rest:

```bash
climo data --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
```

The report says where each variable came from. The folder
`results/prosperous` now holds `forcing.csv`, `metadata.json` and `map.png`.

**Run the model** (step 2) for three snow scenarios:

```bash
climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth 10 --snow-percentage 0 50 100
```

Results land in the same folder: daily values (`climo_daily.csv`), ice dates
per winter (`climo_ice_dates.csv`), maxima per winter (`climo_winters.csv`)
and one summary row per scenario (`climo_summary.csv`). Open them in any
spreadsheet program; the columns are explained in [outputs.md](outputs.md).

**Or do it all in the window:**

```bash
climo gui
```

[how_it_runs.md](how_it_runs.md) explains what each step does and which
options to choose; [workflow.md](workflow.md) has the details of data
folders and running many lakes.

## Updating

In the Miniforge Prompt (or, with a virtual environment, the Command Prompt
on Windows or a terminal on macOS/Linux):

```bat
conda activate climo
cd /d <your CLIMo folder>
git pull
pip install -e ".[data,test]"
```

- With a virtual environment, replace the first line with
  `.venv\Scripts\activate.bat` (Windows) or `source .venv/bin/activate`
  (macOS/Linux), run after the `cd` line.
- Without git, unpack a new ZIP over the old folder instead of `git pull`.
- The `pip install` line is only needed when the dependencies changed, but
  running it anyway does no harm.

Forcing built by an older version is rebuilt the next time you run
`climo data` on it (the version is part of the recorded settings).

## Uninstalling

```bash
conda deactivate
conda env remove -n climo
```

With a virtual environment there is nothing to remove separately: the
environment is the `.venv` folder inside the CLIMo folder.

Then delete the CLIMo folder and, to free the space used by downloads, the
cache folder `~/.cache/climo` (`C:\Users\<you>\.cache\climo` on Windows).

## Troubleshooting

**`…Activate.ps1 cannot be loaded because running scripts is disabled on
this system`.** You are in PowerShell; its line starts with `PS`. Close it
and open the Miniforge Prompt (conda) or the Command Prompt (virtual
environment; press the Windows key, type `cmd`, press Enter). Then go to the
CLIMo folder and activate again, with `conda activate climo` or
`.venv\Scripts\activate.bat`.

If you have to stay in PowerShell, this lets scripts run in that window only,
until you close it:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

**`'climo' is not recognized` / `command not found`.** There are two
possible causes:

- The environment is not active. Run `conda activate climo`, or the
  virtual environment's activate command from step 3. The line should then
  start with `(climo)` or `(.venv)`.
- CLIMo was installed into a different environment. Activate the one you
  want and run `pip install -e ".[data,test]"` again from the CLIMo folder.
  `pip show climo` tells you whether the active environment has it.

**`'conda' is not recognized`.** You are in an ordinary Command Prompt or in
PowerShell. Open the **Miniforge Prompt** from the Start menu instead.

**`'python' is not recognized`, or typing `python` opens the Microsoft
Store** (Windows). Use `py` instead of `python`, as in step 3. `py` is the
Python launcher that the python.org installer adds.

**`cd` does nothing** (Windows). The folder is on another drive. Use
`cd /d D:\path\to\CLIMo`.

**`ERA5 access needs a free Copernicus CDS key`.** The `.cdsapirc` file was
not found or has no `key:` line; the message names the file it looked for.
See [step 5](#5-set-up-era5-access-once).

**`403 Forbidden` or "required licences not accepted" from the CDS.** Accept
the dataset licences (step 5, point 3) while logged in with the same account
as the key.

**FLake downloads take a long time.** ERA5 FLake lake ice (`--flake`, off by
default) comes from the CDS request queue. CLIMo prints the number of
requests and a rough time before starting; requests often wait several
minutes and at busy times an hour or more. Finished requests are cached, so
an interrupted download resumes where it stopped.

**GDAL or PROJ errors such as `Can't load requested DLL`** (Windows). Another
GDAL installation is being picked up, usually through the `GDAL_DATA`,
`GDAL_DRIVER_PATH` or `PROJ_LIB` environment variables. Install geopandas
from conda-forge into the `climo` environment (step 3). Then either remove
those variables from your user environment settings, or clear them in the
window before running CLIMo, for example with `set GDAL_DRIVER_PATH=`.

**Behind a proxy.** Set `HTTPS_PROXY` (and `HTTP_PROXY`) to your proxy's
address before running CLIMo; downloads and the CDS client use them. In the
Command Prompt that is `set HTTPS_PROXY=http://proxy.example.org:8080`
(with your proxy's address), in the same window.

**The graphical interface does not start on Linux.** Install Tk for your
system Python (e.g. `sudo apt install python3-tk`), or use the conda
environment, which includes it.

**Where are downloads kept?** In `~/.cache/climo`; set the `CLIMO_CACHE`
environment variable to use another folder. Deleting it is safe: files are
downloaded again when needed.

**Something else.** Run the failing command again and keep the whole error
message; `pytest` shows whether the installation itself is sound.
