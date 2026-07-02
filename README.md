# SLB_AIDC_Game

This repository provides supplementary materials and reproducible resources for the paper:

**A Stackelberg–Bayesian Capacity-Market Game of Carbon Regulation and Second-Life Battery Investment under AI Data-Center Load Growth**

Submitted to the North American Power Symposium (NAPS) 2026.

## Authors

**Rouzbeh Haghighi**†, **Ali Hassan**†, **Sina Mohammadi**†, **Marcus Chen I Wada**, **Wencong Su**\*
Department of Electrical and Computer Engineering, University of Michigan–Dearborn, Dearborn, MI, USA

† Co-first authors (equal contribution)
\* Corresponding author: Wencong Su

**Paper link:** (to be added)

## Description

This repository includes the full mathematical formulation, parameter tables, and supplementary derivations behind the paper. The manuscript itself is space-limited, so the complete MILP, solver settings, and the equilibrium existence/uniqueness argument are documented here in full to support reproducibility and review.

## Contents

- [I. MILP Formulation](#i-milp-formulation)
- [II. Representative Periods and Capacity Factors](#ii-representative-periods-and-capacity-factors)
- [III. Biomass Emission Factor](#iii-biomass-emission-factor)
- [IV. Regulator (Leader) Problem](#iv-regulator-leader-problem)
- [V. Equilibrium Existence and Uniqueness](#v-equilibrium-existence-and-uniqueness)
- [Reference](#reference)

---

## I. MILP Formulation

Each player's best response and the greenfield planner benchmark are **mixed-integer linear programs** over planning years $t \in \{1,\ldots,T\}$ and representative periods $h \in \mathcal{H}$. We state the planner (cost-minimizing) form below; the player best response replaces the objective with $\max \sum_t \delta_t \, \Pi^p_t$ (the player's own payoff, generator/renewable/storage form) with rivals' capacities fixed and market prices $(\lambda,\pi)$ taken from clearing.

### Decision variables

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

### Objective (planner — minimize total cost)

**Eq. milp-obj**
<img width="600" alt="image" src="https://github.com/user-attachments/assets/e2aed891-ad78-4dbb-9798-09a5b9db5c2e" />

Subsidies are applied as multiplicative capex reductions, `(1-σ)`, matching the follower payoffs (generator/renewable/storage equations in the paper) rather than as separate additive credits — this keeps the planner objective and the equilibrium payoffs consistent. $\mathrm{RV}^k_t$ is a per-year residual-value credit on live (not-yet-retired) capacity, additional to the one-time salvage credit $\rho_k$ taken at retirement.

### Subject to (for all $t,h$)
<img width="600" alt="image" src="https://github.com/user-attachments/assets/ad770f57-8dee-47f3-8930-09742da67643" />


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
| milp-vintage | $\bar K^k_t = \mathrm{CU}^k \sum_{\tau=\max(1,\,t-L_k+1)}^{t} x^k_\tau$; $\mathrm{SoH}^s_t \ge \underline{\mathrm{SoH}}_s$ | — |
| milp-int | $\bar P^{\mathrm{SLB}}_t \le \overline{\mathrm{FS}}_t;\; 0 \le x^k_t \le \bar x^k;\; x^k_t \in \mathbb{Z}_{\ge 0}$ | — |

No system-wide emissions cap is imposed. Carbon is priced solely through the tax term $(c^k+\tau_t\varepsilon_k)E^k_t$ in **milp-obj** — the planner counterpart of the per-generator tax in the follower payoffs — so system emissions are an equilibrium outcome rather than a hard constraint.

### Notes

$H$ is hours per year; $\eta^s_c, \eta^s_d$ are charge/discharge efficiencies; $\overline{\mathrm{FS}}_t$ is the annual SLB feedstock cap. Lifetimes $L_k = \min(L^{\mathrm{cal}}_k, L^{\mathrm{cyc}}_k)$ enter through the vintage window **milp-vintage** (set to $L_k = L^{\mathrm{cal}}_k$ here, with aggregate $\mathrm{SoH}$; the cycle-life leg is a refinement), so a unit built in year $\tau$ leaves $\bar K^k_t$ after $L_k$ years. Capex is annualized by $\mathrm{CRF}_k$ over $L_k$; a salvage credit $\rho_k \mathrm{CU}^k x^k_{t-L_k}$ is taken on capacity retiring in year $t$ (zero when $t - L_k < 1$); the $\mathrm{SoH}$ floor $\underline{\mathrm{SoH}}_s$ enforces retirement. Replacement is **endogenous**: if **milp-adq** still binds after a cohort reaches EOL, the program rebuilds.

Storage state-of-charge **milp-soc** cycles within each representative block in the planner; representative-period clearing and the game instead use a reduced dispatch with an annual discharge (net-energy) budget in place of the SoC recursion:

$$
\sum_h w_h \mathrm{dis}^s_{t,h} \le \mathrm{SoH}^s_t \bar P^s_t d_s
$$

where $d_s$ is storage duration. The cost terms $C^{\mathrm{Inv}}_{s,t}$, $C^{\mathrm{Gen}}_{s,t}$, and $C^{\mathrm{OM}}_{s,t}$ (investment, generation, and O&M) apply over live pre-EOL capacity. **milp-bal** and **milp-adq** carry the duals $\lambda$ and $\pi$ that price energy and capacity; in the market and game layer, $\pi$ follows the CONE curve rather than the planner adequacy shadow price.

The program is linear with integer build variables and is solved with **Gurobi** (HiGHS fallback, via Pyomo). The game-layer best response is the continuous-build, year-by-year relaxation of the equilibrium model; the diagonalization loop (Algorithm 1) calls it once per player per iteration.

---

## II. Representative Periods and Capacity Factors

**Table.** Representative operating periods ($\sum_h w_h = 8\times1095 = 8760$ h).
<img width="600" alt="image" src="https://github.com/user-attachments/assets/a926bfe7-4f2d-48e1-b62d-907758beb2be" />

**Table.** Period-specific dispatch capacity factors $\gamma^m_{t,h}$ (multiplier on live capacity). NGCC, NGCT, and CCHP are fully dispatchable; biomass and nuclear are held at their fixed capacity factor across all periods.
<img width="600" alt="image" src="https://github.com/user-attachments/assets/0c557d53-9a36-4511-a897-63b1a81a7c6a" />

---

## III. Biomass Emission Factor

The generation-technology parameter table in the paper lists biomass at $\varepsilon = 0.20$ tCO$_2$/MWh. Stack combustion for woody biomass is approximately 1.0 tCO$_2$/MWh; the modeled value applies an 80% biogenic offset (net accounting) — between the IPCC's zero-emission treatment of biomass and full stack counting (e.g., EU ETS for non-sustainable feedstocks). Biomass remains eligible for the renewable subsidy $\sigma^{\mathrm{re}}$ but, carrying $\varepsilon > 0$, still pays the carbon tax like any other emitting unit.

---

## IV. Regulator (Leader) Problem

The paper's reported scenarios (S1–S9) fix the policy vector $\boldsymbol{\theta}=(\tau,\sigma^{\mathrm{re}},\sigma^{\mathrm{sl}})$ at nine representative points. More generally, the leader's problem is to choose $\boldsymbol{\theta}$ to minimize expected social cost plus carbon damage at the induced equilibrium $\mathcal{E}(\boldsymbol{\theta})$:

$$
\begin{aligned}
\min_{\boldsymbol{\theta}\ge 0}\;\; & \mathrm{TSC}(\boldsymbol{\theta})
 + \mathrm{scc}\cdot\mathrm{CO_2}(\boldsymbol{\theta})
 + \mu\sum_t\sigma^{\mathrm{sl}}_t\bar P^{\mathrm{sl}}_t \\
\text{s.t.}\;\; & (\cdot)\in\mathcal{E}(\boldsymbol{\theta})
\end{aligned}
$$

where $\mathrm{scc}$ is the social cost of carbon and $\mu$ a subsidy-budget weight. Emissions are internalized through $\tau_t$ in the followers' payoffs and $\mathrm{scc}\cdot\mathrm{CO_2}$ in the leader's objective, rather than through a hard cap. The objective is two-dimensional (cost and emissions), so sweeping this leader problem over $\boldsymbol{\theta}$ traces a cost–emissions frontier; the nine scenarios reported in the paper are representative points on that surface, not an exhaustive sweep.

---

## V. Equilibrium Existence and Uniqueness

Companion material for *"A Stackelberg–Nash Framework for Second-Life Battery Investment under AI Data-Center Load Growth and Carbon Policy in MISO."* This note expands the uniqueness statement that appears in compressed form in the manuscript: it documents the argument and the two numerical checks that back it, so the paper itself can keep the two-sentence summary.

### 1. Setup and notation

For a fixed regulator policy $\boldsymbol{\theta}=(\tau,\sigma^{\mathrm{re}},\sigma^{\mathrm{sl}})$, the lower level is a one-shot game among the investors $p\in\mathcal{P}=\mathcal{R}\cup\mathcal{G}\cup\mathcal{S}$. Within a horizon year $t$:

- $x_p \ge 0$ — player $p$'s build decision (continuous-build relaxation; rounded to modules $\mathrm{CU}^p$ only at the end).
- $\Pi_p(x_p, x_{-p})$ — player $p$'s annual profit (generator/renewable/storage payoff in the paper).
- $\mathrm{CC} = \sum_m \mathrm{cc}^m \bar K^m + \sum_s \mathrm{cc}^s\,\mathrm{SoH}^s \bar P^s$ — system capacity credit.
- $\pi$ — capacity price, set by the CONE demand curve:

$$
\pi(\mathrm{CC}) = \mathrm{CONE}\cdot\mathrm{clip}\!\left(
\frac{(1+\mathrm{RM}^{\max})\,\Delta\hat D - \mathrm{CC}}{(\mathrm{RM}^{\max}-\mathrm{RM}^{\min})\,\Delta\hat D},\,0,\,1\right)
$$

Each investor enters the others' problems **only** through the two scalar prices $(\lambda,\pi)$, both functions of the aggregate dispatch and the aggregate capacity credit. There are no direct payoff couplings.

### 2. Existence (concave game, Rosen 1965)

The joint strategy set is a product of intervals $X=\prod_p[0,\bar x_p]$ — convex and compact. On the **sloped segment** of the CONE curve (i.e., where the `clip` argument is interior, $0<\cdot<1$), $\pi$ is affine and strictly decreasing in $\mathrm{CC}$, so the capacity revenue $\pi(\mathrm{CC})\,\mathrm{cc}^p\,x_p$ is concave in $x_p$ (own builds depress the price they earn). Energy revenue is concave for the same reason through $\lambda$, and all cost terms are convex (linear build costs; convex throughput-degradation cost). Hence each $\Pi_p$ is concave in $x_p$ on a convex compact set, and a pure-strategy Nash equilibrium exists by Rosen's existence theorem [Rosen, 1965].

### 3. Uniqueness via diagonal strict concavity (DSC)

Define the (unit-weighted) **pseudo-gradient** and its Jacobian:

$$
g(x) = \big(\partial \Pi_p/\partial x_p\big)_{p\in\mathcal{P}},
\qquad
G(x) = \nabla g(x),\quad G_{pq}=\frac{\partial^2 \Pi_p}{\partial x_p\,\partial x_q}
$$

**Proposition (conditional uniqueness).** If the symmetric part of the pseudo-gradient Jacobian is negative definite at the candidate equilibrium $x^*$,

$$
\tfrac{1}{2}\big(G(x^*)+G(x^*)^{\top}\big)\;\prec\;0,
$$

then the game satisfies Rosen's *diagonal strict concavity* condition and the Nash equilibrium is unique; the damped best-response iteration of the equilibrium loop converges to it.

**Why DSC holds here.** The structure is what makes the condition checkable:

- **Diagonal entries** $G_{pp}=\partial^2\Pi_p/\partial x_p^2 < 0$. The price-impact channel supplies this curvature: building more lowers $\pi$ (and $\lambda$), so marginal capacity revenue is strictly decreasing in own build. This is the *primary* source of strict concavity — the per-unit build **costs are linear**, so the curvature does not come from the cost side.
- **Off-diagonal entries** $G_{pq}=\partial^2\Pi_p/\partial x_p\partial x_q$ are all channeled through the same scalar maps $\pi(\mathrm{CC})$ and $\lambda$. On the sloped segment they share the single negative slope $\pi'(\mathrm{CC})<0$, which gives $\tfrac12(G+G^\top)$ a dominant-diagonal, negative-definite structure.

**Closing the gap off the slope.** A small strictly convex build-adjustment cost $\tfrac{\kappa}{2}x_p^2$ (with $\kappa>0$) adds $-\kappa$ to every $G_{pp}$ and makes $\tfrac12(G+G^\top)\prec 0$ hold *globally*, not only on the sloped segment. This yields unconditional uniqueness at the cost of one extra parameter; baseline results are reported without it to keep the cost structure transparent.

### 4. The conditional caveat (flat-price corners)

The claim is deliberately **conditional**. Where the CONE curve saturates — $\pi$ pinned at $\mathrm{CONE}$ in deep shortage, or at $0$ in surplus — the slope is zero, $\pi'(\mathrm{CC})=0$, and the price-impact curvature vanishes. In those corners $\Pi_p$ is only weakly concave and the **technology split among same-credit substitutes may not be pinned down**. The aggregates the paper reports — total installed capacity, clearing prices $(\lambda,\pi)$, and system emissions — remain unique, because they are determined by the binding adequacy/price level rather than by the split. Reported metrics are therefore robust even in the degenerate corners.

### 5. Numerical verification

Two independent checks are run at every horizon year, after the damped best-response loop has converged.

**(i) DSC eigenvalue check — `verify_dsc`**

1. Take the converged equilibrium $x^*$.
2. Assemble $G(x^*)$ by **central finite differences** on the per-player best-response gradients, step $\delta = 50$ MW:
   $$
   G_{pq}\approx\frac{g_p(x^*+\delta e_q)-g_p(x^*-\delta e_q)}{2\delta}
   $$
3. Form $S=\tfrac12(G+G^\top)$ and compute its eigenvalues.
4. **Pass** if $\lambda_{\max}(S)<0$ (strictly negative spectrum) → DSC holds at $x^*$.

A non-negative eigenvalue flags a flat-price corner (Section 4); the routine then reports which players span the unpinned subspace.

**(ii) Multi-start convergence — `verify_uniqueness_restarts`**

1. Draw $N=50$ independent initial profiles, each player's start $x_p^{(0)} \sim \mathcal{U}(0,2)\times x_p^{\mathrm{planner}}$.
2. Run the damped best-response iteration $x \leftarrow (1-\omega)\,x + \omega\,\mathrm{BR}(x)$ from each start, with damping $\omega =$ `data.bne.damping` and stopping rule $\max_p|\Delta x_p| <$ `data.bne.tolerance_mw`.
3. **Pass** if all $N$ runs land on the same equilibrium within the build tolerance (50 MW), i.e., $\max_p|x_p^{(k)}-x_p^*|<50$ MW for every start $k$.

Empirical convergence from many randomized starts is the most persuasive evidence for reviewers, and it agrees with the analytic DSC check on the sloped segment.

### 6. Reproduction

| Item | Where |
|---|---|
| Equilibrium loop (damped diagonalization, Algorithm 1) | `S04_Game_BNE.py` (Step 08) |
| Damping / tolerance | `data.bne.damping`, `data.bne.tolerance_mw` |
| DSC eigenvalue check | `verify_dsc` in `S04_Game_BNE.py` |
| Multi-start restarts | `verify_uniqueness_restarts` in `S04_Game_BNE.py` |
| Finite-difference step | $\delta = 50$ MW (central) |
| Restart count / draw | $N=50$, $\mathcal{U}(0,2)\times x^{\mathrm{planner}}$ |

Both routines are called on the converged-equilibrium object returned by the Step-08 loop; the eigenvalue spectrum and multi-start deviations reported in the paper (S6 and S7 at 2035 and 2045) are their output.

---

## Reference

J. B. Rosen, "Existence and uniqueness of equilibrium points for concave $N$-person games," *Econometrica*, vol. 33, no. 3, pp. 520–534, 1965.
