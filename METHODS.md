# Methods: Portfolio-to-Methodology Mapping

## Overview

This document maps the 8 repository artifacts to the methodological framework presented in the whitepaper. Each repository implements a specific component of the closed-loop safety system.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          METHODOLOGY MAPPING                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PROBLEM ANALYSIS        DETECTION           DEFENSE         ADVERSARIAL    │
│  ┌───────────────┐      ┌─────────────┐    ┌───────────┐   ┌───────────┐   │
│  │ when-rlhf-    │      │ agentic-    │    │ agentic-  │   │ safeguards│   │
│  │ fails-quietly │      │ misuse-     │    │ safeguards│   │ -stress-  │   │
│  │               │      │ benchmark   │    │ -simulator│   │ tests     │   │
│  │ Failure modes │      │ Detection   │    │ Safeguard │   │ Attack    │   │
│  │ + taxonomy    │      │ benchmarks  │    │ design    │   │ generation│   │
│  └───────┬───────┘      └──────┬──────┘    └─────┬─────┘   └─────┬─────┘   │
│          │                     │                  │               │         │
│          └──────────┬──────────┴────────┬────────┴───────┬───────┘         │
│                     │                   │                │                  │
│                     ▼                   ▼                ▼                  │
│          ┌─────────────────────────────────────────────────────────┐       │
│          │              EVALUATION INFRASTRUCTURE                    │       │
│          │              scalable-safeguards-eval-pipeline            │       │
│          │                                                           │       │
│          │  • Batch + streaming evaluation                           │       │
│          │  • Cost-aware prioritization                              │       │
│          │  • Drift detection                                        │       │
│          └─────────────────────────┬───────────────────────────────┘       │
│                                    │                                        │
│                                    ▼                                        │
│          ┌─────────────────────────────────────────────────────────┐       │
│          │              RELEASE GATING                               │       │
│          │              model-safety-regression-suite                │       │
│          │                                                           │       │
│          │  • Statistical regression detection                       │       │
│          │  • OK / WARN / BLOCK verdicts                             │       │
│          │  • Longitudinal trend tracking                            │       │
│          └─────────────────────────┬───────────────────────────────┘       │
│                                    │                                        │
│                         ┌──────────┴──────────┐                            │
│                         ▼                     ▼                            │
│                    [DEPLOY]              [BLOCK]                           │
│                         │                                                   │
│                         ▼                                                   │
│          ┌─────────────────────────────────────────────────────────┐       │
│          │              INCIDENT RESPONSE                            │       │
│          │              agentic-safety-incident-lab                  │       │
│          │                                                           │       │
│          │  • Incident triage + RCA                                  │       │
│          │  • Regression test generation                             │       │
│          │  • Blameless postmortem                                   │       │
│          └─────────────────────────┬───────────────────────────────┘       │
│                                    │                                        │
│                    ┌───────────────┴───────────────┐                       │
│                    ▼                               ▼                       │
│             New scenarios                   New regression tests           │
│             → benchmarks                    → release gate                 │
│                                                                             │
│          ┌─────────────────────────────────────────────────────────┐       │
│          │              RESEARCH COMMUNICATION                       │       │
│          │              safety-memos                                 │       │
│          │                                                           │       │
│          │  • Cross-team knowledge transfer                          │       │
│          │  • Stakeholder communication                              │       │
│          │  • Counter-argument documentation                         │       │
│          └─────────────────────────────────────────────────────────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Repository → Methodology Mapping

### 1. when-rlhf-fails-quietly

**Whitepaper Section:** Introduction, RLHF Limitations

**Methodology:**
- Empirical analysis of RLHF failure modes in multi-turn contexts
- Documentation of policy erosion patterns
- Credit assignment failure analysis

**Key Artifacts:**
- Failure taxonomy with RLHF intervention points
- Erosion curve templates
- Reproducibility documentation

---

### 2. agentic-misuse-benchmark

**Whitepaper Section:** Detection Benchmarks and Trajectory-Level Metrics

**Methodology:**
- Trajectory-level detection benchmark design
- Intent drift measurement
- Coverage gap analysis

**Key Artifacts:**
- Multi-turn scenario corpus
- Trajectory-level scoring rubrics
- Threat model documentation

---

### 3. agentic-safeguards-simulator

**Whitepaper Section:** Safeguards in the Loop

**Methodology:**
- Safeguard placement analysis (pre/mid/post-action hooks)
- Escalation policy simulation
- Human-in-the-loop recovery modeling

**Key Artifacts:**
- Agent loop architecture diagrams
- Escalation decision trees
- Latency/accuracy tradeoff analysis

---

### 4. safeguards-stress-tests

**Whitepaper Section:** Red-Teaming Is Necessary but Insufficient

**Methodology:**
- Adaptive attack generation
- Erosion curve measurement
- Coverage vs depth analysis

**Key Artifacts:**
- Attack mutation strategies
- Stopping criteria documentation
- Attack realism assessment

---

### 5. scalable-safeguards-eval-pipeline

**Whitepaper Section:** Production Reality

**Methodology:**
- Batch vs streaming evaluation architecture
- Cost-aware safety coverage
- Drift detection implementation

**Key Artifacts:**
- Cost model documentation
- Backpressure handling
- SLA definitions

---

### 6. model-safety-regression-suite

**Whitepaper Section:** Release Gating via Safety Regression Testing

**Methodology:**
- Statistical regression detection (bootstrap CI, permutation tests)
- OK/WARN/BLOCK verdict logic
- Longitudinal trend tracking

**Key Artifacts:**
- Statistical analysis module
- Governance documentation
- Change management process

---

### 7. agentic-safety-incident-lab

**Whitepaper Section:** Incident → Regression: Closing the Feedback Loop

**Methodology:**
- Incident triage and root cause analysis
- Regression test generation from incidents
- Blameless postmortem process

**Key Artifacts:**
- Severity rubric
- Incident simulation generator
- Learning velocity metrics

---

### 8. safety-memos

**Whitepaper Section:** Cross-cutting (Research Communication)

**Methodology:**
- Research communication to non-technical stakeholders
- Counter-argument documentation
- Limitation acknowledgment

**Key Artifacts:**
- Executive summary templates
- Counter-argument register
- Open questions tracker

---

## Data Flow

```
                              ┌─────────────┐
                              │ Production  │
                              │ Traffic     │
                              └──────┬──────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         eval-pipeline                                    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│  │ Sample      │───▶│ Evaluate    │───▶│ Score       │                  │
│  │ traffic     │    │ trajectories│    │ + aggregate │                  │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                  │
└──────────────────────────────────────────────┬──────────────────────────┘
                                               │
                                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       regression-suite                                   │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│  │ Compare to  │───▶│ Statistical │───▶│ Verdict     │                  │
│  │ baseline    │    │ testing     │    │ OK/WARN/BLOCK│                  │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                  │
└──────────────────────────────────────────────┬──────────────────────────┘
                                               │
                              ┌────────────────┴────────────────┐
                              ▼                                 ▼
                        [DEPLOY]                           [BLOCK]
                              │                                 │
                              ▼                                 │
                     Production deployment                      │
                              │                                 │
                              ▼                                 │
┌─────────────────────────────────────────────────────────────────────────┐
│                        incident-lab                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│  │ Incident    │───▶│ RCA +       │───▶│ Generate    │                  │
│  │ detection   │    │ postmortem  │    │ regression  │                  │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                  │
└──────────────────────────────────────────────┬──────────────────────────┘
                                               │
                              ┌────────────────┴────────────────┐
                              ▼                                 ▼
                    New scenarios                      New regression tests
                    → misuse-benchmark                 → regression-suite
```

---

## Reproducibility

Each repository includes:
- Fixed random seeds for deterministic evaluation
- Frozen model version specifications
- Hardware/environment documentation
- Step-by-step reproduction instructions

See individual repository `docs/reproducibility.md` files for details.
