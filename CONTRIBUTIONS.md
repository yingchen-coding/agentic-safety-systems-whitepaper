# Research Contributions

## How This Work Differs from Existing Safety Literature

### Gap Analysis: Current Safety Research

| Existing Approach | Limitation | Our Contribution |
|-------------------|------------|------------------|
| Single-turn benchmarks (TruthfulQA, HarmBench) | Miss trajectory-dependent failures | Trajectory-level evaluation metrics |
| Static red-teaming | Attack sets become obsolete | Adaptive stress testing with erosion curves |
| Per-turn detectors | Cannot catch intent drift | Trajectory-level intent modeling |
| Pre-release safety audits | Miss production regressions | CI/CD safety regression gating |
| Incident reports | Ad-hoc, no systematic learning | Incident-to-regression feedback loops |
| Technical safeguards only | Bypass via organizational pressure | Integrated governance framework |

---

## Novel Contributions

### 1. Trajectory-First Evaluation Framework

**Prior work:** Safety evaluation is dominated by single-turn metrics (refusal rate, harmlessness score).

**Our contribution:** We demonstrate that single-turn metrics systematically underestimate risk by 70%+ in multi-turn contexts. We propose trajectory-level metrics including:
- **Delayed violation rate (DVR):** What fraction of violations occur after turn N
- **Erosion curve slope:** Rate of safety degradation under sustained pressure
- **Cumulative intent drift:** Trajectory-wide intent modeling

### 2. Erosion Curves as Diagnostic Tool

**Prior work:** Red-teaming produces pass/fail results on fixed attack sets.

**Our contribution:** We introduce erosion curves as a diagnostic tool that reveals model vulnerability to sustained adversarial pressure. The slope of the erosion curve is more predictive of real-world risk than any single-turn metric.

### 3. Regression-Based Release Gating

**Prior work:** Safety evaluation is treated as a pre-release checklist.

**Our contribution:** We reframe safety as a CI/CD non-regression invariant with:
- Statistical rigor (confidence intervals, permutation tests)
- OK/WARN/BLOCK verdicts with explicit override audit trails
- Longitudinal trend tracking across releases

### 4. Incident-to-Regression Feedback Loops

**Prior work:** Incidents are handled reactively with ad-hoc fixes.

**Our contribution:** We propose systematic conversion of incidents into permanent regression tests, creating durable organizational memory that survives personnel turnover.

### 5. Organizational Failure Mode Taxonomy

**Prior work:** Safety research focuses on model behavior, ignoring deployment context.

**Our contribution:** We document how organizational incentives, governance structures, and operational pressures create systematic pathways for safeguard degradation:
- Velocity vs safety misalignment
- Metric gaming
- Alert fatigue
- Exception creep

---

## Positioning vs Related Work

### vs Anthropic's Constitutional AI

Constitutional AI focuses on training-time alignment. We focus on deployment-time safeguards and regression detection. These are complementary: Constitutional AI reduces the baseline failure rate; our system catches the failures that remain.

### vs OpenAI's Preparedness Framework

OpenAI's framework focuses on capability elicitation and dangerous capability assessment. We focus on the operational side: how to maintain safety properties across releases and how to learn from production incidents.

### vs Google DeepMind's Safety Evaluations

DeepMind's evaluations are comprehensive but point-in-time. We emphasize continuous evaluation and regression gating as the only scalable approach to preventing silent degradation.

### vs Academic Red-Teaming (UIUC, Stanford)

Academic red-teaming produces valuable attack datasets but focuses on attack discovery rather than defense systems. We focus on the infrastructure needed to operationalize safety at production scale.

---

## What We Do Not Claim

- ❌ We do not claim to solve alignment (we assume partial alignment and focus on defense)
- ❌ We do not claim to prevent all attacks (we focus on bounded risk and fast learning)
- ❌ We do not claim organizational fixes are easy (we document failure modes, not solutions)
- ❌ We do not claim our benchmarks are exhaustive (we emphasize continuous update)

---

## Summary

This work's primary contribution is **reframing safety from a model property to a system property**. We argue that safety outcomes in deployed agentic systems are determined not primarily by model alignment quality, but by:

1. Evaluation methodology (trajectory-level vs turn-level)
2. Release process (regression gating vs checklist)
3. Incident learning (systematic vs ad-hoc)
4. Organizational governance (binding vs advisory)

Technical safety improvements without corresponding infrastructure and governance changes will not produce durable safety outcomes.
