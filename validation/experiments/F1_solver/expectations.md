# F1_solver: expectations written before the run

Written on 30 October 2026, after the diagnosis below and before the
`surface_solver` option was implemented or `validation/compare.py` was run
with it. Not edited after the run; the results and how they compare with
these expectations are in
[docs/formulations.md](../../../docs/formulations.md#f1-surface-temperature-solver-fall-back-legacy-note-5).

## What is tested

Legacy note 5 ([docs/legacy_notes.md](../../../docs/legacy_notes.md)): when
Newton–Raphson fails to find the surface temperature, the solver brackets
the zero crossing of Ts − T1(Ts) in 1 K steps from the starting temperature,
upward and downward, and bisects. The legacy downward bisection
(`legacy/CLIMo_legacy.py`, lines 1345–1395) is a copy of the upward one
(lines 1296–1342) without its two `return` statements, so it runs 20 times
without exiting and the step falls back to the starting temperature. The
switch `surface_solver=fixed` makes the downward branch bisect and return
exactly as the upward one does (the same function, the bracket the other
way round). Nothing else in the solver changes: the Newton step, the
acceptance at Ts − 0.1 K (note 4), the bracket search, the tolerance
(0.01 K), the 20-iteration limit and its fall-back, and everything outside
the solver.

## Diagnosis (measured before the option existed)

`diagnose.py` runs legacy CLIMo (the E0 configuration) and
A3 on snow and ice (linear, 1 K and 2 K) on the frozen forcing and, at
every Newton–Raphson failure in the scored seasons, repeats the bracket
search outside the model to find which path ended the solver. Means over the
36 cases, per year of ice:

| Configuration | Newton failures | (a) up, bisection converged | (b) down → fall-back (note 5) | (c) no bracket in ±9 K | (d) other |
|---|---|---|---|---|---|
| legacy | 1.05 | 0.62 | 0.43 | 0 | 0 |
| A3 snow and ice, 1 K | 1.46 | 0.76 | 0.70 | 0 | 0 |
| A3 snow and ice, 2 K | 0.61 | 0.07 | 0.54 | 0 | 0 |

- Path (b) is 41 % of the failures in legacy, 48 % with A3 1 K and 88 % with
  A3 2 K. Paths (c) and (d) never occur: every fall-back to the starting
  temperature is note 5, so the fix can address all of them.
- In 82–94 % of the path-(b) steps the starting temperature is the melting
  point (273.15 K): the surface was melting at the previous step and cools
  sharply, so the solution lies below. Most are in March–June; in legacy
  some are mid-winter thaws followed by cold (e.g. Lac des Sables,
  12 February 1981).
- Accepted mismatch |Ts − T1| (median / largest): path (a) 0.006 / 5.4 K in
  legacy (the largest where bisection closes on the albedo jump at the
  melting point; 0.035 K at most with A3), path (b) 0.85 / 19.4 K in legacy,
  1.47 / 9.0 K (A3 1 K), 1.03 / 4.5 K (A3 2 K).
- Surface residual of the step (median / largest, W m⁻²): path (a) 0.1 /
  91 (legacy); path (b) 8.9 / 239 (legacy), 10.4 / 129 (A3 1 K), 10.7 / 102
  (A3 2 K). Daily residual of the day, largest: path (b) 59.8, 32.2 and
  25.5 W m⁻².
- The run's largest daily surface residual falls on a path-(b) day in 6 of
  36 cases in legacy (all four at Lac des Sables with snow, and two others),
  14 with A3 1 K and 6 with A3 2 K. The largest over all legacy cases,
  59.8 W m⁻² (Lac des Sables station 50 %, 12 February 1981), is one.
- Path (b) is almost absent at 0 % snow (0.00–0.04 per year) and occurs
  0.6–1.1 times per year with snow. Cases without a path-(b) step in the
  scored seasons: 6 in legacy (all at 0 % snow), 12 with A3 1 K (all the 0 %
  cases), 7 with A3 2 K.

## What should change

**Only steps where path (b) occurs.** With the fix those steps return a
temperature at which Ts and T1 agree to the bisection's tolerance, instead
of the starting temperature. Every later step of the same run can differ a
little (the state carries the change forward, including through the mixed
layer into the next season). A run with no path-(b) step anywhere (spin-up
included) must be bit-identical to legacy. With the default
(`surface_solver=legacy`) every output must be bit-identical (regression
test, E0).

**Direction.** On a path-(b) step legacy evaluates the surface budget at the
starting temperature (usually 273.15 K), warmer than the solution, so the
surface loses more heat (more emitted long-wave, less sensible heat gained)
than at the consistent temperature, and the conduction step cools the slab
too much. The fix removes that extra cooling: slightly warmer snow and ice
after the step, so marginally less growth (thinner) and, in spring, melt
resuming marginally sooner (earlier break-up). The residuals are one 6-hour
step at a time, about 0.4–0.7 times per year: a median of 9 W m⁻² for 6 h is
about 0.2 MJ m⁻² (0.6 mm of ice melted), the largest (239 W m⁻²) about
5 MJ m⁻² (1.7 cm of ice), much of which goes into the slab's temperature
rather than melt. We are not confident of the direction in every case: the
slab's response over the following days can offset it.

**Expected sizes (legacy against legacy + fix, F1_solver):**

- ice thickness and seasonal maximum: changes under 0.5 cm in every case,
  so all 36 cases "unchanged" within the harness tolerance (0.005 m) for
  MAE and bias; mean change a fraction of a millimetre;
- freeze-up: unchanged (within 0.5 day) in all cases;
- break-up and effective break-up: within 0.5 day in most cases; a few
  (at most about 5 of 36) may move by one time step or a day where the ice
  is close to the threshold; none by more than 2 days; on average slightly
  earlier;
- snow on the ice: within tolerance everywhere;
- snow ice, congelation ice: changes under 0.5 cm.

**Closure (the target of the fix):**

- largest daily surface residual: improved in the 6 cases where a path-(b)
  day sets it (legacy), falling to the size of the run's next-largest days
  (straddle days of the albedo jump, about 15–25 W m⁻² at Lac des Sables);
  the largest over all cases from 59.8 W m⁻² to at most about 32 W m⁻² (the
  largest non-path-(b) maximum in legacy, 32.2 W m⁻² at Baker Lake);
  elsewhere unchanged or changed slightly by the altered trajectory (a few
  "degraded" cases by small amounts are possible);
- mean daily surface residual: improved in most of the 30 cases with
  path-(b) steps, by small amounts (one day in about 1,000 changes);
- mass residuals, bottom residual: unchanged (rounding);
- short-wave residual: unchanged or a hair different (trajectory);
- fall-backs to the starting temperature: from 0.43 per case-year to 0;
  Newton–Raphson failures about the same (1.05 ± 0.1 per case-year; only
  the trajectory changes).

**With A3 (F1_solver_A3_ice, F1_solver_A3_ice_w2, against E3_A3_ice and
E3_A3_ice_w2):**

- fall-backs to the starting temperature from 0.70 (1 K) and 0.54 (2 K) per
  case-year to 0;
- largest daily surface residual improved in the 14 (1 K) and 6 (2 K) cases
  where a path-(b) day sets it; against legacy, the case-by-case count of
  E3_A3_ice (18 improved / 15 degraded) should become clearly favourable
  (at least 24 improved, at most 8 degraded), and E3_A3_ice_w2 (24 / 3)
  should stay as good or improve; the largest over all cases stays near
  32 W m⁻² (it is set by Baker Lake days that are not Newton failures);
- dates and thickness: A3's own changes (break-up 0.1–1.4 days earlier)
  are kept; the fix adds changes of the same small size as above.

## What would count as failure

- any output different from legacy with the default switch, or any
  difference in a run that had no path-(b) step;
- a path-(b) step that still ends at the starting temperature with the fix,
  or new fall-backs (paths (c) or (d)) appearing;
- the largest daily residual not improving in the cases where a path-(b) day
  set it;
- dates changing by more than 2 days, or thickness by more than 1 cm, in any
  case (that would mean the fall-backs mattered far more than their size
  suggests, or that the fix does something else);
- mass or bottom residuals changing beyond rounding.
