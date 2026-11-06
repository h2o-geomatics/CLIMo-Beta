# CLIMo Classic: model description

This page documents the equations implemented in CLIMo Classic and where each
one lives in the code. Symbols follow Duguay et al. (2003). Fluxes are in
W m⁻² and positive downward.

## Structure

- One snow layer (when snow is thicker than 1 mm) and ice layers, 5 layers
  in total (`NLAY`), with temperatures at the layer interfaces.
- Time step of 6 h (`NTSPD = 4` steps per day). Daily forcing is linearly
  interpolated from the previous day's value to the current day's value.
- When ice is present, the surface temperature is found by Newton–Raphson
  iteration (bisection as a fallback) so that the surface energy budget and
  heat conduction agree (`model.solve_surface_temperature`). As in legacy
  CLIMo, a solution bracketed below the starting temperature falls back to
  the start ([legacy note 5](legacy_notes.md)); when Newton–Raphson
  converges on its step size (about 15 % of the ice time steps) or its
  derivative vanishes, the evaluation 0.1 K below the estimate is accepted
  ([legacy note 4](legacy_notes.md)); and the Newton step is unbounded
  ([legacy note 11](legacy_notes.md)). The option `fixes = "on"` corrects
  all three together: the downward bracket is bisected as one above, the
  converged estimate is accepted (a vanishing derivative goes to the
  bisection), and a step longer than 20 K is shortened to 20 K (see
  [options.md](options.md#fixes) and
  [formulations.md, F1–F3](formulations.md#the-fixes-option-f1--f2--f3)).
- When there is no ice, the lake is a mixed layer of fixed depth whose heat
  content changes with the surface energy budget (`growth.open_water`).

## Initial state and spin-up — `model.py`, `forcing.py`

Every run starts on 1 January of the first spin-up year from a fixed state
(`constants.py`): ice 1 m thick (`INITIAL_ICE`) under 5 cm of snow
(`INITIAL_SNOW`, at the dry snow density), slab temperatures varying linearly
from 245 K at the surface (`INITIAL_TSURF`) to the freezing point at the
base, and no heat stored in the mixed layer ($H_{mix} = 0$). The spin-up
years (`Parameters.spinup_years`, default 1, as in the legacy code) are
copies of the start year, dated start − N to start − 1, prepended to the
forcing by `forcing.add_spinup_year`. Days and ice events before the start
year are dropped from the outputs. `model.run` issues a `SpinUpWarning`
when no break-up happens before the start year, i.e. the initial ice was
never melted and the first seasons still depend on it. How many spin-up
years are needed is discussed in
[how_it_runs.md](how_it_runs.md#spin-up-years).

## Heat conduction — `conduction.py`

$$
\rho C_p \frac{\partial T}{\partial t} = \frac{\partial}{\partial z}\left(k \frac{\partial T}{\partial z}\right) + F_{sw} i_0 (1-\alpha) \kappa e^{-\kappa z}
$$

Solved with a generalized Crank–Nicolson scheme (`THETA = 1`, fully implicit)
and a tridiagonal solver (Press et al., 1986). Boundary conditions:

- top: the net surface flux $F_0$, or the melting temperature (273.15 K) when
  the surface is melting and the flux is into the surface;
- bottom: the freezing temperature (273.15 K).

When the ice is thinner than 0.2 m (`HLAYMIN`), a linear temperature profile
with the conductivity of pure ice is used instead.

**Snow** (Duguay et al., 2003, Eqs. 8–9, after Ebert and Curry, 1993):

$$
k_s = 2.845\times10^{-6}\rho_s^2 + 2.7\times10^{-4}\thinspace 2^{(T-233)/5}
$$

$$
\rho C_p = \rho_s (92.88 + 7.364\thinspace T)
$$

**Ice** (Ebert and Curry, 1993, with nominal salinity $S = 1$):

$$
k_i = 2.034 + 0.1172\thinspace S/(T - T_m), \quad \rho C_{p,i} = 1.883\times10^6 + 1.715\times10^7 S/(T-T_m)^2
$$

limited to $k_i \ge 2.034/5$ and $\rho C_{p,i} \le 100 \times 1.883\times10^6$.

**Penetrating short-wave**: a fraction $i_0 = 0.17$ of the absorbed short-wave
penetrates the surface when snow is thinner than 0.1 m, and is absorbed with
the bulk extinction coefficient $\kappa = 1.5$ m⁻¹.

## Surface energy budget — `surface.py`

$$
F_0 = F_{lw} - \varepsilon\sigma T_s^4 + (1-\alpha)(1-i_0)F_{sw} + F_{lat} + F_{sens}
$$

| Term | Formulation | Reference |
|---|---|---|
| $F_{lw}$ | $\varepsilon\sigma T_a^4 \cdot 0.7855\thinspace (1 + 0.2232\thinspace c^{2.75})$ | Maykut and Church (1973) |
| $F_{sw}$ clear sky | $S_0\cos^2 Z / [(\cos Z + 1)\thinspace e_a 10^{-5} + \cos Z + 0.046]$, hourly over the day | Shine (1984) |
| $F_{sw}$ cloudy | $(53.5 + 1274.5\cos Z)\sqrt{\cos Z} / [1 + 0.139(1 - 0.9345\alpha)\tau]$, $\tau = 7$ | Shine (1984); Curry and Ebert (1992) |
| total $F_{sw}$ | $(1-c)F_{clear} + c\thinspace F_{cloudy}$ | Shine (1984) |
| $F_{sens}$, $F_{lat}$ | bulk formulae, $C_T = 1.3\times10^{-3}$ with Richardson-number stability correction ($b' = 4.7$) | Ebert and Curry (1993); Louis (1979) |
| $q_{sat}$ | polynomial in $T$ | Maykut (1978) |

$c$ is cloud fraction, $e_a$ vapour pressure (Pa), $S_0 = 1367$ W m⁻²
(Iqbal, 1983).

**Albedo** (Duguay et al., 2003, Eq. 10):

| Surface | Albedo |
|---|---|
| open water ($h_i < 0.001$ m) | 0.05 |
| cold ice | $0.44\thinspace h_i^{0.28} + 0.08$ (Maykut, 1982) |
| melting ice | $\min(0.075\thinspace h_i^2 + 0.15,\ 0.55)$ (Heron and Woo, 1994; Langleben, 1971) |
| snow ($h_s > 0.1$ m) | 0.75 cold, 0.65 melting |
| thin snow ($h_s \le 0.1$ m) | linear between ice and snow: $\alpha_i + h_s(\alpha_s - \alpha_i)/0.1$ |

"Melting" means a surface temperature at 273.15 K: the albedos jump from
their cold to their melting values there. The surface temperature is solved
for every time step and never stored above 273.15 K (a melting surface is
held there). The options `melt_albedo_transition` and
`melt_albedo_transition_ice` (spec A3, `linear` or `logistic`) let the snow
and the ice albedo approach their melting values over an interval below the
melting point instead (default 1 K, as in the CCSM3 sea-ice albedo); see
[formulations.md](formulations.md#a3-smooth-melt-state-albedo-transition).
The option
`thin_snow_albedo = "exponential"` (spec A1) replaces the thin-snow line by
$\alpha_i + (\alpha_s - \alpha_i)[1 - \exp(-h_s/h_\alpha)]$; see
[formulations.md](formulations.md#a1-thin-snow-albedo) for its sources,
the science and the validation. The option `ice_type_albedo` (spec A2,
`surface_threshold` or `thickness_weighted`) makes the ice albedo depend on
the snow ice at the top of the ice: snow-free ice with more than 0.05 m of
snow ice on top takes 0.75 cold, 0.55 melting (or a share-weighted value);
see [formulations.md](formulations.md#a2-clear-ice-versus-snow-ice-albedo).

## Growth, melt and snow ice — `growth.py`

- **Bottom**: $\Delta h = -(F_{bottom} + F_w)\thinspace \Delta t / (\rho_i L_f)$, where
  $F_w$ is the heat flux from the water (zero, plus heat stored in the mixed
  layer).
- **Surface melt**: when the surface is at the melting point, the flux
  divergence melts snow first, then ice.
- **Snow ice**: when the snow load submerges the ice surface
  ($m_s > (\rho_w - \rho_i)\thinspace m_i/\rho_i$), the submerged snow floods and
  freezes, and the latent heat of the pore water warms the top ice layer.
  The outputs `SnowIce` and `CongelationIce` are the ice grown per season;
  the snow ice still at the top (`State.surface_snow_ice`: grown snow ice
  minus melt at the top, at most the ice thickness) is kept only for the
  A2 option and changes nothing in legacy runs.
- **Snow density**: `rho_snow_cold` when the mean snow-layer temperature is
  below 0 °C, otherwise `rho_snow_warm` (Duguay et al., 2003), chosen anew
  at every time step; the wet value is almost never used
  ([legacy note 6](legacy_notes.md)). The option
  `snow_density_scheme = "bulk_relaxation"` (`climo/snow.py`) makes it a
  state variable of the snow layer instead: new snow is mixed in (with
  legacy's mass) and the density relaxes towards `rho_snow_cold`, rising
  towards `rho_snow_warm` within 1 K of melting, never downwards (see
  [formulations.md](formulations.md#s1-bulk-snow-density-evolution)).
- **Open water**: the mixed-layer temperature is
  $T_{mix} = T_f + H_{mix}/(\rho c_w d_{mix})$; ice forms when it falls below
  freezing.
- **Freeze-up / break-up**: the first day with ice mass above / below
  0.01 kg m⁻².

## References

- Curry, J. A., and Ebert, E. E. (1992). Annual cycle of radiation fluxes over the Arctic Ocean. *Journal of Climate*, 5, 1267–1280.
- Duguay, C. R., et al. (2003). *Hydrological Processes*, 17, 3465–3483. https://doi.org/10.1002/hyp.1394
- Ebert, E. E., and Curry, J. A. (1993). An intermediate one-dimensional thermodynamic sea ice model for investigating ice-atmosphere interactions. *Journal of Geophysical Research*, 98(C6), 10085–10109.
- Flato, G. M., and Brown, R. D. (1996). *Journal of Geophysical Research*, 101(C10), 25767–25777.
- Heron, R., and Woo, M.-K. (1994). Decay of a High Arctic lake-ice cover: observations and modelling. *Journal of Glaciology*, 40(135), 283–292.
- Iqbal, M. (1983). *An Introduction to Solar Radiation*. Academic Press.
- Langleben, M. P. (1971). Albedo of melting sea ice in the southern Beaufort Sea. *Journal of Glaciology*, 10(58), 101–104.
- Louis, J.-F. (1979). A parametric model of vertical eddy fluxes in the atmosphere. *Boundary-Layer Meteorology*, 17, 187–202.
- Maykut, G. A. (1978). Energy exchange over young sea ice in the central Arctic. *Journal of Geophysical Research*, 83(C7), 3646–3658.
- Maykut, G. A. (1982). Large-scale heat exchange and ice production in the central Arctic. *Journal of Geophysical Research*, 87(C10), 7971–7984.
- Maykut, G. A., and Church, P. E. (1973). Radiation climate of Barrow, Alaska, 1962–66. *Journal of Applied Meteorology*, 12, 620–628.
- Press, W. H., et al. (1986). *Numerical Recipes*. Cambridge University Press.
- Shine, K. P. (1984). Parametrization of the shortwave flux over high albedo surfaces as a function of cloud thickness and surface albedo. *Quarterly Journal of the Royal Meteorological Society*, 110, 747–764.
