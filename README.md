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

### I. Model summary (MILP appendix)

Each player's best response and the greenfield planner solve a **mixed-integer linear program** over planning years \(t \in \{1,\ldots,T\}\) and representative periods \(h \in \mathcal{H}\). The planner minimizes total system cost; players maximize discounted profit with rivals' capacity fixed and \((\lambda, \pi)\) from market clearing.

Core constraints include:

- **Energy balance** — dispatch, storage net injection, and non-served energy meet incremental demand \(\Delta D^E_{t,h}\).
- **Capacity adequacy** — live capacity (EOL-adjusted) plus SoH-weighted storage meets the planning reserve band; dual \(\pi_t\) is the capacity payment.
- **Emissions cap** — \(\sum_h w_h \sum_{k \in \mathcal{G}^f} \varepsilon_k P^k_{t,h} \le \Gamma_t\).
- **Vintage / EOL** — live capacity sums only cohorts within lifetime \(L_k\); SLB SoH fade and retirement floors enforced.

The full mathematical formulation:

Each player's best response and the greenfield planner benchmark are **mixed-integer linear programs** over horizon \(t \in \{1,\ldots,T\}\) and representative periods \(h \in \mathcal{H}\). We state the planner (cost-minimizing) form; the player best response replaces the objective by \(\max \sum_t \delta_t \, \pi^p_t\) with rivals' capacities fixed and \((\lambda, \pi)\) from market clearing.

**Decision variables**

- \(x^k_t \in \mathbb{Z}_{\ge 0}\): integer build units of technology \(k\) in year \(t\)
- \(\bar K^k_t = \mathrm{CU}^k \sum_{\tau=\max(1,\,t-L_k+1)}^{t} x^k_\tau\): **live** cumulative capacity (vintages past \(L_k\) drop out)
- \(\bar P^s_t, \bar E^s_t\): live storage power and energy (defined analogously)
- \(P^k_{t,h} \ge 0\): dispatch; \(\mathrm{ch}^s_{t,h}, \mathrm{dis}^s_{t,h}, \mathrm{soc}^s_{t,h} \ge 0\): storage charge/discharge/state of charge; \(\mathrm{SoH}^s_t\): state of health
- \(\mathrm{ns}_{t,h} \ge 0\): non-served energy; \(E^k_t = \sum_h w_h P^k_{t,h}\): annual energy

**Objective (planner — minimize total cost)**

$$
\min \sum_t \delta_t \Big[
\sum_k \mathrm{CRF}_k I^k \bar K^k_t
+ \sum_k (c^k + \tau_t \varepsilon_k) E^k_t
+ \sum_k O^k \bar K^k_t
- \sum_{r \in \mathcal{R}} \sigma^{\mathrm{re}}_t \mathrm{CRF}_r I^r \bar K^r_t
+ \sum_s (C^{\mathrm{Inv}}_{s,t} + C^{\mathrm{Gen}}_{s,t} + C^{\mathrm{O\&M}}_{s,t})
- \sum_s \sigma^s_t \bar P^s_t
- \sum_k \rho_k \mathrm{CU}^k x^k_{t-L_k}
+ \mathrm{VOLL} \sum_h w_h \mathrm{ns}_{t,h}
\Big]
$$

**Subject to** (for all \(t, h\)):

| Eq. | Constraint | Dual |
|-----|------------|------|
| (bal) | \(\sum_k P^k_{t,h} + \sum_s (\mathrm{dis}^s_{t,h} - \mathrm{ch}^s_{t,h}) + \mathrm{ns}_{t,h} = \Delta D^E_{t,h}\) | \(\lambda_{t,h}\) |
| (adq) | \((1+\mathrm{RM}^{\min}_t)\Delta\hat D_t \le \sum_k \mathrm{cc}^k \bar K^k_t + \sum_s \mathrm{cc}^s \mathrm{SoH}^s_t \bar P^s_t\) | \(\pi_t\) |
| (adqU) | \(\sum_k \mathrm{cc}^k \bar K^k_t + \sum_s \mathrm{cc}^s \mathrm{SoH}^s_t \bar P^s_t \le (1+\mathrm{RM}^{\max}_t)\Delta\hat D_t\) | — |
| (pcap) | \(0 \le P^k_{t,h} \le \bar K^k_t\); \(P^r_{t,h} \le \gamma^r_{t,h} \bar K^r_t\) for \(r \in \mathcal{R}\) | — |
| (cf) | \(\sum_h w_h P^k_{t,h} \le \mathrm{CF}^k \bar K^k_t H\) | — |
| (soc) | \(\mathrm{soc}^s_{t,h} = \mathrm{soc}^s_{t,h-1} + \eta^s_c \mathrm{ch}^s_{t,h} - \mathrm{dis}^s_{t,h}/\eta^s_d\) | — |
| (slim) | \(0 \le \mathrm{soc}^s_{t,h} \le \mathrm{SoH}^s_t \bar E^s_t\); \(\mathrm{ch}^s_{t,h}, \mathrm{dis}^s_{t,h} \le \bar P^s_t\) | — |
| (soh) | \(\mathrm{SoH}^s_t = \mathrm{SoH}^s_0 - \alpha^s \sum_{t' \le t} \sum_h w_h (\mathrm{ch}^s_{t',h} + \mathrm{dis}^s_{t',h})\) | — |
| (cap) | \(\sum_h w_h \sum_{k \in \mathcal{G}^f} \varepsilon_k P^k_{t,h} \le \Gamma_t\) | — |
| (vintage) | \(\bar K^k_t = \mathrm{CU}^k \sum_{\tau=\max(1,\,t-L_k+1)}^{t} x^k_\tau\); \(\mathrm{SoH}^s_t \ge \underline{\mathrm{SoH}}_s\) | — |
| (int) | \(\bar P^{\mathrm{SLB}}_t \le \overline{\mathrm{FS}}_t\); \(0 \le x^k_t \le \bar x^k\); \(x^k_t \in \mathbb{Z}_{\ge 0}\) | — |

**Notes**

- \(H\): hours per year; \(\eta^s_c, \eta^s_d\): charge/discharge efficiencies; \(\overline{\mathrm{FS}}_t\): annual SLB feedstock cap.
- Lifetimes \(L_k = \min(L^{\mathrm{cal}}_k, L^{\mathrm{cyc}}_k)\) enter through the vintage window (implementation uses \(L_k = L^{\mathrm{cal}}_k\) with aggregate SoH).
- A cohort built in year \(\tau\) leaves \(\bar K^k_t\) after \(L_k\) years; capex is annualized by \(\mathrm{CRF}_k\); salvage credit \(\rho_k \mathrm{CU}^k x^k_{t-L_k}\) applies in the EOL year.
- If adequacy still binds after a vintage retires, replacement is **endogenous** (new \(x^k_t > 0\)).
- In the planner, the full SoC recursion **(soc)** is retained; clearing and the game use a reduced annual discharge budget \(\sum_h w_h \mathrm{dis}^s_{t,h} \le \mathrm{SoH}^s_t \bar P^s_t d_s\).
- Energy dual \(\lambda\) comes from **(bal)**; capacity price \(\pi\) follows the CONE curve in the market/game layer (planner uses the adequacy shadow price).
- Solved with **Gurobi** (HiGHS fallback); the game calls one best response per player per BNE iteration.

See **`Manuscript_Ver06.tex`** (Appendix A) for equation labels and full notation.



