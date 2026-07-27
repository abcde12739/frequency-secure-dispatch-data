# Test-system data for frequency-secure AC dispatch in inverter-dominated power systems

This repository accompanies the paper and holds its supplementary material
(`supplementary.pdf`) together with the test-system data. The data define the
two test systems used in the case studies: modified IEEE 118-bus and IEEE
300-bus networks in which part of the synchronous generation is replaced by
inverter-based resources (IBRs). Only the input data needed to reconstruct the
systems are included; results and code are not.

## Layout

```
case_data/
├── supplementary.pdf                  supplementary material for the paper
├── ibr_capability.csv                 shared voltage-dependent IBR derating curve
├── ieee118/
│   ├── pglib_opf_case118_ieee.m       base network (MATPOWER)
│   ├── synchronous_generators.csv     synchronous fleet (steady-state model)
│   ├── inverter_based_resources.csv   IBR fleet
│   ├── penetration_levels.csv         IBR penetration levels
│   ├── sg_dynamic_parameters.csv      synchronous-machine dynamic parameters
│   └── contingencies.csv              N-1 contingency list
├── ieee300/                           same six files
└── multi_period/                      inputs for the multi-period study (IEEE 118)
    ├── bess.yaml                      storage fleet (energy shifting + FFR)
    ├── inverter_based_resources_51pct.csv   IBR fleet at 51.2% penetration
    ├── forecast_error_ar1.yaml        AR(1) forecast-error model
    └── profiles/                      seven representative days of load/wind/solar
```

## Supplementary material

`supplementary.pdf` is the supplementary document that accompanies the paper. It
provides the derivations, parameter tables, and additional results referenced
from the main text.

## Base networks

`pglib_opf_case118_ieee.m` and `pglib_opf_case300_ieee.m` carry the IEEE 118-
and 300-bus cases from the PGLib-OPF benchmark library, in MATPOWER format
(`mpc.version = 2`, `baseMVA = 100`); the electrical data (buses, branches,
loads, shunts, thermal ratings) are those of the PGLib-OPF cases. The generator fleet used in this study is
given by the CSV files below and supersedes the generator records in the `.m`
file.

## Synchronous generators

`synchronous_generators.csv`, one row per unit:

| column | meaning |
|---|---|
| `gen_id`, `bus` | identifier and bus number |
| `p_max`, `p_min` | active-power limits (MW) |
| `q_max`, `q_min` | reactive-power limits (MVAr) |
| `H_g` | inertia constant (s) |
| `R_g`, `K_droop` | governor droop and droop gain |
| `r_gov_max` | governor reserve limit (MW) |
| `cost_p_a`, `cost_p_b`, `cost_p_c` | quadratic active-power cost coefficients |
| `cost_r` | reserve cost coefficient |

## Inverter-based resources

`inverter_based_resources.csv`, one row per unit:

| column | meaning |
|---|---|
| `ibr_id`, `bus` | identifier and bus number |
| `type` | `GFM` (grid-forming) or `GFL` (grid-following) |
| `S_rated_MVA` | apparent-power rating (MVA) |
| `p_max`, `p_min`, `q_max`, `q_min` | active/reactive limits (MW, MVAr) |
| `in_fr_set` | participates in frequency reserve (`True`/`False`) |
| `fr_mode` | frequency-response mode (`VSG` or `droop`) |
| `hvirt_max` | maximum virtual inertia (s) |
| `tau_resp` | response time constant (s) |
| `delivery_time`, `sustain_time` | reserve delivery and sustain times (s) |
| `energy_available` | energy available for reserve (p.u.) |
| `cost_p_a`, `cost_p_b`, `cost_p_c`, `cost_r` | cost coefficients (as above) |
| `penetration_level` | penetration level at which the unit enters |
| `replaces_sg` | synchronous generator replaced by this unit |

## Penetration levels

Each system is studied at three IBR penetration levels, listed in
`penetration_levels.csv` (`penetration_level_pct`, the exact share `actual_pct`,
and the cumulative installed IBR capacity `cumulative_ibr_mw`):

- IEEE 118: 34 %, 70 %, 97 %
- IEEE 300: 31 %, 50 %, 75 %

IBRs are added cumulatively. A unit with `penetration_level = L` is present at
level `L` and above. At a given level the active fleet is every IBR whose
`penetration_level` is at or below that level, together with every synchronous
generator except those named in the `replaces_sg` column of the active IBRs.

## Synchronous-machine dynamic parameters

`sg_dynamic_parameters.csv` gives the electromechanical and control parameters
per generator, following standard synchronous-machine, AVR, and governor
conventions: rated power `Sn_MVA`; inertia `H_s` and damping `D_pu`;
armature and axis reactances (`R_s`, `X_d`, `X_q`, `X_d_prime`, `X_q_prime`,
`X_d_pp`, `X_q_pp`, `X_l`); open-circuit time constants (`T_d0_prime`,
`T_q0_prime`, `T_d0_pp`, `T_q0_pp`); saturation coefficients (`S_10`, `S_12`);
AVR settings (`avr_Ka`, `avr_Ta`, `avr_Tb`); and governor settings (`gov_R`,
`gov_T1`, `gov_T2`, `gov_T3`, `gov_Dt`). IBR frequency-response parameters are
in `inverter_based_resources.csv`.

## IBR capability curve

`ibr_capability.csv` is a piecewise-linear, voltage-dependent current-derating
curve shared by both systems. On segment `seg_id`, for squared bus voltage
`W` in `[w_low, w_high]` (per unit), the derating factor `t` satisfies
`t <= a_coeff * W + b_coeff`.

## Contingencies

`contingencies.csv` lists the single-unit outages (N-1):

| column | meaning |
|---|---|
| `cont_id` | identifier |
| `cont_type` | contingency type (`unit_trip`) |
| `lost_unit_id`, `unit_type` | tripped unit and its type (`SG` or `IBR`) |
| `valid_from` | lowest penetration level at which the contingency applies (`base` = all levels) |

A contingency applies only while its unit is in service. When a synchronous
generator is replaced by an IBR at a given penetration level, it is no longer
tripped at that level and above, and the replacing IBR's outage (with
`valid_from` set to that level) takes its place.

## Multi-period study

`multi_period/` holds the additional inputs for the multi-period study on the
IEEE 118 system, run at a single 51.2 % IBR penetration with battery storage
that provides both energy shifting and fast frequency response (FFR). The
multi-period case retains 30 % of the displaced synchronous-condenser inertia.

`inverter_based_resources_51pct.csv` lists the seven IBRs online at 51.2 %
penetration (3334 MW cumulative): the four units of the 34 % fleet (`IBR_G30a`,
`IBR_G30b`, `IBR_G37`, `IBR_G5`) plus `IBR_G12`, `IBR_G45a`, and `IBR_G45b`. Its
columns match `ieee118/inverter_based_resources.csv` except that
`penetration_level` is dropped, since every row belongs to this single fleet.
The online synchronous generators are all those except the ones named in the
`replaces_sg` column.

`bess.yaml` defines the storage fleet: two stations at buses 26 and 80, each
775 MW / 6000 MWh (1550 MW / 12000 MWh in total). Each station carries a
converter rating (`s_rating_mva`), round-trip efficiency (`eta_charge`,
`eta_discharge`), a minimum state-of-charge floor (`e_min_fraction`), a
daily-neutral initial state of charge (`e_initial_fraction`), and zero reactive
base power (`q_base_mvar`). The per-interval self-discharge fraction
(`self_discharge_per_interval`) equals a 5 % standing loss over a 30-day month.
Both stations also provide FFR up to `ffr_max_mw` (80 MW each, 160 MW in total),
with a response time constant `ffr_response_delay_s`, a sustained-delivery
duration `ffr_hold_minutes`, and an offer price `ffr_cost_per_mw_h`. The two
stations receive equal base-power and FFR participation. The interval length is
5 minutes, matching the profiles.

`profiles/` contains seven representative days of CAISO-derived load, wind, and
solar at 5-minute resolution, spanning spring, summer, and autumn and commonly
scaled to the annual-peak day. Each file `caiso_YYYY-MM-DD.csv` has columns: `ts_gmt`
(timestamp); day-ahead load, wind, and solar forecasts (`*_forecast_dam_mw`);
real-time wind and solar forecasts (`*_forecast_rtpd_mw`); realized values
(`*_actual_mw`); and the `scale_factor` applied to the raw series.

`forecast_error_ar1.yaml` gives the AR(1) forecast-error model for load, wind,
and solar (lag-1 autocorrelation `rho` and innovation standard deviation `sigma`).

## Conventions

- Active power in MW, reactive power in MVAr, apparent power in MVA.
- Voltage magnitudes and `W` (squared voltage magnitude) are in per unit on each
  bus's nominal voltage base.
- The system power base is 100 MVA (`baseMVA` in the `.m` files).
