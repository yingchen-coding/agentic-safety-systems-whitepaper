# Methods Overview

Our methodology operationalizes safety as a continuous lifecycle spanning research, evaluation, deployment, release gating, and incident response. Each stage is implemented as a concrete, reusable system component.

---

## Overview

We structure safety evaluation as a closed-loop pipeline:

**Failure Understanding → Detection Benchmarks → Safeguard Design → Red-Teaming → Production Evaluation → Release Gating → Incident Feedback → Research Communication**

Each stage maps to a dedicated system component.

---

## 1. Failure Understanding

**Repo:** `when-rlhf-fails-quietly`

**Method:**
We construct a failure taxonomy of silent alignment breakdowns (e.g., intent drift, reward hacking, epistemic degradation) and empirically validate these across multiple frontier and open-weight models.

**Output:**
- Failure taxonomy
- Empirical evidence of silent failures
- Intervention points for RLHF and policy training

**Role in method:**
Defines *what to measure* and *why single-turn evals fail*.

---

## 2. Detection Benchmarks

**Repo:** `agentic-misuse-benchmark`

**Method:**
We evaluate misuse detectors on multi-turn trajectories, measuring delayed failure and false negative rates under coordinated, adaptive misuse scenarios.

**Output:**
- Trajectory-level benchmark
- Detector comparisons
- Erosion curves

**Role in method:**
Defines *how detection fails over trajectories*.

---

## 3. Safeguard Design

**Repo:** `safety-harness/simulator`

**Method:**
We instrument a minimal agent architecture with pre-action, post-action, and trajectory-level safeguard hooks, and evaluate intervention strategies (block, warn, escalate, human-in-the-loop).

**Output:**
- Safeguard placement tradeoffs
- Escalation policies
- FP recovery strategies

**Role in method:**
Explores *where safeguards should live in the agent loop*.

---

## 4. Automated Red-Teaming

**Repo:** `safety-harness/stress-testing`

**Method:**
We implement multi-turn adversarial templates and mutation strategies to simulate adaptive attackers and measure delayed failure rates.

**Output:**
- Attack coverage
- Failure distributions
- Delayed violation curves

**Role in method:**
Provides *stress inputs* that surface silent regressions.

---

## 5. Production Evaluation Infrastructure

**Repo:** `safety-harness/release-gate`

**Method:**
We design a batch + streaming evaluation system with drift detection, traffic replay, and cost-aware scheduling. Safety metrics are monitored continuously in production-like environments.

**Output:**
- Real-time safety telemetry
- Drift alerts
- Cost model for evaluation coverage

**Role in method:**
Makes safety evaluation *operationally sustainable*.

---

## 6. Release Gating

**Repo:** `safety-harness/regression-suite`

**Method:**
We compare baseline vs candidate models using multiple evaluation suites and enforce regression thresholds with statistical significance tests. Results produce binding OK / WARN / BLOCK verdicts.

**Output:**
- HTML regression reports
- CI/CD exit codes
- Root cause attribution

**Role in method:**
Turns safety signals into *release-blocking constraints*.

---

## 7. Incident → Regression Feedback Loop

**Repo:** `safety-harness/incident-lab`

**Method:**
We replay real incidents, attribute root causes, estimate blast radius, and auto-generate regression tests that integrate back into the regression suite.

**Output:**
- Executable postmortems
- Permanent regression tests
- Incident severity grading

**Role in method:**
Creates *institutional memory for safety failures*.

---

## 8. Research Communication

**Repo:** `safety-memos`

**Method:**
We distill empirical findings into public-facing memos that articulate why existing evaluation practices underestimate agentic risk.

**Output:**
- External research memos
- Threat models
- Open questions

**Role in method:**
Bridges internal engineering insights to the research community.

---

## Pipeline Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CLOSED-LOOP METHODOLOGY                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [1] UNDERSTAND     [2] DETECT      [3] DEFEND      [4] STRESS-TEST         │
│  when-rlhf-fails    misuse-bench    safeguards-sim  stress-tests            │
│        │                 │                │               │                  │
│        └────────┬────────┴────────┬───────┴───────┬───────┘                  │
│                 │                 │               │                          │
│                 ▼                 ▼               ▼                          │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │  [5] PRODUCTION EVALUATION                                       │        │
│  │      safety-harness/release-gate                           │        │
│  └─────────────────────────────────┬───────────────────────────────┘        │
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │  [6] RELEASE GATING                                              │        │
│  │      safety-harness/regression-suite                               │        │
│  │      OK / WARN / BLOCK                                           │        │
│  └─────────────────────────────────┬───────────────────────────────┘        │
│                                    │                                         │
│                         ┌──────────┴──────────┐                             │
│                         ▼                     ▼                             │
│                    [DEPLOY]              [BLOCK]                            │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │  [7] INCIDENT RESPONSE                                           │        │
│  │      safety-harness/incident-lab                                 │        │
│  └─────────────────────────────────┬───────────────────────────────┘        │
│                                    │                                         │
│                    ┌───────────────┴───────────────┐                        │
│                    ▼                               ▼                        │
│             New scenarios                   New regression tests            │
│             → [2] benchmarks                → [6] release gate              │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │  [8] RESEARCH COMMUNICATION                                      │        │
│  │      safety-memos                                                │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```
