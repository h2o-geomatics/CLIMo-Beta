# Legacy behaviours and known issues

CLIMo Classic reproduces the model physics of the legacy Python CLIMo exactly.
Clear bugs in the output bookkeeping are fixed directly in Classic and listed
under *Fixed in CLIMo Classic*. The remaining behaviours below are reproduced
as they are, so users can interpret Classic outputs; any change to them would
be a separate, documented option (so far: notes 4, 5 and 11, corrected
together by `fixes` on; note 6, `snow_density_scheme`; see
[options.md](options.md)).

## Fixed in CLIMo Classic

The regression test checks that each fix changes only what it should.

- **Season boundaries for `SnowIce` and `CongelationIce`.** Both columns are
  derived from cumulative growth since the start of the run by resetting at
  each season boundary. Legacy boundaries were any pause in bottom growth for
  congelation ice, so 1–3 day pauses at melt onset (late March–April in the
  test cases) restarted the season while the ice was still there; and pauses
  of at least 150 days for snow ice, which would fail for lakes with short
  ice-free seasons. A boundary is now a pause in growth that includes at
  least one ice-free day.
- **`SnowIce` in the last season.** The legacy code did not reset snow ice
  after the last season boundary, so the final (usually partial) season
  reported the snow ice grown over the whole run: 2.15 m in the `YL_70` test
  case, where total ice never exceeds 1.01 m, and 0.85 m instead of 0.13 m
  in `S3A_209` at 50 % snow.
- **`SnowDepth` density.** Snow depth was always computed with the dry snow
  density; it now uses the density the model is using at the time (dry or
  wet), consistent with `L1SnowDepth`. (In the test cases the wet density is
  almost never active at the end of a day, see note 6.)
- **`Ice-Duration`.** The legacy leap-year correction edited a copy of the
  table and had no effect, so every duration assumed a 365-day previous
  year; durations after leap years were one day short. Also, when a year had
  no freeze-up, the next year's duration was counted from the freeze-up two
  years earlier. Durations are now computed from the freeze-up and break-up
  dates of the same ice cover (both days included, as before).
- **Layer depth columns.** The legacy columns changed meaning with the snow
  percentage: with snow, `L1SnowDepth` and `L2Thickness`–`L5Thickness`
  (depths below the snow/ice interface, not thicknesses); at 0 % snow,
  `L1Thickness`–`L5Thickness` (depths below the surface). A table combining
  scenarios would mix the two. They are replaced by `L1Depth`–`L5Depth`,
  depths below the surface in every run (mapping in [outputs.md](outputs.md)).
- **Ice dates by ice season instead of calendar year.** `ice_dates` paired,
  for each calendar year, the last ice formation and the first ice loss, and
  listed a year only if it had both. This suits lakes that freeze in autumn
  and break up in spring, but for lakes that freeze after 1 January (e.g.
  some years at Qinghai Lake), or when the next freeze-up fell after the end
  of the run, a year's break-up was dropped: a Qinghai Lake cell that froze
  on 2015 day 364 and broke up on 2016 day 89, and next froze in 2017, had
  no 2016 row. Durations could also pair the wrong events. `ice_dates` now
  has one row per ice season (1 August–31 July in the Northern Hemisphere,
  1 February–31 January in the Southern) with the freeze-up and break-up of
  the season's longest ice cover, as day of year and as dates; seasons
  without ice or with events outside the run keep their row with the value
  missing ([outputs.md](outputs.md)). Legacy year Y's break-up is the
  break-up of season Y−1–Y and its freeze-up the freeze-up of season
  Y–Y+1; in the regression cases every freeze-up and break-up day is
  unchanged under this mapping. The legacy table dropped the spin-up year;
  Classic keeps the season ending in the start year with its freeze-up
  missing, which holds the legacy first-year break-up.

## Outputs

1. **`SnowIce` and `CongelationIce` are ice grown, not ice remaining.** They
   show the growth since the season's growth began, do not decrease as ice
   melts, and are set to zero once growth stops (which for snow ice can be
   mid-winter). The snow ice still in the column is kept separately, for the
   ice-type albedo option only (`State.surface_snow_ice`; see
   [formulations.md](formulations.md#a2-clear-ice-versus-snow-ice-albedo)).
2. **Layer depths during open water** keep their last ice-covered values.
3. **Freeze-up threshold** is an ice mass of 0.01 kg m⁻² (about 11 µm of ice).

## Physics and numerics

4. **Newton–Raphson acceptance.** When the iteration converges on the step
   size, the accepted temperature profile and fluxes are those computed at
   Ts − 0.1 K (the last evaluation, used for the derivative), not at the new
   estimate; the same evaluation is accepted when the centred derivative
   "vanishes", whatever its mismatch (`legacy/CLIMo_legacy.py`, lines
   1187–1258, the `return`s marked `#987`). The first happens at about 15 %
   of all ice time steps and leaves a mismatch Ts − T1 of about 0.3 K
   (up to 5 K at the melting point) and a surface residual of about
   4 W m⁻² for the step: it is almost all of the mean surface residual of
   legacy runs (0.65 W m⁻²). The second is rare (0.07 per year) but can
   accept a mismatch of 7–11 K. With note 5 fixed, these steps set the
   largest remaining surface residuals in most cases. Probably original
   Flato and Brown behaviour, not a translation fault (label 987 was most
   likely a plain return). **Option:** `fixes` on (`--fixes on`, with
   notes 5 and 11) re-evaluates at the converged estimate and sends a
   vanishing derivative to the bisection (F2; the internal `Parameters`
   field `newton_acceptance`, `final_estimate` or `fixed`, does it alone,
   for the validation experiments). With note 5 fixed it changes dates
   and thickness by under 0.6 day and 6 mm, cuts the mean daily surface
   residual from 0.63 to 0.02 W m⁻² in every case, and with A3 the largest
   from 32.8 to 4.2 W m⁻² (see
   [formulations.md, F2](formulations.md#f2-newton-raphson-acceptance-legacy-note-4)).
5. **Downward bisection has no exit.** When the zero crossing is bracketed
   below the starting surface temperature, the legacy bisection runs 20
   iterations without returning and the model falls back to the starting
   temperature (`legacy/CLIMo_legacy.py`, lines 1345–1395: a copy of the
   upward bisection, lines 1296–1342, without its two `return` statements;
   its comment `goto 887` points to the upward block, so this is most likely
   a fault of the translation from Fortran). Newton–Raphson fails (so the
   bracket search runs) about once per year of ice in the validation runs,
   and this downward fall-back ends 41 % of those failures (0.43 per year),
   leaving the surface budget and the conduction step up to 19 K apart; it
   sets the largest surface energy residual of the legacy runs (59.8 W m⁻²).
   With the A3 option (an albedo continuous in the surface temperature near
   melting) it happens more often and sets A3's largest residuals.
   **Option:** `fixes` on (`--fixes on`, with notes 4 and 11) bisects
   and returns the downward bracket as the upward one (F1; alone, the
   internal `Parameters` field `surface_solver="fixed"`); it removes every
   fall-back, changes dates and thickness by under 0.1 day and 0.3 mm at
   the validation sites, and lowers the largest residual to 32.2 W m⁻² (see
   [formulations.md, F1](formulations.md#f1-surface-temperature-solver-fall-back-legacy-note-5)).
6. **The wet snow density is rarely used.** Snow takes the wet density only
   when the mean of the surface and snow/ice-interface temperatures reaches
   0 °C. The interface stays below freezing until the snow is nearly gone,
   so in the test cases the wet density was used in 2 of about 6,000
   snow-covered ice time steps (`S3A_209`, 50 %) and in none (`YL_70`): snow
   melts at the dry density, and `rho_snow_warm` has almost no effect. This
   agrees with Rafat et al. (2025), who found CLIMo's density stayed at the
   dry value until melt. When it does switch, snow depth changes abruptly
   (snow mass is conserved). Measured on the validation runs: 462 of 435,674
   snow-covered ice time steps (0.11 %), depth jumps of up to 14 cm for one
   step; when the surface melts, the mean is still a median 0.6 K below
   0 °C. **Option:** `snow_density_scheme="bulk_relaxation"`
   (`--snow-density-scheme bulk_relaxation`, spec S1) makes the density a
   state variable with memory (new snow mixed in, relaxing upward towards
   `rho_snow_cold`, and towards `rho_snow_warm` within 1 K of melting), with
   the same snow mass; with its defaults it changes no observed quantity
   beyond tolerance (see
   [formulations.md, S1](formulations.md#s1-bulk-snow-density-evolution)).
7. **Penetrating short-wave factor.** Absorption in the slab is computed as
   PenSW·κ·(e^(−κz₁) − e^(−κz₂)), and the short-wave reaching the mixed layer
   as PenSW·κ·e^(−κh)·Δt. Beer's law would give these without the factor κ
   (1.5 m⁻¹). To be checked against the Fortran code of Flato and Brown (1996).
8. **Snow accumulates only below 273.00 K** (not 273.15 K), and only on ice.
9. **Surface melt without snow layer.** When there is no snow layer (snow
   thinner than 1 mm) and the surface is above melting, the excess heat
   first removes snow at the ice density, and if any snow remains no ice is
   melted.
10. **Thin ice with snow.** Below 0.2 m of ice, the linear-profile shortcut
    uses the ice thickness only and ignores the snow layer's insulation.
11. **Unbounded Newton–Raphson step.** The surface-temperature solver's
    Newton step, Tsnew = Ts − f/f′ with a centred derivative over ±0.1 K
    (`legacy/CLIMo_legacy.py`, lines 1244–1245 and 1258), has no limit, and
    nothing in the surface budget or the conduction bounds Ts (the 273.15 K
    caps act on the conduction's boundary condition, chosen from the
    temperature at the start of the step, and on the slab after the solver).
    The mismatch f = Ts − T1 has a kink at the melting point (legacy's albedo
    jump and the conduction's melting-surface switch): when a step starts at
    0 °C, the derivative's two evaluations (273.25 and 273.05 K) can straddle
    it and give a slope near zero, and the step is then tens to hundreds of
    kelvin (0.01–0.06 steps beyond 20 K per year of ice in the validation
    runs, most starting at 273.15 K). Usually the iteration then fails and
    the bracketing fall-back runs, but once (S1 with fresh snow at
    150 kg m⁻³, Lac des Sables station 50 %, 19 January 1990, with F1 and
    F2) it converged on a spurious root of the discrete equations, −72.5 K,
    followed by 30 K and 109 K on the next steps (largest daily surface
    residual of that run, 51.3 W m⁻²). The same day in legacy is a ±117 K
    oscillation ending in the note-5 fall-back. Most likely original
    behaviour (a plain Newton iteration), not a translation fault.
    **Option:** `fixes` on (`--fixes on`, with notes 4 and 5) shortens a
    step longer than 20 K to that length (F3; alone, the internal
    `Parameters` fields `newton_step="limited"` and `newton_step_limit`); it changes
    only those calls, removes every implausible accepted temperature, and
    needs F1 for the calls it cannot rescue to end on the solution (see
    [formulations.md, F3](formulations.md#f3-newton-step-limit)).

## Changes in the port that do not affect results

- The legacy tridiagonal solver checked divisors against the emissivity
  (0.99) because the name `epsilon` was reused; it would stop the program if
  a divisor fell below 0.99. The port stops only for a non-positive divisor.
  This never occurred in the reference runs.
- The legacy flux bottom boundary condition had an indexing error; it is
  never used (the bottom is always at the freezing temperature) and is
  written correctly in the port.
- Unused accumulators (top melt, internal melt, stored heat) are not kept.
- The legacy code read a hardcoded file name; forcing is now an argument.
- If the forcing file contained data before the start year, the legacy code
  ran both that data and the spin-up copy. The port uses only the years
  start..end.
- SMRT-specific outputs were removed.

## References

- Rafat, A., Cheng, B., and Kheyrollah Pour, H. (2025). Exploring the physics of two thermodynamic lake ice models. *Water Resources Research*, 61(5), e2024WR038615. https://doi.org/10.1029/2024WR038615
