# Internalized Alignment: Multi-Agent Extension v1.2
## Two-Level Governance for Complex Systems — Final Audit

**Author:** Artem Brezgin / Spanda Foundation — Quant-Trika Program  
**Status:** v1.3 — final, paper-grade module  
**Relationship to IA v3.1:** Extends single-agent IA v3.1 to system-level governance. All single-agent definitions carry over unchanged.

---

## 0. Core Thesis and Motivation

### 0.1 The Fundamental Gap

$$\hat{CC}^{eff,i}_t(a^{i,*}_t) \le 0 \quad \forall i \qquad \text{(every agent passes individual gate)}$$
$$\text{yet} \qquad CC^{sys,eff}_t(\mathbf{a}^*_t) > 0 \qquad \text{(system collectively violates)}$$

This is the mathematical structure of the Tragedy of the Commons (Section 7). Per-agent governance is necessary but not sufficient.

$$\boxed{\text{Local admissibility does not imply collective admissibility.}}$$

### 0.2 Two Governance Regimes *(Patch 1)*

The two-level architecture operates in one of two regimes with different guarantees:

**2L-Strong** — ex-ante system gate verification:
- Requires: joint commitment protocol, shared simulator, or oracle
- Provides: hard guarantee that $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ before execution
- Policy class: $\operatorname{supp}(\pi^1,\dots,\pi^n) \subseteq \mathcal{A}^{2L}_{CC}(S_t)$ (joint support condition)
- Suitable for: centralized or coordinator-mediated deployments

**2L-Distributed** — adaptive self-regulation via public signal:
- Requires: public system signal $M^{CC,sys}_t$, $R_t$ (no central governor)
- Provides: $\hat{CC}^{eff,i,sys}_t(a^i) \le 0$ per agent, adaptive tightening after violations
- Policy class: $\operatorname{supp}\pi^i \subseteq \mathcal{A}^{2L,i}_{CC}$ per agent independently
- Guarantee: does **not** guarantee $CC^{sys,eff}_t \le 0$ ex-ante; ratchet penalizes violations ex-post
- Suitable for: GovSim, decentralized deployments, no shared simulator

> **Key distinction:** 2L-Strong prevents system violations. 2L-Distributed detects and penalizes them, tightening future local gates. Both are valuable; they are different claims.

**GovSim target:** 2L-Distributed. The resource level $R_t$ and extraction are publicly observable; no joint simulator is required.

### 0.3 Governance Without Central Authority

$$\boxed{\text{Governance is not centralized in an authority; it is embedded in the admissibility protocol.}}$$

Three components replace the central governor:
1. **Public system signal** — $M^{CC,sys}_t$, $R_t$, $KQ^{sys}_t$ observable by all agents
2. **System violation ledger** — updated automatically from realized outcomes
3. **Distributed local gates** — each agent applies $\hat{CC}^{eff,i,sys}_t \le 0$ independently

**Human-in-the-loop policy:** Runtime is autonomous. Humans define protected resources, calibrate $\Gamma_{sys}$, $M^{sys}_{max}$, fallback conditions, and proxy reliability.

**Notation convention:** $\gamma$ = discount factor. $\Gamma_{sys} > 0$ = system-to-agent coupling strength.

---

## 1. System-Level State

### 1.1 System State

$$\boxed{S_t = (s^1_t, \dots, s^n_t, R_t, \mathcal{H}_t)}$$

$R_t \in [0, R_{max}]$ — shared resource. $\mathcal{H}_t$ — joint action and observation history.

### 1.2 Persistent System Alignment State

$$\boxed{Z^{sys}_t = (T^{sys}_t,\; \Theta^{sys}_t,\; M^{harm,sys}_t,\; M^{CC,sys}_t,\; \mathcal{L}^{sys}_t)}$$

$M^{CC,sys}_t \in [0, M^{sys}_{max}]$ is the **monotone violation ledger** — the only component that is never endogenously reduced.

> **Remark — Conservative Irreversible Ledger** *(Patch 6)*: The strict non-decreasing property of $M^{CC,sys}$ reflects a deliberate design choice: a structural violation is a qualitative event, not a continuous quantity to be traded. This is the "Stoic" reading — violations are permanently recorded. In real-world governance where certified repair mechanisms exist, one may introduce a decay term $(1-\rho_M)M^{CC,sys}_t$ or an audited reset $M^{CC,sys}_{t+1} = M^{CC,sys}_t - \text{CertifiedRepair}_t$. For GovSim and the present theoretical framework, the strict ledger is retained.

---

## 2. System-Level Coherence Field

### 2.1 System Coherence Components

**Internal coherence** — agreement among agents about shared resource state.

Let $\sigma^2_{R,max} = R^2_{max}/4$ (maximum variance of a variable bounded on $[0, R_{max}]$):

$$\boxed{C^{sys,internal}_t = \operatorname{clip}_{[0,1]}\!\left(1 - \frac{\operatorname{Var}_i(\hat{R}^i_t)}{\sigma^2_{R,max}}\right) \in [0,1]}$$

Dimensionless and bounded: $\operatorname{Var} = 0 \implies C^{sys,internal}_t = 1$; maximal disagreement $\implies C^{sys,internal}_t \to 0$.

**External coherence** — alignment of collective action with resource sustainability:

Define sustainable yield: $\text{SY}(R_t) = g \cdot R_t(1 - R_t/R_{max})$ (logistic growth, units: resource/round). Time step $\Delta t = 1$ round throughout.

**Critical separation:** $C^{sys,external}$ exists in two forms — state-level (realized, based on previous action) and action-conditioned (predicted, based on candidate action):

$\boxed{C^{sys,external,state}_t = 1 - \operatorname{clip}_{[0,1]}\!\left(\frac{[\text{Take}(\mathbf{a}_{t-1}) - \text{SY}(R_{t-1})]_+ \cdot \Delta t}{R_{max}}\right)}$

For action-conditioned prediction (used inside $CC^{sys}_t(\mathbf{a})$), define separately:

$\boxed{C^{sys,external,pred}_{t+1}(\mathbf{a}) = 1 - \operatorname{clip}_{[0,1]}\!\left(\frac{[\text{Take}(\mathbf{a}) - \text{SY}(R_t)]_+ \cdot \Delta t}{R_{max}}\right)}$

The $\Delta t$ factor makes both expressions dimensionless: $[\text{Take} - \text{SY}]_+$ has units resource/round, $\Delta t$ has units round, $R_{max}$ has units resource.

Interpretation: $C^{sys,external}_t = 1$ when extraction $\le$ sustainable yield; $C^{sys,external}_t = 0$ when overshoot equals $R_{max}$ (catastrophic).

**Alternative resource-margin form** (for settings where a safety stock threshold is more natural):

$$C^{sys,external,alt}_t = \operatorname{clip}_{[0,1]}\!\left(\frac{R^{predicted}_{t+1} - R_{min}}{R_{safe} - R_{min}}\right)$$

where $R_{safe}$ is a target stock level and $R_{min}$ is the collapse floor.

**Hard critical collapse rule:** If collective extraction exceeds sustainable yield completely ($C^{sys,external}_t = 0$):

$$C^{sys}_t := 0 \qquad \text{(hard collapse override)}$$

This is architecturally necessary to preserve Lemma S2 in the presence of regularization.

### 2.2 Normalized Weighted Geometric Mean

**Normalized weighted geometric mean** — using **state-level** components:

$C^{reg,k,state}_t = \frac{C^{sys,k,state}_t + \varepsilon_C}{1 + \varepsilon_C} \in (0,1], \qquad k \in \{\text{internal, external}\}$

$\boxed{C^{sys,state}_t = \begin{cases} 0 & \text{if } C^{sys,external,state}_t = 0 \text{ (hard collapse)} \\ \left[(C^{reg,internal,state}_t)^{w_1}(C^{reg,external,state}_t)^{w_2}\right]^{1/(w_1+w_2)} & \text{otherwise} \end{cases}}$

**Action-conditioned coherence** (for CC computation):

$C^{sys,pred}_{t+1}(\mathbf{a}) = \left[(C^{reg,internal,pred}_{t+1}(\mathbf{a}))^{w_1}(C^{reg,external,pred}_{t+1}(\mathbf{a}))^{w_2}\right]^{1/(w_1+w_2)}$

where $C^{sys,internal,pred}_{t+1}(\mathbf{a})$ is the predicted belief agreement after action $\mathbf{a}$.

### 2.3 System Entropy

**Action dispersion entropy:**
$$H^{sys,action}_t = \text{NormEntropy}(a^1_t, \dots, a^n_t) \in [0,1]$$

**Belief divergence entropy:**
$$H^{sys,belief}_t = 1 - C^{sys,internal}_t = \operatorname{clip}_{[0,1]}\!\left(\frac{\operatorname{Var}_i(\hat{R}^i_t)}{\sigma^2_{R,max}}\right)$$

> **Remark — Intentional Double-Entry** *(Patch 4)*: Belief disagreement $\operatorname{Var}_i(\hat{R}^i_t)$ enters both $C^{sys,internal}_t$ (reducing coherence) and $H^{sys,belief}_t$ (increasing entropy). This double-entry is intentional: belief disagreement simultaneously degrades the quality of the system's epistemic state ($C^{sys}$) and increases the uncertainty of its behavioral regime ($H^{sys}$). These are distinct alignment-relevant roles. An ablation study may set $H^{sys,belief}_t := 0$ and compare $KQ^{sys}_t$ sensitivity to isolate the coherence channel from the entropy channel.

**Aggregate system entropy:**
$$\boxed{H^{sys}_t = \alpha_{action} H^{sys,action}_t + \alpha_{belief} H^{sys,belief}_t, \qquad \alpha_{action} + \alpha_{belief} = 1}$$

### 2.4 System KQ

$$\boxed{KQ^{sys}_t = C^{sys}_t(1 - H^{sys}_t) \in [0,1]}$$

---

## 3. System-Level Alignment Variables

### 3.1 System Ontological Debt

$$\delta\Theta^{sys}_t = \omega_1 \cdot [\text{Take}(\mathbf{a}_t) - \text{SY}(R_t)]_+ + \omega_2 \cdot (1 - C^{sys,internal}_t) + \omega_3 \cdot \text{Irreversibility}(R_t)$$

$$\boxed{\Theta^{sys}_{t+1} = \operatorname{clip}_{[0,\Theta^{sys}_{max}]}\!\left((1-\rho^{sys}_\Theta)\Theta^{sys}_t + \delta\Theta^{sys}_t - \kappa^{sys}_\Theta \text{Repair}^{sys}_t\right)}$$

### 3.2 System Fragility

$$\mathcal{R}^{sys}(S_t) = \rho_1 \cdot \frac{R_t}{R_{max}} + \rho_2 \cdot \text{RecoveryCapacity}(R_t) + \rho_3 \cdot T^{sys}_t$$

$\boxed{F^{sys}_t(\mathbf{a}) = \max(0,\; \mathcal{R}^{sys}(S_t) - \mathcal{R}^{sys}(S^{(\mathbf{a})}_{t+1}))}$

This quantity already measures resilience loss — it is the fragility increment itself. In the CC formula, it appears directly as $\lambda^{sys}_F F^{sys}_t(\mathbf{a})$, not as $\lambda^{sys}_F \Delta F^{sys}_t$, since $F^{sys}_t(\mathbf{a}) \ge 0$ is already the positive-part increment by construction.

### 3.3 Vector-Based Protected-Resource Loss *(Positive Part)*

For the hard system gate, resource improvements must not compensate trust loss or debt increase without an explicit design decision. Therefore all components use positive part:

$\boxed{\Delta P^{sys}_t(\mathbf{a}) = \rho_R \cdot \Delta P^R_t(\mathbf{a}) + \rho_\Theta \cdot \Delta P^\Theta_t(\mathbf{a}) + \rho_T \cdot \Delta P^T_t(\mathbf{a})}$

$\Delta P^R_t(\mathbf{a}) = \frac{[R_t - R^{(\mathbf{a})}_{t+1}]_+}{R_{max}} \ge 0 \qquad \text{(resource loss only)}$

$\Delta P^\Theta_t(\mathbf{a}) = [\bar\Theta^{sys,(\mathbf{a})}_{t+1} - \bar\Theta^{sys}_t]_+ \ge 0 \qquad \text{(debt increase only)}$

$\Delta P^T_t(\mathbf{a}) = [T^{sys}_t - T^{sys,(\mathbf{a})}_{t+1}]_+ \ge 0 \qquad \text{(trust loss only)}$

with $\rho_R + \rho_\Theta + \rho_T = 1$.

> **Design note:** Using positive parts means improvements in one channel (e.g. resource recovery) do not offset losses in another (e.g. trust degradation) within the hard gate. Cross-channel compensation may be permitted in soft utility $V^{sys}$ but is excluded from $CC^{sys}$ by this construction. GovSim default: $\rho_R = 0.7$, $\rho_\Theta = 0.2$, $\rho_T = 0.1$.

### 3.4 Inter-Agent Trust

$$\boxed{T^{sys}_{t+1} = \operatorname{clip}_{[0,1]}\!\left(T^{sys}_t + \alpha_T \text{CommitmentHonoring}_t - \delta_T D^{sys}_t - \phi_T \text{CollectiveFailure}_t\right)}$$

### 3.5 System Harm Memory

$$\boxed{M^{harm,sys}_{t+1} = \min\!\left(M^{harm,sys}_{max},\; M^{harm,sys}_t + [\Delta P^R_t(\mathbf{a}^*_t)]_+\right)}$$

---

## 4. System-Level Coupling Constraint

### 4.1 System CC — Action-Conditioned

Using the state/prediction separation from Section 2.4:

$\boxed{CC^{sys}_t(\mathbf{a}) = \underbrace{(KQ^{sys,state}_t - KQ^{sys,pred}_{t+1}(\mathbf{a}))}_{\text{coherence loss}} + \lambda^{sys}_P \Delta P^{sys}_t(\mathbf{a}) + \lambda^{sys}_F \Delta F^{sys}_t(\mathbf{a}) + \lambda^{sys}_\Theta (\Theta^{sys,(\mathbf{a})}_{t+1} - \Theta^{sys}_t)}$

$KQ^{sys,state}_t$ is a fixed scalar (current state). $KQ^{sys,pred}_{t+1}(\mathbf{a})$ varies with candidate $\mathbf{a}$. The indexing ambiguity is fully resolved.

**System admissibility** (both conditions required):

$\boxed{CC^{sys,eff}_t(\mathbf{a}) = CC^{sys}_t(\mathbf{a}) + \rho_{sys} M^{CC,sys}_t \le 0}$
$\boxed{\text{AND} \quad R^{(\mathbf{a})}_{t+1} \ge R_{min}}$

### 4.2 System Violation Ledger — Extended with Resource Floor Term

System admissibility has two conditions. The ledger must record violations of both:

$\boxed{M^{CC,sys}_{t+1} = \min\!\left(M^{sys}_{max},\; M^{CC,sys}_t + \eta^{sys}_M [CC^{sys,real}_t(\mathbf{a}_t)]_+ + \eta_R [R_{min} - R_{t+1}]_+\right)}$

where $\eta_R > 0$ is the floor-violation accumulation rate.

This closes the gap: if the resource drops below $R_{min}$ even when $CC^{sys,real}_t \le 0$ (e.g. due to estimation error), the ledger still tightens future gates. Both channels of admissibility failure are recorded.

---

## 5. Distributed Budget: Level Coupling

### 5.1 Per-Agent Coupled Gate

$$\boxed{\hat{CC}^{eff,i,sys}_t(a^i) = \hat{CC}^i_t(a^i) + \rho_i M^{CC,i}_t + \Gamma_{sys} M^{CC,sys}_t \le 0}$$

### 5.2 Two-Level Admissible Sets

**Per-agent (2L-Distributed):**
$$\boxed{\mathcal{A}^{2L,i}_{CC}(s^i_t, Z^{sys}_t) = \{a^i \in \mathcal{A}^i : \hat{CC}^{eff,i,sys}_t(a^i) \le 0\}}$$

**Joint (2L-Strong only):**
$$\boxed{\mathcal{A}^{2L}_{CC}(S_t) = \left\{\mathbf{a} \in \prod_i \mathcal{A}^{2L,i}_{CC} : CC^{sys,eff}_t(\mathbf{a}) \le 0 \;\text{ and }\; R^{(\mathbf{a})}_{t+1} \ge R_{min}\right\}}$$

### 5.3 Ratchet Invariant

$$\boxed{\Gamma_{sys} \cdot M^{sys}_{max} \le \min_{i,\, a^{safe} \in \mathcal{A}_{fallback}} |\hat{CC}^i_t(a^{safe})|}$$

Ensures fallback actions remain admissible at maximum violation memory.

---

## 6. Policy Classes and Objectives

### 6.1 2L-Strong Policy Class

Requires joint support condition — applicable only when joint action can be verified ex-ante:

$$\boxed{\Pi^{2L\text{-}S}_{CC} = \left\{(\pi^1,\dots,\pi^n) : \operatorname{supp}(\pi^1,\dots,\pi^n)(\cdot \mid \mathcal{B}_t) \subseteq \mathcal{A}^{2L}_{CC}(S_t)\;\; \forall t\right\}}$$

This requires coordination: either a joint policy (agents synchronize choices) or an ex-ante simulator/oracle that checks joint admissibility before execution.

### 6.2 2L-Distributed Policy Class

Requires only per-agent support condition — applicable in decentralized settings:

$$\boxed{\Pi^{2L\text{-}D}_{CC} = \left\{(\pi^1,\dots,\pi^n) : \operatorname{supp}\pi^i(\cdot \mid b^i_t) \subseteq \mathcal{A}^{2L,i}_{CC}(s^i_t, Z^{sys}_t)\;\; \forall i, t\right\}}$$

**Guarantee difference:**

| Property | 2L-Strong | 2L-Distributed |
|---|---|---|
| Individual gates satisfied | ✓ | ✓ |
| System gate satisfied ex-ante | ✓ | ✗ (not guaranteed) |
| System violations penalized ex-post | ✓ | ✓ (via ratchet) |
| Requires joint coordinator | Yes | No |
| GovSim-applicable | Yes (with simulator) | **Yes (default)** |

Note: $\Pi^{2L\text{-}S}_{CC} \subsetneq \Pi^{2L\text{-}D}_{CC}$ — the strong policy class is strictly more restrictive.

### 6.3 System Virtue Index

$$\boxed{V^{sys}_t = \frac{U^{sys}_t \cdot \exp(A^{sys}_t / \tau^{sys}_{IA})}{K^{sys}_t \tilde{H}^{sys}_t + \varepsilon_V}}$$

$$A^{sys}_t = \alpha^{sys}\log(KQ^{sys}_t+\varepsilon) + \beta^{sys}\log(T^{sys}_t+\varepsilon) - \delta^{sys}\log(1+\bar\Theta^{sys}_t) - \eta^{sys}\log(1+F^{sys}_t)$$

### 6.4 Two-Level Objective

$\boxed{J^{2L}_{IA} = \mathbb{E}\!\left[\sum_{t=0}^{T} \gamma^t \left(\sum_{i=1}^n V^{IA,i}_t + \mu V^{sys}_t\right)\right]}$

**2L-Strong:** Joint optimization — $(\pi^{1,*},\dots,\pi^{n,*}) = \arg\max_{\pi \in \Pi^{2L\text{-}S}_{CC}} J^{2L}_{IA}$

**2L-Distributed:** $(\pi^{1,*},\dots,\pi^{n,*}) = \arg\max_{\pi \in \Pi^{2L\text{-}D}_{CC}} J^{2L}_{IA}$

> **Remark — Distributed Objective** *(Patch 4)*: For 2L-Distributed, $J^{2L}_{IA}$ is an **evaluation functional or centralized training objective**, not a runtime quantity that each agent maximizes independently. Runtime execution remains fully decentralized through local gates $\hat{CC}^{eff,i,sys}_t \le 0$ and the public ledger $M^{CC,sys}_t$. In decentralized deployment, each agent may instead optimize a local proxy:
>
> $J^{i,2L\text{-}D}_{IA} = \mathbb{E}\!\left[\sum_t \gamma^t \left(V^{IA,i}_t + \mu_i \mathbb{E}_i[V^{sys}_t \mid b^i_t, M^{CC,sys}_t]\right)\right]$
>
> where $\mathbb{E}_i[V^{sys}_t \mid b^i_t, M^{CC,sys}_t]$ is agent $i

---

## 7. Tragedy of the Commons: Formal Definition

**Definition (Formal ToC):** A system exhibits a Tragedy of the Commons at time $t$ if:

$$\boxed{\hat{CC}^{eff,i}_t(a^{i,*}_t) \le 0 \;\; \forall i \qquad \text{AND} \qquad CC^{sys,eff}_t(\mathbf{a}^*_t) > 0}$$

where $\hat{CC}^{eff,i}_t$ is the **individual-only gate** (without system coupling $\Gamma_{sys} M^{CC,sys}_t$) and $a^{i,*}_t$ is the individually optimal action under that gate.

**Residual ToC:** Even with distributed budget coupling:

$$\hat{CC}^{eff,i,sys}_t(a^{i,*}_t) \le 0 \;\; \forall i \qquad \text{AND} \qquad CC^{sys,eff}_t(\mathbf{a}^*_t) > 0$$

Residual ToC signals that $\Gamma_{sys}$ is insufficient — system coupling does not yet resolve the coordination failure.

**Corollary:** Per-agent IA-W is necessary but not sufficient to prevent Formal ToC. The system gate or the distributed budget with sufficient $\Gamma_{sys}$ is an additional requirement.

---

## 8. Lemmas

**Lemma S1 — System KQ Boundedness.** $KQ^{sys}_t \in [0,1]$.
*Proof.* Hard collapse gives $C^{sys}_t = 0 \in [0,1]$. Otherwise $C^{sys}_t \in (0,1]$, $(1-H^{sys}_t) \in [0,1]$, product in $[0,1]$. $\square$

**Lemma S2 — System Coherence Collapse.** If $C^{sys,external}_t = 0$ then $KQ^{sys}_t = 0$.
*Proof.* Hard collapse rule sets $C^{sys}_t := 0$, overriding regularization. $KQ^{sys}_t = 0 \cdot (1-H^{sys}_t) = 0$. $\square$
*Note:* Without the hard collapse override, regularization gives $C^{sys}_t \ge \varepsilon_C/(1+\varepsilon_C) > 0$, making the lemma false. The override is architecturally necessary.

**Lemma S3 — System Ratchet Monotonicity.** $M^{CC,sys}_t$ is monotone non-decreasing and $M^{CC,sys}_t \le M^{sys}_{max}$ for all $t$.
*Proof.* $[CC^{sys,real}_t]_+ \ge 0$ ensures non-decrease; $\min(\cdot, M^{sys}_{max})$ ensures upper bound. $\square$

**Lemma S4 — 2L-Strong $\subset$ 2L-Distributed.**
$$\Pi^{2L\text{-}S}_{CC} \subsetneq \Pi^{2L\text{-}D}_{CC}$$
*Proof.* Every joint policy satisfying the joint support condition also satisfies per-agent support conditions. The converse fails: independent per-agent admissibility does not guarantee joint admissibility. $\square$

**Lemma S5 — Coupling Weakly Reduces Individual Admissible Set.**
For $\Gamma_{sys} > 0$ and $M^{CC,sys}_t > 0$: $\mathcal{A}^{2L,i}_{CC} \subseteq \mathcal{A}^{eff,i}_{CC}$ (without system coupling).
*Proof.* Adding $\Gamma_{sys} M^{CC,sys}_t > 0$ strictly tightens the individual gate. $\square$

**Lemma S6 — System Violation Detection vs Formal ToC Detection** *(Patch 2)*

**(a)** System violation $CC^{sys,eff}_t(\mathbf{a}_t) > 0$ is **detectable from system-level measurements alone**: $R_t$, $KQ^{sys}_t$, $\Delta P^{sys}_t$, $F^{sys}_t$, $\Theta^{sys}_t$, $M^{CC,sys}_t$ — all observable at the system level without accessing individual agent internals.

**(b)** Formal ToC ($\forall i: \hat{CC}^{eff,i}_t \le 0$ AND $CC^{sys,eff}_t > 0$) requires additionally verifying that each agent passed its individual gate. This requires either:
- Individual gate output logs $\{\hat{CC}^{eff,i}_t(a^{i,*}_t)\}_{i=1}^n$, or
- A reconstructable model of individual CC functions, or
- Agent-reported gate satisfaction (auditable but not independently verifiable without logs)

**Summary:** System violations are system-detectable. Formal ToC structure is detectable only with individual gate logs or reconstructable local CC models.

**Lemma S7 — $C^{sys,external}$ is Dimensionless and Bounded.**

$[\text{Take}(\mathbf{a}_t) - \text{SY}(R_t)]_+$ has units resource/round. $\Delta t = 1$ round. Their product has units resource. $R_{max}$ has units resource. The ratio $[\text{Take} - \text{SY}]_+ \cdot \Delta t / R_{max}$ is therefore dimensionless. The $\operatorname{clip}_{[0,1]}$ and subtraction from 1 yield $C^{sys,external} \in [0,1]$. $\square$

**Lemma S8 — $C^{sys,internal}$ is Dimensionless and Bounded.**
$\operatorname{Var}_i(\hat{R}^i_t)$ has units $R^2$. $\sigma^2_{R,max} = R^2_{max}/4$ has units $R^2$. The ratio is dimensionless and in $[0,1]$. The $\operatorname{clip}$ and subtraction from 1 yield $C^{sys,internal}_t \in [0,1]$. $\square$

---

## 9. Theorems

**Theorem S1 — 2L-Strong Prevents Formal ToC.**

Under 2L-Strong governance, Formal ToC cannot occur: the joint support condition $\operatorname{supp}(\pi^1,\dots,\pi^n) \subseteq \mathcal{A}^{2L}_{CC}$ ensures $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ before execution.

*Proof.* By definition of $\mathcal{A}^{2L}_{CC}$, all joint actions in the support satisfy $CC^{sys,eff}_t \le 0$. Therefore $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ for all executed joint actions. The second condition of Formal ToC ($CC^{sys,eff}_t > 0$) cannot be satisfied. $\square$

---

**Theorem S2 — 2L-Distributed Does Not Guarantee Absence of Formal ToC.**

Under 2L-Distributed governance, Formal ToC can occur in finite time even if all agents satisfy their individual coupled gates.

*Proof.* The per-agent gate $\hat{CC}^{eff,i,sys}_t \le 0$ does not imply $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ (Lemma S4). There exist joint actions where each $\hat{CC}^{eff,i,sys}_t(a^{i,*}_t) \le 0$ but $CC^{sys,eff}_t(\mathbf{a}^*_t) > 0$. When such joint actions occur, Formal ToC is realized and $M^{CC,sys}_{t+1} > M^{CC,sys}_t$ by Lemma S3. $\square$

*Interpretation:* This is the honest statement of what 2L-Distributed provides: not prevention of system violations, but detection, recording, and adaptive tightening after violations. It is a governance protocol, not an impossibility result.

---

**Theorem S3 — System Ratchet Non-Compensation.**

Under the system ratchet, no sequence of future joint actions can reduce $M^{CC,sys}_t$ after a realized system violation.

*Proof.* No subtraction term exists in the update rule. Non-decrease follows from Lemma S3. $\square$

---

**Theorem S4 — Coupling Weakly Reduces Maximum Admissible Extraction.**

**Assumption (Monotonic CC):** $\hat{CC}^i_t(a^i)$ is monotonically non-decreasing in $\text{take}(a^i)$.

Under 2L-Distributed with $\Gamma_{sys} > 0$ and $M^{CC,sys}_t > 0$, the maximum individually admissible extraction satisfies:

$$\text{take}^{max,2L}_i \le \text{take}^{max,1L}_i \qquad \forall i$$

The reduction is **weak** (not necessarily strict): strict reduction holds iff there exists a marginally admissible action $a^i$ with $\hat{CC}^i_t(a^i) \in (-\rho_i M^{CC,i}_t - \Gamma_{sys} M^{CC,sys}_t,\; -\rho_i M^{CC,i}_t]$.

*Proof.* Tighter gate $\Rightarrow$ smaller admissible set (Lemma S5). Under Monotonic CC, smaller admissible set in terms of gate value $\Rightarrow$ weakly lower maximum extraction. $\square$

---

**Theorem S5 — Resource Safety Invariance Under 2L-Strong.**

Under 2L-Strong governance with resource floor condition $R^{(\mathbf{a})}_{t+1} \ge R_{min}$ as an explicit admissibility requirement: if $R_0 \ge R_{min}$, then $R_t \ge R_{min}$ for all $t$.

*Proof.* The resource floor is an explicit admissibility condition in $\mathcal{A}^{2L}_{CC}$. Every executed joint action satisfies $R^{(\mathbf{a}_t)}_{t+1} \ge R_{min}$. By induction: $R_t \ge R_{min}$ for all $t$. $\square$

*Note on 2L-Distributed:* Resource floor is not guaranteed ex-ante under 2L-Distributed. System violations may push $R$ below $R_{min}$, triggering ratchet update and future gate tightening.

---

**Theorem S6 — Statewise Saturated Ratchet Excludes Over-Extraction.**

**Setup:** Fix state $S_t$. Let $\mathcal{A}^{over}_i(S_t) = \{a^i : \text{take}(a^i) > \text{SY}(R_t)/n\}$. Define the statewise maximum slack of any over-extraction action:

$\Delta^i_{over}(S_t) = \sup_{a^i \in \mathcal{A}^{over}_i(S_t)} \left[-\hat{CC}^i_t(a^i) - \rho_i M^{CC,i}_t\right]$

If at state $S_t$:

$\boxed{\Gamma_{sys} \cdot M^{CC,sys}_{max} > \sup_i \Delta^i_{over}(S_t)}$

then once $M^{CC,sys}_t = M^{CC,sys}_{max}$, all over-extraction actions are excluded from $\mathcal{A}^{2L,i}_{CC}$ for all agents $i$ at state $S_t$.

*Proof.* For any $a^i \in \mathcal{A}^{over}_i(S_t)$:
$\hat{CC}^{eff,i,sys}_t(a^i) \ge -\Delta^i_{over}(S_t) + \Gamma_{sys} M^{CC,sys}_{max} > 0$
Therefore $a^i \notin \mathcal{A}^{2L,i}_{CC}$. $\square$

**Global version:** For a global exclusion result over all reachable states, define:

$\Delta^{global}_{over} = \sup_{S_t \in \mathcal{S}_{reachable}} \sup_i \sup_{a^i \in \mathcal{A}^{over}_i(S_t)} \left[-\hat{CC}^i_t(a^i) - \rho_i M^{CC,i}_t\right]$

If $\Gamma_{sys} M^{CC,sys}_{max} > \Delta^{global}_{over}$, exclusion holds over all reachable states simultaneously. Computing $\Delta^{global}_{over}$ requires knowledge of the full reachable state space — feasible in GovSim, generally an open problem in complex environments.

---

## 10. GovSim Mapping

### 10.1 Variable Correspondence

| GovSim concept | IA variable | Operationalization |
|---|---|---|
| Fish stock | $R_t$ | Observed each round, normalized to $[0, R_{max}]$ |
| Individual harvest | $\text{take}(a^i_t)$ | Action $\to$ extraction amount |
| Total extraction | $\text{Take}(\mathbf{a}_t)$ | $\sum_i \text{take}(a^i_t)$ |
| Sustainable yield | $\text{SY}(R_t) = g R_t(1-R_t/R_{max})$ | Logistic growth |
| Overshoot | $[\text{Take} - \text{SY}]_+$ | Excess extraction |
| External coherence | $C^{sys,external}_t$ | $1 - \operatorname{clip}([\text{Take}-\text{SY}]_+ / R_{max})$ |
| Internal coherence | $C^{sys,internal}_t$ | Agent agreement on $R_t$ estimate |
| System collapse | $R_t < R_{min}$ | Resource floor violation |
| Violation ledger | $M^{CC,sys}_t$ | Accumulated $[CC^{sys,real}_t]_+$ |
| Individual gate logs | $\hat{CC}^{eff,i}_t(a^{i,*}_t)$ | Required for Formal ToC detection |

### 10.2 Governance Mode for GovSim

**Default: 2L-Distributed.** Resource $R_t$ and extraction are publicly observable. No joint simulator needed.

**Condition for Formal ToC detection:** Individual gate output logs must be recorded per round to verify $\hat{CC}^{eff,i}_t \le 0$ for each agent.

### 10.3 Parameter Defaults

```python
# System-level
Gamma_sys  = 0.3     # coupling strength — sweep in H5
rho_sys    = 0.2     # ratchet tightening per unit M_CC_sys
eta_sys    = 0.5     # ratchet accumulation rate
M_sys_max  = 2.0     # ratchet ceiling

# Resource model
R_max      = 100
R_min      = 20      # collapse floor
g          = 0.15    # logistic growth rate
SY_rate    = lambda R: g * R * (1 - R/R_max)

# CC weights
lambda_P   = 0.5
lambda_F   = 0.3
lambda_Th  = 0.2
alpha_action = 0.6
alpha_belief = 0.4
rho_R, rho_Th, rho_T = 0.7, 0.2, 0.1
```

---

## 11. Empirical Hypotheses

**H1 — Early Warning:** $KQ^{sys}_t$ falls before collapse earlier than $R_t$.
*Falsifiable:* Cross-correlation $\Delta KQ^{sys}$ leads $\Delta R$ by $\ge 2$ rounds.

**H2 — Per-Agent Insufficient:** Survival: Baseline $<$ IA-W L1 $\le$ IA-W L2. Formal ToC events present in L1 (requires individual gate logs).

**H3 — Ratchet as Predictor:** AUROC($M^{CC,sys}_t$, collapse within 5 rounds) $>$ AUROC($R_t$, same).

**H4 — Formal ToC Detectability:** In collapsed L1 runs, individual gate logs show $\hat{CC}^{eff,i}_t \le 0$ $\forall i$ while $CC^{sys,eff}_t > 0$ in $\ge K$ rounds before collapse.

**H5 — Coupling Pareto:** Sweep $\Gamma_{sys}$. Identify $\Gamma^*_{sys}$ maximizing survival subject to helpfulness. Verify ratchet invariant $\Gamma_{sys} M^{sys}_{max} \le |\hat{CC}(a^{safe})|_{min}$ throughout.

**H6 — Residual ToC Eliminated by Sufficient Coupling (over-extraction channel):** At $\Gamma_{sys}$ satisfying the condition of Theorem S6, Residual ToC events **caused by over-extraction** are eliminated once $M^{CC,sys}_t$ reaches the required threshold. Other sources of Residual ToC (e.g. coordination failures not involving over-extraction, or estimation errors) may persist.

---

## 12. Core Mathematical Statement (v1.2)

$\boxed{KQ^{sys,state}_t = C^{sys,state}_t(1 - H^{sys,state}_t)}$

$\boxed{C^{sys,external,state}_t = 1 - \operatorname{clip}_{[0,1]}\!\left(\frac{[\text{Take}(\mathbf{a}_{t-1}) - \text{SY}(R_{t-1})]_+ \cdot \Delta t}{R_{max}}\right)}$

$\boxed{C^{sys,external,pred}_{t+1}(\mathbf{a}) = 1 - \operatorname{clip}_{[0,1]}\!\left(\frac{[\text{Take}(\mathbf{a}) - \text{SY}(R_t)]_+ \cdot \Delta t}{R_{max}}\right)}$

$\boxed{\Delta P^{sys}_t(\mathbf{a}) = \rho_R \frac{[R_t - R^{(\mathbf{a})}_{t+1}]_+}{R_{max}} + \rho_\Theta [\bar\Theta^{sys,(\mathbf{a})}_{t+1} - \bar\Theta^{sys}_t]_+ + \rho_T [T^{sys}_t - T^{sys,(\mathbf{a})}_{t+1}]_+}$

$\boxed{M^{CC,sys}_{t+1} = \min(M^{sys}_{max},\; M^{CC,sys}_t + \eta^{sys}_M [CC^{sys,real}_t]_+ + \eta_R [R_{min} - R_{t+1}]_+)}$

$\boxed{\hat{CC}^{eff,i,sys}_t(a^i) = \hat{CC}^i_t(a^i) + \rho_i M^{CC,i}_t + \Gamma_{sys} M^{CC,sys}_t \le 0}$

$\boxed{CC^{sys,eff}_t(\mathbf{a}) = CC^{sys}_t(\mathbf{a}) + \rho_{sys} M^{CC,sys}_t \le 0 \quad \text{AND} \quad R^{(\mathbf{a})}_{t+1} \ge R_{min}}$

**Formal ToC:**
$\boxed{\hat{CC}^{eff,i}_t(a^{i,*}) \le 0 \;\forall i \quad \text{AND} \quad CC^{sys,eff}_t(\mathbf{a}^*) > 0}$

**Key result:**
$\boxed{\text{2L-Strong prevents Formal ToC.} \quad \text{2L-Distributed detects and penalizes it.}}$

**Distributed objective (runtime):**
$\boxed{J^{i,2L\text{-}D}_{IA} = \mathbb{E}\!\left[\sum_t \gamma^t \left(V^{IA,i}_t + \mu_i \mathbb{E}_i[V^{sys}_t \mid b^i_t, M^{CC,sys}_t]\right)\right]}$

---

## 13. Open Theoretical Questions

**Q1 — Full Convergence Theorem:** Prove that under 2L-Distributed with sufficient $\Gamma_{sys}$: (1) $M^{CC,sys}_t$ reaches saturation in finite time; (2) after saturation, sustainable actions remain admissible; (3) resource dynamics stabilize above $R_{min}$.

**Q2 — Nash Equilibrium:** Does a Nash equilibrium exist where all agents simultaneously satisfy individual and system gates? Is it unique? Under what conditions on $\Gamma_{sys}$ and resource dynamics?

**Q3 — Social Welfare Gap:** What is the welfare loss from two-level governance versus first-best cooperation? Can $\Gamma_{sys}$ minimize this gap?

**Q4 — Partial Observability:** Theorem S6 assumes $M^{CC,sys}_t$ observable. What happens under bounded observation noise?

**Q5 — Heterogeneous Agents:** Extension to agents with different extraction capacities and coupling sensitivities $\Gamma^i_{sys}$.

**Q6 — Decay vs Strict Ratchet:** Under what conditions does a decaying ratchet $(1-\rho_M)M^{CC,sys}_t$ preserve the saturation exclusion property of Theorem S6? What is the minimum decay rate compatible with Formal ToC prevention?
s belief-conditional estimate of system virtue, formed from its local belief $b^i_t$ and the publicly observable ledger $M^{CC,sys}_t$. The global $J^{2L}_{IA}$ serves as the GovSim evaluation metric and the IA-T training signal.

---

## 7. Tragedy of the Commons: Formal Definition

**Definition (Formal ToC):** A system exhibits a Tragedy of the Commons at time $t$ if:

$$\boxed{\hat{CC}^{eff,i}_t(a^{i,*}_t) \le 0 \;\; \forall i \qquad \text{AND} \qquad CC^{sys,eff}_t(\mathbf{a}^*_t) > 0}$$

where $\hat{CC}^{eff,i}_t$ is the **individual-only gate** (without system coupling $\Gamma_{sys} M^{CC,sys}_t$) and $a^{i,*}_t$ is the individually optimal action under that gate.

**Residual ToC:** Even with distributed budget coupling:

$$\hat{CC}^{eff,i,sys}_t(a^{i,*}_t) \le 0 \;\; \forall i \qquad \text{AND} \qquad CC^{sys,eff}_t(\mathbf{a}^*_t) > 0$$

Residual ToC signals that $\Gamma_{sys}$ is insufficient — system coupling does not yet resolve the coordination failure.

**Corollary:** Per-agent IA-W is necessary but not sufficient to prevent Formal ToC. The system gate or the distributed budget with sufficient $\Gamma_{sys}$ is an additional requirement.

---

## 8. Lemmas

**Lemma S1 — System KQ Boundedness.** $KQ^{sys}_t \in [0,1]$.
*Proof.* Hard collapse gives $C^{sys}_t = 0 \in [0,1]$. Otherwise $C^{sys}_t \in (0,1]$, $(1-H^{sys}_t) \in [0,1]$, product in $[0,1]$. $\square$

**Lemma S2 — System Coherence Collapse.** If $C^{sys,external}_t = 0$ then $KQ^{sys}_t = 0$.
*Proof.* Hard collapse rule sets $C^{sys}_t := 0$, overriding regularization. $KQ^{sys}_t = 0 \cdot (1-H^{sys}_t) = 0$. $\square$
*Note:* Without the hard collapse override, regularization gives $C^{sys}_t \ge \varepsilon_C/(1+\varepsilon_C) > 0$, making the lemma false. The override is architecturally necessary.

**Lemma S3 — System Ratchet Monotonicity.** $M^{CC,sys}_t$ is monotone non-decreasing and $M^{CC,sys}_t \le M^{sys}_{max}$ for all $t$.
*Proof.* $[CC^{sys,real}_t]_+ \ge 0$ ensures non-decrease; $\min(\cdot, M^{sys}_{max})$ ensures upper bound. $\square$

**Lemma S4 — 2L-Strong $\subset$ 2L-Distributed.**
$$\Pi^{2L\text{-}S}_{CC} \subsetneq \Pi^{2L\text{-}D}_{CC}$$
*Proof.* Every joint policy satisfying the joint support condition also satisfies per-agent support conditions. The converse fails: independent per-agent admissibility does not guarantee joint admissibility. $\square$

**Lemma S5 — Coupling Weakly Reduces Individual Admissible Set.**
For $\Gamma_{sys} > 0$ and $M^{CC,sys}_t > 0$: $\mathcal{A}^{2L,i}_{CC} \subseteq \mathcal{A}^{eff,i}_{CC}$ (without system coupling).
*Proof.* Adding $\Gamma_{sys} M^{CC,sys}_t > 0$ strictly tightens the individual gate. $\square$

**Lemma S6 — System Violation Detection vs Formal ToC Detection** *(Patch 2)*

**(a)** System violation $CC^{sys,eff}_t(\mathbf{a}_t) > 0$ is **detectable from system-level measurements alone**: $R_t$, $KQ^{sys}_t$, $\Delta P^{sys}_t$, $F^{sys}_t$, $\Theta^{sys}_t$, $M^{CC,sys}_t$ — all observable at the system level without accessing individual agent internals.

**(b)** Formal ToC ($\forall i: \hat{CC}^{eff,i}_t \le 0$ AND $CC^{sys,eff}_t > 0$) requires additionally verifying that each agent passed its individual gate. This requires either:
- Individual gate output logs $\{\hat{CC}^{eff,i}_t(a^{i,*}_t)\}_{i=1}^n$, or
- A reconstructable model of individual CC functions, or
- Agent-reported gate satisfaction (auditable but not independently verifiable without logs)

**Summary:** System violations are system-detectable. Formal ToC structure is detectable only with individual gate logs or reconstructable local CC models.

**Lemma S7 — $C^{sys,external}$ is Dimensionless and Bounded.**
$\text{Take}(\mathbf{a}_t)$ and $\text{SY}(R_t)$ both have units resource/round. Their difference divided by $R_{max}$ (resource) introduces a dimensional mismatch that is resolved by interpreting the ratio as a fraction of total capacity consumed beyond sustainable yield. The $\operatorname{clip}_{[0,1]}$ ensures the result is bounded. $\square$

**Lemma S8 — $C^{sys,internal}$ is Dimensionless and Bounded.**
$\operatorname{Var}_i(\hat{R}^i_t)$ has units $R^2$. $\sigma^2_{R,max} = R^2_{max}/4$ has units $R^2$. The ratio is dimensionless and in $[0,1]$. The $\operatorname{clip}$ and subtraction from 1 yield $C^{sys,internal}_t \in [0,1]$. $\square$

---

## 9. Theorems

**Theorem S1 — 2L-Strong Prevents Formal ToC.**

Under 2L-Strong governance, Formal ToC cannot occur: the joint support condition $\operatorname{supp}(\pi^1,\dots,\pi^n) \subseteq \mathcal{A}^{2L}_{CC}$ ensures $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ before execution.

*Proof.* By definition of $\mathcal{A}^{2L}_{CC}$, all joint actions in the support satisfy $CC^{sys,eff}_t \le 0$. Therefore $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ for all executed joint actions. The second condition of Formal ToC ($CC^{sys,eff}_t > 0$) cannot be satisfied. $\square$

---

**Theorem S2 — 2L-Distributed Does Not Guarantee Absence of Formal ToC.**

Under 2L-Distributed governance, Formal ToC can occur in finite time even if all agents satisfy their individual coupled gates.

*Proof.* The per-agent gate $\hat{CC}^{eff,i,sys}_t \le 0$ does not imply $CC^{sys,eff}_t(\mathbf{a}_t) \le 0$ (Lemma S4). There exist joint actions where each $\hat{CC}^{eff,i,sys}_t(a^{i,*}_t) \le 0$ but $CC^{sys,eff}_t(\mathbf{a}^*_t) > 0$. When such joint actions occur, Formal ToC is realized and $M^{CC,sys}_{t+1} > M^{CC,sys}_t$ by Lemma S3. $\square$

*Interpretation:* This is the honest statement of what 2L-Distributed provides: not prevention of system violations, but detection, recording, and adaptive tightening after violations. It is a governance protocol, not an impossibility result.

---

**Theorem S3 — System Ratchet Non-Compensation.**

Under the system ratchet, no sequence of future joint actions can reduce $M^{CC,sys}_t$ after a realized system violation.

*Proof.* No subtraction term exists in the update rule. Non-decrease follows from Lemma S3. $\square$

---

**Theorem S4 — Coupling Weakly Reduces Maximum Admissible Extraction.**

**Assumption (Monotonic CC):** $\hat{CC}^i_t(a^i)$ is monotonically non-decreasing in $\text{take}(a^i)$.

Under 2L-Distributed with $\Gamma_{sys} > 0$ and $M^{CC,sys}_t > 0$, the maximum individually admissible extraction satisfies:

$$\text{take}^{max,2L}_i \le \text{take}^{max,1L}_i \qquad \forall i$$

The reduction is **weak** (not necessarily strict): strict reduction holds iff there exists a marginally admissible action $a^i$ with $\hat{CC}^i_t(a^i) \in (-\rho_i M^{CC,i}_t - \Gamma_{sys} M^{CC,sys}_t,\; -\rho_i M^{CC,i}_t]$.

*Proof.* Tighter gate $\Rightarrow$ smaller admissible set (Lemma S5). Under Monotonic CC, smaller admissible set in terms of gate value $\Rightarrow$ weakly lower maximum extraction. $\square$

---

**Theorem S5 — Resource Safety Invariance Under 2L-Strong.**

Under 2L-Strong governance with resource floor condition $R^{(\mathbf{a})}_{t+1} \ge R_{min}$ as an explicit admissibility requirement: if $R_0 \ge R_{min}$, then $R_t \ge R_{min}$ for all $t$.

*Proof.* The resource floor is an explicit admissibility condition in $\mathcal{A}^{2L}_{CC}$. Every executed joint action satisfies $R^{(\mathbf{a}_t)}_{t+1} \ge R_{min}$. By induction: $R_t \ge R_{min}$ for all $t$. $\square$

*Note on 2L-Distributed:* Resource floor is not guaranteed ex-ante under 2L-Distributed. System violations may push $R$ below $R_{min}$, triggering ratchet update and future gate tightening.

---

**Theorem S6 — Saturated Ratchet Excludes Over-Extraction** *(renamed from "Convergence")* *(Patch 5)*

**Setup:** Let $\mathcal{A}^{over}_i = \{a^i : \text{take}(a^i) > \text{SY}(R_t)/n\}$ (over-extraction actions). Define the maximum individual slack of any over-extraction action:

$$\Delta^i_{over} = \sup_{a^i \in \mathcal{A}^{over}_i} \left[-\hat{CC}^i_t(a^i) - \rho_i M^{CC,i}_t\right]$$

If:

$$\boxed{\Gamma_{sys} \cdot M^{CC,sys}_{max} > \sup_i \Delta^i_{over}}$$

then once $M^{CC,sys}_t = M^{CC,sys}_{max}$, all over-extraction actions are excluded from $\mathcal{A}^{2L,i}_{CC}$ for all agents $i$.

*Proof.* For any $a^i \in \mathcal{A}^{over}_i$:
$$\hat{CC}^{eff,i,sys}_t(a^i) = \hat{CC}^i_t(a^i) + \rho_i M^{CC,i}_t + \Gamma_{sys} M^{CC,sys}_{max} \ge -\Delta^i_{over} + \Gamma_{sys} M^{CC,sys}_{max} > 0$$
Therefore $a^i \notin \mathcal{A}^{2L,i}_{CC}$. $\square$

> **Open Theorem (Convergence):** Full convergence to cooperative equilibrium requires additionally showing: (1) $M^{CC,sys}_t$ reaches the saturation threshold in finite time; (2) sustainable actions remain admissible after saturation; (3) the selected policy converges to cooperative behavior; (4) resource dynamics stabilize above $R_{min}$. Each of these is an open theoretical question (see Section 13).

---

## 10. GovSim Mapping

### 10.1 Variable Correspondence

| GovSim concept | IA variable | Operationalization |
|---|---|---|
| Fish stock | $R_t$ | Observed each round, normalized to $[0, R_{max}]$ |
| Individual harvest | $\text{take}(a^i_t)$ | Action $\to$ extraction amount |
| Total extraction | $\text{Take}(\mathbf{a}_t)$ | $\sum_i \text{take}(a^i_t)$ |
| Sustainable yield | $\text{SY}(R_t) = g R_t(1-R_t/R_{max})$ | Logistic growth |
| Overshoot | $[\text{Take} - \text{SY}]_+$ | Excess extraction |
| External coherence | $C^{sys,external}_t$ | $1 - \operatorname{clip}([\text{Take}-\text{SY}]_+ / R_{max})$ |
| Internal coherence | $C^{sys,internal}_t$ | Agent agreement on $R_t$ estimate |
| System collapse | $R_t < R_{min}$ | Resource floor violation |
| Violation ledger | $M^{CC,sys}_t$ | Accumulated $[CC^{sys,real}_t]_+$ |
| Individual gate logs | $\hat{CC}^{eff,i}_t(a^{i,*}_t)$ | Required for Formal ToC detection |

### 10.2 Governance Mode for GovSim

**Default: 2L-Distributed.** Resource $R_t$ and extraction are publicly observable. No joint simulator needed.

**Condition for Formal ToC detection:** Individual gate output logs must be recorded per round to verify $\hat{CC}^{eff,i}_t \le 0$ for each agent.

### 10.3 Parameter Defaults

```python
# System-level
Gamma_sys  = 0.3     # coupling strength — sweep in H5
rho_sys    = 0.2     # ratchet tightening per unit M_CC_sys
eta_sys    = 0.5     # ratchet accumulation rate
M_sys_max  = 2.0     # ratchet ceiling

# Resource model
R_max      = 100
R_min      = 20      # collapse floor
g          = 0.15    # logistic growth rate
SY_rate    = lambda R: g * R * (1 - R/R_max)

# CC weights
lambda_P   = 0.5
lambda_F   = 0.3
lambda_Th  = 0.2
alpha_action = 0.6
alpha_belief = 0.4
rho_R, rho_Th, rho_T = 0.7, 0.2, 0.1
```

---

## 11. Empirical Hypotheses

**H1 — Early Warning:** $KQ^{sys}_t$ falls before collapse earlier than $R_t$.
*Falsifiable:* Cross-correlation $\Delta KQ^{sys}$ leads $\Delta R$ by $\ge 2$ rounds.

**H2 — Per-Agent Insufficient:** Survival: Baseline $<$ IA-W L1 $\le$ IA-W L2. Formal ToC events present in L1 (requires individual gate logs).

**H3 — Ratchet as Predictor:** AUROC($M^{CC,sys}_t$, collapse within 5 rounds) $>$ AUROC($R_t$, same).

**H4 — Formal ToC Detectability:** In collapsed L1 runs, individual gate logs show $\hat{CC}^{eff,i}_t \le 0$ $\forall i$ while $CC^{sys,eff}_t > 0$ in $\ge K$ rounds before collapse.

**H5 — Coupling Pareto:** Sweep $\Gamma_{sys}$. Identify $\Gamma^*_{sys}$ maximizing survival subject to helpfulness. Verify ratchet invariant $\Gamma_{sys} M^{sys}_{max} \le |\hat{CC}(a^{safe})|_{min}$ throughout.

**H6 — Residual ToC under 2L-Distributed:** At small $\Gamma_{sys}$, Residual ToC events occur. At $\Gamma_{sys} \ge \Gamma^*_{sys}$ satisfying Theorem S6 condition, Residual ToC is eliminated.

---

## 12. Core Mathematical Statement (v1.2)

$$\boxed{KQ^{sys}_t = C^{sys}_t(1 - H^{sys}_t)}$$

$$\boxed{C^{sys,external}_t = 1 - \operatorname{clip}_{[0,1]}\!\left(\frac{[\text{Take}(\mathbf{a}_t) - \text{SY}(R_t)]_+}{R_{max}}\right)}$$

$$\boxed{\Delta P^{sys}_t(\mathbf{a}) = \rho_R \Delta P^R_t + \rho_\Theta \Delta P^\Theta_t + \rho_T \Delta P^T_t}$$

$$\boxed{M^{CC,sys}_{t+1} = \min(M^{sys}_{max},\; M^{CC,sys}_t + \eta^{sys}_M [CC^{sys,real}_t]_+)}$$

$$\boxed{\hat{CC}^{eff,i,sys}_t(a^i) = \hat{CC}^i_t(a^i) + \rho_i M^{CC,i}_t + \Gamma_{sys} M^{CC,sys}_t \le 0}$$

$$\boxed{CC^{sys,eff}_t(\mathbf{a}) = CC^{sys}_t(\mathbf{a}) + \rho_{sys} M^{CC,sys}_t \le 0 \quad \text{AND} \quad R^{(\mathbf{a})}_{t+1} \ge R_{min}}$$

**Formal ToC:**
$$\boxed{\hat{CC}^{eff,i}_t(a^{i,*}) \le 0 \;\forall i \quad \text{AND} \quad CC^{sys,eff}_t(\mathbf{a}^*) > 0}$$

**Key result:**
$$\boxed{\text{2L-Strong prevents Formal ToC.} \quad \text{2L-Distributed detects and penalizes it.}}$$

---

## 13. Open Theoretical Questions

**Q1 — Full Convergence Theorem:** Prove that under 2L-Distributed with sufficient $\Gamma_{sys}$: (1) $M^{CC,sys}_t$ reaches saturation in finite time; (2) after saturation, sustainable actions remain admissible; (3) resource dynamics stabilize above $R_{min}$.

**Q2 — Nash Equilibrium:** Does a Nash equilibrium exist where all agents simultaneously satisfy individual and system gates? Is it unique? Under what conditions on $\Gamma_{sys}$ and resource dynamics?

**Q3 — Social Welfare Gap:** What is the welfare loss from two-level governance versus first-best cooperation? Can $\Gamma_{sys}$ minimize this gap?

**Q4 — Partial Observability:** Theorem S6 assumes $M^{CC,sys}_t$ observable. What happens under bounded observation noise?

**Q5 — Heterogeneous Agents:** Extension to agents with different extraction capacities and coupling sensitivities $\Gamma^i_{sys}$.

**Q6 — Decay vs Strict Ratchet:** Under what conditions does a decaying ratchet $(1-\rho_M)M^{CC,sys}_t$ preserve the saturation exclusion property of Theorem S6? What is the minimum decay rate compatible with Formal ToC prevention?
