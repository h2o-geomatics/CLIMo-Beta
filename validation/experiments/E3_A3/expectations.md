# E3_A3: expectations written before the run

Written on 29 October 2026, after the option was implemented and unit-tested
and before `validation/compare.py` was run with it. Not edited after the
run; the results and how they compare with these expectations are in
[docs/formulations.md](../../../docs/formulations.md#a3-smooth-melt-state-albedo-transition).

## What is tested

Spec A3: the snow (and, separately, the ice) albedo goes from its cold to its
melting value over an interval of surface temperature below the melting
point, instead of jumping at it. Everything else is legacy CLIMo; the cold
and melting values are legacy's (snow 0.75 / 0.65; ice
max(0.44 h^0.28 + 0.08, 0.05) / min(0.075 h² + 0.15, 0.55, cold value)).

- Legacy (`step`): melting values when T_s ≥ 273.15 K, cold values below.
- `linear`: w = clip((T_s − T_1) / (T_2 − T_1), 0, 1), albedo = (1 − w) cold +
  w melting, with T_2 = 273.15 K and T_1 = T_2 − width.
- `logistic`: the same interval, S-shaped (a logistic going from 5 to 95 %
  across it, rescaled to reach exactly 0 and 1 at its ends).

**T_2 is the melting point, not a temperature above it.** The model never
stores a surface above 273.15 K (a melting surface is held at it, and any
excess melts snow or ice), so a transition centred on 0 °C, e.g. −0.5 to
+0.5 °C, would leave a melting surface only half-way to the melting albedo
and change legacy's melting endpoint. The interval therefore lies entirely
below 0 °C. Consequence: at every surface temperature the A3 albedo is equal
to legacy's or lower (never higher). A3 can only add absorbed short-wave; it
cannot make the surface brighter on a day legacy "flickered" to the melting
value, because legacy's melting days keep the melting value.

Width 1 K (T_1 = −1 °C): the interval of the CCSM3 sea-ice albedo (Briegleb
et al., 2004; CICE/Icepack `ccsm3` albedo, `dT_melt = 1`, checked in the
code). Not tuned to the validation sites. The other literature intervals
(CICE delta-Eddington snow grains 1.5 K; JULES snow 2 K) are covered by the
sensitivity runs.

Runs:

| Folder | Options | Role |
|---|---|---|
| E3_A3 | `melt_albedo_transition=linear` (1 K), snow only | the spec's A3, first experiment |
| E3_A3_ice | snow and ice `linear`, 1 K | "apply the same structure to ice" |
| E3_A3_logistic | snow and ice `logistic`, 1 K | does the shape matter? (against E3_A3_ice) |
| E3_A3_w0.5, E3_A3_w2 | snow only, 0.5 K and 2 K | sensitivity to the interval |
| E3_A3_ice_w0.5, E3_A3_ice_w2 | snow and ice, 0.5 K and 2 K | sensitivity to the interval |
| E3_A3_A2 | snow and ice `linear` 1 K, with A2 `surface_threshold` | A2 made the melting-point jump larger (0.75 ↔ 0.55 on white ice); an extra experiment, not the A3 result |

## How legacy switches now (measured before the run)

From the legacy runs on the frozen forcing (every 6-hour time step with
ice, the seasons the harness scores), made with
`melt_switch_baseline.py` (output
[`melt_switch_baseline.csv`](melt_switch_baseline.csv)). Spring = March to
July. Ranges over the sites and both forcings.

| | 0 % snow | 50 % snow | 100 % snow |
|---|---|---|---|
| spring ice time steps at the melting point | 33–50 % | 32–55 % | 32–60 % |
| within 1 K below it (where A3 with 1 K acts) | 5–13 % | 2–4 % | 2–4 % |
| within 1–2 K below it (added by 2 K) | 2–6 % | 2–5 % | 2–5 % |
| switches cold ↔ melting albedo per spring | 2.6–5.4 | 4.1–7.7 | 4.2–8.1 |
| albedo jump at a switch | 0.12–0.29 (ice) | 0.11–0.14 | 0.11–0.13 |
| time steps per year where the solver's two surface temperatures straddle 0 °C | 0.3–0.8 | 0.9–2.8 | 0.9–2.9 |
| surface residual on those steps / on all others (W m⁻²) | 16–40 / 0.2–0.4 | 8–25 / 0.5–1.6 | 8–26 / 0.5–1.4 |
| largest daily surface residual of the run on such a day | 10 of 12 cases | 7 of 12 | 6 of 12 |

So the switch is not a frequent flicker (2–8 switches per spring, i.e. one
to four excursions below and back), but it is a strong one, and it carries
the run's largest closure error. Two examples from the legacy runs:

- Yellowknife, station, 50 % snow, 13–15 May 1996: bare melting ice
  (albedo 0.28) cools to −0.004 °C; the albedo jumps to the cold value 0.56,
  the surface stays 0.04–0.10 °C below melting for two days with the cold
  albedo, then jumps back. The daily surface residual on 13 May is 14.7
  W m⁻² (−58.7 W m⁻² at the time step where the solver's trial surface was at
  0 °C and the conduction step returned −0.004 °C). A surface a few
  hundredths of a degree below melting gets the full cold albedo: the step
  makes a stable "just-below-melting, bright" state.
- Lac des Sables, station, 50 %: the run's largest residual (59.8 W m⁻², the
  largest of all 36 cases) is on 12 February 1981, a mid-winter thaw, on a
  straddle day.

The largest residuals that are *not* on straddle days are at Baker Lake
(early-winter cold spells, 1980 and 2019) and a few spring days at Gimli and
Big Trout: A3 cannot change those.

First-order size (legacy trajectory, no feedback: A3's albedo at each time
step's own surface temperature, snow and ice): extra short-wave absorbed per
spring

| | snow only (E3_A3) | snow and ice (E3_A3_ice) |
|---|---|---|
| 0 % snow | 0 | 11–26 MJ m⁻² (3.7–8.5 cm of ice) |
| 50 and 100 % snow | 0.9–2.2 MJ m⁻² | 1.6–6.6 MJ m⁻² (0.5–2.2 cm of ice) |

## Expected effect, by metric (candidate minus baseline)

The direction is fixed by the construction (lower albedo near melting, never
higher): more short-wave absorbed in spring, earlier snow-off, earlier
break-up, slightly less ice in spring. The first-order energy is small; the
removal of the "just-below-melting, bright" state (above) adds a feedback:
with A3 a surface at −0.05 °C already has 95 % of the melting albedo, so it
warms to melting instead of staying stuck below it. That could double the
first-order effect, not more.

Spring melt rates in the legacy runs: ice 4–5.5 cm per day; snow at roughly
25 kg m⁻² per day near the end.

| Metric | Expected direction | E3_A3 (snow only) | E3_A3_ice (snow and ice) | Why |
|---|---|---|---|---|
| Break-up (bias) | earlier | 0 at 0 % (identical); −0.1 to −0.8 day with snow, many cases within the 0.5-day tolerance | −1 to −4 days at 0 %; −0.3 to −2 days with snow | First-order energy above, times up to 2 for the feedback. At 0 % snow the ice option acts on bare ice all spring; with snow it acts after snow-off and through the thin-snow blend. |
| Break-up (MAE) | improves where legacy is late, degrades where early | few verdicts (changes mostly under 0.5 day) | improves: Yellowknife (all 6 cases, late +1.8 to +7.3), Baker 0 % (+2.7, +3.1), Lac des Sables 100 % (+3.1, +7.1); degrades: the 26 early cases where the change passes 0.5 day (Big Trout, Gimli, Island, Lac des Sables 0 and 50 %, Baker 50 and 100 %) | Legacy break-up is early in 26 of 36 cases and late in 10. A3 can only move it earlier. |
| Effective break-up | earlier | as break-up | as break-up | Same mechanism. |
| Freeze-up | unchanged | 0 | 0, perhaps +1 day in a few cases | Autumn sun is low; the ice option lowers the albedo of new ice near 0 °C slightly; E2 showed the 10 m mixed layer does not carry a week of earlier ice-off to autumn. |
| Ice thickness (bias, MAE) | slightly thinner | 0 to −0.3 cm | 0 to −1 cm, most at 0 % | The extra energy arrives in spring, after most measurements and the maximum; spring measurements see a little less ice. Most cases are too thick, so the MAE improves slightly where it changes; Baker 50/100 % and Gimli (too thin) degrade slightly. Mostly within the 0.5 cm tolerance. |
| Seasonal maximum | unchanged | under 0.2 cm | under 0.5 cm | The maximum is reached before the near-melting days, except after mid-winter thaws in the south. |
| Snow depth on the ice | slightly less | under 0.3 cm | under 0.3 cm | Earlier spring melt-out; only spring measurements see it. |
| Snow ice, congelation ice | unchanged | | | Formation is a winter process. |
| Closure: largest daily surface residual | improves where it is a straddle day | improves in a few snow cases (only where the straddle is on snow); 0 % identical | improves in most of the 23 cases where it is a straddle day (by a factor of several: the albedo difference between the solver's two temperatures, 0.004–0.1 K apart, becomes a fraction of a percent instead of 0.1–0.3); unchanged where it is not (Baker Lake) | The step-0 diagnosis: the largest residuals are the 0 °C albedo jump. |
| Closure: other | mean daily surface residual slightly lower; mass residuals unchanged | | | No bookkeeping changes. |
| Solver | Newton-Raphson fails (legacy fall-back) somewhat more often, still rare | | | Inside the interval the albedo rises steeply as the surface cools (0.1–0.3 per K): a positive feedback of 30–90 W m⁻² K⁻¹ at spring sun, as large as the restoring terms (long-wave, turbulent, conduction), so the surface balance can have more than one root there. In a test run on the unit-test forcing Newton failed at 11 of 3234 time steps instead of 4; a failure falls back to legacy's starting temperature and can leave a single large residual. So a few cases may show a new largest residual on such a day. |
| Energy terms | FSWout down, FSWin down slightly | under 1 W m⁻² | about −1 W m⁻² in the means over ice time steps | Lower albedo; CLIMo's cloudy-sky short-wave falls with the albedo, and the ice ends in less sunny days. |

**The specification's warning** (break-up improving while winter thickness
degrades) is not expected to fire: break-up should degrade in more cases
than it improves, and the thickness, if anything, improves.

## North against south, and the expected "helps / hurts"

At the matching snow scenario (50 %; 0 % at Baker Lake):

- *helps*: Yellowknife (62.5° N, late +1.8 / +3.6 days) and Baker Lake 0 %
  (64.3° N, late +2.7 / +3.1), both with the ice option;
- *hurts*: Gimli (50.6° N), Big Trout (53.8° N), Island Lake (53.9° N) and
  Lac des Sables 50 % (46.1° N), all early, again mainly with the ice option;
- with snow only (E3_A3), mostly within tolerance everywhere.

As for A2 the deciding factor should be the sign of legacy's break-up error,
not latitude itself: A3 moves break-up the opposite way to A2.

## Logistic and the sensitivity runs (not calibration)

None of these is chosen from the results.

- *Logistic* (E3_A3_logistic, against E3_A3_ice): within ±0.3 day of the
  linear form. The S is flatter near −1 °C and steeper near −0.5 °C; the
  near-melting time steps (most at −0.1 to 0 °C) get w ≈ 0.95–1.0 either way.
  Possibly a few more Newton failures (slope 1.6 times the line's in the
  middle).
- *0.5 K* (E3_A3_w0.5, E3_A3_ice_w0.5): about half to two-thirds of the 1 K
  effect. The "just-below-melting" removal is kept (−0.05 °C is inside
  0.5 K), the time spent in the interval halves; steeper albedo slope, more
  Newton failures.
- *2 K* (E3_A3_w2, E3_A3_ice_w2): about 1.5–2 times the 1 K effect (the
  1–2 K band holds about as many spring time steps as the 0–1 K band);
  gentler slope, fewer Newton failures, the closure improvement kept.

## A3 with A2 (E3_A3_A2)

A2 alone (E2_A2) delayed break-up by +0.4 to +10 days, and 10 of its
largest daily surface residuals got worse at the 0.75 ↔ 0.55 white-ice
jump. With A3 on snow and ice the white-ice albedo also goes smoothly from
0.75 to 0.55 over the last kelvin. Expected against E2_A2: break-up 0.5–2
days earlier (more where there is more white ice), so slightly less of A2's
overshoot at Lac des Sables 100 % and Island 100 % and slightly less of its
help at Big Trout and Gimli; the largest daily surface residuals of the
cases E2 flagged better than in E2_A2. Against legacy (what the harness
reports): still mostly A2's pattern.

## What cannot be checked here

A3 changes the albedo as a function of surface temperature. There are no
albedo measurements and no surface-temperature measurements at the six
validation sites, so E3 can show only the knock-on effect on dates and
thickness and the closure, not whether the albedo near melting or the
surface temperature is realistic. The spec asks to calibrate the interval
against observed surface temperature and albedo: not possible here.
