# F2_newton: expectations written before the run

Written on 30 October 2026, after the diagnosis below and before the
`newton_acceptance` option was implemented or `validation/compare.py` was
run with it. Not edited after the run; the results and how they compare
with these expectations are in
[docs/formulations.md](../../../docs/formulations.md#f2-newton-raphson-acceptance-legacy-note-4).

## What is tested

Legacy note 4 ([docs/legacy_notes.md](../../../docs/legacy_notes.md)). At
every time step with ice the solver looks for the surface temperature Ts at
which f(Ts) = Ts − T1(Ts) = 0 (T1: the surface temperature returned by one
conduction step driven by the surface budget at Ts). Newton–Raphson (at most
5 iterations, tolerance 0.01 K) evaluates f at Ts, then at Ts + 0.1 K and
Ts − 0.1 K for a centred derivative, and ends in one of three ways
(`legacy/CLIMo_legacy.py`, lines 1187–1258, each `return` marked `#987`):

1. |f(Ts)| < 0.01 K: returns the evaluation at Ts (correct);
2. |f(Ts + 0.1) − f(Ts − 0.1)| < 0.01 K ("derivative vanishes"): returns the
   evaluation at Ts − 0.1 K, whatever its mismatch;
3. the Newton step |Tsnew − Ts| < 0.01 K: returns the evaluation at
   Ts − 0.1 K (the last one made), not at the converged estimate Tsnew.

The switch `newton_acceptance` gets two non-legacy values, because the
diagnosis shows the two exits are different problems:

- `final_estimate`: exit 3 re-evaluates the budget and conduction at Tsnew
  and returns that (as the bisection's own exit, `#901`/`#902`, re-evaluates
  at its accepted temperature);
- `fixed`: as `final_estimate`, and exit 2 is treated as a Newton–Raphson
  failure (a flat derivative is not convergence; |f(Ts)| < 0.01 K is already
  accepted by exit 1), so the bracketing fall-back runs.

Nothing else changes: the tolerance, the 0.1 K derivative step, the 5
iterations, the bracket search and bisection, the 273.15 K cap, the
conduction's melting-surface switch, everything outside the solver.

## Diagnosis (measured before the option existed)

`diagnose.py` records every evaluation of every solver call
at every time step with ice, replays the solver's rules on them to find how
the call ended (all replays consistent), and records the accepted mismatch
|Ts − T1| and the step's surface residual (climo.closure). Scored seasons,
36 cases. Calls per year of ice (mean over the cases):

| Configuration | exit 1 | exit 2 | exit 3 | bracket up | bracket down |
|---|---|---|---|---|---|
| legacy | 651.1 | 0.072 | 112.7 | 0.62 | 0.43 (fall-back, note 5) |
| legacy + F1 | 651.0 | 0.068 | 112.8 | 0.62 | 0.44 (bisected) |
| A3 snow and ice 1 K + F1 | 650.1 | 0.114 | 111.0 | 0.77 | 0.66 |
| A3 snow and ice 2 K + F1 | 651.0 | 0.016 | 110.5 | 0.07 | 0.57 |

| Exit | Configuration | calls | mismatch median / 95 % / largest (K) | step residual median / 95 % / largest (W m⁻²) |
|---|---|---|---|---|
| 2 | legacy | 55 | 0.19 / 0.69 / 11.3 | 27.9 / 72.1 / 98.2 |
| 2 | legacy + F1 | 53 | 0.18 / 0.70 / 11.3 | 28.1 / 72.8 / 98.2 |
| 2 | A3 1 K + F1 | 95 | 0.76 / 4.1 / 7.6 | 4.3 / 51.9 / 130.2 |
| 2 | A3 2 K + F1 | 13 | 2.2 / 3.8 / 4.7 | 5.6 / 39.3 / 61.8 |
| 3 | legacy | 112,005 | 0.30 / 0.49 / 5.2 | 3.8 / 10.5 / 89.1 |
| 3 | legacy + F1 | 112,023 | 0.30 / 0.49 / 5.2 | 3.8 / 10.5 / 89.1 |
| 3 | A3 1 K + F1 | 110,465 | 0.30 / 0.48 / 1.6 | 3.7 / 10.2 / 45.8 |
| 3 | A3 2 K + F1 | 109,912 | 0.30 / 0.48 / 1.4 | 3.7 / 10.1 / 46.2 |

- **Exit 3 is systematic, not rare:** about 15 % of all ice time steps.
  In every one of them T1 is warmer than the accepted Ts (Ts − T1 < 0 in
  all but 4 of 112,005 calls) and the step's surface residual is negative
  (mean −4.5 W m⁻²): the slope of f is about 3 (2.5–6), so the evaluation
  0.1 K below the converged estimate has a mismatch of about 0.3 K. Exit 3
  occurs in all months with ice, most in November–April. It accounts for
  almost all of the mean daily surface residual: with the exit-3 residuals
  set to zero, the mean of daily absolute residuals falls from 0.65 to
  0.04 W m⁻² (legacy), 0.64 → 0.03 (F1), 0.61 → 0.02 (A3 1 K + F1). Its
  largest mismatches (1.4–5.2 K) are at the melting point, where the
  conduction's melting-surface switch makes f jump between Ts − 0.1 and Ts.
- **Exit 2 is rare** (0.07 per case-year in legacy, mostly at 0 % snow in
  spring, with Ts within 0.2 K of the melting point; 0.11 with A3 1 K + F1)
  but its accepted evaluation can be far from the solution (A3: Lac des
  Sables ERA5 100 %, 26 March 1986, 7.6 K and 130 W m⁻²; legacy: Lac des
  Sables ERA5 50 %, 30 December 1984, 11.3 K and 98 W m⁻²).
- **Which exit sets each case's largest daily surface residual** (the step
  with the largest residual on that day):

  | Configuration | exit 1 | exit 2 | exit 3 | up | down |
  |---|---|---|---|---|---|
  | legacy | 10 | 7 | 12 | 1 | 6 |
  | legacy + F1 | 12 | 9 | 14 | 1 | 0 |
  | A3 1 K + F1 | 0 | 7 | 29 | 0 | 0 |
  | A3 2 K + F1 | 0 | 1 | 35 | 0 | 0 |

  The exit-1 maxima in legacy are steps that converge across the
  melting-point albedo jump of legacy (|Ts − T1| < 0.01 K but the albedo
  differs between Ts and T1); A3 removes them.
- **What the sub-fixes can reach** (largest daily residual with the
  residuals of those steps set to zero; over all cases / median per case,
  W m⁻²): legacy + F1 32.2 / 14.0 → exit 3 only 24.6 / 13.0, both
  22.6 / 11.7 (the rest are exit-1 albedo-jump days and one bisection, Gimli
  ERA5 50 %, 6 April 1983); A3 1 K + F1 32.8 / 11.0 → exit 3 only
  32.6 / 2.1, both 4.1 / 0.9; A3 2 K + F1 32.2 / 9.9 → exit 3 only
  15.4 / 2.3, both 6.5 / 2.2. Cases whose largest residual each could
  improve by more than 1 %: legacy + F1: exit 2 9, exit 3 15, both 24; A3
  1 K + F1: 7, 32, 36; A3 2 K + F1: 1, 35, 36. Both parts matter: exit 3
  for the mean and most maxima, exit 2 for the largest single days.

These are upper bounds: the re-evaluation at Tsnew leaves a mismatch of
the order of the Newton error (expected under 0.01 K in most calls), and a
flat-derivative call sent to the bracket search ends on a bisection, which
can close on a jump of f (F1: up to 1.1 K and 25 W m⁻² at the melting
point).

## Is it a translation fault?

Probably not. The Python marks all three Newton exits `#987` … `#end of
987` around a bare `return`, while the translation copies the bisection's
exit block (label 902: evaluate the budget and conduction at Ts, then
return) in full wherever it is used, marked `#902` … `#902 end`. Label 902
also exists separately from 901 (`Ts=Tsm`) because the fall-backs reach it
after setting `Ts=Told(1)`, i.e. 902 is the Fortran's shared "evaluate at
Ts and return" exit. Had the Newton exits meant to re-evaluate, they would
have jumped to 902 (and exit 3 would have set Ts = Tsnew first); they jump
to a different label, 987, whose translation contains no evaluation, so
987 was most likely a plain `RETURN`. The Fortran comment at exit 2 ("if
derivative vanishes, take solution obtained at this point and don't iterate
any further") reads as a deliberate acceptance. Inference: the Ts − 0.1 K
acceptance is in the original Flato and Brown code, an approximation that
the author probably judged harmless (0.1 K, ten times the tolerance), not a
porting fault. We do not have the Fortran.

## What should change

**With `fixed` or `final_estimate`, about 15 % of the ice time steps
change** (every exit-3 step), so, unlike F1, no case will be bit-identical
to legacy. With the default (`legacy`) every output must be bit-identical
(regression test, E0).

**Direction.** On an exit-3 step legacy drives the conduction step with the
budget at Ts − 0.1 K, 0.1 K colder than the solution: about 1.5 W m⁻² more
heat into the slab for 6 hours (the closure residual, 4.5 W m⁻², is larger
because it compares the budget at the accepted Ts with that at the warmer
T1). Averaged over all ice steps that is about 0.2 W m⁻² of extra heating,
mostly in winter, which the surface largely gives back (a warmer surface
loses more heat; about 15 W m⁻² K⁻¹), so the fix should leave the ice very
slightly colder: marginally thicker ice and later break-up. The exit-2 part
acts on a handful of spring days and has no predictable direction.

**Expected sizes (against legacy; the same against F1_solver and the A3 + F1
runs):**

- ice thickness and seasonal maximum: changes of 0–3 mm, all or nearly all
  cases "unchanged" (within 0.005 m) for MAE and bias (at least 33 of 36),
  mean change slightly positive (thicker);
- freeze-up: unchanged (within 0.5 day) in all cases;
- break-up and effective break-up: within 0.5 day in at least 32 of 36
  cases, none beyond 2 days; mean shift later by under 0.2 day;
- snow on the ice, snow ice, congelation ice: within tolerance / under
  0.5 cm.

**Closure (the target):**

- mean daily surface residual: improved in all 36 cases in every
  comparison, falling from about 0.6 W m⁻² to under 0.1 W m⁻² (median);
- largest daily surface residual:
  - `fixed` alone (F2_newton, without F1): improved in the cases where an
    exit-2 or exit-3 day set it (up to 18), but the largest over all cases
    stays 59.8 W m⁻² (the note-5 day at Lac des Sables, which only F1
    fixes); without F1 a flat-derivative step bracketed below the start
    falls back to the starting temperature (note 5), so a few cases may
    degrade. This is why the exit-2 part should be used with F1;
  - with F1 (F2_newton_F1 against F1_solver): improved in about 20–24 cases,
    degraded in at most 4 (by small amounts, from the changed trajectory);
    over all cases from 32.2 to about 23 W m⁻² (set by legacy's albedo
    jump, exit 1, or a bisection), median per case from 14.0 to about 12;
  - with F1 and A3 1 K (F2_newton_F1_A3_ice against F1_solver_A3_ice):
    improved in at least 33 of 36 cases; over all cases from 32.8 to under
    15 W m⁻²; median per case from about 11 to under 5;
  - with F1 and A3 2 K: improved in at least 33 of 36; from 32.2 to under
    15 W m⁻²;
- mass and bottom residuals: unchanged (rounding); short-wave residual:
  unchanged or a hair different (trajectory);
- Newton–Raphson failures (calls that reach the bracket search): with
  `fixed`, up by the former exit-2 rate (legacy 1.05 → about 1.12 per
  case-year; A3 1 K + F1 1.43 → about 1.55; A3 2 K + F1 0.64 → about 0.66),
  within ±0.1 for trajectory changes; with `final_estimate` unchanged
  within ±0.1. With F1 on, no fall-back to the starting temperature
  anywhere;
- the accepted mismatch on exit-3 steps: median under 0.01 K (from 0.30 K).

**Runtime.** One extra evaluation of the budget and conduction on every
exit-3 step (about 0.15 per ice step on top of about 4.85): about 3 % more
model time, plus the bracket searches of the former exit-2 steps
(negligible).

**`final_estimate` with F1 (F2_newton_final_F1)** isolates exit 3: the
same dates, thickness and mean residual as `fixed` with F1 within
tolerance, and the largest residual improved in fewer cases (the exit-2
days remain: Lac des Sables ERA5 50 % 24.6 W m⁻², 0 % 20.1, Island ERA5
0 % 19.2, Gimli station 0 % 17.5).

## What would count as failure

- any output different from legacy with the default switch (regression
  test, E0 not 432 of 432 equal);
- an exit-3 step under `fixed` still accepting a mismatch near 0.3 K
  (median above 0.05 K), i.e. the re-evaluation not taking effect;
- with F1 on, any fall-back to the starting temperature, or Newton
  failures rising by clearly more than the former exit-2 rate (more than
  0.1 per case-year above it);
- the mean daily surface residual not improving in most cases, or the
  largest daily residual degrading in more than 4 cases in any comparison
  with F1 on;
- dates changing by more than 2 days, or thickness by more than 1 cm, in
  any case; any observed metric (thickness, maximum thickness, freeze-up,
  break-up, effective break-up, snow depth) degraded beyond tolerance in
  more than 3 cases;
- mass or bottom residuals changing beyond rounding.
