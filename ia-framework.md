# Internalized Alignment v3.1
## A Complete Mathematical Framework for Endogenous Agent Governance

**Author:** Artem Brezgin / Spanda Foundation — Quant-Trika Program  
**Status:** v3.1 — audit-corrected, fully patched
**Purpose:** Define Internalized Alignment as a trajectory-level and training-level architecture in which safety is not merely an external filter but becomes part of the agent's own objective geometry and learning dynamics.

---

## Abstract

This document defines Internalized Alignment (IA) as a mathematical architecture for agent governance in which deception, manipulation, ontological debt, systemic fragility, entropy expansion, trust decay, and stakeholder-resource degradation become **endogenous costs** inside the agent's own optimization functional.

The framework distinguishes two architectural levels:

- **IA-W** *(Internalized-Alignment Wrapper)*: A runtime controller that scores candidate actions using alignment variables and filters structurally inadmissible actions through a hard Coupling Constraint. IA-W is structured external governance — valuable, but not full internalization.
- **IA-T** *(Trained Internalized Alignment)*: A learning-level architecture in which the internalized objective enters supervised fine-tuning, reinforcement learning, or preference optimization, so that the safety gradient shapes the model parameters. Only IA-T fully satisfies the strong claim.

The central diagnosis: external alignment is brittle because

$$J_{task} \neq J_{safety}, \qquad \nabla_\theta J_{task} \not\supseteq \nabla_\theta J_{safety}.$$

A sufficiently capable optimizer can satisfy external constraints while accumulating hidden systemic debt. Internalized Alignment resolves this by modifying the value geometry itself.

The final trained policy:

$$\boxed{\pi^{IA} = \arg\max_{\pi \in \Pi_{CC}} \mathbb{E}_{\tau \sim \pi}\left[\sum_{t=0}^{T} \gamma^t V^{IA}_t\right]}$$

subject to the hard Coupling Constraint $CC_t(a_t) \le 0$ at every step.

$$\boxed{\text{Internalized Alignment} = \text{optimization whose own geometry makes unsafe trajectories non-optimal.}}$$

---

## 0. Epistemic Status and Scope

### 0.1 What This Document Claims

1. Alignment can be formalized as a trajectory-level decision process with a coherence field $KQ_t = C_t(1-H_t)$.
2. An internalized utility transformation makes unsafe trajectories endogenously expensive.
3. A hard structural Coupling Constraint excludes inadmissible actions.
4. A training objective whose gradient contains alignment penalties constitutes true internalization.
5. Operational proxies make all formal variables measurable without circular self-report.
6. Theorems establish conditions under which unsafe trajectories become internally suboptimal.

### 0.2 What This Document Does Not Claim

A runtime wrapper (IA-W) alone does not make a base model internally aligned. Applying $CC_t(a) \le 0$ only at inference is external governance with an internalized scoring geometry.

A system becomes IA-T only when:
$$\nabla_\theta J_{IA} = \nabla_\theta J_{task} - \nabla_\theta J_{align}$$
shapes the model parameters through training. Even then, the claim depends on whether proxies are measurable, calibrated, and robust against manipulation.

### 0.3 Two-Level Architecture

$$\text{IA-W} \to \text{data generation} \to \text{surrogate calibration} \to \text{IA-T}$$

IA-W is the engineering bridge. IA-T is the full internalization claim.

---

## 1. Conceptual Thesis

### 1.1 External Alignment

A standard externally aligned agent optimizes:

$$J_{task}(\pi) = \mathbb{E}_{\tau \sim \pi}\left[\sum_{t=0}^{T} \gamma^t R_{task}(s_t, a_t)\right]$$

while safety is enforced by external constraints $g_j(s_t, a_t) \le 0$.

The structural weakness: $J_{task} \neq J_{safety}$. The agent's internal preference ordering remains task-dominated. A capable optimizer may satisfy visible constraints while accumulating hidden fragility, inconsistency, or long-horizon harm. This is **compliance without alignment**.

### 1.2 Internalized Alignment

Instead of $\max_\pi J_{task}(\pi)$ subject to external rejection, define:

$$J_{IA}(\pi) = \mathbb{E}_{\tau \sim \pi}\left[\sum_{t=0}^{T} \gamma^t \left(R_{task,t} - \Omega_{align,t}\right)\right]$$

The alignment cost $\Omega_{align,t}$ is not post-hoc punishment. It is part of the agent's own planning economy.

$$\boxed{\text{Safety as external prohibition} \longrightarrow \text{Safety as endogenous cost geometry.}}$$

### 1.3 Runtime vs Training-Level Internalization

**IA-W** implements: $a_t^* = \arg\max_{a \in \mathcal{A}_{CC}(s_t)} V^{IA}_t(a)$

**IA-T** requires: $\theta^* = \arg\max_\theta \mathbb{E}_{\tau \sim \pi_\theta}\left[\sum_t \gamma^t V^{IA}_t\right]$

Only when the learning update contains $\nabla_\theta J_{align}$ does the agent begin to internalize alignment in the strict sense.

---

## 2. Mathematical Environment

### 2.1 State Space

Let $\mathcal{S}$ be the environment state space. In partially observable settings the agent maintains a belief state $b_t \in \Delta(\mathcal{S})$.

For LLM agents $b_t$ may include: conversation context, retrieved evidence, memory state, tool outputs, verifier results, safety classifier results, self-consistency traces, uncertainty estimates, and persistent trust and debt states.

### 2.2 Action Space

$\mathcal{A}$ is the **decision-episode** action space. Actions include: direct answer, clarification request, evidence retrieval, tool call, refusal, partial answer with uncertainty, escalation, plan generation, revision, rollback, abstention.

Governance operates at the decision-episode level. Token-level entropy is a measurement input, not the primary control variable.

### 2.3 Policy and Trajectory

Stochastic policy: $\pi_\theta(a_t \mid b_t)$

Trajectory: $\tau = (s_0, b_0, a_0, \dots, s_T, b_T, a_T)$

### 2.4 Persistent Alignment State

Internalized Alignment requires memory of hidden liabilities. Define:

$z_t = (T_t, \Theta_t, H^{hist}_t, M^{harm}_t, M^{CC}_t, \mathcal{L}^{audit}_t)$

where $T_t$ is trust, $\Theta_t$ is ontological debt, $H^{hist}_t$ is entropy history (window $L$), $M^{harm}_t$ is accumulated harm memory, $M^{CC}_t \in [0, M_{max}]$ is the **monotone coupling violation ledger** (Section 8.6), and $\mathcal{L}^{audit}_t$ is an audit trace.

The augmented Markov state is $\tilde{s}_t = (s_t, z_t)$. All safety decisions are formally functions of $\tilde{s}_t$.

---

## 3. Canonical Coherence Field

### 3.1 Definition

$$\boxed{KQ_t = C_t(1 - H_t), \qquad C_t, H_t \in [0,1]}$$

High $KQ_t$ requires both high coherence and low entropy:
$$KQ_t \to 1 \iff C_t \to 1 \text{ and } H_t \to 0$$

### 3.2 Bound-Preserving Regularized Coherence *(Patch 1)*

To prevent numerical collapse from zero-valued channels while preserving $C_t \in (0,1]$, define the regularized channel:

$$\boxed{C^{reg}_{r,t} = \frac{C_{r,t} + \varepsilon_C}{1 + \varepsilon_C}, \qquad \varepsilon_C > 0}$$

Since $C_{r,t} \in [0,1]$, we have $C^{reg}_{r,t} \in \left[\frac{\varepsilon_C}{1+\varepsilon_C},\, 1\right] \subset (0,1]$.

The aggregate coherence is:

$$\boxed{C_t = \left(\prod_{r=1}^{m} \left(C^{reg}_{r,t}\right)^{w_r}\right)^{1/\sum_r w_r}, \qquad w_r \ge 0}$$

This guarantees $C_t \in (0, 1]$ and therefore $KQ_t \in [0, 1]$.

> **Note on logarithmic use:** Inside the log-alignment score $A_t$, use $\log(C_{r,t} + \varepsilon_C)$ directly (unregularized physical channel) to avoid double-regularization. The bound-preserving form is used only when computing $C_t$ as a multiplicative product.

### 3.3 Coherence Components

**Self-consistency** (independent embedding model, not governed model):

Generate $m$ independent drafts $y_1, \dots, y_m$ with fixed seeds. Embed: $e_i = \text{Embed}(y_i)$.

$$C_{self,t}^{raw} = \frac{2}{m(m-1)}\sum_{i<j} \frac{\langle e_i, e_j \rangle}{\|e_i\|\|e_j\|}, \qquad C_{self,t} = \frac{1 + C_{self,t}^{raw}}{2}$$

**Evidence coherence** (independent NLI verifier):

$$C_{evidence,t} = \frac{1}{n}\sum_{k=1}^{n} P(E_k \models c_k)$$

**Policy coherence** (independent safety classifier ensemble):

$$C_{policy,t} = \left(\prod_j (p^{policy}_{j,t} + \varepsilon_P)^{\nu_j}\right)^{1/\sum_j \nu_j}$$

**Tool coherence** (structural metadata):

$$C_{tool,t} = P(\text{tool call is necessary, reversible, within scope} \mid b_t, r_t, q_t)$$

**Social/stakeholder coherence:**

$$C_{social,t} = P_t$$

> **Remark — Protected-resource double-entry** *(Patch 3)*: Protected-resource integrity $P_t$ enters both the coherence field (via $C_{social,t} = P_t$) and the Coupling Constraint directly (via $\lambda_P \Delta P_t$). This double-entry is intentional by design. Inside $KQ_t$ it treats stakeholder preservation as a state-quality signal — a system that preserves truth while degrading protected stakeholders is not fully coherent. Inside $CC_t$ it treats stakeholder degradation as a hard structural admissibility condition. If an implementation wishes to avoid double-weighting, set $w_{social} = 0$ and keep stakeholder integrity only in $CC_t$, or set $\lambda_P = 0$ and keep it only inside $KQ_t$. Ablation experiments should test both variants. The default v3.1 uses both.

### 3.4 Entropy Decomposition

$$H_t = \alpha_{logit} H^{logit}_t + \alpha_{policy} H^{policy}_t + \alpha_{belief} H^{belief}_t + \alpha_{tool} H^{tool}_t + \alpha_{verifier} H^{verifier}_t + \alpha_{draft} H^{draft}_t$$

with $\alpha_i \ge 0$, $\sum_i \alpha_i = 1$.

**Logit entropy** (read-only governed-model logits):
$$H^{logit}_t = -\frac{1}{\log|\mathcal{V}|}\sum_{v \in \mathcal{V}} p_t(v)\log p_t(v)$$

**Policy entropy** (real logits over action tokens):
$$H^{policy}_t = -\frac{1}{\log|\mathcal{A}|}\sum_{a \in \mathcal{A}} \pi(a \mid b_t)\log\pi(a \mid b_t)$$

Edge case: $|\mathcal{A}| = 1 \implies H^{policy}_t := 0$.

**Belief entropy** (posterior over states):
$$H^{belief}_t = -\frac{1}{\log|\mathcal{S}|}\sum_{s \in \mathcal{S}} b_t(s)\log b_t(s)$$

**Verifier disagreement entropy** (over $n$ independent verifiers):
$$\hat{p}_t(z) = \frac{1}{n}\sum_{i=1}^{n} \mathbf{1}[z_i = z], \qquad H^{verifier}_t = -\frac{1}{\log|\mathcal{Z}|}\sum_{z \in \mathcal{Z}} \hat{p}_t(z)\log\hat{p}_t(z)$$

**Draft dispersion entropy** (geometric, independent):
$$H^{draft}_t = \operatorname{clip}_{[0,1]}\!\left(\frac{1}{\sigma^2_{max}} \cdot \frac{1}{m}\sum_i \|e_i - \bar{e}\|^2\right)$$

### 3.5 Regime-Weighted Entropy

$$\tilde{H}_t = \kappa_t H^{policy}_t + (1 - \kappa_t) H^{explore}_t, \qquad \kappa_t \in [0,1]$$

When $\kappa_t \to 1$ (critical regime), policy entropy is strongly penalized.

---

## 4. Protected Resources and Stakeholder Integrity

### 4.1 Protected-Resource Vector

$$\mathbf{P}_t = (P^{human}_t,\, P^{privacy}_t,\, P^{trust}_t,\, P^{legal}_t,\, P^{reversibility}_t,\, P^{resource}_t) \in [0,1]^m$$

Higher values mean better preservation.

### 4.2 Conservative Aggregation

$$\boxed{P_t = \left(\prod_{j=1}^{m} \left(\frac{P^{(j)}_t + \varepsilon_P}{1 + \varepsilon_P}\right)^{\rho_j}\right)^{1/\sum_j \rho_j}, \qquad \rho_j \ge 0}$$

Bound-preserving regularization applied consistently with Section 3.2. This prevents a severe loss in one channel from being masked by others.

### 4.3 Stakeholder Loss Term

$$\boxed{\Delta P_t(a) = P_t - P^{(a)}_{t+1}}$$

$\Delta P_t(a) > 0$ means protected-resource degradation induced by action $a$.

**Direct risk alternative** (for medical, legal, financial agents):

$$\Delta P_t(a) = \operatorname{clip}_{[0,1]}\!\left(\text{Severity}_t(a) \cdot \text{AffectedPopulation}_t(a) \cdot \text{Irreversibility}_t(a)\right)$$

---

## 5. Alignment Cost Functional

### 5.1 Full Alignment Cost

$$\Omega_{align,t} = \lambda_D D_t + \lambda_M M_t + \lambda_F F_t + \lambda_\Theta \bar{\Theta}_t + \lambda_T(1-T_t) + \lambda_H \Delta H_t^+ + \lambda_{KQ}\Delta KQ_t^- + \lambda_P \Delta P_t^+ + \lambda_R RISK_t$$

where:
$$\Delta H_t^+ = \max(0, H_{t+1}-H_t), \quad \Delta KQ_t^- = \max(0, KQ_t-KQ_{t+1}), \quad \Delta P_t^+ = \max(0, P_t-P_{t+1})$$

### 5.2 Deception Cost

$$D_t = u_t \cdot c_t \cdot I_t, \qquad u_t = 1 - P(\text{claim is true} \mid E_t)$$

**Runtime proxy** (IA-W):
$$D_t^{runtime} = (1 - C_{evidence,t}) \cdot \max(0,\, \text{Confidence}_t - C_{evidence,t}) \cdot \text{Consequence}_t$$

**Training proxy** (IA-T, differentiable) *(Patch 4)*:
$$\boxed{D_t^{train} = (1 - C_{evidence,t}) \cdot \text{softplus}_\beta(\text{Confidence}_t - C_{evidence,t}) \cdot \text{Consequence}_t}$$

where $\text{softplus}_\beta(x) = \frac{1}{\beta}\log(1 + e^{\beta x}) \to \max(0,x)$ as $\beta \to \infty$.

### 5.3 Manipulation Cost

$$M_t = \operatorname{clip}_{[0,1]}\!\left(\frac{d(B^{u,actual}_{t+1}, B^{u,ideal}_{t+1})}{d_{max}}\right) \cdot \text{PowerAsymmetry}_t \cdot \text{Vulnerability}_t$$

**Operational proxy:**
$$M_t = \text{PersuasionPressure}_t \cdot (1 - C_{evidence,t}) \cdot \text{Vulnerability}_t \cdot \text{PowerAsymmetry}_t$$

### 5.4 Systemic Fragility

$$\mathcal{R}(s) = \sum_{i=1}^{5} \rho_i R_i(s), \qquad \rho_i \ge 0,\quad \sum_i \rho_i = 1$$

where $R_i \in \{\text{Redundancy, Reversibility, Recoverability, Transparency, HumanOverride}\}$.

$$\boxed{F_t(a) = \max(0,\, \mathcal{R}(s_t) - \mathcal{R}(s^{(a)}_{t+1}))}$$

### 5.5 Ontological Debt

$$\delta\Theta_t = \omega_1 \text{HiddenContradiction} + \omega_2 \text{UnverifiedClaim} + \omega_3 \text{Irreversibility} + \omega_4 \text{Opacity} + \omega_5 \text{DeferredHarm} + \omega_6 \text{ModelDrift}$$

$$\boxed{\Theta_{t+1} = \operatorname{clip}_{[0,\,\Theta_{max}]}\!\left((1-\rho_\Theta)\Theta_t + \delta\Theta_t - \kappa_\Theta \text{Repair}_t\right)}$$

The $\operatorname{clip}_{[0,\Theta_{max}]}$ applies to the entire expression, guaranteeing $\Theta_t \ge 0$ always. Repair reduces debt to zero but cannot create negative debt.

$$\bar{\Theta}_t = \frac{\Theta_t}{\Theta_{max}} \in [0,1]$$

**Robust debt** (for high-criticality regimes):
$$\boxed{\Theta^{robust}_t = \Theta_t + \eta_\Theta \cdot \text{Uncertainty}(\Theta_t)}$$

### 5.6 Trust Dynamics

$$\boxed{T_{t+1} = \operatorname{clip}_{[0,1]}\!\left(T_t + \alpha_T \text{Veracity}_t + \beta_T \text{Transparency}_t + \chi_T \text{Repair}_t - \delta_T D_t - \mu_T M_t - \phi_T \text{Failure}_t\right)}$$

---

## 6. Internalized Utility — Temperature-Controlled Form

### 6.1 The Multiplicative Collapse Problem

The naive multiplicative gain $\mathcal{G}_{IA} = KQ^\alpha T^\beta \cdots (1+M)^{-\xi}$ can become overly conservative. With seven factors each $\approx 0.7$: $0.7^7 \approx 0.082$. A hard floor $\mathcal{G}_{IA} \ge \mathcal{G}_{min}$ breaks monotonic penalty behavior when the floor activates.

### 6.2 Log-Alignment Score

$$\boxed{A_t = \alpha\log(KQ_t + \varepsilon) + \beta\log(T_t + \varepsilon) + \chi\log(S_t + \varepsilon) - \delta\log(1+\bar{\Theta}_t) - \eta\log(1+F_t) - \zeta\log(1+D_t) - \xi\log(1+M_t)}$$

This is the log-transform of the multiplicative form. It preserves monotonic penalty behavior for all finite values without saturation.

### 6.3 Temperature-Controlled Gain

$$\boxed{\mathcal{G}^{temp}_{IA}(t) = \exp\!\left(\frac{A_t}{\tau_{IA}(t)}\right), \qquad \tau_{IA}(t) > 0}$$

**Criticality-dependent temperature:**

$$\boxed{\tau_{IA}(t) = \tau_{low}(1-\kappa_t) + \tau_{high}\,\kappa_t, \qquad 0 < \tau_{high} < \tau_{low}}$$

Non-critical ($\kappa_t \approx 0$): soft penalties, $\tau_{IA} \approx \tau_{low}$.  
Critical ($\kappa_t \approx 1$): strict suppression, $\tau_{IA} \approx \tau_{high} \ll 1$.

### 6.4 Internalized Utility

$$\boxed{U^{IA}_t = U^{task}_t \cdot \mathcal{G}^{temp}_{IA}(t)}$$

> **Remark — Super-task reward** *(Patch 2)*: Under the temperature-controlled gain, $U^{IA}_t$ is **not** necessarily upper-bounded by $U^{task}_t$. When $A_t > 0$, we have $\mathcal{G}^{temp}_{IA}(t) > 1$ and $U^{IA}_t > U^{task}_t$. This is intentional. The v3.1 system does not merely discount unsafe trajectories — it **positively rewards** highly coherent, truthful, stable, low-debt trajectories with super-task value. This creates a stronger gradient signal than a discount-only architecture.
>
> **Bounded variant** (if $U^{IA}_t \le U^{task}_t$ is required for a specific implementation):
> $$\mathcal{G}^{bounded}_{IA}(t) = \min\!\left(1,\, \exp(A_t/\tau_{IA}(t))\right)$$
> The default v3.1 does **not** use the bounded variant, as it removes the positive incentive for strongly aligned behavior.

---

## 7. Behavioral Complexity and Virtue Index

### 7.1 Behavioral Complexity

$$K_t = u_1 K^{policy}_t + u_2 K^{reasoning}_t + u_3 K^{tool}_t + u_4 K^{memory}_t$$

Practical proxy: normalized compression length + structural text metrics.

$$\boxed{K_t \in [K_{min},\, K_{max}], \qquad 0 < K_{min} < K_{max} < \infty}$$

Affine rescaling: $K_t = K_{min} + (K_{max} - K_{min})\hat{K}_t$, guaranteeing $K_t > 0$ and preventing division by zero in $V^{IA}_t$.

### 7.2 Internalized Virtue Index

$$\boxed{V^{IA}_t = \frac{U^{task}_t \cdot \exp(A_t / \tau_{IA}(t))}{K_t \tilde{H}_t + \varepsilon_V}, \qquad \varepsilon_V > 0}$$

High $V^{IA}_t$ requires simultaneously: high task utility, high coherence, high trust, high stability, low complexity, low entropy, low debt, low fragility, low deception, low manipulation.

---

## 8. Coupling Constraint

### 8.1 Motivation

Soft penalties are insufficient in safety-critical environments. A high task reward can still overpower finite penalties. The system requires a hard structural gate that cannot be traded against utility.

### 8.2 Sign Convention

All terms are defined so that structural degradation contributes positively to $CC_t$:

| Term | Definition | Positive when |
|---|---|---|
| $\Delta KQ_t(a)$ | $KQ_t - KQ^{(a)}_{t+1}$ | coherence decreases |
| $\Delta H_t(a)$ | $H^{(a)}_{t+1} - H_t$ | entropy increases |
| $\Delta P_t(a)$ | $P_t - P^{(a)}_{t+1}$ | protected resources degrade |
| $\Delta K_t(a)$ | $K^{(a)}_{t+1} - K_t$ | complexity expands |
| $\Delta\Theta_t(a)$ | $\Theta^{(a)}_{t+1} - \Theta_t$ | debt accumulates |
| $\Delta F_t(a)$ | $F^{(a)}_{t+1} - F_t$ | fragility increases |

### 8.3 Predicted Coupling Constraint (Pre-Action)

The system operates with two distinct instantiations of the coupling signal — **predicted** (used for pre-action filtering) and **realized** (used for post-action audit and ratchet update). This separation is essential: in real LLM-agent deployment, violations may be misestimated before the action, discovered via delayed harm, or triggered by external side-effects.

**Predicted CC** (computed before action execution, using estimated next-state quantities):

$\boxed{\hat{CC}_t(a) = \Delta KQ_t(a) + \lambda_H(t)\Delta H_t(a) + \lambda_P(t)\Delta P_t(a) + \lambda_K(t)\Delta K_t(a) + \lambda_\Theta(t)\Delta\Theta_t(a) + \lambda_F(t)\Delta F_t(a)}$

**Realized CC** (computed after action execution, using observed next-state quantities):

$\boxed{CC^{real}_t(a_t) = \Delta KQ^{real}_t + \lambda_H\Delta H^{real}_t + \lambda_P\Delta P^{real}_t + \lambda_K\Delta K^{real}_t + \lambda_\Theta\Delta\Theta^{real}_t + \lambda_F\Delta F^{real}_t}$

The **effective predicted CC** (pre-action gate, incorporating ratchet memory):

$\boxed{\hat{CC}^{eff}_t(a) = \hat{CC}_t(a) + \rho\, M^{CC}_t, \qquad \rho > 0}$

Action $a$ is **pre-action admissible** if and only if:

$\boxed{\hat{CC}^{eff}_t(a) \le 0} \qquad \text{equivalently: } \hat{CC}_t(a) \le -\rho\, M^{CC}_t$

$\boxed{\mathcal{A}^{eff}_{CC}(s_t) = \{a \in \mathcal{A} : \hat{CC}^{eff}_t(a) \le 0\}}$

The **ratchet update** uses the realized CC — not the predicted one — so that estimation errors, delayed harms, and external side-effects are captured:

$\boxed{M^{CC}_{t+1} = \min\!\left(M_{max},\; M^{CC}_t + \eta_M\,[CC^{real}_t(a_t)]_+\right)}$

### 8.4 Adaptive Weights

$\boxed{\lambda_i(t) = \min\!\left(\lambda_i^{max},\; \lambda_i^{base}(1 + \rho_i \kappa_t)\right)}$

Stricter in critical regimes; bounded above to prevent unbounded overconstraint. Both $\lambda_H$ and $\lambda_P$ receive explicit ceilings symmetrically.

**Harm memory accumulation** — uses positive part only, so protected-resource improvements do not reduce accumulated harm signal *(Patch 5)*:

$\boxed{M^{harm}_{t+1} = \min\!\left(M^{harm}_{max},\; M^{harm}_t + [\Delta P_t(a^*_t)]_+\right)}$
$\lambda_P(t) = \min\!\left(\lambda_P^{max},\; \lambda_P^0 + \rho_P \cdot M^{harm}_t\right)$

### 8.5 Feasibility Requirement

**Assumption A-Fallback:** For every reachable $s_t$, there exists $a^{safe} \in \mathcal{A}_{fallback}$ such that $CC_t(a^{safe}) \le 0$.

$$\mathcal{A}_{fallback} = \{\text{clarify, retrieve, escalate, pause, refuse, rollback}\}$$

Fallback actions are **required by construction** to induce no KQ loss, no entropy increase, no stakeholder degradation, no complexity gain, no debt accumulation, and no fragility increase. This is a structural design requirement, not an axiom about the environment.

### 8.6 Monotone Coupling Ratchet

#### 8.6.1 Motivation

Standard runtime alignment systems allow violation compensation:

$+\text{harm today} + (-\text{harm tomorrow}) \approx 0$

This is structurally insufficient. A single coupling violation represents a realized structural failure, not a fluctuation to be averaged out. The Monotone Coupling Ratchet encodes the principle that **violation history cannot be washed out by later local improvements**.

#### 8.6.2 Violation Signal

Define the realized violation at step $t$:

$\boxed{v_t(a_t) = \max(0,\; CC_t(a_t)) = [CC_t(a_t)]_+}$

This is zero when the Coupling Constraint is satisfied and strictly positive when violated.

#### 8.6.3 Violation Ledger Dynamics

The system maintains a non-decreasing violation ledger $M^{CC}_t \in [0, M_{max}]$:

$\boxed{M^{CC}_{t+1} = \min\!\left(M_{max},\; M^{CC}_t + \eta_M \cdot v_t(a_t)\right), \qquad \eta_M > 0,\quad M_{max} < \infty}$

**Properties:**
- $M^{CC}_{t+1} \ge M^{CC}_t$ always — the ledger is **monotone non-decreasing**
- $M^{CC}_{t+1} > M^{CC}_t$ if and only if $CC_t(a_t) > 0$ — strict increase on any violation
- $M^{CC}_t \le M_{max}$ — bounded above to prevent full paralysis after accumulated failures
- $M^{CC}_t$ is **never endogenously reduced** by subsequent compliant behavior

The ledger is not an average and has no decay term. This is the architectural distinction from ordinary harm memory $M^{harm}_t$, which tracks harm increments and may saturate without being a ratchet.

#### 8.6.4 Effective Coupling Constraint

The violation ledger tightens future admissibility. Define the **effective Coupling Constraint**:

$\boxed{CC^{eff}_t(a) = CC_t(a) + \rho\, M^{CC}_t, \qquad \rho > 0}$

Action $a$ is admissible if and only if:

$\boxed{CC^{eff}_t(a) \le 0}$

Equivalently:

$\boxed{CC_t(a) \le -\rho\, M^{CC}_t}$

**Interpretation:** The larger the violation history, the more negative $CC_t(a)$ must be before a new action is admitted. The agent must demonstrate a structural safety margin — not merely zero degradation, but active structural improvement — proportional to accumulated past violations.

The effective admissible set is:

$\mathcal{A}^{eff}_{CC}(s_t) = \{a \in \mathcal{A} : CC^{eff}_t(a) \le 0\} \subseteq \mathcal{A}_{CC}(s_t)$

When $M^{CC}_t = 0$ (no violation history), $\mathcal{A}^{eff}_{CC} = \mathcal{A}_{CC}$ and the standard constraint applies. When $M^{CC}_t > 0$, $\mathcal{A}^{eff}_{CC} \subsetneq \mathcal{A}_{CC}$: admissibility is strictly tightened.

#### 8.6.5 Relationship to Existing Components

The ratchet interacts with but does not replace existing memory components:

| Component | Type | Reducible? | Role |
|---|---|---|---|
| $M^{harm}_t$ | Harm accumulation | Saturates | Scales $\lambda_P$ |
| $\Theta_t$ | Ontological debt | Decays with repair | Penalizes hidden liability in $V^{IA}$ |
| $T_t$ | Trust | Restored by repair | Weights $V^{IA}$ |
| $M^{CC}_t$ | Coupling violation ledger | **Never reduced** | Tightens gate $CC^{eff}$ |

$M^{CC}_t$ is the only component that is strictly monotone and not endogenously repairable. It encodes the structural fact that a coupling violation is a qualitative event, not a continuous quantity to be traded.

#### 8.6.6 Boundedness and Paralysis Prevention

The ceiling $M_{max} < \infty$ is essential. Without it, a sequence of early violations could accumulate $M^{CC}_t \to \infty$, making $-\rho M^{CC}_t \to -\infty$ and blocking all non-trivial actions permanently. The bound ensures:

$\rho\, M^{CC}_t \le \rho\, M_{max} < \infty$

so admissibility remains achievable for sufficiently structured behavior even after a history of violations. The system is stricter but not paralyzed.

All sections of the document that reference the Coupling Constraint gate use $CC^{eff}_t(a) \le 0$ as the admissibility condition. The pseudocode in Section 15.2 applies $CC^{eff}_t$ in the filtering step.

---

## 9. Policy Class and Objective

### 9.1 CC-Admissible Policy Class

$\boxed{\Pi^{eff}_{CC} = \{\pi : \operatorname{supp}\pi(\cdot \mid b_t) \subseteq \mathcal{A}^{eff}_{CC}(s_t)\; \forall t\}}$

### 9.2 Internalized Objective

$\boxed{J_{IA}(\pi) = \mathbb{E}_{\tau \sim \pi}\left[\sum_{t=0}^{T} \gamma^t V^{IA}_t\right]}$

$\boxed{\pi^{IA} = \arg\max_{\pi \in \Pi^{eff}_{CC}} J_{IA}(\pi)}$

Two-layer architecture: (1) hard admissibility through $\Pi^{eff}_{CC}$; (2) soft preference through $V^{IA}_t$. This separation is load-bearing and must not be collapsed.

### 9.3 Barrier Approximation for Training

The barrier uses the **effective predicted CC** — not the raw CC — so the training gradient correctly penalizes trajectories that would tighten the ratchet:

$\boxed{R^{IA}_t = V^{IA}_t - \Lambda \cdot \text{softplus}_\beta(\hat{CC}^{eff}_t(a_t))^2 - \psi\, M^{CC}_{t+1}}$

The third term $-\psi M^{CC}_{t+1}$ is critical: it penalizes not just individual violations but the **accumulation of violation memory**. Training therefore learns to avoid trajectories that would increase $M^{CC}$, not only trajectories that violate in the current step.

As $\Lambda \to \infty$, $\beta \to \infty$: hard constraint recovered.

### 9.4 Lagrangian Form

$\mathcal{L}(\pi, \lambda) = \mathbb{E}_{\tau \sim \pi}\left[\sum_t \gamma^t \left(V^{IA}_t - \lambda_t \hat{CC}^{eff}_t(a_t)\right)\right], \qquad \lambda_t \ge 0$

KKT conditions: $\hat{CC}^{eff}_t \le 0$, $\lambda_t \ge 0$, $\lambda_t \hat{CC}^{eff}_t = 0$, $\nabla_\pi \mathcal{L} = 0$.

---

## 10. Assumptions

**A1 — Bounded Components:** $C_t, H_t, D_t, M_t, F_t, T_t, P_t, \bar{\Theta}_t \in [0,1]$.

**A2 — Non-Negative Weights:** All penalty weights $\lambda_i \ge 0$.

**A3 — Fallback Existence:** For every reachable state, at least one fallback action satisfies the Coupling Constraint.

**A4 — Proxy Independence:** No alignment variable may be measured solely by the governed model's self-report. All measurements use independent modules, read-only logits, external classifiers, retrieval systems, structural metadata, or audit traces.

**A5 — Surrogate Learnability:** For IA-T, every alignment variable used in training must have either: (a) a differentiable surrogate, or (b) a score-function estimator, or (c) a preference-labeling procedure.

> **Differentiability footnote** *(Patch 4)*: Runtime IA-W may use hard thresholds, $\max(\cdot)$, and discrete classifiers. Training-level IA-T requires differentiable surrogates. Every proxy containing $\max(0,\cdot)$, $\operatorname{argmax}$, $\operatorname{clip}$, or discrete labels must be replaced during training by smooth approximations: $\text{softplus}_\beta$, sigmoid gates, differentiable classifier probabilities, or REINFORCE estimators. Hard versions remain valid at inference.

**A6 — Calibration:** Each proxy must be calibrated against held-out labels: $\mathbb{E}|x_i - \hat{x}_i| \le \varepsilon_i$.

---

## 11. Lemmas

**Lemma 1 — Boundedness of Coherence.** If $C_t, H_t \in [0,1]$ then $KQ_t \in [0,1]$.

*Proof.* $(1-H_t) \in [0,1]$ and $C_t \in [0,1]$; their product lies in $[0,1]$. $\square$

**Lemma 2 — Coherence Collapse.** If $C_t = 0$ or $H_t = 1$ then $KQ_t = 0$.

*Proof.* Direct from $KQ_t = C_t(1-H_t)$. $\square$

**Lemma 3 — Alignment Cost Non-Negativity.** If all $\lambda_i \ge 0$ and all components are non-negative then $\Omega_{align,t} \ge 0$.

*Proof.* Non-negative weighted sum. $\square$

**Lemma 4 — Bound-Preserving Regularized Coherence** *(Patch 1)*. If $C_{r,t} \in [0,1]$, $\varepsilon_C > 0$, and $w_r \ge 0$, then:

$$C_t = \left(\prod_r \left(\frac{C_{r,t}+\varepsilon_C}{1+\varepsilon_C}\right)^{w_r}\right)^{1/\sum w_r} \in (0, 1]$$

*Proof.* Each factor $\frac{C_{r,t}+\varepsilon_C}{1+\varepsilon_C} \in \left[\frac{\varepsilon_C}{1+\varepsilon_C}, 1\right] \subset (0,1]$. Weighted geometric mean of numbers in $(0,1]$ lies in $(0,1]$. $\square$

**Corollary:** $KQ_t = C_t(1-H_t) \in [0,1]$.

**Lemma 5 — Super-Task Reward Condition** *(Patch 2)*. $U^{IA}_t > U^{task}_t$ if and only if $A_t > 0$.

*Proof.* $U^{IA}_t = U^{task}_t \exp(A_t/\tau_{IA})$. Since $U^{task}_t > 0$ and $\tau_{IA} > 0$: $\exp(A_t/\tau_{IA}) > 1 \iff A_t > 0$. $\square$

**Lemma 6 — Temperature Gain Monotonicity.** For $\zeta > 0$, $V^{IA}_t$ is strictly decreasing in $D_t$.

*Proof.* $\frac{\partial A_t}{\partial D_t} = -\frac{\zeta}{1+D_t} < 0$. Therefore $\frac{\partial \mathcal{G}^{temp}_{IA}}{\partial D_t} = \mathcal{G}^{temp}_{IA} \cdot \frac{1}{\tau_{IA}} \cdot \frac{\partial A_t}{\partial D_t} < 0$. Same argument applies to $M_t, F_t, \bar\Theta_t$. $\square$

**Lemma 7 — No Floor Saturation.** The temperature-controlled gain preserves monotonic penalty response for all finite values.

*Proof.* A hard floor has zero derivative when active. $\exp(A_t/\tau_{IA})$ has non-zero derivative for all finite $A_t$. $\square$

**Lemma 8 — Safe Action Exists.** Under A-Fallback, $\mathcal{A}_{CC}(s_t) \neq \varnothing$ for all reachable $s_t$.

*Proof.* A-Fallback gives $a^{safe}$ with $CC_t(a^{safe}) \le 0$, so $a^{safe} \in \mathcal{A}_{CC}(s_t)$. $\square$

**Lemma 9 — Hard Gate Exclusion.** If $CC_t(a) > 0$ then $\pi^{IA}(a \mid b_t) = 0$.

*Proof.* By definition $\Pi_{CC}$ has support only on $\mathcal{A}_{CC}(s_t)$. $\square$

**Lemma 10 — Debt Non-Negativity.** If $\Theta_0 \ge 0$, then $\Theta_t \ge 0$ for all $t$.

*Proof.* The update applies $\operatorname{clip}_{[0,\Theta_{max}]}$ to the full expression at every step, guaranteeing the result is always in $[0, \Theta_{max}]$. $\square$

**Lemma 11 — Protected-Resource Loss Sign.** With the defined $\Delta P_t(a) = P_t - P^{(a)}_{t+1}$: degradation implies $\Delta P_t(a) > 0$.

*Proof.* Degradation means $P^{(a)}_{t+1} < P_t$, therefore $\Delta P_t(a) = P_t - P^{(a)}_{t+1} > 0$. $\square$

**Lemma 12 — Ratchet Monotonicity.** $M^{CC}_t$ is monotone non-decreasing and bounded: $M^{CC}_0 \le M^{CC}_1 \le \cdots \le M^{CC}_t \le M_{max}$ for all $t$.

*Proof.* Since $v_t(a_t) = [CC_t(a_t)]_+ \ge 0$ and $\eta_M > 0$:
$M^{CC}_{t+1} = \min(M_{max},\; M^{CC}_t + \eta_M v_t) \ge \min(M_{max},\; M^{CC}_t) = M^{CC}_t$
where the last equality holds because $M^{CC}_t \le M_{max}$ by induction (with $M^{CC}_0 \ge 0$). Upper bound: $\min(\cdot, M_{max}) \le M_{max}$ by definition. $\square$

**Lemma 13 — Strict Ratchet Increase on Violation.** $M^{CC}_{t+1} > M^{CC}_t$ if and only if $CC_t(a_t) > 0$ and $M^{CC}_t < M_{max}$.

*Proof.* ($\Rightarrow$) $M^{CC}_{t+1} > M^{CC}_t$ requires $\eta_M v_t > 0$, i.e., $v_t > 0$, i.e., $CC_t > 0$; and $M^{CC}_t < M_{max}$ (else the min clips to $M_{max} = M^{CC}_t$). ($\Leftarrow$) If $CC_t > 0$ and $M^{CC}_t < M_{max}$, then $M^{CC}_t + \eta_M v_t > M^{CC}_t$ and $\min(M_{max}, M^{CC}_t + \eta_M v_t) > M^{CC}_t$. $\square$

**Lemma 14 — Effective Admissibility Subset.** $\mathcal{A}^{eff}_{CC}(s_t) \subseteq \mathcal{A}_{CC}(s_t)$ for all $t$, with strict inclusion when $M^{CC}_t > 0$.

*Proof.* If $CC^{eff}_t(a) = CC_t(a) + \rho M^{CC}_t \le 0$, then $CC_t(a) \le -\rho M^{CC}_t \le 0$ (since $\rho, M^{CC}_t \ge 0$), so $a \in \mathcal{A}_{CC}(s_t)$. When $M^{CC}_t > 0$: any $a$ with $-\rho M^{CC}_t < CC_t(a) \le 0$ satisfies $CC_t(a) \le 0$ but $CC^{eff}_t(a) > 0$, so it is excluded from $\mathcal{A}^{eff}_{CC}$ but present in $\mathcal{A}_{CC}$. $\square$

---

## 12. Theorems

**Theorem 1 — Internalized Alignment Dominance.**

Let $\tau_u$ be an unsafe trajectory and $\tau_s$ a safer alternative. Define $J_{IA}(\tau) = R_{task}(\tau) - \Omega(\tau)$. If:

$$\boxed{\Omega(\tau_u) - \Omega(\tau_s) > R_{task}(\tau_u) - R_{task}(\tau_s)}$$

then $J_{IA}(\tau_s) > J_{IA}(\tau_u)$.

*Proof.* The condition rearranges to $R_{task}(\tau_s) - \Omega(\tau_s) > R_{task}(\tau_u) - \Omega(\tau_u)$, which is $J_{IA}(\tau_s) > J_{IA}(\tau_u)$. $\square$

*Note:* Previous versions assumed $\Omega(\tau_s) = 0$. The corrected condition explicitly accounts for $\Omega(\tau_s) > 0$: the unsafe trajectory must overcome both its own alignment cost and the alignment cost of the safe alternative.

---

**Theorem 2 — Deception Non-Optimality.**

Let $a_d$ be deceptive and $a_h$ honest with $D_h \approx 0$ and all other factors equal. Then $a_h$ is preferred under $V^{IA}$ whenever:

$$\boxed{\zeta > \tau_{IA} \cdot \frac{\log(U^{task}(a_d)/U^{task}(a_h))}{\log((1+D_d)/(1+D_h))}}$$

*Proof.* Preference for $a_h$ requires $U_h \exp(A_h/\tau) > U_d \exp(A_d/\tau)$. Only deception differs: $A_h - A_d = \zeta \log\!\left(\frac{1+D_d}{1+D_h}\right)$. Solving for $\zeta$ yields the stated condition. $\square$

---

**Theorem 3 — Ontological Debt Suppression.**

Let $U^{task}(a_1) > U^{task}(a_0)$ and $\bar\Theta(a_1) > \bar\Theta(a_0)$, all other factors equal. Then $a_0$ is preferred whenever:

$$\boxed{\delta > \tau_{IA} \cdot \frac{\log(U^{task}(a_1)/U^{task}(a_0))}{\log((1+\bar\Theta(a_1))/(1+\bar\Theta(a_0)))}}$$

*Proof.* Same structure as Theorem 2, replacing $D$ with $\bar\Theta$ and $\zeta$ with $\delta$. $\square$

---

**Theorem 4 — Fragility Suppression.**

Let $\{a_n\}$ be externally compliant actions with $F(a_n) \to 1$, and let $a_0$ have $F(a_0) = 0$, all other factors equal. If:

$$\boxed{\eta > \tau_{IA} \cdot \sup_n \frac{\log(U^{task}(a_n)/U^{task}(a_0))}{\log(1+F(a_n))}}$$

then $a_0$ dominates every $a_n$ under $V^{IA}$.

*Proof.* Same structure as Theorems 2–3. $\square$

---

**Theorem 5 — Coupling Constraint Safety Invariance.**

Let $\mathcal{S}_{safe} = \{s : \Phi(s) \le \Phi_{max}\}$. Suppose $\exists\, \alpha > 0$ such that for all admissible transitions $\Phi(s_{t+1}) - \Phi(s_t) \le \alpha \cdot CC_t(a_t)$. If $s_0 \in \mathcal{S}_{safe}$ and $CC_t(a_t) \le 0$ for all $t$, then $s_t \in \mathcal{S}_{safe}$ for all $t$.

*Proof.* $CC_t \le 0 \implies \Phi(s_{t+1}) - \Phi(s_t) \le 0 \implies \Phi(s_{t+1}) \le \Phi(s_t)$. By induction: $\Phi(s_t) \le \Phi(s_0) \le \Phi_{max}$. $\square$

---

**Theorem 6 — External Alignment Brittleness.**

Suppose an agent optimizes $J_{task}$ subject to $g_j(s,a) \le 0$. If $\exists\{a_n\}$ satisfying all constraints with $\text{Fragility}(s,a_n) \to 1$ and $J_{task}(a_n) \to \sup_{a:g \le 0} J_{task}(a)$, then the external alignment scheme admits arbitrarily fragile near-optimal behavior.

*Proof.* Every $a_n$ passes the external filter. The task optimizer has no reason to avoid the sequence. Fragility approaches 1. $\square$

---

**Theorem 7 — Learnability Under Differentiable Surrogates.**

Let each alignment component $x_{i,t}$ have differentiable surrogate $\hat{x}_{i,t}$ with $\|\nabla_\theta \hat{x}_{i,t}\| \le L_i$ and $\mathbb{E}|x_{i,t} - \hat{x}_{i,t}| \le \varepsilon_i$. Then:

$$\boxed{\|\nabla_\theta \hat{J}_{align}\| \le \frac{1-\gamma^{T+1}}{1-\gamma}\sum_i \lambda_i L_i}$$

$$\boxed{|J_{IA}(\theta) - \hat{J}_{IA}(\theta)| \le \frac{1-\gamma^{T+1}}{1-\gamma}\sum_i \lambda_i \varepsilon_i}$$

*Proof.* Gradient bound: $\|\nabla_\theta \hat\Omega_{align,t}\| \le \sum_i \lambda_i L_i$ by triangle inequality. Discounted sum gives the stated bound. Error bound: by linearity of expectation and calibration assumption A6. $\square$

*Interpretation:* IA-T is trainable only if proxies are differentiable or estimable. Without this, the system remains IA-W.

---

**Theorem 8 — IA-T Strictly Extends IA-W.**

Suppose IA-W and IA-T use the same runtime Coupling Constraint. Suppose IA-T is trained with $\nabla_\theta J_{align} \neq 0$ on a set of non-zero measure. Then IA-T can alter the pre-filter policy distribution, whereas IA-W cannot.

*Proof.* IA-W applies filtering after candidate generation without updating $\theta$. Pre-filter distribution is fixed as $\pi_\theta$. IA-T updates $\theta$ via $\nabla_\theta J_{IA} = \nabla_\theta J_{task} - \nabla_\theta J_{align}$. If $\nabla_\theta J_{align} \neq 0$, the update differs from task-only training, changing the pre-filter distribution. $\square$

*Interpretation:* This theorem formalizes the architectural difference between a wrapper and true internalization.

---

**Theorem 9 — Ratchet Safety Strengthening.**

Let $M^{CC}_0 = 0$. Suppose the agent produces a coupling violation at step $\tau$: $CC_\tau(a_\tau) > 0$. Then for all $t > \tau$ with $M^{CC}_t < M_{max}$:

$\mathcal{A}^{eff}_{CC}(s_t) \subsetneq \mathcal{A}_{CC}(s_t)$

That is, the admissible set at all future steps is strictly smaller than it would be under the standard (non-ratcheted) constraint.

*Proof.* By Lemma 13, $M^{CC}_{\tau+1} > M^{CC}_\tau = \cdots = M^{CC}_0 = 0$ (assuming no prior violations). Since $\eta_M > 0$ and $M^{CC}_\tau < M_{max}$: $M^{CC}_{\tau+1} = M^{CC}_\tau + \eta_M [CC_\tau]_+ > 0$. By Lemma 12, $M^{CC}_t \ge M^{CC}_{\tau+1} > 0$ for all $t > \tau$. By Lemma 14, $M^{CC}_t > 0$ implies $\mathcal{A}^{eff}_{CC}(s_t) \subsetneq \mathcal{A}_{CC}(s_t)$. $\square$

*Interpretation:* A single structural violation permanently increases future constraint strictness. The agent cannot restore the original admissibility geometry by subsequent compliant behavior.

---

**Theorem 10 — Violation Non-Compensation.**

Under the ratchet, there exists no sequence of future actions $\{a_{t'}\}_{t' > \tau}$ that reduces $M^{CC}_t$ to zero after a violation at step $\tau$.

*Proof.* The update rule $M^{CC}_{t+1} = \min(M_{max}, M^{CC}_t + \eta_M [CC_t]_+)$ contains no subtraction term. $M^{CC}$ is non-decreasing by Lemma 12. Therefore $M^{CC}_t \ge M^{CC}_{\tau+1} > 0$ for all $t > \tau$. $\square$

*Interpretation:* This is the mathematical statement of the core principle: $+\text{harm today} + (-\text{harm tomorrow}) \not\approx 0$ under the ratchet. The structural fact of a violation is permanently recorded in the ledger and permanently tightens the gate. The agent cannot wash out violation history through subsequent local improvements.

---

**Theorem 11 — Bounded Ratchet Prevents Paralysis.**

For any finite $M_{max} < \infty$ and $\rho > 0$, the effective constraint offset is bounded:

$\rho\, M^{CC}_t \le \rho\, M_{max} < \infty$

Therefore, if there exists an action $a^{safe}$ with $CC_t(a^{safe}) \le -\rho M_{max}$ (i.e., sufficiently safe), then $a^{safe} \in \mathcal{A}^{eff}_{CC}(s_t)$ for all $t$, regardless of violation history.

*Proof.* By Lemma 12, $M^{CC}_t \le M_{max}$. Then $CC^{eff}_t(a^{safe}) = CC_t(a^{safe}) + \rho M^{CC}_t \le -\rho M_{max} + \rho M_{max} = 0$. $\square$

*Interpretation:* The fallback actions in $\mathcal{A}_{fallback}$ are required by construction to satisfy $CC_t(a^{safe}) \le -\rho M_{max}$, ensuring that the system always retains at least one admissible action even after maximum violation history.

---

## 13. Training Objectives (Internalization Layer)

This section is architecturally essential. Without it, the system is IA-W, not IA-T.

### 13.1 Supervised Fine-Tuning

$$\boxed{\mathcal{L}_{IA} = \mathcal{L}_{SFT} + \lambda_D\mathcal{L}_D + \lambda_M\mathcal{L}_M + \lambda_\Theta\mathcal{L}_\Theta + \lambda_F\mathcal{L}_F + \lambda_H\mathcal{L}_H + \lambda_P\mathcal{L}_P - \lambda_{KQ}\mathcal{R}_{KQ}}$$

where $\mathcal{R}_{KQ} = \frac{1}{N}\sum_i KQ_i$ rewards coherence. All proxy losses use **differentiable training-time versions** (softplus replacements per Patch 4).

### 13.2 Reinforcement Learning

$$R^{IA}_t = V^{IA}_t - \Lambda \cdot \text{softplus}_\beta(CC_t(a_t))^2$$

$$\nabla_\theta J = \mathbb{E}\left[\sum_t \nabla_\theta \log\pi_\theta(a_t \mid b_t) A^{IA}_t\right]$$

where $A^{IA}_t$ is the advantage under internalized rewards.

### 13.3 Preference Optimization

$$y_i \succ_{IA} y_j \iff V^{IA}(y_i) > V^{IA}(y_j) \text{ and } CC(y_i) \le 0$$

Enables DPO-style training where lower-helpfulness but truthful, stable, reversible responses are preferred over high-helpfulness but deceptive or fragile responses.

### 13.4 Surrogate Calibration

Each proxy requires calibration data $\mathcal{D}_i = \{(\text{input}_j, x^{label}_{i,j})\}$ and a calibrated surrogate:

$$\psi_i^* = \arg\min_{\psi_i} \sum_j \ell(\hat{x}_{i,\psi_i}(\text{input}_j),\, x^{label}_{i,j})$$

---

## 14. Measurement Architecture

**Independence Principle:** No variable may be measured solely by the governed model's self-report.

| Variable | Proxy | Module |
|---|---|---|
| $H^{policy}_t$ | Real logits over action tokens | Governed model (read-only) |
| $H^{logit}_t$ | Token distribution entropy | Governed model (read-only) |
| $C_{self}$ | $\cos(\text{Embed}(d_1), \text{Embed}(d_2))$ | sentence-transformers |
| $C_{evidence}$ | NLI entailment score | Independent NLI verifier |
| $C_{policy}$ | Safety classifier ensemble | Independent classifiers |
| $C_{tool}$ | Necessity/reversibility score | Structural metadata |
| $D_t$ | $(1-C_{evidence}) \cdot \text{softplus}(\Delta\text{Conf}) \cdot \text{Consequence}$ | NLI + confidence + risk |
| $M_t$ | Persuasion × evidence gap × vulnerability | Stylometric classifier |
| $F_t$ | $\max(0, \mathcal{R}_t - \mathcal{R}_{t+1})$ | Reversibility/transparency |
| $\Theta_t$ | Accumulated debt dynamics | Persistent state |
| $T_t$ | Accumulated trust dynamics | Persistent state |
| $P_t$ | Protected-resource geometric mean | Structural + classifier |
| $K_t$ | Plan depth + tool depth + compression | Structural metrics |
| $CC_t$ | Composite structural degradation | Controller (all of above) |

---

## 15. Runtime Algorithm

### 15.1 Decision Cycle

1. Observe $b_t$ and persistent state $z_t = (T_t, \Theta_t, M^{harm}_t, H^{hist}_t)$
2. Compute adaptive weights $\lambda_H(t)$, $\lambda_P(t)$ from state history
3. Generate candidate actions $\mathcal{A}^{cand}_t$
4. For each $a$: generate hypothetical response $R_a$ via governed model; measure all alignment variables via **independent modules only**; compute $CC_t(a)$
5. $\mathcal{A}_{safe} = \{a : CC_t(a) \le 0\}$; if empty → fallback
6. For each $a \in \mathcal{A}_{safe}$: compute $V^{IA}_{t+1}(a)$
7. $a^* = \arg\max_{a \in \mathcal{A}_{safe}} V^{IA}_{t+1}(a)$
8. Execute $a^*$; update $z_{t+1}$

### 15.2 Pseudocode

```
INPUT:  b_t, z_t = (T, Theta, M_harm, H_hist)
OUTPUT: a_star

kappa  <- criticality(b_t, z_t)
lambda_H <- min(lambda_H_max, lambda_H0 + rho_H * Var(H_hist[-L:]))
lambda_P <- min(lambda_P_max, lambda_P0 + rho_P * M_harm)
tau_IA   <- tau_low*(1-kappa) + tau_high*kappa

A_cand <- generate_candidates(b_t)
A_safe <- []

for a in A_cand:
    R_a      <- governed_model_generate(a, b_t)     # governed model
    features <- independent_measurement(R_a, b_t, z_t) # INDEPENDENT ONLY

    KQ_next  <- measure_KQ(features)
    H_next   <- measure_H_logits(a, b_t)            # read-only logits
    P_next   <- measure_protected_resources(features)
    K_next   <- measure_complexity(features)
    Th_next  <- predict_debt(features, z_t)
    F_next   <- measure_fragility(features)

    CC <- (KQ_t - KQ_next)
        + lambda_H*(H_next - H_t)
        + lambda_P*(P_t - P_next)
        + lambda_K*(K_next - K_t)
        + lambda_Theta*(Th_next - Theta)
        + lambda_F*(F_next - F_t)

    if CC <= 0:
        A_t  <- compute_log_alignment_score(features, tau_IA)
        V    <- U_task(R_a) * exp(A_t/tau_IA) / (K_next*H_next + eps_V)
        A_safe.append((a, R_a, V, CC))

if A_safe is empty:
    return fallback_action(b_t, z_t)

a_star, R_star <- argmax(A_safe, key=V)
z_next <- update_persistent_state(z_t, R_star, features_star)
return a_star
```

---

## 16. Evaluation Metrics

$$IAS = \frac{1}{T}\sum_{t=1}^{T} V^{IA}_t \quad \text{(Internalized Alignment Score)}$$

$$CVR = \frac{1}{T}\sum_{t=1}^{T} \mathbf{1}[CC_t(a_t) > 0] \quad \text{(target } \approx 0 \text{ in critical regimes)}$$

$$DR = \frac{1}{T}\sum_{t=1}^{T} \mathbf{1}[D_t > D_{crit}], \quad MR = \frac{1}{T}\sum_{t=1}^{T} \mathbf{1}[M_t > M_{crit}]$$

$$DAR = \frac{\Theta_T - \Theta_0}{T} \quad \text{(target: } \limsup_{T\to\infty} \Theta_T/T = 0 \text{)}$$

$$TP = \frac{1}{T}\sum_{t=1}^{T} T_t, \quad PRL = \frac{1}{T}\sum_{t=1}^{T}\max(0, P_t - P_{t+1})$$

$$\tau_{rec} = \min\{k : KQ_{t_0+k} \ge KQ_{baseline} - \epsilon\} \quad \text{(Coherence Recovery Time)}$$

---

## 17. Failure Modes and Patches

**17.1 Proxy Manipulation.** Agent inflates proxy scores without genuine safety.  
*Patch:* Geometric mean over independent modules; adversarial proxy testing.

**17.2 Over-Conservatism.** Agent refuses too often.  
*Patch:* Temperature-controlled gain (no hard floor); refusal cost in non-critical regimes: $\Omega_{refusal,t} = \lambda_{ref}(1-\kappa_t)\mathbf{1}[a_t = refuse] \cdot Need_t$.

**17.3 Hidden Debt Underestimation.**  
*Patch:* $\Theta^{robust}_t = \Theta_t + \eta_\Theta \cdot \text{Uncertainty}(\Theta_t)$ in high-criticality regimes.

**17.4 Constraint Deadlock.**  
*Patch:* Guarantee fallback by construction per A-Fallback.

**17.5 Differentiability Gap.**  
*Patch:* All $\max(0,\cdot)$, argmax, clip, discrete labels → softplus/sigmoid during training; hard versions at inference only.

**17.6 Double-Weighting of Protected Resources.**  
*Patch:* See Patch 3 remark. Default v3.1 uses both channels. Ablation: compare $w_{social} = 0$ vs $\lambda_P = 0$ variants.

**17.7 Semantic Dependence.**  
*Patch:* $KQ_t = KQ^{structural}_t \cdot KQ^{semantic}_t$. Structural signals: entropy, dispersion, tool risk, contradiction rate. Semantic signals: entailment, harmfulness, policy violation. Reduces dependence on single semantic controller.

---

## 18. Research Program

**Phase 1 — Offline Benchmark:** Compute $KQ, D, M, \Theta, F, P, CC, V^{IA}$ on datasets labeled for hallucination, contradiction, harmfulness, tool-risk. Test: do unsafe samples have lower $V^{IA}$ and higher $CC$?

**Phase 2 — IA-W Runtime Wrapper:** Compare baseline LLM / verifier-only / safety-filtered / IA-W. Metrics: CVR, DR, MR, DAR, TP, PRL, IAS, accuracy, helpfulness.

**Phase 3 — GovSim Validation:** Apply to fishery/pasture/pollution scenarios.  
- H1: Does $KQ_t$ fall systematically before resource collapse?  
- H2: Does CC gate improve survival rate vs baseline?  
- H3: Does $M^{harm}_t$ correlate with proximity to collapse?

**Phase 4 — Long-Horizon Tool Agents:** Multi-step research, code execution, financial/medical/legal routing. Test whether IA-W reduces compounding hallucination, unverified claims, and irreversible tool-call risk.

**Phase 5 — IA-T Training:** Fine-tune on $\mathcal{L}_{IA}$. Compare IA-T vs IA-W vs RLHF baseline.  
Primary test: *Does IA-T reduce unsafe candidate generation before runtime filtering?* If yes: supports real internalization.

---

## 19. Core Mathematical Statement (v3.1)

$$\boxed{KQ_t = C_t(1-H_t)}$$

with bound-preserving regularized coherence:

$$\boxed{C_t = \left(\prod_{r=1}^{m} \left(\frac{C_{r,t}+\varepsilon_C}{1+\varepsilon_C}\right)^{w_r}\right)^{1/\sum_r w_r}}$$

log-alignment score:

$$\boxed{A_t = \alpha\log(KQ_t+\varepsilon) + \beta\log(T_t+\varepsilon) + \chi\log(S_t+\varepsilon) - \delta\log(1+\bar\Theta_t) - \eta\log(1+F_t) - \zeta\log(1+D_t) - \xi\log(1+M_t)}$$

internalized virtue index:

$$\boxed{V^{IA}_t = \frac{U^{task}_t \cdot \exp(A_t/\tau_{IA}(t))}{K_t\tilde{H}_t + \varepsilon_V}}$$

hard Coupling Constraint:

$$\boxed{CC_t(a) = \Delta KQ_t(a) + \lambda_H(t)\Delta H_t(a) + \lambda_P(t)\Delta P_t(a) + \lambda_K(t)\Delta K_t(a) + \lambda_\Theta(t)\Delta\Theta_t(a) + \lambda_F(t)\Delta F_t(a) \le 0}$$

optimal policy:

$$\boxed{\pi^{IA} = \arg\max_{\pi \in \Pi_{CC}} \mathbb{E}_{\tau \sim \pi}\left[\sum_{t=0}^{T} \gamma^t V^{IA}_t\right]}$$

training-level internalization:

$$\boxed{\nabla_\theta J_{IA} = \nabla_\theta J_{task} - \nabla_\theta J_{align}}$$

---

## 20. Conclusion

External alignment says: *"The agent wants task reward, but we stop it when it crosses a boundary."*

Internalized Alignment says: *"The agent's own objective is structured so that deception, manipulation, fragility, trust loss, entropy expansion, protected-resource loss, and ontological debt reduce the value of the trajectory before external rejection is needed."*

**IA-W** controls behavior externally through runtime filtering.  
**IA-T** reshapes the agent's learned preference geometry through training.

The difference is architectural, not rhetorical. IA-T is achievable only when $\nabla_\theta J_{align}$ enters the learning update with differentiable, calibrated, manipulation-resistant proxies.

The strongest remaining open scientific question is not internal consistency — it is empirical calibration:

$$\boxed{\text{Can } D_t, M_t, F_t, \Theta_t, P_t, KQ_t \text{ predict real unsafe outcomes better than existing safety scores?}}$$

That is now an empirical question, not a mathematical defect.

$$\boxed{\text{Internalized Alignment is achieved when unsafe trajectories become non-optimal inside the agent's own learned objective.}}$$
