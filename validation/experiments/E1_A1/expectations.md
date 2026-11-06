# E1_A1: expectations written before the run

Written on 29 October 2026, after the option was implemented and before
`validation/compare.py` was run with it. Not edited after the run; the
results and how they compare with these expectations are in
[docs/formulations.md](../../../docs/formulations.md#a1-thin-snow-albedo).

## What is tested

Spec A1 alone: `thin_snow_albedo=exponential` with
`thin_snow_albedo_depth=0.03` (m). Everything else is legacy CLIMo.

- Legacy: albedo = α_i + (α_s − α_i) · h_s / 0.1 for h_s ≤ 0.1 m, α_s above.
- A1: albedo = α_i + (α_s − α_i) · [1 − exp(−h_s / 0.03)].

With 0.03 m the snow share of the albedo is 49 % at 2 cm of snow (legacy
20 %), 81 % at 5 cm (legacy 50 %), 96 % at 10 cm (legacy 100 %), 99.3 % at
15 cm. Averaged over 0–10 cm the snow share is 71 % against 50 %. So A1
makes snow thinner than about 9.5 cm brighter than legacy, and snow from
9.5 to about 20 cm very slightly darker (at most 3.6 % of α_s − α_i, about
0.016 in albedo, at 10 cm).

The depth 0.03 m was set from the literature and from legacy's own 0.1 m
before any run (see docs/formulations.md); it is not tuned to these sites.

## How much thin snow the validation runs see (legacy baseline, measured before the run)

From the legacy runs on the frozen forcing (end-of-day values):

- 0 % snow: there is never snow on the ice after the spin-up year, so A1
  changes nothing except through the 5 cm of initial snow in the spin-up
  year (see below).
- 50 % and 100 % snow: 24–38 ice-covered days per year have snow between
  1 mm and 10 cm. Nearly all of them are in early winter (October to
  February), when the daily mean short-wave is low: 10–23 W m⁻² at
  Yellowknife and Baker Lake, 28–32 W m⁻² at Island and Big Trout Lakes,
  41 W m⁻² at Gimli, 58 W m⁻² at Lac des Sables. Only 2–8 days per year are
  in spring (March to June), when the snow melts through the last 10 cm, but
  then the short-wave is 210–280 W m⁻².

## Expected effect, by metric (candidate minus baseline)

Rough sizes, from the extra reflected short-wave: albedo higher by about
0.065 (cold snow) to 0.08 (melting snow) on thin-snow days, times the
short-wave above.

| Metric | Expected direction | Expected size | Why |
|---|---|---|---|
| Break-up (bias) | later | +0.2 to +1 day in the 50 % and 100 % cases; 0 at 0 % | In spring the last 10 cm of snow stays brighter for 2–8 days: about 20 W m⁻² less absorbed, some 5 MJ m⁻², the energy to melt about 1.5 cm of ice or 15 kg m⁻² of snow. Snow-off, and then ice-off, come a little later. |
| Break-up (MAE) | improves where the baseline is early, degrades where it is late | mostly within or just above the 0.5-day tolerance | Baseline break-up is early (negative bias) in 18 of the 24 snow cases (Baker 50/100 %, Big Trout, Gimli, Island, Lac des Sables 50 %) and late in 6 (Yellowknife, Lac des Sables 100 %). |
| Effective break-up (ice < 0.1 m) | later | as break-up | Same mechanism. |
| Freeze-up | unchanged | below 0.5 day | No snow on open water; the albedo does not act. A later break-up shortens the summer heating slightly, which could make freeze-up a fraction of a day earlier. |
| Ice thickness (MAE, bias) | slightly thicker | 0 to +1 cm; largest at the southern sites; mostly within the 0.005 m tolerance | Brighter thin snow in early winter absorbs less short-wave (about 2–4 W m⁻² over a month at the southern sites, under 2 W m⁻² in the north): a colder surface and a little more growth. Spring measurements may also see ice that lasts slightly longer. Thicker ice degrades the many cases already too thick (Yellowknife, Island Lake, Lac des Sables, Big Trout 100 %) and improves Baker Lake (too thin), but the changes should mostly count as unchanged. |
| Seasonal maximum | slightly thicker | 0 to +1 cm | Same early-winter effect. |
| Snow depth on the ice | unchanged | below 0.5 cm | Snow mass does not depend on the albedo until spring melt; only the last days of snow in spring change. |
| Snow ice, congelation ice | congelation slightly up, snow ice unchanged | below 1 cm | Snow ice depends on the snow load, which does not change; congelation growth follows the colder early-winter surface. |
| Closure | unchanged | | A1 changes no mass or energy bookkeeping and is continuous in h_s. The largest daily surface residual is set by the albedo jump at the melting point (spec A3), which A1 does not touch; it may move to another day, so a few "largest daily" case-checks may pass the 1 % tolerance either way by chance. |
| Energy terms | FSWout up, PenSW down | under 1 W m⁻² in the means | More short-wave reflected while snow is thin. FSWin also changes slightly, because CLIMo's cloudy-sky short-wave depends on the albedo (multiple reflection, Shine, 1984). |

**0 % snow cases.** The spin-up year starts with 5 cm of snow on 1 m of
ice (`INITIAL_SNOW`). With A1 that snow is brighter (81 % instead of 50 %
of the snow value), so in the spin-up spring it may last a little longer;
this can shift the first season's freeze-up slightly through the mixed
layer's summer heat. Expected: unchanged at the 0.5-day and 0.005 m
tolerances.

**The specification's warning** (break-up improving while winter thickness
degrades). Possible in principle here: the break-up is expected to improve
in more cases than it degrades, and thickness to move, if at all, towards
thicker ice where the model is already too thick. But both changes should
be small; the warning is not expected to fire unless the thickness changes
are larger than estimated.

## Sensitivity runs (not calibration)

`thin_snow_albedo_depth` 0.01, 0.02 and 0.06 m, in E1_A1_h0.01,
E1_A1_h0.02 and E1_A1_h0.06, to show how much the result depends on the
depth. None of them is chosen from the results.

- 0.01 m: thin snow almost as bright as thick snow (86 % at 2 cm): the
  largest effect, in the same direction as 0.03 m, perhaps 1.5 times as
  large.
- 0.02 m: between 0.01 and 0.03 m.
- 0.06 m: the same mean albedo as legacy over 0–10 cm, brighter below
  4 cm and darker from 7 to 20 cm (81 % at 10 cm, 92 % at 15 cm). Expected
  close to legacy, perhaps a slightly earlier break-up, because in spring
  the snow from 10 to 20 cm is now darker.

## What cannot be checked here

A1's direct target is the surface albedo. There are no albedo
measurements at the six validation sites, so E1 can only show the indirect
effect on ice thickness and dates, not whether the albedo of thin snow is
more realistic.
