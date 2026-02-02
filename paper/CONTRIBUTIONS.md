# Research Contributions

This work contributes a lifecycle-oriented framework for evaluating and governing safety in agentic systems. Unlike prior safety research that focuses on model-centric, single-turn benchmarks, our contributions operate at the system level and emphasize longitudinal, trajectory-level risk that emerges only in deployment.

---

## 1. Reframing Safety as a Non-Regression Property of Systems

**Prior work:**
Most safety benchmarks evaluate static model snapshots using one-off red-teaming or single-turn prompts, implicitly assuming safety is a model property.

**Our contribution:**
We formalize safety as a *non-regression invariant* over time and releases. Safety is evaluated longitudinally across versions and enforced via regression gating (OK / WARN / BLOCK) with statistical significance. This reframes safety from a pre-release checklist into a continuous release constraint.

**Why this matters:**
This directly addresses silent safety regressions introduced by capability upgrades, prompt changes, or infrastructure drift—failure modes largely unmeasured by existing benchmarks.

---

## 2. Trajectory-Level Failure Modes as First-Class Evaluation Targets

**Prior work:**
Most benchmarks measure per-turn refusal accuracy or classifier performance, which underestimates risk in agentic settings.

**Our contribution:**
We define and operationalize *trajectory-level safety metrics*, including delayed violation rate, policy erosion curves, and intent drift over multi-turn interactions. These metrics surface failures that are invisible to single-turn detectors.

**Why this matters:**
This provides empirical grounding for why agentic systems fail silently even when per-turn safety metrics appear healthy.

---

## 3. Integrating Red-Teaming with Release Gating

**Prior work:**
Red-teaming is typically episodic and decoupled from release decisions.

**Our contribution:**
We integrate automated multi-turn red-teaming directly into a regression suite that gates releases. Red-team failures become binding signals in CI/CD, rather than advisory reports.

**Why this matters:**
This shifts red-teaming from post-hoc analysis to a release-blocking control surface.

---

## 4. Closing the Loop: From Incidents to Permanent Regression Tests

**Prior work:**
Postmortems often remain documentation artifacts and do not feed back into systematic evaluation.

**Our contribution:**
We introduce an incident → replay → root cause → regression test pipeline, ensuring that real-world failures permanently expand evaluation coverage. Incidents become executable tests, not just narratives.

**Why this matters:**
This creates institutional memory for safety failures and prevents repeated regressions of known issues.

---

## 5. Production-Grade Safety Evaluation Infrastructure

**Prior work:**
Safety research is frequently demonstrated in notebooks or offline benchmarks.

**Our contribution:**
We design and implement a scalable evaluation pipeline with batch + streaming evaluation, drift detection, traffic replay, and cost-aware monitoring. Safety evaluation is treated as production infrastructure, not research tooling.

**Why this matters:**
This bridges the gap between safety research and real-world deployment constraints, where cost, latency, and operational reliability determine what safety coverage is sustainable.

---

## 6. Organizational Failure Modes as a Safety Risk Factor

**Prior work:**
Most safety research treats deployment context as exogenous.

**Our contribution:**
We model organizational incentives, metric gaming, and governance gaps as first-class contributors to safety erosion. We propose governance mechanisms (threshold change control, audit trails, human override policy) to mitigate these failure modes.

**Why this matters:**
This acknowledges that many safety failures are socio-technical, not purely model-level.
