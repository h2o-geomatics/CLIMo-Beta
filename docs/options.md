# Model options

CLIMo Classic reproduces legacy CLIMo exactly: **with every option at its
default, a run is bit-identical to the legacy model** (regression test,
validation experiment E0). The options below are switched on one by one, on
the command line, in a sites file, in the GUI or in Python. One of them,
`fixes`, corrects numerical faults of the legacy model; the others replace a
single legacy equation with new physics.

This page is the practical guide: what each option does, how to turn it on,
where it helped and where it hurt at the six validation lakes, and what it is
most likely useful for. The full science record (sources, equations,
diagnosis, every validation experiment) is in
[formulations.md](formulations.md), linked from each section.

**Read every "helps / hurts" below with two cautions.** The options were
judged at six Canadian lakes (46–64° N), on one frozen set of forcing
(station and ERA5) and three snow scenarios (0, 50 and 100 % of snowfall on
the ice), against ECCC ice thickness and ice dates
([validation.md](validation.md)). None of the sites has albedo, surface
temperature or snow-density measurements, so the albedo and density options
are judged only by their knock-on effect on thickness and dates. And none of
the parameter values was tuned to these results.

## Overview

| Option | Type | Default | Effect at the six lakes | Recommended use |
|---|---|---|---|---|
| [`fixes`](#fixes) | correction of legacy numerics (the surface-temperature solver; legacy notes 4, 5, 11) | `off` | dates and thickness: break-up within 0.33 day, thickness within 6 mm (one freeze-up season moved 6 days); energy closure: mean daily surface residual 0.65 → 0.02 W m⁻², largest 59.8 → 24.6 W m⁻² | **any new work**; always with A3 and S1 |
| [`thin_snow_albedo`](#a1-thin-snow-albedo) (A1) | new physics: literature form, engineering proposal for CLIMo | `legacy` | small: break-up 0.2–0.9 day later, ice 0.1–0.8 cm thicker (snow cases only) | optional; physically sounder for snow under a few cm, too small to matter for dates |
| [`ice_type_albedo`](#a2-ice-type-albedo) (A2) | new physics: literature-backed premise, engineering proposal in form and values | `legacy` | large: break-up 0.4–10 days later (snow cases), most where the model makes most snow ice; nothing at 0 % snow | lakes where legacy breaks up early and the ice has much white ice (here 50–54° N) |
| [`melt_albedo_transition`, `melt_albedo_transition_ice`](#a3-melt-albedo-transition) (A3) | new physics: continuous transition used in other models; the interval is an engineering proposal | `step` | snow only: under 0.5 day; snow and ice: break-up 0.1–1.4 days earlier; removes the melting-point energy residuals (with `fixes`) | energy-budget work; lakes where legacy breaks up late; snow and ice, 2 K, with `fixes` |
| [`snow_density_scheme`](#s1-snow-density) (S1) | new physics: scheme from land-snow models, engineering proposal for CLIMo | `legacy` | defaults: nothing beyond 2.6 mm and 0.22 day; lighter fresh snow (150–200 kg m⁻³): ice 4–11 cm thinner, break-up 1–3 days earlier at every lake | sensitivity studies, with `fixes`; do not tune it alone |

## Recommended combinations

| Purpose | Options | Why |
|---|---|---|
| **Classic reproduction**: compare with published CLIMo results or the legacy code | all defaults | bit-identical to legacy CLIMo |
| **Recommended for new work** | `fixes on` | corrects the solver without changing the physics; observed quantities move by at most a few millimetres and a fraction of a day, the energy budget closes |
| Energy-budget analysis (surface fluxes, closure) | `fixes on` + A3 on snow and ice, `linear`, 2 K | the largest daily surface residual falls to a few W m⁻² (4.2 at 1 K; 6.5 at 2 K, measured with F1 + F2); break-up moves 0.1–1.4 days earlier |
| A lake where legacy breaks up clearly late (here Yellowknife; Baker Lake at 0 % snow) | `fixes on` + A3 on snow and ice, 2 K | A3 can only advance break-up; small effect (at most 1.4 days) |
| A lake where legacy breaks up clearly early (more than about 5 days) and the ice is known to have much white (snow) ice | `fixes on` + A2 `surface_threshold`; bracket with `--snow-ice-albedo 0.46 --snow-ice-albedo-melt 0.36` | A2 can only delay break-up; the size depends mostly on the melting snow-ice albedo |
| Snow-density sensitivity | `fixes on` + S1 (preferably + A3 on snow and ice) | lighter packs make the legacy solver fail more; S1's fresh-snow density is to be decided together with the snow mass conversion and conductivity (spec S2, S3), not tuned |
| Thin-snow albedo | `fixes on` + A1 | sounder for very thin snow; changes dates by under a day |

Run legacy first and compare with observations before choosing A2 or A3:
whether they help depends on the sign of legacy's own break-up error at your
lake, not on latitude.

## How to turn an option on

Every option has the same name everywhere: a switch (e.g. `ice_type_albedo`)
and, for some values, parameters (e.g. `snow_ice_albedo`).

| Where | How |
|---|---|
| Command line, `climo run` and `climo batch` | `--fixes on`, `--ice-type-albedo surface_threshold --snow-ice-albedo 0.71`, ... (underscores become hyphens); `climo run --help` lists them |
| Sites file of `climo batch` | a column per switch and parameter, e.g. `fixes`, `ice_type_albedo`, `snow_ice_albedo`; empty cells take the command-line values ([workflow.md](workflow.md#the-sites-csv)) |
| GUI, Model tab | the **Model options** box: the Fixes box, a list per switch, and an entry per parameter, enabled when the chosen value uses it ([gui.md](gui.md#model)) |
| Python | `Parameters(..., fixes="on", ice_type_albedo="surface_threshold")`; `run_scenarios`, `run_grid` and `run_sites` take `formulations={"fixes": "on", ...}` |

A parameter given without a switch value that uses it stops a command-line
run. The run's log lists the options in use; gridded results record them in
the netCDF attributes, batch runs in `batch.json`.

## Fixes

**What it does.** At every time step with ice, CLIMo solves for the surface
temperature at which the surface energy budget and the heat conduction
agree. The legacy solver has three faults, which `fixes on` corrects
together:

- **F1** ([legacy note 5](legacy_notes.md)): when Newton–Raphson fails and
  the solution lies *below* the starting temperature, legacy finds it by
  bisection and then throws it away, keeping the starting temperature (most
  likely a slip in the translation from Fortran). Fixed: the solution is
  returned, as it already is above the start.
- **F2** ([legacy note 4](legacy_notes.md)): when Newton–Raphson converges
  on its step size (about one ice time step in seven), legacy keeps the
  calculation made 0.1 K below the solution, about 0.3 K out of balance;
  when the slope is too flat it keeps a calculation that can be several
  kelvin out. Fixed: the converged temperature is used, and a flat slope
  goes to the bisection.
- **F3** ([legacy note 11](legacy_notes.md)): the Newton–Raphson step is
  unbounded, so at the melting point it can jump hundreds of kelvin, and
  once settled on −72.5 K. Fixed: a step longer than 20 K is shortened to
  20 K.

**Type:** a correction of legacy numerics, not new physics: no equation of
the model changes, and there is no parameter to choose (the 20 K limit is
fixed). It is off by default only because CLIMo Classic reproduces legacy
CLIMo exactly by default.

**How to turn it on:** `--fixes on`; sites column `fixes` (`off` or `on`);
GUI Model tab, **Fixes**; Python `Parameters(..., fixes="on")`.

**Where it helps and where it hurts** (largest daily surface energy
residual over the lake's 6 cases; break-up bias at 50 % snow, Baker Lake at
0 %, its best-matching scenario, station / ERA5 forcing; from the
`comparison.csv` of `validation/experiments/FIXES`, `FIXES_A3` and
`E3_A3_ice`, identical to the F1–F3
results in [formulations.md](formulations.md#the-fixes-option-f1--f2--f3)):

| Lake | Latitude | Legacy fall-backs per year | Largest daily residual, legacy → fixes (W m⁻²) | With A3 (snow and ice, 1 K): A3 → A3 + fixes | Break-up bias, legacy → fixes (days) | Verdict |
|---|---|---|---|---|---|---|
| Lac des Sables | 46.1° N | 1.07 | 59.8 → 22.2 | 32.5 → 2.3 | −1.6 / −4.5 → −1.5 / −4.5 | helps most (mid-winter thaws) |
| Gimli (Lake Winnipeg) | 50.6° N | 0.33 | 24.0 → 18.9 | 28.5 → 2.9 | −10.3 / −12.3 → −10.2 / −12.2 | helps |
| Big Trout Lake | 53.8° N | 0.40 | 16.0 → 12.6 | 23.4 → 2.1 | −12.1 / −12.4 → −12.0 / −12.4 | helps |
| Island Lake | 53.9° N | 0.22 | 19.2 → 14.0 | 17.0 → 1.3 | −6.7 / −5.6 → −6.6 / −5.6 | helps |
| Yellowknife (Back Bay) | 62.5° N | 0.26 | 14.7 → 24.6 | 25.5 → 1.4 | +3.6 / +1.8 → +3.6 / +1.8 | hurts alone (3 cases: legacy's albedo jump at 0 °C); helps with A3 |
| Baker Lake | 64.3° N | 0.28 | 32.2 → 17.6 | 32.8 → 4.2 | +3.1 / +2.7 → +3.1 / +2.7 | helps |

The mean daily surface residual falls at every lake, from 0.5–1.2 to
0.02–0.03 W m⁻². Fall-backs to the starting temperature disappear. Where it
helps most is set by how often the surface cools sharply from 0 °C (thaws,
then cold), not by latitude. The only cost: with legacy's step albedo at
0 °C, a few spring steps now land on the warm side of the jump instead of
the cold side, so at Yellowknife the largest single day gets worse; A3
removes that jump.

**Most likely application:** any new work. Use it always with A3 (without
it, A3's continuous albedo makes the legacy solver fail more often and leave
large single-day errors) and with S1 (lighter snow packs exposed the
unbounded Newton step). Leave it off only to reproduce legacy CLIMo.

**Cautions.** It is judged by energy closure, not by observations: it barely
changes observed quantities (at most 5.6 mm of thickness and 0.33 day of
break-up; one freeze-up season moved 6 days, a threshold effect on thin
early ice).

**Advanced: the three corrections one by one.** `Parameters` keeps them as
separate fields, `surface_solver` (`legacy`, `fixed`), `newton_acceptance`
(`legacy`, `final_estimate`, `fixed`) and `newton_step` (`legacy`,
`limited`, with `newton_step_limit`), so that the validation harness can
reproduce the experiments that judged them separately (F1_solver,
F2_newton_F1, F3_F1F2, ...). They are not on the command line, in sites
files or in the GUI. `fixes="on"` sets them to `fixed`, `fixed` and
`limited` at 20 K; with it, leave them at their defaults (or at those
values): any other value raises an error. See
[formulations.md](formulations.md#the-fixes-option-f1--f2--f3).

## A1: thin-snow albedo

**What it does.** Legacy blends the albedo of snow thinner than 0.1 m along
a straight line from the ice albedo to the snow albedo, so 2 cm of fresh
snow counts as 80 % ice. A1 uses an exponential blend instead (e-folding
depth `thin_snow_albedo_depth`, 0.03 m), so a few centimetres of snow are
already nearly as bright as deep snow.

**Type:** new physics. The exponential form is from the literature (glacier
measurements, sea-ice models); using it in CLIMo, and the 0.03 m depth, is
an engineering proposal. Details:
[formulations.md, A1](formulations.md#a1-thin-snow-albedo).

**How to turn it on:** `--thin-snow-albedo exponential` (and
`--thin-snow-albedo-depth M`); sites columns `thin_snow_albedo`,
`thin_snow_albedo_depth`; GUI **Thin-snow albedo**; Python
`Parameters(..., thin_snow_albedo="exponential")`.

**Where it helps and where it hurts** (break-up bias in days, negative =
CLIMo too early; station / ERA5 forcing, 50 % snow; from
`validation/experiments/E1_A1/comparison.csv`):

| Lake | Latitude | Legacy | A1 | Verdict |
|---|---|---|---|---|
| Lac des Sables | 46.1° N | −1.6 / −4.5 | −1.0 / −3.6 | helps slightly; ice thickest change (+0.55 cm), seasonal maximum degrades at 100 % |
| Gimli (Lake Winnipeg) | 50.6° N | −10.3 / −12.3 | −9.5 / −12.0 | helps slightly (still 10–12 days early) |
| Big Trout Lake | 53.8° N | −12.1 / −12.4 | −11.4 / −12.2 | helps slightly |
| Island Lake | 53.9° N | −6.7 / −5.6 | −6.3 / −5.1 | helps slightly |
| Yellowknife (Back Bay) | 62.5° N | +3.6 / +1.8 | +3.8 / +2.3 | hurts slightly (already late) |
| Baker Lake | 64.3° N | −6.1 / −5.6 | −5.9 / −5.1 | helps slightly at 50 %; nothing at 0 %, its best scenario |

A1 delays break-up a little wherever there is snow (0.2–0.9 day), which
helps where legacy is early and hurts where it is late; the ice grows
0.1–0.8 cm thicker, most in the south, where it is already too thick. The
changes are a few per cent of the errors. The sign of the effect depends on
the depth: 0.06 m makes break-up earlier, not later.

**Most likely application:** studies where very thin snow on the ice matters
(early winter, light spring snowfalls), as the physically sounder blend.
For dates and thickness it makes little difference.

**Cautions:** the depth comes from a glacier and sea-ice models, not lake
snow; there are no albedo measurements at the sites to test it.

## A2: ice-type albedo

**What it does.** Legacy gives snow-free ice the albedo of clear Arctic ice
whatever the ice is made of. Lake ice often has white snow ice on top
(frozen slush), which reflects much more sunlight. With
`surface_threshold`, ice with more than `snow_ice_albedo_thickness`
(0.05 m) of snow ice at the top takes the snow-ice albedo (`snow_ice_albedo`
0.75 cold, `snow_ice_albedo_melt` 0.55 melting); `thickness_weighted` blends
by the snow-ice share of the ice instead (a much weaker effect). Break-up
comes later.

**Type:** new physics. That white ice is brighter, and that brighter ice
fixed CLIMo's early ice-off at two Ontario lakes, is in the literature
(Robinson et al., 2021); letting CLIMo's modelled snow ice set the albedo,
the threshold and the use of those two lakes' values everywhere are an
engineering proposal. Details:
[formulations.md, A2](formulations.md#a2-clear-ice-versus-snow-ice-albedo).

**How to turn it on:** `--ice-type-albedo surface_threshold` (or
`thickness_weighted`), with `--snow-ice-albedo`, `--snow-ice-albedo-melt`,
`--snow-ice-albedo-thickness`; sites columns `ice_type_albedo`,
`snow_ice_albedo`, `snow_ice_albedo_melt`, `snow_ice_albedo_thickness`; GUI
**Ice-type albedo**; Python `Parameters(..., ice_type_albedo="surface_threshold")`.

**Where it helps and where it hurts** (break-up bias in days, negative =
CLIMo too early; `surface_threshold`, station / ERA5 forcing, 50 % snow;
from [formulations.md](formulations.md#where-it-helps-and-where-it-hurts)):

| Lake | Latitude | Legacy | A2 | Break-up MAE, legacy → A2 | Verdict |
|---|---|---|---|---|---|
| Lac des Sables | 46.1° N | −1.6 / −4.5 | +5.6 / +1.8 | 3.8 → 7.7 / 4.9 → 4.8 | hurts: overshoots (at 100 %: +17 / +12 days late) |
| Gimli (Lake Winnipeg) | 50.6° N | −10.3 / −12.3 | −8.1 / −11.1 | 10.3 → 8.1 / 12.3 → 11.1 | helps a little (at 100 %: −0.9 / −5.9) |
| Big Trout Lake | 53.8° N | −12.1 / −12.4 | −8.8 / −9.7 | 12.3 → 9.4 / 12.5 → 10.0 | helps |
| Island Lake | 53.9° N | −6.7 / −5.6 | −2.9 / −2.7 | 7.3 → 5.9 / 6.6 → 5.0 | helps (overshoots at 100 %: +4.5 / +4.8) |
| Yellowknife (Back Bay) | 62.5° N | +3.6 / +1.8 | +5.5 / +2.3 | 3.8 → 5.5 / 2.5 → 2.9 | hurts (already late) |
| Baker Lake | 64.3° N | −6.1 / −5.6 | −5.6 / −3.8 | 7.9 → 7.6 / 10.2 → 9.1 | small help at 50 %; none at 0 %, its best scenario |

**Why:** A2 can only make break-up later, and the delay grows with the
amount of snow ice the run makes (more snow on the ice, more delay). So it
helps where legacy breaks up early (Gimli, Big Trout, Island Lake) and hurts
where legacy is already close or late (Lac des Sables, Yellowknife). This is
a rule about the model's error, not about latitude: Robinson et al.'s
Ontario lakes at 45° N improved with brighter ice. Winter thickness grows by
up to 1.8 cm, at lakes where it is already too thick.

**Most likely application:** lakes where legacy CLIMo breaks up
consistently more than about 5 days early and the ice is known to have much
white ice (heavy snow, slushing); in our set, the 50–54° N lakes. Bracket
the result with the lowest literature albedos (`--snow-ice-albedo 0.46
--snow-ice-albedo-melt 0.36`), which give a quarter (south) to a tenth
(north) of the delay.

**Cautions:** the snow ice CLIMo makes is not checked against observations
(it is likely too much in the High Arctic); the result depends mostly on
the melting snow-ice albedo, measured on five days at two lakes. A2 makes
the albedo jump at 0 °C larger; use it with `fixes`, and consider A3 on the
ice (together they take about half a day off A2's delay).

## A3: melt-albedo transition

**What it does.** Legacy switches the snow and ice albedo from the cold to
the melting value in one step, the moment the surface reaches 0 °C (snow
0.75 → 0.65; 1 m of bare ice 0.52 → 0.225). A3 makes the change continuous
over a width below 0 °C (`linear` or `logistic`; width 1 K by default),
for the snow (`melt_albedo_transition`) and, separately, for the ice
(`melt_albedo_transition_ice`). The albedo near melting is a little lower,
so break-up comes a little earlier, and the energy residuals that the jump
caused disappear.

**Type:** new physics. Snow and ice darkening towards melting, and
continuous temperature-based albedos, are in the literature and other
models (CCSM3/CICE, FLake, JULES); using it in CLIMo, the 1 K width (from a
sea-ice model) and applying it to the ice are an engineering proposal.
Details: [formulations.md, A3](formulations.md#a3-smooth-melt-state-albedo-transition).

**How to turn it on:** `--melt-albedo-transition linear
--melt-albedo-transition-ice linear` (widths `--melt-albedo-transition-width`,
`--melt-albedo-transition-ice-width`, K); sites columns
`melt_albedo_transition`, `melt_albedo_transition_ice` and their `_width`
columns; GUI **Snow melt-albedo transition**, **Ice melt-albedo
transition**; Python `Parameters(..., melt_albedo_transition="linear",
melt_albedo_transition_ice="linear")`.

**Where it helps and where it hurts** (break-up bias in days, negative =
CLIMo too early; 1 K, station / ERA5 forcing, 50 % snow, Baker Lake at 0 %;
from [formulations.md](formulations.md#where-a3-helps-and-where-it-hurts)):

| Lake | Latitude | Legacy | A3 snow | A3 snow and ice | Verdict |
|---|---|---|---|---|---|
| Lac des Sables | 46.1° N | −1.6 / −4.5 | −1.9 / −4.8 | −1.9 / −5.1 | hurts slightly (at 100 %, late: helps slightly) |
| Gimli (Lake Winnipeg) | 50.6° N | −10.3 / −12.3 | −10.6 / −12.6 | −10.6 / −12.8 | hurts slightly |
| Big Trout Lake | 53.8° N | −12.1 / −12.4 | −12.1 / −12.9 | −12.2 / −13.1 | hurts slightly |
| Island Lake | 53.9° N | −6.7 / −5.6 | −6.9 / −6.0 | −7.3 / −6.1 | hurts slightly |
| Yellowknife (Back Bay) | 62.5° N | +3.6 / +1.8 | +3.2 / +1.7 | +3.1 / +1.6 | helps slightly |
| Baker Lake, 0 % | 64.3° N | +3.1 / +2.7 | +3.1 / +2.7 | +2.0 / +2.3 | helps (at 50 %, early: hurts slightly) |

**Why:** A3 can only lower the albedo, so it can only advance break-up: it
helps where legacy breaks up late (Yellowknife, Baker Lake at 0 %, Lac des
Sables at 100 %) and hurts where legacy is early. The effect is largest with
no snow and with the ice switch, smallest for snow only (under half a day).
The dates barely depend on the width (2 K gives 110–130 % of the 1 K change)
or the shape (logistic within 0.1 day of linear).

**Energy closure:** with `fixes`, A3 on snow and ice brings the largest
daily surface residual down to 1.3–4.2 W m⁻² at every lake (1 K; 6.5 W m⁻²
over all cases at 2 K, measured with F1 + F2). Without `fixes`, A3 makes the
legacy solver fail more often (at 1 K) and the closure is mixed.

**Most likely application:** energy-budget analyses (it removes the 0 °C
albedo jump that sets the largest residuals), and lakes where legacy breaks
up late. Use it on snow and ice, at 2 K (the width the solver handles best),
with `fixes`. It is not a remedy for early break-up.

**Cautions:** the width cannot be checked without albedo and surface
temperature measurements; a temperature-based albedo has no memory (a cold
night restores the cold albedo).

## S1: snow density

**What it does.** Legacy gives all snow on the ice one of two densities at
every time step: `rho_snow_cold` (330 kg m⁻³), or `rho_snow_warm` (450)
when the mean snow temperature reaches 0 °C, which almost never happens.
S1 (`bulk_relaxation`) makes the density a state that carries over: new
snow is mixed in at `snow_density_fresh`, and the pack then relaxes towards
330 kg m⁻³ (rising towards 450 near melting, within
`snow_density_transition_width`) with an e-folding time
`snow_density_timescale` (100 h); it never gets lighter without new snow.
The snow mass is unchanged; only its depth, conductivity and flooding
change.

**Type:** new physics. The relaxation scheme is that of land-surface snow
models (CLASS, ISBA, FSM); using it in CLIMo, with CLIMo's two densities as
equilibria and the 1 K band, is an engineering proposal. Details:
[formulations.md, S1](formulations.md#s1-bulk-snow-density-evolution).

**How to turn it on:** `--snow-density-scheme bulk_relaxation` (with
`--snow-density-fresh`, `--snow-density-timescale`,
`--snow-density-transition-width`); sites columns `snow_density_scheme`,
`snow_density_fresh`, `snow_density_timescale`,
`snow_density_transition_width`; GUI **Snow density**; Python
`Parameters(..., snow_density_scheme="bulk_relaxation")`. Not the same as
`--snow-density` of `climo data`, which converts ERA5 snowfall.

**Where it helps and where it hurts.** With its defaults (fresh snow at
330 kg m⁻³) S1 changes nothing beyond tolerance at any lake (at most 2.6 mm
of thickness, 0.22 day of break-up). With lighter fresh snow, a sensitivity
(fresh 150 kg m⁻³, with the solver fixes; legacy bias at 50 % snow, change
by S1 averaged over 50 and 100 %; from
[formulations.md](formulations.md#where-s1-helps-and-where-it-hurts)):

| Lake | Latitude | Legacy bias: seasonal maximum / break-up | Change: maximum / break-up | Verdict |
|---|---|---|---|---|
| Lac des Sables | 46.1° N | +14 cm / −3.1 d | −10.2 cm / −2.9 d | helps thickness (too thick), hurts break-up (early) |
| Gimli (Lake Winnipeg) | 50.6° N | +2 cm / −11.3 d | −9.3 cm / −2.2 d | hurts (break-up already 11 days early) |
| Big Trout Lake | 53.8° N | 0 cm / −12.3 d | −10.5 cm / −2.5 d | hurts |
| Island Lake | 53.9° N | +23 cm / −6.1 d | −10.5 cm / −2.6 d | mixed |
| Yellowknife (Back Bay) | 62.5° N | +10 cm / +2.7 d | −9.4 cm / −2.1 d | helps (ice too thick, break-up late) |
| Baker Lake | 64.3° N | −51 cm / −5.8 d | −7.9 cm / −2.0 d | hurts (ice already too thin with snow) |

**Why:** lighter snow insulates more, so the ice grows thinner and breaks up
earlier, by about the same amount everywhere. Whether that helps depends
only on legacy's own bias at each lake. This is the pattern of a uniform
insulation knob, which the snow-percentage scenarios can already produce.

**Most likely application:** sensitivity studies of snow insulation, with
`fixes` (and preferably A3 on snow and ice). The defaults are safe but do
almost nothing.

**Cautions:** do not tune `snow_density_fresh` against thickness or dates
on its own; decide it together with the conversion of snowfall to snow mass
and the snow conductivity (spec S3 and S2, planned), and against on-ice
snow-density measurements where they exist. Without `fixes`, light packs
make the legacy solver fail (once −72.5 K). The relaxation constants come
from land snow, not lake snow.

## Combining options

Options combine freely; each replaces a different equation. Combinations
validated so far: `fixes` with A3 and with S1 (the F3 experiments), A2 with
A3 (E3_A3_A2: A3 takes about half a day off A2's delay). The albedo package
A1 + A2 + A3 is planned. A combination is not
the sum of its parts where the options act on the same days: judge it with
the validation harness ([validation.md](validation.md#experiment-harness-judging-the-step-3-options)),
e.g.

```bash
python validation/compare.py --name MY_RUN --option fixes=on --option ice_type_albedo=surface_threshold
```
