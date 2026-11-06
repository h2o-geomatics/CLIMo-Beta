# E2_A2: expectations written before the run

Written on 29 October 2026, after the option was implemented and before
`validation/compare.py` was run with it. Not edited after the run; the
results and how they compare with these expectations are in
[docs/formulations.md](../../../docs/formulations.md#a2-clear-ice-versus-snow-ice-albedo).

## What is tested

Spec A2 alone: `ice_type_albedo=surface_threshold` with
`snow_ice_albedo=0.75` (cold), `snow_ice_albedo_melt=0.55` (melting) and
`snow_ice_albedo_thickness=0.05` m. Everything else is legacy CLIMo.

- Legacy: one ice albedo, a function of ice thickness only: cold
  0.44 h^0.28 + 0.08 (0.52 at 1 m), melting min(0.075 h² + 0.15, 0.55)
  (0.15–0.225 below 1 m). Then the thin-snow blend towards the snow albedo.
- A2: when more than 0.05 m of snow ice is at the top of the ice, the ice
  albedo is 0.75 (cold) or 0.55 (melting); otherwise legacy's value (the
  clear-ice albedo). The thin-snow blend and everything after it are
  unchanged.

The values were set from the literature before any run (Robinson et al.,
2021, measured on white ice at two Ontario lakes that are not validation
sites; the thickness from optics; see docs/formulations.md). They are not
tuned to these sites.

"Snow ice at the top" is the new diagnostic `State.surface_snow_ice`: snow
ice formed this season minus the ice melted at the top (and inside the snow
ice layer), never more than the ice thickness. It is not legacy's `SnowIce`
output, which is ice grown and never decreases.

A second run, E2_A2_weighted, tests `ice_type_albedo=thickness_weighted`
(same albedos): albedo = (1 − f) α_ci + f α_si, f = snow ice at the top /
ice thickness.

## How much snow ice the legacy runs have (measured before the run)

From the legacy runs on the frozen forcing (end-of-day values, the seasons
the harness scores), made with
`snow_ice_baseline.py` (output
[`snow_ice_baseline.csv`](snow_ice_baseline.csv)):

| Site | Snow | Snow ice grown, seasonal max (m) | Share of max ice | Snow ice at the top when the snow is gone in spring (m) | Days per year with snow < 0.1 m and > 0.05 m of snow ice at the top: winter / spring (Mar–Jul) |
|---|---|---|---|---|---|
| Yellowknife | 50 % | 0.06–0.14 | 4–10 % | 0.04–0.11 | 0.1–0.6 / 1.6–5.0 |
| Yellowknife | 100 % | 0.29–0.46 | 21–31 % | 0.25–0.39 | 4–5 / 9–13 |
| Baker Lake | 50 % | 0.08–0.15 | 5–9 % | 0.06–0.12 | 0.1–1.0 / 1.8–4.3 |
| Baker Lake | 100 % | 0.36–0.53 | 21–30 % | 0.30–0.45 | 5–6 / 9–12 |
| Island Lake | 50 % | 0.16–0.23 | 15–19 % | 0.13–0.19 | 1.6–2.5 / 7–9 |
| Island Lake | 100 % | 0.49–0.61 | 38–46 % | 0.40–0.48 | 6–7 / 14–17 |
| Big Trout Lake | 50 % | 0.18–0.24 | 15–20 % | 0.15–0.19 | 3–7 / 7–9 |
| Big Trout Lake | 100 % | 0.52–0.64 | 39–46 % | 0.44–0.52 | 5–8 / 14–16 |
| Gimli | 50 % | 0.09–0.14 | 9–14 % | 0.07–0.11 | 0–1.4 / 4–5 |
| Gimli | 100 % | 0.33–0.43 | 30–37 % | 0.26–0.34 | 2–3 / 9–12 |
| Lac des Sables | 50 % | 0.39–0.50 | 44–53 % | 0.31–0.39 | 9–10 / 17 |
| Lac des Sables | 100 % | 0.91–1.14 | 78–88 % | 0.67–0.84 | 11–12 / 21–23 |

(ranges: station and ERA5 forcing). At 0 % snow there is no snow ice at all,
so A2 cannot act. For comparison, Small Lake (High Arctic) had under 4–7 %
white ice and the Haliburton lakes 25–73 % (Robinson et al., 2021, p. 2).

Most snow ice is still at the top when the spring snow is gone: the snow
melts first, and little ice has melted at the top by then. Ice-off follows
20–30 days later. In the legacy runs the spring days with exposed white ice
are mostly at the melting point (the albedo then is 0.28–0.49 on average,
including the days with a little snow), with daily short-wave of 210–290
W m⁻². Winter days with exposed white ice have 8–62 W m⁻².

## Expected effect, by metric (candidate minus baseline)

Rough sizes: on a melting, snow-free day with white ice at the top, A2
raises the albedo from about 0.2 to 0.55, so about 0.35 × 250 W m⁻² × 0.83
(the part not penetrating) ≈ 70 W m⁻², 6 MJ m⁻² per day, less absorbed: the
energy to melt about 2 cm of ice. Legacy melts the ice in spring at 4–5.5 cm
per day, so each such day delays ice-off by roughly a third of a day to half
a day, and the delay feeds itself (slower melt keeps the white layer
longer). Multiplying by the legacy number of such days gives a lower
estimate; the feedback could double it.

| Metric | Expected direction | Expected size | Why |
|---|---|---|---|
| Break-up (bias) | later, in the 50 and 100 % cases; 0 at 0 % | north 50 %: +1 to +3 days; north 100 % and Gimli 100 %: +3 to +8; Island, Big Trout 100 %: +5 to +12; Lac des Sables: +5 to +15 (100 %); south 50 %: +2 to +8 | Spring days with a bright white surface after the snow has gone, above. Largest where the model makes most snow ice. |
| Break-up (MAE) | improves where the baseline is early, degrades where late or where the delay overshoots | | Baseline break-up is early in 18 of the 24 snow cases (Baker 50/100 %, Big Trout, Gimli, Island, Lac des Sables 50 %), late in 6 (Yellowknife 50/100 %, Lac des Sables 100 %). Expected to improve: Baker 50 %, Big Trout, Gimli, Lac des Sables ERA5 50 %, probably Baker station 100 % and Island 50 %. Expected to degrade: Yellowknife, Lac des Sables 100 %, Island 100 % (baseline −1.4 to −1.8 days, likely overshoot), Lac des Sables station 50 % (−1.6). Baker ERA5 100 % (−1.8) uncertain. |
| Effective break-up (ice < 0.1 m) | later | as break-up, slightly less | Same mechanism; the last 10 cm of ice is usually clear ice by then. |
| Freeze-up | earlier where break-up moves most | 0 to −3 days (largest at Lac des Sables, Island, Big Trout 100 %); 0 at 0 % | A later ice-off leaves less summer to warm the 10 m mixed layer: about 0.3 K less heat for 10 days of high sun, and the lake cools 0.1–0.2 K per day in autumn. |
| Ice thickness (MAE, bias) | thicker | +0.5 to +3 cm in the bias of the snow cases, most in the south; 0 at 0 % | Mostly ice lasting longer through the spring measurements; a little from winter days with exposed white ice (colder surface). Thicker ice degrades the cases already too thick (most of them) and improves Baker 50/100 % (too thin by 0.34–0.39 m). Some MAE changes will pass the 0.005 m tolerance. |
| Seasonal maximum | slightly thicker | 0 to +3 cm, most in the south | Bright white ice in March before the maximum; the maximum is mostly reached before the snow goes. |
| Snow depth on the ice | nearly unchanged | below 0.5 cm | Snow albedo unchanged; thin snow over white ice is brighter (the blend goes to 0.75 instead of 0.5), so the last snow lasts a little longer, and late snowfalls land on ice that lasts longer. |
| Snow ice, congelation ice | nearly unchanged | below 1–2 cm | Snow-ice formation depends on the snow load and the freeboard, not on albedo; late snowfalls on longer-lasting ice may add a little snow ice in the south. |
| Closure | unchanged in substance | | No mass or energy bookkeeping changes; surface_snow_ice is a diagnostic. The albedo jumps when the snow ice at the top crosses 0.05 m, but between time steps, not inside the surface-temperature solver. As in E1, a few "largest daily" residuals may move to another day and pass the 1 % tolerance. Mass residuals should be unchanged. |
| Energy terms | FSWout up, FSWin up slightly | about +1 to +3 W m⁻² in the means over ice time steps | More reflected short-wave in spring; FSWin also rises because CLIMo's cloudy-sky short-wave grows with the albedo, and because ice lasts into sunnier days. |

**0 % snow cases** (12): identical to legacy in every metric (no snow ice
forms; the 5 cm of initial snow on 1 m of ice does not flood).

**North against south.** The north (Yellowknife, Baker Lake) should get
later break-up in its snow cases, which harms Yellowknife (already late)
and helps Baker Lake 50 % (early). Baker Lake's matching-snow scenario is
0 %, where nothing changes. In the south, where legacy breaks up 2–12 days
early (Island Lake, Big Trout, Gimli, Lac des Sables 50 %), A2 should help
most, except where the model already makes so much snow ice that it
overshoots (Lac des Sables 100 %, Island Lake 100 %).

**The specification's warning** (break-up improving while winter thickness
degrades) is expected to fire: break-up should improve in more cases than
it degrades, while the thickness, already too thick in most cases, moves
thicker by more than the 0.5 cm tolerance in several southern cases.

## thickness_weighted (E2_A2_weighted)

Same albedos, weighted by the snow-ice share (above: at snow-off, 3–10 % of
the ice in the north at 50 %, 17–26 % at 100 %; 25–40 % at Gimli, Island
and Big Trout 100 %; 35–65 % at Lac des Sables). The albedo change is the
threshold's times that share, and falls gradually as the top melts, instead
of staying until less than 5 cm is left. Expected: the same directions,
smaller: about 10–30 % of the threshold's break-up delay in the north,
30–70 % in the south, most at Lac des Sables.

## Sensitivity runs (not calibration)

None of these is chosen from the results.

- `snow_ice_albedo` 0.71 and 0.84 (E2_A2_a0.71, E2_A2_a0.84; melting value
  0.55 kept): the range Robinson et al. measured on cold white ice. Expected
  close to E2_A2 (within about a day): the cold value acts on winter and
  cold spring days, when the short-wave is low or the snow is thin; the
  delay comes mostly from melting days, which use 0.55 in all three.
  *Added before the run, after the unit tests:* legacy's thin-snow blend
  never gives more than the snow albedo (0.75 cold), so a snow-ice albedo of
  0.84 acts as 0.75: E2_A2_a0.84 is expected to be identical to E2_A2. It is
  run anyway to show this.
- `snow_ice_albedo` 0.46 with `snow_ice_albedo_melt` 0.36 (E2_A2_a0.46): the
  lower end of the literature (Bolsenga, 1969, snow ice on the Great Lakes,
  and a drop of about 0.1 when snow ice becomes moist, Bolsenga, 1980).
  Expected: about half or less of E2_A2's delays, because the melting
  albedo rises from about 0.2 to 0.36 instead of 0.55.
- `snow_ice_albedo_thickness` 0.02 and 0.10 m (E2_A2_h0.02, E2_A2_h0.10):
  0.02 m keeps the white albedo longer in spring (larger delays, by perhaps
  1–2 days); 0.10 m drops it earlier and switches it on later in winter
  (smaller delays, and no effect where the snow ice at the top stays under
  0.10 m: Yellowknife, Baker and Gimli at 50 %).

## What cannot be checked here

A2's direct target is the albedo of snow-free ice, and its premise is the
amount of snow ice. There are no albedo measurements and no ice-composition
(snow ice / clear ice) measurements at the six validation sites, so E2 can
only show the indirect effect on ice thickness and dates, not whether the
albedo or the snow ice is realistic.
