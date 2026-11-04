# Quick start

From download to your first lake in about 15 minutes. You need 64-bit Windows 11, about 1 GB of disk space and
an internet connection. Python is not needed. It should also work on Windows 10, which has not been tested yet, so
please tell us how it goes.

## 1. Download and unpack

1. Under [Releases](https://github.com/h2o-geomatics/CLIMo-Beta/releases), download `CLIMo-<version>-windows-x64.zip` from the newest release.
2. Right-click the zip, choose **Extract All...** and pick a folder of your own, such as your Documents folder.
   **The program does not work from inside the zip,** so do unpack it.
3. In the unpacked folder, double-click **`CLIMo-GUI.exe`**. Leave the `_internal` folder beside it.

The first start can take up to a minute while antivirus software checks the files. Later starts take seconds.

## 2. The Windows warning

CLIMo is not signed with a paid certificate yet, so Windows may show **"Windows protected your PC"**. To continue,
choose **More info**, then **Run anyway**.

To check that your copy is the one we published, compare its checksum with the `.sha256` file in the release. In the
Command Prompt, in the folder with the zip:

```bat
certutil -hashfile CLIMo-<version>-windows-x64.zip SHA256
```

If there is no **Run anyway**, your computer's settings block unsigned programs (Smart App Control, or your
organisation's rules), and you may need your IT department. If antivirus software removes a file, please
[tell us](https://github.com/h2o-geomatics/CLIMo-Beta/issues/new/choose). Please do not switch off your protection for this.

## 3. Set up ERA5 (once)

ERA5 fills in whatever no weather station near the lake provides, so most runs need it. It needs a free key from the
Copernicus Climate Data Store. In CLIMo, open **Help > ERA5 key...** and follow the three steps:

1. Register or log in, and copy your personal access token.
2. Accept the ERA5 and ERA5-Land licences, logged in with the same account.
3. Paste the token and press **Save**.

The key stays on your computer, in the file `.cdsapirc` in your home folder. It is never written to CLIMo's log.
Do not post it in an issue.

## 4. Your first lake

**On the Data tab:**

1. Type a lake name, such as `Prosperous Lake`, and press **Search**. The first search downloads a 79 MB table of
   lakes (HydroLAKES), once.
2. Set the **Start date** and **End date**. Try two years first, for example `2015-01-01` and `2016-12-31`.
3. Press **Build forcing**. CLIMo looks for weather stations near the lake and uses ERA5 for the rest. It reports
   where each variable came from.

**On the Model tab:** the forcing is already filled in.

1. Set the **Mixing depth**. The HydroLAKES mean depth shown is a guide.
2. Set **Snow percentages**, for example `0, 50, 100`.
3. Press **Run**. The **Results** tab opens with the plots.

Results are in **Documents\CLIMo\results**, as CSV files you can open in any spreadsheet program
([what the columns mean](docs/outputs.md)).

## The command line

`climo.exe`, in the same folder, takes the same commands as the [documentation](docs/how_it_runs.md). Open the
Command Prompt in the program's folder:

```bat
climo --help
climo data --lake-name "Prosperous Lake" --start 2015-01-01 --end 2016-12-31 --out-dir results/prosperous
climo run --forcing results/prosperous --start 2015 --end 2016 --mixing-depth 10 --snow-percentage 0 50 100
```

Double-clicking `climo.exe` only flashes a window: it is meant to be started from the Command Prompt.

## Where things are

| What | Where | Safe to delete? |
|---|---|---|
| The program | The folder you unpacked | Yes, this uninstalls it. To update, unpack a new version beside it |
| Your results | `Documents\CLIMo` | They are yours |
| Downloads | `C:\Users\<you>\.cache\climo` | Yes, they download again when needed |
| Log | `C:\Users\<you>\.cache\climo\logs\climo-gui.log` (**Help > Open log folder**) | Yes |
| ERA5 key | `C:\Users\<you>\.cdsapirc` | Yes, if you no longer need ERA5 |

CLIMo writes nothing else, and changes no Windows settings.

## Something went wrong?

Choose **Help > Copy report information**, then **Help > Report a problem**, and paste it in. Attaching the log helps
a great deal. Look through it first: it shows lake names and folder names. If your study sites are private, write to
info@h2ogeomatics.com instead, because issues here are public.

Common problems are in the [install guide's troubleshooting section](docs/install.md#troubleshooting). Those about
ERA5 keys and licences apply to the program too.
