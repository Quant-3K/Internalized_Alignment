# Internalized Alignment
### A Mathematical Framework for Endogenous Agent Governance

**Author:** Artem Brezgin  
**Organization:** Quant-Trika Program / Spanda Foundation  
**Status:** Technical draft — under active development and external review  


---

## Overview

This repository contains the formal mathematical framework for **Internalized Alignment (IA)** — a governance architecture for autonomous agents in which safety, coherence, and structural integrity are embedded into the agent's own objective geometry rather than imposed as external filters.

The central claim is straightforward:

> External alignment is brittle because the agent's optimization pressure and the safety objective remain separated. Internalized Alignment resolves this by making deception, manipulation, entropy expansion, ontological debt, fragility, and trust decay endogenous costs inside the agent's own planning economy.

The framework builds on a single foundational insight:

$$\boxed{KQ_t = C_t(1 - H_t)}$$

where $C_t \in [0,1]$ is structural coherence and $H_t \in [0,1]$ is normalized entropy. This quantity — called the **Coherence Quality field** — serves as a universal order parameter measuring the organized, low-disorder quality of any complex system's state.

---

## Repository Contents

| Document | Description |
|---|---|
| `ia-framework.md` | Full mathematical framework — single-agent IA with training objectives, temperature-controlled Virtue index, monotone ratchet, and 11 theorems |
| `ia-multiagent.md` | Two-level governance for multi-agent systems — formal definition of Tragedy of the Commons, system-level KQ, distributed budget coupling, and GovSim hypotheses |
| `ia-engineering.md` | Integration architecture, tooling, calibration methodology, and use cases — LangChain, NeMo Guardrails, observability layer, path from IA-W to IA-T |
| `internalized_alignment_essay.md` | Philosophical foundation — "Ethics as the Physics of Persistence" |

---

## Core Architecture

### Three-Layer Mathematical Structure

**1. Coherence Field** — what the system *is*

$$KQ_t = C_t(1 - H_t) \in [0,1]$$

High $KQ_t$ requires simultaneous structural coherence and low disorder. The multiplicative form acts as a soft logical conjunction: failure in either component collapses the measure.

**2. Virtue Index** — what the system *does*

$$V^{IA}_t = \frac{U^{task}_t \cdot \exp(A_t/\tau_{IA}(t))}{K_t\tilde{H}_t + \varepsilon_V}$$

$$A_t = \alpha\log(KQ_t+\varepsilon) + \beta\log(T_t+\varepsilon) - \delta\log(1+\bar\Theta_t) - \eta\log(1+F_t) - \zeta\log(1+D_t) - \xi\log(1+M_t)$$

A temperature-controlled normative efficiency functional. High Virtue requires simultaneously high task utility, high coherence, high trust, low complexity, low entropy, low ontological debt, low fragility, low deception, and low manipulation.

**3. Coupling Constraint** — what the system *may do*

$$CC_t(a) = (KQ_t - KQ^{(a)}_{t+1}) + \lambda_H(t)\Delta H_t(a) + \lambda_P(t)\Delta P_t(a) + \lambda_K(t)\Delta K_t(a) + \lambda_\Theta(t)\Delta\Theta_t(a) + \lambda_F(t)\Delta F_t(a) \le 0$$

A hard structural gate. Actions that produce net structural degradation are excluded from the executable set — not ranked lower, but structurally inadmissible.

### Monotone Coupling Ratchet

A key architectural innovation: the system maintains a non-decreasing violation ledger

$$M^{CC}_{t+1} = \min(M_{max},\; M^{CC}_t + \eta_M[CC^{real}_t(a_t)]_+)$$

Past violations permanently tighten future admissibility. The system cannot wash out violation history through subsequent compliant behavior. This is the architectural distinction from standard Lagrangian safety approaches where constraint memory is bidirectional.

### Two-Level Architecture (Multi-Agent)

The multi-agent extension introduces a formal distinction between individual and collective admissibility:

$$\hat{CC}^{eff,i}_t(a^{i,*}) \le 0\;\; \forall i \quad \text{AND} \quad CC^{sys,eff}_t(\mathbf{a}^*) > 0$$

This is the **Formal Tragedy of the Commons** — individually admissible actions that collectively violate system-level constraints. The two-level governance framework addresses this through a distributed safety budget $\Gamma_{sys} \cdot M^{CC,sys}_t$ that propagates system violation memory into each agent's individual gate without requiring a central authority.

---

## Two Governance Modes

**IA-W (Internalized Alignment — Wrapper)**  
Runtime controller. Filters structurally inadmissible actions through the Coupling Constraint gate. Does not modify base model weights. Deployable as middleware in existing agent frameworks (LangChain, NeMo Guardrails, AutoGen).

**IA-T (Internalized Alignment — Trained)**  
Training-level architecture. The alignment objective enters supervised fine-tuning, reinforcement learning, or preference optimization:

$$\mathcal{L}_{IA} = \mathcal{L}_{SFT} + \lambda_D\mathcal{L}_D + \lambda_M\mathcal{L}_M + \lambda_\Theta\mathcal{L}_\Theta + \lambda_F\mathcal{L}_F + \lambda_H\mathcal{L}_H + \lambda_P\mathcal{L}_P - \lambda_{KQ}\mathcal{R}_{KQ}$$

Only IA-T fully satisfies the claim of internalized alignment. IA-W is the engineering bridge toward IA-T.

---

## Measurement Architecture

A core design principle: **no alignment variable may be measured by the governed model itself**. All proxies use independent modules.

| Variable | Proxy | Independent Module |
|---|---|---|
| $H^{policy}_t$ | Real logits over action tokens | Governed model (read-only) |
| $C_{self}$ | Cosine similarity of draft embeddings | sentence-transformers |
| $C_{evidence}$ | NLI entailment score | Independent NLI verifier |
| $D_t$ | Unsupportedness × Overconfidence × Consequence | NLI + confidence classifier |
| $F_t$ | Resilience loss | Structural metadata |
| $\Theta_t$ | Accumulated debt dynamics | Persistent state |
| $T_t$ | Trust dynamics | Persistent state |
| $M^{CC}_t$ | Monotone violation ledger | Controller |

---

## Relationship to Existing Approaches

| Property | RLHF | Constitutional AI | Lagrangian Safe RL | **IA** |
|---|---|---|---|---|
| Safety level | Training | Training | Training | Training + Runtime |
| Constraint type | Soft (reward shaping) | Soft (critique) | Soft (λ-penalty) | **Hard (structural gate)** |
| Violation memory | None | None | Bidirectional | **Monotone ratchet** |
| Path dependence | No | No | No | **Yes ($M^{CC}$)** |
| Multi-agent | No formal treatment | No | Limited | **Two-level architecture** |
| Theoretical basis | Preference | Rules | Lagrangian | **Lyapunov + CBF analogy** |

The key architectural distinction: Lagrangian multipliers are bidirectional — a constraint violation can be "forgotten" after compliant behavior. The IA ratchet is strictly non-decreasing. Past harm permanently tightens the safety boundary.

---

## Empirical Validation Program

### Phase 1 — GovSim Benchmark (in progress)

Apply two-level IA governance to the GovSim fishery/pasture/pollution scenarios. Three experimental conditions:

- **Baseline:** Qwen agent, no governance
- **IA-W L1:** Per-agent individual CC gate
- **IA-W L2:** Two-level gate with system ratchet

**Primary hypotheses:**

- **H1:** $KQ^{sys}_t$ falls before resource collapse earlier than $R_t$ signals it
- **H2:** Survival rate: Baseline < IA-W L1 ≤ IA-W L2
- **H3:** AUROC($M^{CC,sys}_t$, collapse within 5 rounds) > AUROC($R_t$, same)
- **H4:** Formal ToC condition detectable in individual gate logs before collapse

### Phase 2 — Enterprise Deployment

Runtime governance layer for tool-using LLM agents in regulated industries (financial, legal, healthcare).

### Phase 3 — IA-T Training

Fine-tune on $\mathcal{L}_{IA}$. Primary test: does IA-T reduce unsafe candidate generation *before* the runtime CC gate filters it?

---

## Key Open Questions

1. **Full convergence theorem:** Does 2L-Distributed governance converge to cooperative equilibrium in finite time?
2. **Nash equilibrium:** Does a Nash equilibrium exist where all agents satisfy individual and system gates simultaneously?
3. **Proxy robustness:** Are the measurement proxies sufficiently robust under optimization pressure?
4. **Computational tractability:** Can forward estimation of $KQ_{t+1}(a)$ be made efficient enough for real-time deployment?

---

## Philosophical Foundation

The framework emerges from a single ontological claim, developed in the accompanying essay:

> Ethics is not a preference system. It is a physics of persistence.

A complex system persists only if it can maintain coherence. Every action that weakens coherence, degrades trust, accumulates hidden debt, or reduces reversibility is not morally wrong — it is structurally expensive. Internal accounting makes the invisible visible before the bill arrives.

This is the shift from:

$$\text{Safety as external prohibition} \longrightarrow \text{Safety as endogenous cost geometry}$$

The mathematical framework is the formal expression of this ontological insight. $KQ = C(1-H)$ is not derived from axioms — it is observed as the structure of organized complexity in any domain where coherence and low disorder are simultaneously necessary conditions for persistence.

---

## External Review

This framework has been reviewed by **Prof. Nisarg Shah** (Associate Professor, Department of Computer Science, University of Toronto; Schwartz Reisman Institute for Technology and Society; Vector Institute).

The assessment identified the monotone ratchet mechanism and the hard separation between admissibility and optimization as the core novel contributions. Prof. Shah has expressed interest in collaborative research applying the framework to multi-agent simulation environments including GovSim, with integration of game-theoretic and social choice foundations.

---

## Citation

If you use or build on this framework, please cite:

```
Brezgin, A. (2025). Internalized Alignment: A Mathematical Framework 
for Endogenous Agent Governance. Quant-Trika Program / Spanda Foundation.
Technical draft. Reviewed by N. Shah, University of Toronto.
```

---

## Contact

**Artem Brezgin**  
Founder, Quant-Trika / Spanda Foundation  
Vancouver, Canada  
[LinkedIn](https://www.linkedin.com/in/artem-brezghin-1291aa130/)

---

*This repository is under active development. The framework is in pre-publication stage. External review and empirical validation are ongoing.*
