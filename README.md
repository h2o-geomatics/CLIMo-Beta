# CLIMo beta

**CLIMo**, the Canadian Lake Ice Model, is a one-dimensional thermodynamic model of lake ice. It simulates
freeze-up and break-up dates, ice thickness and composition, and snow on the ice, driven by daily weather. It was
developed by Duguay et al. (2003).

This repository is where the beta is shared with invited testers: a ready-made Windows program, its documentation
and a place to report problems and request features. You do not need Python.

**[Get started: QUICKSTART.md](QUICKSTART.md)** · [Download](https://github.com/h2o-geomatics/CLIMo-Beta/releases) · [Documentation](docs/overview.md) ·
[Report a problem or request a feature](https://github.com/h2o-geomatics/CLIMo-Beta/issues/new/choose)

## What is here, and what is not

| | |
|---|---|
| **The program** | Under [Releases](https://github.com/h2o-geomatics/CLIMo-Beta/releases), as a zip for 64-bit Windows 11. It should also work on Windows 10, which has not been tested yet |
| **Documentation** | [docs/](docs/overview.md), with the science and the validation behind each option |
| **Validation data and results** | [validation/](validation/), which the documentation links to |
| **Source code** | Not published yet. It is developed in a separate repository and is planned for this one later |

## This is a beta

- **Results are not final.** Check them before you rely on them. With every option at its default, the model is
  tested to reproduce the legacy CLIMo model results bit for bit, inside the packaged program too. The tools around
  it, such as the forcing builder and the window, have had less use.
- **The program is not signed,** so Windows warns about it the first time. The quick start shows what to do.
  Managed computers at some organisations may block it outright; please tell us if yours does.
- **Windows only** for now.
- **The documentation is still being reviewed.** Some pages describe installing or running from the source code,
  which does not apply to the program yet.
- **It needs the internet** for lake search, weather stations and ERA5, and sends nothing but those data requests.
  Running the model on forcing you already have works offline. There is no telemetry.

## Licence

CLIMo is free for research, teaching and other non-commercial use under the
[PolyForm Noncommercial License 1.0.0](LICENSE). Commercial use needs a licence from H2O Geomatics
(info@h2ogeomatics.com). Other software in the program keeps its own licence, listed in `THIRD_PARTY_NOTICES.txt`
in the program's folder, and data keep their terms, listed in [NOTICE.md](NOTICE.md) and
[docs/data.md](docs/data.md).

Anyone can read and fork what is public here, but only the maintainers can change this repository. If you want a
change, please [open an issue](https://github.com/h2o-geomatics/CLIMo-Beta/issues/new/choose).

## How to cite

Please cite the model description, and this software ([CITATION.cff](CITATION.cff)):

Duguay, C. R., Flato, G. M., Jeffries, M. O., Ménard, P., Morris, K., and Rouse, W. R. (2003). Ice-cover variability
on shallow lakes at high latitudes: model simulations and observations. *Hydrological Processes*, 17(17), 3465–3483.
https://doi.org/10.1002/hyp.1394
