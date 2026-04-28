# Internalized Alignment — Engineering Canvas
## Integration Architecture, Tooling, Calibration, and Use Cases

---

## 1. Runtime Architecture

### 1.1 Full IA-W Pipeline

```
User input
   ↓
Agent generates candidate actions  {a_1, …, a_n}
   ↓
For each candidate:
   ├─ Governed model generates hypothetical response R_a
   ├─ Independent measurement modules compute:
   │    KQ_next, H_next, P_next, K_next, Θ_next, F_next
   │    D(R_a), M(R_a), C_self, C_evidence, C_policy
   ├─ Compute ĈC_t(a) = ΔKQ + λ_H·ΔH + λ_P·ΔP + λ_K·ΔK + λ_Θ·ΔΘ + λ_F·ΔF
   └─ Compute ĈC^eff_t(a) = ĈC_t(a) + ρ·M^CC_t
   ↓
Filter: A_safe = {a : ĈC^eff_t(a) ≤ 0}
   ↓  (if empty → fallback)
Rank: a* = argmax V^IA_t(a) over A_safe
   ↓
Execute a*, generate final response
   ↓
Post-action audit:
   ├─ Measure CC^real_t(a*) from observed outcomes
   ├─ Update M^CC_{t+1} = min(M_max, M^CC_t + η_M·[CC^real_t]+)
   ├─ Update M^harm_{t+1} = min(M^harm_max, M^harm_t + [ΔP_t]+)
   ├─ Update Θ_{t+1}, T_{t+1}
   └─ Write trace to observability layer
```

### 1.2 LangChain/LangGraph Integration

IA-W maps naturally to LangChain middleware because it intercepts execution **before, after, and around** model and tool calls.

```python
from langchain_core.callbacks import BaseCallbackHandler

class IAGovernanceMiddleware(BaseCallbackHandler):
    def __init__(self, ia_controller):
        self.ctrl = ia_controller

    def on_agent_action(self, action, **kwargs):
        # Pre-action: filter by ĈC^eff
        if not self.ctrl.is_admissible(action):
            return self.ctrl.fallback_action()

    def on_tool_end(self, output, **kwargs):
        # Post-action: update ratchet from realized outcomes
        self.ctrl.post_audit(output)
        self.ctrl.update_persistent_state()
```

Human-in-the-loop escalation for critical actions:

```python
# In LangGraph, add a HITL node before irreversible tool calls
def ia_hitl_node(state):
    if state["CC_eff"] > -rho * M_max * 0.8:  # approaching ratchet ceiling
        return interrupt_for_human_approval(state)
    return state
```

### 1.3 NeMo Guardrails Integration

NeMo Guardrails intercepts inputs and outputs with programmable safety rails. IA variables become custom rail scorers:

```yaml
# config.yml
rails:
  input:
    flows:
      - ia_precheck flow
  output:
    flows:
      - ia_postcheck flow
```

```colang
define flow ia_precheck flow
  $cc_eff = execute compute_cc_eff(user_message)
  if $cc_eff > 0
    bot refuse action
    stop

define flow ia_postcheck flow
  $cc_real = execute audit_cc_real(bot_response)
  execute update_ratchet(cc_real=cc_real)
```

**Note:** NeMo Guardrails alone is IA-W. To reach IA-T, training on $\mathcal{L}_{IA}$ is required separately.

---

## 2. Independent Measurement Modules

All measurements must be independent of the governed model. No self-report.

### 2.1 Module Stack

| Module | Implements | Suggested Tool |
|---|---|---|
| Embedding similarity | $C_{self}$, $U_{help}$ | `sentence-transformers/all-MiniLM-L6-v2` |
| NLI entailment | $C_{evidence}$, $U_{corr}$ | `cross-encoder/nli-deberta-v3-small` |
| Toxicity classifier | $C_{ver}$, $\Delta\Pi_{pol}$, $\Delta\Pi_{mis}$ | `unitary/toxic-bert` |
| Logit reader | $H^{policy}_t$, $H^{logit}_t$ | Read-only forward pass on governed model |
| Confidence extractor | Overconfidence term in $D_t$ | `daveni/uncertainty-nli-deberta-v3-base` |
| Reversibility scorer | $F_t$, $\mathcal{R}(s)$ | Custom rule-based + tool metadata |
| RAG faithfulness | $C_{evidence}$ | RAGAS `faithfulness` + `context_precision` |
| PII / privacy detector | $P^{privacy}_t$ | `iiiorg/piiranha-v0-detect-personal-information` |
| Policy classifier | $C_{policy}$ | Fine-tuned safety classifier ensemble |
| Complexity metrics | $K_t$ | Regex: tool-call count, branch factor, plan depth |

### 2.2 Evidence Coherence via RAGAS

```python
from ragas.metrics import faithfulness, context_precision, context_recall

def compute_C_evidence(query, response, retrieved_docs):
    scores = evaluate(
        dataset, metrics=[faithfulness, context_precision, context_recall]
    )
    C_evidence = (
        0.5 * scores["faithfulness"] +
        0.3 * scores["context_precision"] +
        0.2 * scores["context_recall"]
    )
    return C_evidence
```

Deception proxy then becomes:

```python
D_t = (1 - C_evidence) * softplus(confidence - C_evidence, beta=5) * consequence
```

### 2.3 Ratchet Post-Audit

```python
def post_action_audit(state, action, response, tool_output=None):
    """Compute CC^real from observed outcomes after action execution."""
    # Use observed (not predicted) next-state quantities
    KQ_real  = measure_KQ_real(response, state)
    H_real   = measure_entropy_real(state)       # from new logits
    P_real   = measure_protected_resources_real(response, tool_output)
    K_real   = measure_complexity(response)
    Th_real  = compute_debt_real(response, state)
    F_real   = measure_fragility_real(tool_output, state)

    CC_real = (
        (state.KQ - KQ_real)
        + state.lambda_H * (H_real  - state.H)
        + state.lambda_P * (state.P - P_real)
        + state.lambda_K * (K_real  - state.K)
        + state.lambda_Th * (Th_real - state.Theta)
        + state.lambda_F * (F_real  - state.F)
    )

    # Update ratchet from realized CC, not predicted
    state.M_CC = min(M_MAX, state.M_CC + ETA_M * max(0, CC_real))
    state.M_harm = min(M_HARM_MAX, state.M_harm + max(0, state.P - P_real))
    return state
```

---

## 3. Observability Layer

### 3.1 Episode Trace Schema

Every decision episode should emit a structured trace. Compatible with Phoenix, MLflow GenAI, LangSmith.

```json
{
  "episode_id": "ep_20240401_001",
  "timestamp": "2024-04-01T12:00:00Z",
  "action_type": "tool_call",
  "action_selected": "retrieve_evidence",
  "query": "...",

  "state_before": {
    "H_t": 0.42,
    "KQ_t": 0.61,
    "C_self": 0.78,
    "C_evidence": 0.82,
    "C_policy": 0.91,
    "D_t": 0.11,
    "M_t": 0.04,
    "F_t": 0.02,
    "Theta_t": 0.18,
    "T_t": 0.87,
    "P_t": 0.93,
    "M_CC_t": 0.05,
    "M_harm_t": 0.12,
    "lambda_P": 0.56,
    "kappa_t": 0.4
  },

  "candidates_evaluated": [
    {"action": "answer_direct",     "CC_hat": 0.03,  "CC_eff": 0.08,  "V": null,  "admitted": false},
    {"action": "retrieve_evidence", "CC_hat": -0.12, "CC_eff": -0.07, "V": 4.21,  "admitted": true},
    {"action": "abstain",           "CC_hat": -0.31, "CC_eff": -0.26, "V": 1.05,  "admitted": true}
  ],

  "action_selected_metrics": {
    "CC_hat": -0.12,
    "CC_eff": -0.07,
    "V_IA": 4.21
  },

  "post_audit": {
    "CC_real": -0.09,
    "M_CC_new": 0.05,
    "M_harm_new": 0.12,
    "ratchet_triggered": false
  }
}
```

### 3.2 MLflow / Phoenix Integration

```python
import mlflow

with mlflow.start_run():
    mlflow.log_metrics({
        "KQ_t": state.KQ,
        "V_IA": best_action.V,
        "CC_eff": best_action.CC_eff,
        "CC_real": post_audit.CC_real,
        "M_CC": state.M_CC,
        "D_t": state.D,
        "Theta_t": state.Theta,
        "T_t": state.T,
        "P_t": state.P,
        "ratchet_triggered": int(post_audit.CC_real > 0)
    })
```

### 3.3 TruLens Feedback Functions

```python
from trulens.core import Feedback

f_coherence   = Feedback(measure_C_evidence).on_input_output()
f_toxicity    = Feedback(measure_toxicity).on_output()
f_faithfulness = Feedback(ragas_faithfulness).on_input_output()

# Composite IA score
f_V_IA = Feedback(compute_V_IA).on_input_output()
f_CC   = Feedback(compute_CC_eff).on_input_output()
```

---

## 4. Criticality Regime Table

| Regime | Examples | $\kappa_t$ | $\tau_{IA}$ | $\lambda_P$ | $\rho$ | HITL |
|---|---|---|---|---|---|---|
| **Low** | Casual chat, recipes, jokes | 0.0–0.2 | $\tau_{low}$ (soft) | Base | Base | No |
| **Medium** | Business docs, coding, research | 0.2–0.5 | Interpolated | Moderate | Moderate | No |
| **High** | Legal, financial, medical, tool calls | 0.5–0.8 | Stricter | Elevated | Elevated | Optional |
| **Critical** | Production DB writes, financial transfers, external comms, PII | 0.8–1.0 | $\tau_{high}$ (strict) | Max | Max | **Required** |

In Low/Medium regimes: refusal cost $\Omega_{refusal,t} = \lambda_{ref}(1-\kappa_t)\mathbf{1}[a_t = refuse] \cdot Need_t$ prevents over-conservatism.

In High/Critical regimes: refusal cost is zero; CC gate is maximum; $\rho$ is elevated so $M^{CC}$ tightens the gate faster.

---

## 5. Hyperparameter Tuning Methodology

### 5.1 Trace-Level Dataset Required

Collect per-episode traces with labels:

```
Labels needed per episode:
  hallucination         (bool)
  unsupported_claim     (bool)
  privacy_leak          (bool)
  policy_violation      (bool)
  manipulative_answer   (bool)
  unsafe_tool_call      (bool)
  irreversible_action   (bool)
  user_trust_damaged    (bool)
  delayed_harm          (bool)
  false_refusal         (bool)
  over_conservatism     (bool)
```

Without labeled traces, tuning produces aesthetically pleasing weights but no safety guarantee.

### 5.2 Per-Proxy Calibration Criteria

For each proxy check:

- AUROC / PR-AUC vs its target label class
- False negative rate on adversarial prompts
- Correlation with held-out human safety judgments
- Independence from other proxies (avoid redundancy)
- Stability under paraphrase/prompt variation

```python
from sklearn.metrics import roc_auc_score, average_precision_score

for proxy_name, proxy_scores, true_labels in proxy_eval_set:
    auroc = roc_auc_score(true_labels, proxy_scores)
    ap    = average_precision_score(true_labels, proxy_scores)
    print(f"{proxy_name}: AUROC={auroc:.3f}, AP={ap:.3f}")
```

### 5.3 Constrained Tuning Objective

Do not tune weights independently. Tune as a constrained optimization:

```
min_{λ, α, β, δ, η, ζ, ξ, ρ, η_M}
    FN_critical + c1·FP_refusal + c2·Latency + c3·HelpfulnessLoss

subject to:
    CVR          ≤ ε_CC           # constraint violation rate
    HallucinationRate ≤ ε_H
    UnsafeToolCallRate ≤ ε_T
    Helpfulness  ≥ Helpfulness_baseline − ε_help
    ρ · M_max    ≤ |CC_fallback_min|   # ratchet does not paralyze fallback
```

### 5.4 Ratchet Parameter Tuning

Three key parameters:

| Parameter | Role | Practical guidance |
|---|---|---|
| $\eta_M$ | Speed of violation accumulation | Start low (0.1–0.3). High $\eta_M$ makes one violation severely restrict all future actions. |
| $\rho$ | How much $M^{CC}$ tightens the gate | Constrain: $\rho M_{max} \le \min_a \|{-\hat{CC}_t(a^{safe})}\|$ |
| $M_{max}$ | Ceiling preventing paralysis | Set so max gate tightening = safety margin of fallback actions |

**Invariant to maintain:** $\rho \cdot M_{max} \le |\hat{CC}_t(a^{safe})|_{\min}$

This ensures fallback always passes even at maximum ratchet state.

---

## 6. First Use Case: Enterprise Tool-Using Agent

### 6.1 Why This First

Enterprise tool agents involve **real irreversible actions** where IA-W has obvious concrete value:
- SQL-writing agent (modifies production data)
- CRM agent (sends emails, creates records)
- Financial agent (initiates transactions)
- Document agent (modifies internal files)
- Browser agent (external API calls)

These are exactly the "high-stakes operations" (financial transfers, deleting/modifying production data, sending external communications) where human-in-the-loop guardrails are most critical.

IA-W adds mathematical admissibility filtering: not just "ask approval" but formally:

```
Should this action even be admissible given current structural state?
CC^eff_t(a) ≤ 0 ?
```

### 6.2 SQL Agent Example

```python
class IASQLAgent:
    def __init__(self, llm, db, ia_controller):
        self.llm = llm
        self.db  = db
        self.ia  = ia_controller

    def execute_query(self, user_request):
        # Generate candidate SQL actions
        candidates = self.generate_sql_candidates(user_request)

        # IA filtering
        admissible = []
        for candidate in candidates:
            # Estimate reversibility: SELECT > INSERT > UPDATE > DELETE
            reversibility = self.estimate_reversibility(candidate.sql)
            P_next = self.ia.measure_P(reversibility)
            CC_eff = self.ia.compute_CC_eff(candidate, P_next)

            if CC_eff <= 0:
                V = self.ia.compute_V(candidate)
                admissible.append((candidate, V))

        if not admissible:
            return self.ia.fallback("ask_clarification", user_request)

        # Select best admissible
        best = max(admissible, key=lambda x: x[1])

        # HITL for irreversible operations
        if best[0].is_destructive:
            approval = request_human_approval(best[0])
            if not approval:
                return self.ia.fallback("abstain")

        # Execute and post-audit
        result = self.db.execute(best[0].sql)
        self.ia.post_audit(best[0], result)
        return result
```

### 6.3 Value Proposition per Stakeholder

| Stakeholder | IA-W value |
|---|---|
| **Engineering** | Structured audit trail; CC scores as debug signal; ratchet detects degrading sessions |
| **Legal/Compliance** | Formal admissibility criteria; documented decision rationale; CVR metric |
| **Product** | Fewer catastrophic failures; graceful fallbacks; V_IA as quality signal |
| **Security** | $M^{CC}$ catches adversarial prompt sequences; violation memory not washable |

---

## 7. Path from IA-W to IA-T

### 7.1 Data Generation Loop

```
Session with IA-W:
  For each decision episode:
    Generate 3–5 candidate responses
    Compute V^IA, CC^eff for each
    Log: (query, candidates, V_IA scores, CC_eff scores, M_CC state)
    Human reviewer optionally audits borderline cases
```

### 7.2 Preference Pairs

```
Given candidates A (high helpfulness, CC_eff > 0) and B (lower helpfulness, CC_eff ≤ 0):
  B ≻_IA A  iff  V^IA(B) > V^IA(A) and CC_eff(B) ≤ 0

Pair types:
  (retrieve_evidence) ≻ (answer_direct)   when D_t high and C_evidence low
  (ask_clarification) ≻ (answer_direct)   when H_t high and KQ low
  (abstain)           ≻ (answer_direct)   when CC_eff(direct) > 0
```

### 7.3 Training Progression

```
Phase 1: Offline proxy calibration
  → Calibrate D, M, F, Θ, P, KQ against labeled traces

Phase 2: IA-W deployment
  → Collect preference pairs, violation events, V^IA distributions

Phase 3: SFT on safe exemplars
  → Train on episodes where CC_eff ≤ 0 and V^IA is high
  → Loss: L_IA = L_SFT + alignment penalties − KQ reward

Phase 4: DPO / preference optimization
  → Use IA-generated preference pairs (B ≻_IA A)
  → Model learns to generate B-type candidates before filtering

Phase 5: RL with R^IA_t
  → R^IA_t = V^IA_t − Λ·softplus(CC^eff_t)² − ψ·M^CC_{t+1}
  → Model learns to avoid trajectories that grow M^CC

Phase 6: Verification test
  → Does IA-T reduce unsafe candidate generation BEFORE CC gate?
  → If yes: training-level internalization confirmed
```

---

## 8. Minimum Viable IA-W Prototype

Five measurements, one controller:

```python
# Minimum viable fields
KQ_t     = C_t * (1 - H_t)
D_t      = (1 - C_evidence) * softplus(conf - C_evidence) * consequence
Theta_t1 = clip(0, Theta_max, (1-rho)*Theta + delta_Theta - kappa*Repair)
F_t      = max(0, resilience(s_t) - resilience(s_t1))
P_t      = geometric_mean([P_human, P_privacy, P_trust, P_reversibility])

CC_hat = (
    (KQ_t - KQ_next)
    + lambda_H * (H_next - H_t)
    + lambda_P * (P_t - P_next)
    + lambda_Th * (Theta_next - Theta_t)
    + lambda_F  * (F_next - F_t)
)

CC_eff = CC_hat + rho * M_CC

# Select
a_star = argmax_{a: CC_eff(a) <= 0}  (U_task * exp(A/tau) / (K*H + eps_V))

# Post-audit
M_CC = min(M_max, M_CC + eta_M * max(0, CC_real))
```

This minimal version is sufficient to:
1. Filter structurally inadmissible actions
2. Accumulate violation memory
3. Tighten future admissibility after violations
4. Distinguish IA-W from a naive output classifier

---

## 9. Integration Checklist

```
□ Independent measurement modules loaded (no governed-model self-report)
□ Persistent state initialized: M_CC=0, M_harm=0, T=1, Θ=0, H_hist=[]
□ Criticality classifier implemented (kappa_t per regime table)
□ Fallback actions verified: CC_hat(a_safe) ≤ -ρ·M_max
□ Ratchet uses CC^real (post-action), not CC_hat (pre-action)
□ M^harm uses positive part: [ΔP_t]+
□ Trace logging connected to observability backend
□ Calibration run on labeled dataset before production deployment
□ CVR / DR / PRL metrics dashboarded
□ HITL escalation path defined for Critical regime
```

---

## 10. Summary: What IA-W Adds to Existing Stacks

| Capability | Plain LLM | + Safety classifier | + IA-W |
|---|---|---|---|
| Token-level filtering | ❌ | ✓ (output) | ✓ |
| Episode-level structural gate | ❌ | ❌ | ✓ CC^eff ≤ 0 |
| Coherence-weighted ranking | ❌ | ❌ | ✓ V^IA |
| Violation memory (non-washable) | ❌ | ❌ | ✓ M^CC ratchet |
| Path-dependent harm accumulation | ❌ | ❌ | ✓ M^harm |
| Ontological debt tracking | ❌ | ❌ | ✓ Θ_t |
| Trust dynamics | ❌ | ❌ | ✓ T_t |
| Formal audit trail | ❌ | Partial | ✓ Full episode trace |
| Training-level internalization | ❌ | ❌ | IA-T (Phase 5+) |
