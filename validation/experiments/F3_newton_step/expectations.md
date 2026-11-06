# F3_newton_step: expectations written before the run

Written on 30 October 2026 (about 19:40 UTC), after the diagnosis below and
before the `newton_step` option was implemented or `validation/compare.py`
was run with it. Not edited after the run; the results and how they compare
with these expectations are in
[docs/formulations.md](../../../docs/formulations.md#f3-newton-step-limit).

## What is tested

The surface-temperature solver (`climo.model.solve_surface_temperature`;
`legacy/CLIMo_legacy.py`, `NR_iterate`, lines 1187–1258) looks for the Ts at
which f(Ts) = Ts − T1(Ts) = 0, T1 being the surface temperature returned by
one conduction step driven by the surface budget at Ts. Its Newton–Raphson
step is Tsnew = Ts − f(Ts)/f′(Ts) with a centred derivative over ±0.1 K
(line 1245), and nothing bounds it: not the step, not the iterate, not the
surface budget or the conduction (the only 273.15 K caps act on the
conduction's boundary condition, chosen from the temperature at the start of
the step, and on the slab after the solver, in the melt).

f is not smooth at the melting point. When the step starts with the surface
at 0 °C, the two derivative evaluations (273.25 and 273.05 K) straddle
legacy's albedo jump at 273.15 K and the conduction's melting-surface switch
(a surface at 273.15 K with a non-negative net flux is held there, so T1 is
273.15 K whatever Ts), and the centred slope can be almost zero (0.07–0.2,
just above the 0.05 "vanishing" test) while f is 10–20 K. The Newton step is
then hundreds of kelvin.

## Diagnosis (measured before the option existed)

`diagnose.py` runs the 36 cases with a traced copy of the
solver (checked call by call against the model's own solver: 0 mismatched
calls) and records every Newton step and what each call accepted. Scored
seasons, 924 case-years per configuration.

**Step sizes |Tsnew − Ts| (K), all cases:**

| Configuration | steps | median | 99 % | 99.9 % | 99.99 % | largest | > 5 K | > 9 K | > 20 K |
|---|---|---|---|---|---|---|---|---|---|
| legacy | 998,050 | 0.23 | 2.8 | 4.4 | 10.6 | 145 | 435 | 123 | 32 |
| F1 + F2 | 996,268 | 0.23 | 2.9 | 4.4 | 11.5 | 210 | 442 | 141 | 40 |
| F1 + F2 + A3 (snow and ice, linear 1 K) | 987,455 | 0.24 | 3.0 | 4.9 | 16.0 | 72 | 935 | 275 | 58 |
| F1 + F2 + S1 defaults | 995,807 | 0.23 | 2.9 | 4.4 | 10.8 | 107 | 456 | 130 | 30 |
| F1 + F2 + S1 fresh 150 | 1,012,041 | 0.23 | 3.0 | 4.6 | 13.0 | 1581 | 631 | 142 | 52 |
| F1 + F2 + S1 fresh 200 | 1,005,678 | 0.23 | 2.9 | 4.5 | 12.6 | 173 | 535 | 140 | 48 |
| S1 fresh 150, τ 4800 h (no fixes) | 1,068,683 | 0.23 | 3.4 | 5.5 | 12.3 | 345 | 1786 | 158 | 60 |
| F1 + F2 + S1 fresh 150, τ 4800 h | 1,069,069 | 0.22 | 3.4 | 5.4 | 9.9 | 346 | 1709 | 121 | 52 |

- Steps beyond 20 K occur in 9–39 calls per configuration (0.01–0.04 per
  case-year). In every configuration without A3, 74–100 % of them start at
  exactly 273.15 K (the kink); with A3, 20 %. Almost all of these calls then
  fail (5 iterations without convergence) and the bracket search and
  bisection find the solution (with F1) or fall back (legacy note 5).
- **Implausible accepted temperatures** (outside 200–280 K): one episode in
  all eight configurations: F1 + F2 + S1 fresh 150, Lac des Sables station
  50 %, 19 January 1990. The first step of the day starts at 273.15 K with
  f = 21.2 K and a centred slope of 0.075; the Newton step is −284 K, to
  −11 K (negative kelvin), and the iteration converges (exit 1) on a
  spurious root of the discrete equations at −72.5 K. The closure residual
  of that step is zero (T1 = Ts). The next three steps accept 30.0 K
  (exit 1), 30.0 K (no bracket, fall-back) and 109.3 K (fall-back), with T1
  = 109 and 225 K; the day's residual is 51.3 W m⁻², the case's largest.
  Many more calls *evaluate* the budget at absurd temperatures (down to
  −1488 K; 1–6 calls per configuration below 200 K) but fail and are
  rescued by the bracket search.
- The same day in legacy: Newton oscillates ±116.7 K around 273.15 K (f is
  V-shaped there), fails, the bracket is below the start and the legacy
  fall-back (note 5) returns 273.15 K with T1 = 253.7 K: daily residual
  54.0 W m⁻² (not the case's largest, 59.8 W m⁻² on 12 February 1981).
- **Plausible changes are large in stiff packs.** The largest change from
  the starting temperature among plausible accepted solutions is 7–10 K
  with the legacy density, but 24–25 K with fresh 150 (light, insulating
  snow: one 6-hour step can move the surface by 20 K or more); the 99.99th
  percentile of the first Newton step reaches 9.6 K.
- **12 May 1973, Big Trout Lake ERA5 100 %, S1 fresh 150, τ 4800 h, no
  fixes** (the 2209 W m⁻² day; before the scored seasons, so it shows only
  in all-year closure): *not the same mechanism.* Step 3 of the day starts
  at 273.15 K; f = 0.08, 1.35 and 12.1 K at 273.15, 273.25 and 273.05 K
  (the cold albedo below the jump), so the slope is −54 and the Newton step
  0.0015 K: exit 3, and legacy accepts the evaluation at 273.05 K with
  T1 = 260.9 K (legacy note 4; F2 fixes it). The next step starts at
  260.9 K with f = −81.7 K (T1 = 342.6 K: the light pack responds by
  80 K to 126 W m⁻²); Newton steps +18.7 K, then back, and runs out of its
  5 iterations at 271.6 K, 0.03 K from converging; the solution is 10.6 K
  above the start, beyond the 9 K bracket search, so the solver falls back
  to the start and keeps T1 = 342.6 K: step residual −8992 W m⁻². With
  F1 + F2 the run's largest all-year residual is 50.0 W m⁻². No step beyond
  20 K is involved.

**Candidate designs** (traced copy, same 36 cases; limit 9 K, the bracket
search's reach, or 20 K, beyond the 99.99th percentile of steps in every
configuration):

| Design, limit | F1 + F2 + S1 fresh 150 | S1 fresh 150, τ 4800 h, no fixes | other F1 + F2 configurations | calls changed |
|---|---|---|---|---|
| (a) reject, 9 K | fixed (largest 51.3 → 28.2) | breaks: runaway to 150 K surfaces, 135 implausible calls, one run stops (overflow) | unchanged; with τ 4800 h + F1 + F2 also breaks | 35–942 |
| (a) reject, 20 K | fixed | 9 implausible calls, residual 4988 W m⁻² | unchanged | 10–42 |
| (b) clamp, 9 K | fixed | 4 implausible calls, 4765 W m⁻² | unchanged | 35–94 |
| **(b) clamp, 20 K** | **fixed** | **unchanged (largest 67.4 vs 67.3 W m⁻², scored)** | **unchanged** | **9–39** |
| (c) safeguard (clamp 9 K + bisect inside a known bracket) | fixed | 4 implausible calls | F1 + F2: largest 24.6 → 38.0 W m⁻² | 290–601 |

A limit of 9 K cuts legitimate steps in stiff (light-snow) packs; rejecting
a step hands the call to a bracket search that cannot reach a solution more
than 9 K away; the safeguard's bracket rule also acts on ordinary steps. A
clamp at 20 K changes only the steps that are beyond any change seen in a
plausible solution, never evaluates the budget below 236 K in any
configuration, and does not depend on F1.

**Chosen:** (b), a damped Newton step: a step longer than
`newton_step_limit` (20 K) is shortened to that length in the same
direction and the iteration continues. Nothing else changes: toler, dTemp,
the 5 iterations, the vanishing-derivative test, the exits, the bracket
search and bisection, the albedo, everything outside the solver. On the
19 January 1990 step the clamped iterate (253.15 K) sits on the far side of
the kink, where T1 is pinned at 273.15 K (slope exactly 1), so Newton
returns to 273.15 K and cycles; the call fails as legacy's does and the
bracket search finds the solution (269.6 K with F1). The limit therefore
needs no F1 to be safe, but with legacy's fall-back the solution below the
start is still lost (note 5): the useful configuration is F1 + F2 + F3.

## What should change

With `legacy` (the default) every output must be bit-identical (regression
test, E0 432 of 432 equal, self-test 0 differences).

Only calls with a Newton step beyond 20 K change, 9–39 calls in 36 cases
(0.01–0.04 per case-year), so most cases should be bit-identical:

- **F3 alone (F3_newton_step, against legacy):** 10 calls change, all start
  at 273.15 K and failed already; with the clamp they fail the same way and
  reach the same bracket search. Expected: every metric and closure number
  unchanged in all 36 cases (bit-identical or within rounding); at most 2
  cases with any difference.
- **F3 + F1 + F2 (F3_F1F2, against F2_newton_F1):** 11 calls, all failing
  already: bit-identical or within rounding in all 36 cases.
- **+ A3 (F3_F1F2_A3_ice, against F2_newton_F1_A3_ice):** 39 calls in 21
  cases; the diagnosis shows the largest daily residual changing in 2 cases
  by under 0.01 W m⁻². Expected: all observed metrics unchanged within
  tolerance in all 36 cases; closure within 1 %; up to about 20 cases not
  bit-identical.
- **+ S1 fresh 150 (F3_F1F2_S1_f150, against E4_S1_f150_F1F2):** the
  triggering case (Lac des Sables station 50 %): largest daily surface
  residual 51.3 → about 10 W m⁻² (the case's next-largest day), improved;
  over all cases 51.3 → 28.2 W m⁻² (Yellowknife ERA5 50 %, an exit-3 step
  at legacy's albedo jump). Its thickness and dates change only from
  19 January 1990 on (one season of 16): within tolerance (5 mm, 0.5 day on
  the case's means); at most that one case changes beyond rounding.
  The other 35 cases bit-identical or within rounding (15 limited calls).
- **+ S1 fresh 200 (F3_F1F2_S1_f200, against E4_S1_f200_F1F2):** 15 calls,
  all failing already: bit-identical or within rounding in all 36 cases.
- **Newton failures and fall-backs:** unchanged within ±0.01 per case-year;
  no fall-back with F1.
- **Diagnosis rerun with the option:** no accepted surface temperature
  outside 200–280 K, and no evaluation below about 220 K, in any
  configuration with F3; the model's solver and the traced copy agree on
  every call.
- **Runtime:** unchanged within noise (one comparison per Newton step;
  the clamped calls make the same number of evaluations or fewer).

## What would count as failure

- any change with the default `legacy`;
- an implausible accepted temperature remaining with F3 in any of the
  diagnosis configurations;
- an observed metric beyond tolerance, or a closure number degraded by more
  than 1 %, in any case of F3_F1F2, F3_F1F2_A3_ice or F3_F1F2_S1_f200 against
  their references;
- the largest residual of Lac des Sables station 50 % with fresh 150 not
  improving;
- runtime more than 5 % above F1 + F2.
