# SLB_AIDC_Game

This repository provides supplementary materials and reproducible resources for the paper:

**A Stackelberg–Nash Framework for Second-Life Battery Investment under AI Data-Center Load Growth and Carbon Policy**


## Authors

**Rouzbeh Haghighi**†, **Ali Hassan**†, **Sina Mohammadi**†, **Marcus Chen I Wada**, **Wencong Su**\*  
Department of Electrical and Computer Engineering, University of Michigan–Dearborn, Dearborn, MI, USA  

† Co-first authors (equal contribution)  
\* Corresponding author: Wencong Su

**Paper link:** (to be added)

## Description

This repository includes mathematical formulations, tables, and figures discussed in the paper. It aims to provide additional resources and data to support the research findings and facilitate further studies.

## Contents

### I. MILP formulation

Each player's best response and the greenfield planner benchmark are **mixed-integer linear programs** over planning years $t \in \{1,\ldots,T\}$ and representative periods $h \in \mathcal{H}$. We state the planner (cost-minimizing) form below; the player best response replaces the objective by $\max \sum_t \delta_t \, \pi^p_t$ with rivals' capacities fixed and market prices $(\lambda,\pi)$ from clearing.

#### Decision variables

| Symbol | Description |
|--------|-------------|
| $x^k_t \in \mathbb{Z}_{\ge 0}$ | Integer build units of technology $k$ in year $t$ |
| $\bar K^k_t = \mathrm{CU}^k \sum_{\tau=\max(1,\,t-L_k+1)}^{t} x^k_\tau$ | **Live** cumulative capacity (only assets not yet at EOL; retired vintages drop out) |
| $\bar P^s_t,\, \bar E^s_t$ | Live storage power and energy (defined analogously) |
| $P^k_{t,h} \ge 0$ | Dispatch |
| $\mathrm{ch}^s_{t,h},\, \mathrm{dis}^s_{t,h},\, \mathrm{soc}^s_{t,h} \ge 0$ | Storage charge, discharge, and state of charge |
| $\mathrm{SoH}^s_t$ | Storage state of health |
| $\mathrm{ns}_{t,h} \ge 0$ | Non-served energy |
| $E^k_t = \sum_h w_h P^k_{t,h}$ | Annual energy |

#### Objective (planner — minimize total cost)

**Eq. milp-obj**

$$
\min \sum_t \delta_t \Big[
\sum_k \mathrm{CRF}_k I^k \bar K^k_t
+ \sum_k (c^k + \tau_t \varepsilon_k) E^k_t
+ \sum_k O^k \bar K^k_t
+ \sum_{r \in \mathcal{R}} \big({-}\sigma^{\mathrm{re}}_t \mathrm{CRF}_r I^r \bar K^r_t\big)
+ \sum_s \big(C^{\mathrm{Inv}}_{s,t} + C^{\mathrm{Gen}}_{s,t} + C^{\mathrm{OM}}_{s,t}\big)
+ \sum_s \big({-}\sigma^s_t \bar P^s_t\big)
+ \sum_k \big({-}\rho_k \mathrm{CU}^k x^k_{t-L_k}\big)
+ \mathrm{VOLL} \sum_h w_h \mathrm{ns}_{t,h}
\Big]
$$

#### Subject to (for all $t,h$)

| Eq. | Constraint | Dual |
|-----|------------|------|
| milp-bal | $\sum_k P^k_{t,h} + \sum_s (\mathrm{dis}^s_{t,h} - \mathrm{ch}^s_{t,h}) + \mathrm{ns}_{t,h} = \Delta D^E_{t,h}$ | $\lambda_{t,h}$ |
| milp-adq | $(1+\mathrm{RM}^{\min}_t)\Delta\hat D_t \le \sum_k \mathrm{cc}^k \bar K^k_t + \sum_s \mathrm{cc}^s \mathrm{SoH}^s_t \bar P^s_t$ | $\pi_t$ |
| milp-adqU | $\sum_k \mathrm{cc}^k \bar K^k_t + \sum_s \mathrm{cc}^s \mathrm{SoH}^s_t \bar P^s_t \le (1+\mathrm{RM}^{\max}_t)\Delta\hat D_t$ | — |
| milp-pcap | $0 \le P^k_{t,h} \le \bar K^k_t$; $P^r_{t,h} \le \gamma^r_{t,h} \bar K^r_t$ for $r \in \mathcal{R}$ | — |
| milp-cf | $\sum_h w_h P^k_{t,h} \le \mathrm{CF}^k \bar K^k_t H$ | — |
| milp-soc | $\mathrm{soc}^s_{t,h} = \mathrm{soc}^s_{t,h-1} + \eta^s_c \mathrm{ch}^s_{t,h} - \mathrm{dis}^s_{t,h}/\eta^s_d$ | — |
| milp-slim | $0 \le \mathrm{soc}^s_{t,h} \le \mathrm{SoH}^s_t \bar E^s_t$; $\mathrm{ch}^s_{t,h}, \mathrm{dis}^s_{t,h} \le \bar P^s_t$ | — |
| milp-soh | $\mathrm{SoH}^s_t = \mathrm{SoH}^s_0 - \alpha^s \sum_{t' \le t} \sum_h w_h (\mathrm{ch}^s_{t',h} + \mathrm{dis}^s_{t',h})$ | — |
| milp-cap | $\sum_h w_h \sum_{k \in \mathcal{G}^f} \varepsilon_k P^k_{t,h} \le \Gamma_t$ | — |
| milp-eol | $\bar K^k_t = \mathrm{CU}^k \sum_{\tau=\max(1,\,t-L_k+1)}^{t} x^k_\tau$; $\mathrm{SoH}^s_t \ge \underline{\mathrm{SoH}}_s$ | — |
| milp-int | $\bar P^{\mathrm{SLB}}_t \le \overline{\mathrm{FS}}_t$; $0 \le x^k_t \le \bar x^k$; $x^k_t \in \mathbb{Z}_{\ge 0}$ | — |

#### Notes

$H$ is hours per year; $\eta^s_c, \eta^s_d$ are charge/discharge efficiencies; $\overline{\mathrm{FS}}_t$ is the annual SLB feedstock cap. Lifetimes $L_k = \min(L^{\mathrm{cal}}_k, L^{\mathrm{cyc}}_k)$ enter through the EOL window **milp-eol** (set to $L_k = L^{\mathrm{cal}}_k$ here, with aggregate $\mathrm{SoH}$; the cycle-life leg is a refinement), so a unit built in year $\tau$ leaves $\bar K^k_t$ after $L_k$ years. Capex is annualized by $\mathrm{CRF}_k$ over $L_k$; a salvage credit $\rho_k \mathrm{CU}^k x^k_{t-L_k}$ is taken on capacity retiring in year $t$ (zero when $t - L_k \lt 1$); the $\mathrm{SoH}$ floor $\underline{\mathrm{SoH}}_s$ enforces retirement. Replacement is **endogenous**: if **milp-adq** still binds after a cohort reaches EOL, the program rebuilds.

Storage state-of-charge **milp-soc** cycles within each representative block in the planner; representative-period clearing and the game instead use a reduced dispatch with an annual discharge (net-energy) budget in place of the SoC recursion:

$$
\sum_h w_h \mathrm{dis}^s_{t,h} \le \mathrm{SoH}^s_t \bar P^s_t d_s
$$

where $d_s$ is storage duration. The cost terms $C^{\mathrm{Inv}}_{s,t}$, $C^{\mathrm{Gen}}_{s,t}$, and $C^{\mathrm{OM}}_{s,t}$ (investment, generation, and O&amp;M) apply over live pre-EOL capacity. **milp-bal** and **milp-adq** carry the duals $\lambda$ and $\pi$ that price energy and capacity; in the market and game layer, $\pi$ follows the CONE curve rather than the planner adequacy shadow price.

The program is linear with integer build variables and is solved with **Gurobi** (HiGHS fallback). The game-layer best response is the continuous-build, year-by-year relaxation of the equilibrium model; BNE diagonalization calls it once per player per iteration.
