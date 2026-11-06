# E4_S1: expectations written before the run

Written on 30 October 2026, after the diagnosis below and before the
`snow_density` option was implemented or `validation/compare.py` was run
with it. Not edited after the run; the results and how they compare with
these expectations are in
[docs/formulations.md](../../../docs/formulations.md#s1-bulk-snow-density-evolution).

## What is tested

Spec S1 (bulk snow-density evolution with snowfall mixing). Legacy CLIMo
gives the snow on the ice one of two densities at every time step, chosen
from the mean of the surface and snow/ice-interface temperatures
Tsav = (T1 + T2)/2: `rho_snow_cold` (330 kg m⁻³) below 273.15 K,
`rho_snow_warm` (450) at or above it (`climo/model.py`, `simulate`). The
density has no memory. S1 makes it a prognostic state variable of the
single snow layer:

1. **Snowfall mixing** (once a day, when snow is added): the new snow keeps
   legacy's mass, `rho_snow_cold × Snow × pct` (below 273.00 K, on ice,
   legacy note 8), and enters the pack at `snow_density_fresh`; the pack's
   density becomes the mass over the summed depths,
   ρ* = (m + Δm) / (m/ρ + Δm/ρ_fresh), i.e. the spec's
   (ρ h + ρ_new Δh)/(h + Δh) with h = m/ρ, Δh = Δm/ρ_fresh. Mass and depth
   are both conserved.
2. **Relaxation** (every 6-hour time step, before the step, where legacy
   chooses its density): ρ ← ρ + max(0, ρ_eq − ρ)·(1 − exp(−Δt/τ)), with
   ρ_eq = ρ_cold + (ρ_warm − ρ_cold)·g and
   g = min(1, max(0, 1 − (273.15 − Tsav)/W)), linear over the width W below
   melting, on legacy's own temperature Tsav. Density only increases
   (snow does not un-compact; FSM 1.0 does the same); it can fall only by
   mixing in lighter new snow.
3. **Reset:** when the snow mass is zero the density is set to
   `snow_density_fresh` (the next snowfall then starts a new pack).

Defaults, fixed before the run: `snow_density_fresh` = 330 kg m⁻³ (=
`rho_snow_cold`, so new snow has legacy's depth), `snow_density_timescale`
τ = 100 h (CLASS, Verseghy 1991, 0.01 h⁻¹, and ISBA, Douville et al. 1995,
0.24 d⁻¹: both checked in model code), `snow_density_transition_width`
W = 1 K. The density stays between min(ρ_fresh, ρ_cold) and
max(ρ_fresh, ρ_warm) (convex combinations only), at most 600 kg m⁻³ (the
largest seasonal-snow bulk density in Sturm et al., 2010) and below the ice
density.

**What S1 does not change:** the snow mass reaching the ice (S3's subject),
the snowfall rule (273.00 K, on ice), snow melt by mass, the snow-ice
flooding criterion (by mass), every equation of the conduction, surface and
growth code. S1 changes only the density those equations are given, hence
the snow depth, its conductivity (2.845e-6 ρ² + …), the pore water of new
snow ice (ρ_i − ρ_s)/ρ_i, and the snow depth seen by the thin-snow albedo
and the short-wave absorption. The snow layer's heat capacity per unit area
(ρ c(T) × m/ρ) does not depend on the density, so a density change moves no
heat.

## Diagnosis (measured before the option existed)

`snow_density_baseline.py` ran legacy (as E0) and
legacy + F1 + F2 (`surface_solver=fixed`, `newton_acceptance=fixed`) on the
frozen forcing, 50 % and 100 % snow (0 % has no snow), and recorded the
slab temperatures at the start of every time step with a snow layer on the
ice (scored seasons; outputs `snow_density_baseline.csv` and
`snow_density_months.csv`).

- **The wet density is almost never used:** 462 of 435,674 snow-covered
  ice time steps (0.11 %; 0.08–1.6 per year per case), 432 with F1 + F2.
  When it is, the snow depth drops by 0.01–0.03 m on average and up to
  0.14 m (Baker Lake ERA5 100 %) for that step and comes back at the next.
  Legacy note 6 is confirmed at every site.
- **Candidate temperatures for g** (share of snow-covered ice steps; mean
  of the 24 legacy cases; F1 + F2 within 0.002):

  | Temperature | at melting | within 0.5 K | within 1 K | within 2 K |
  |---|---|---|---|---|
  | Tsav (legacy's mean snow temperature) | 0.14 % | 3.6 % | 8.4 % | 13.4 % |
  | T1 (surface) | 7.8 % | 0.4 % | 1.9 % | 4.4 % |
  | T2 (snow/ice interface) | 1.7 % | 4.4 % | 9.3 % | 19.5 % |
  | air (day's forcing value) | 8.0 % | 1.1 % | 2.4 % | 4.8 % |

  By month (pooled): Tsav within 1 K in 0.2 % of January steps, 5.6 % in
  March, 21 % in April, 33 % in May, 82 % in June; and 4.9 % in October,
  2.0 % in November (warm spells on thin early ice). By lake: 4.6 %
  (Yellowknife) and 5.0 % (Baker Lake) to 15.9 % (Lac des Sables).
- **When the surface is melting, Tsav is a little below melting:** on steps
  with T1 at 273.15 K, Tsav is a median 0.61 K below it (0.34 K at Lac des
  Sables, 0.81 K at Yellowknife), within 1 K in 87 % of them (74–99 % by
  lake) and within 2 K in 99.9 %. The interface stays below freezing while
  the top melts, which is why legacy's step at 0 °C on Tsav almost never
  switches.

**Choice of the temperature for g (decided from this):** legacy's Tsav.
It is the bulk temperature of the single layer, the analogue of the bulk
snow temperature CLASS uses to choose its cold or melting maximum density
(`TSNOW < −0.01 °C`, CLASSIC code checked), and the spec asks for "the same
snow-temperature information already used by CLIMo". A step at 0 °C on
Tsav (as CLASS) would leave S1 inert in CLIMo (0.14 % of steps); hence the
linear band below melting, W = 1 K, which covers 87 % of the steps with a
melting surface, with g ≈ 0.4 on a typical one (0.6 K below). T1 alone
would push the whole pack towards the wet density whenever the top
melts, however cold the base; T2 reaches melting in mid-winter (1.1 % of
January steps), when flooding releases the latent heat of the pore water at
the interface (`climo/growth.py`), which is not a densification signal of
the pack; the air temperature is forcing, not the state of the snow. W is
an engineering choice (no source); W = 0.1 K and 2 K are run as sensitivity.

## What should change

### (a) Defaults (`snow_density=bulk_relaxation`, fresh 330, τ 100 h, W 1 K)

New snow enters at 330 and the cold equilibrium is 330, so in mid-winter
nothing changes (expected bit-identical stretches of December–February).
The density rises only while Tsav is within 1 K of melting: in spring melt
(April–June; on a typical melting step ρ_eq ≈ 330 + 120 × 0.4 ≈ 380, after
several such days about 360–400 kg m⁻³), and briefly in October–November
warm spells on thin early ice, where the denser snow then stays (no
decrease), diluted by later snowfall. Effects:

- **spring:** snow depth lower by up to 10–20 % during melt; snow
  conductivity up by 20–45 % (ρ²); thermal resistance (∝ 1/ρ³ at fixed
  mass) down by a third or more, but the pack is then near-isothermal, so
  little extra conduction. The thinner pack reaches the 0.1 m thin-snow
  albedo blend earlier: slightly earlier snow-off and break-up;
- **autumn:** slightly less insulation of early snow, slightly thicker ice
  (millimetres);
- **snow ice:** denser snow makes slightly less pore water when it floods,
  but flooding happens mostly in mid-winter, when nothing changes:
  negligible.

Expected sizes (36 cases, against legacy):

- ice thickness and seasonal maximum: within ±5 mm (unchanged) in at
  least 30 of 36 cases, none beyond 1.5 cm; 0 % snow cases bit-identical in
  dates and thickness (no snow, nothing to densify);
- freeze-up: unchanged in all cases;
- break-up and effective break-up: within 0.5 day in at least 28 of 36,
  mean shift earlier by 0–0.5 day, none beyond 2 days;
- snow depth on the ice (observed, ECCC): small decrease of the modelled
  depth (spring measurements most), MAE and bias unchanged in most cases,
  improved where legacy is too deep (Baker Lake +0.2 m) only marginally;
- wet-density depth jumps (diagnosis) gone: the depth changes smoothly.

### (b) Lighter fresh snow (sensitivity, fresh 150 and 200, τ 100 h)

The same snow mass enters at 150 (200) kg m⁻³, so its depth is 2.2 (1.65)
times the input snowfall depth (and legacy's), and compacts towards 330
with an e-folding time of 100 h (4.2 days). With snowfall every few days
the bulk density of the mid-winter pack should settle around 270–320
(150) and 290–325 (200): snow 5–25 % deeper than legacy, thermal
resistance 15–80 % higher. Mass is unchanged, so this is a sensitivity of
the density path at fixed mass, not a reinterpretation of the snowfall
input (that is S3).

- **congelation ice thinner** (more insulation), **snow ice a little
  more** (thinner ice floods sooner; pore water (ρ_i − ρ_s)/ρ_i up by
  3–10 %): total ice thinner, seasonal maximum by 1–6 cm at 50–100 % snow
  (larger with 150 than 200), partly compensated by snow ice at 100 %;
  0 % unchanged;
- thickness bias improves where legacy is too thick (Island, Lac des
  Sables 100 %, Yellowknife, Big Trout and Gimli 100 %) and degrades where
  it is too thin (Baker Lake 50–100 %);
- break-up earlier by 0.5–3 days (thinner ice), against slightly later
  snow-off (deeper snow keeps the snow albedo longer): net earlier at most
  sites; degrades where legacy is already early (Gimli, Big Trout, Lac des
  Sables 0–50 %), improves Yellowknife;
- freeze-up unchanged;
- snow depth on the ice: deeper by 2–8 cm on average; degrades in most
  50–100 % cases (legacy is already too deep at all sites but Lac des
  Sables 50 %); this is the observed quantity S1 acts on most directly.

### (c) Timescale (τ 200 h, the FSM 1.0 value; and with fresh 150)

- with fresh 330: half the spring densification rate: effects of (a)
  about halved, all within tolerance;
- with fresh 150 and τ 200 h: slower compaction, lower winter bulk density
  (about 250–300), so the (b) changes larger by about half;
- with fresh 150 and τ 4800 h (200 days, of the order of the seasonal
  densification rates k2 = 0.0031–0.0049 d⁻¹ fitted by Sturm et al., 2010;
  a bracket, not a relaxation-model value): new snow barely compacts, the
  pack stays near 150–200, snow about twice legacy's depth; the largest
  changes of all runs (maximum thickness down by 5–15 cm at 100 % snow,
  snow depth degraded nearly everywhere). Shows the size of the density
  lever, not a candidate.

### (d) Transition width

- W = 0.1 K (close to CLASS's step): effects of (a) mostly vanish (Tsav
  rarely within 0.1 K);
- W = 2 K: every melting step inside the band (g ≈ 0.7), more autumn
  steps: effects of (a) about doubled, still mostly within tolerance.

### Closure (all runs)

- snow and ice mass residuals unchanged (rounding, under 1e-6 kg m⁻²):
  mixing and relaxation change no mass;
- bottom residual unchanged (rounding);
- surface residual: the solver works within a step at fixed density, so S1
  adds no solver error; changes only through the trajectory, mean daily
  residual within a few per cent, largest daily residual improved and
  degraded in similar numbers (noise of the 1 % tolerance). With F1 + F2
  the mean daily residual stays about 0.02 W m⁻² (versus F2_newton_F1);
- short-wave residual: changes slightly with the snow-layer thickness (the
  κ factor of note 7 depends on the layer geometry), either sign.

### With the solver fixes (F1 + F2 on, against F2_newton_F1)

S1's own effect, isolated from the solver's: the same directions and sizes
as (a) and (b) within the tolerances (F1 + F2 change dates and thickness by
under 0.6 day and 6 mm); cleaner closure verdicts.

## What would count as failure

- any output different from legacy with the default `snow_density=legacy`
  (regression test, E0 not 432 of 432 equal, self-test not 0);
- snow or ice mass residual beyond rounding (above 1e-6 kg m⁻²) in any
  case;
- a density outside [min(ρ_fresh, ρ_cold), max(ρ_fresh, ρ_warm)], above
  600 kg m⁻³, or decreasing without new snow;
- defaults changing any 0 % snow case, any mid-winter (December–February)
  day before the first warm-band step of the season, or dates by more than
  2 days, or thickness by more than 1.5 cm;
- closure (surface mean daily residual) degraded in more than a third of
  the cases with F1 + F2 on;
- lighter fresh snow making the ice thicker on average, or the snow
  shallower (that would mean the density path is not what is described).
