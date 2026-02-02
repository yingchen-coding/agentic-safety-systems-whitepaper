> **Portfolio**: [Safety Memo](https://yingchen-coding.github.io/safety-memos/) · [when-rlhf-fails-quietly](https://github.com/yingchen-coding/when-rlhf-fails-quietly) · [agentic-misuse-benchmark](https://github.com/yingchen-coding/agentic-misuse-benchmark) · [agentic-safeguards-simulator](https://github.com/yingchen-coding/agentic-safeguards-simulator) · [safeguards-stress-tests](https://github.com/yingchen-coding/safeguards-stress-tests) · [scalable-safeguards-eval-pipeline](https://github.com/yingchen-coding/scalable-safeguards-eval-pipeline) · [model-safety-regression-suite](https://github.com/yingchen-coding/model-safety-regression-suite) · [agentic-safety-incident-lab](https://github.com/yingchen-coding/agentic-safety-incident-lab)

# Silent Failures in Agentic Systems

> Why Single-Turn Safety Evaluations Systematically Underestimate Risk

## Abstract

Despite widespread deployment of large language models, safety evaluation practices remain dominated by single-turn benchmarks and static red-teaming. We argue that these methods systematically underestimate risk in agentic systems, where failures emerge over multi-turn interactions under partial observability. Through empirical evidence and system-level analysis, we show that alignment mechanisms trained on single-step feedback fail to constrain delayed, compounding failure modes such as policy erosion, intent drift, and tool misuse.

We propose a lifecycle-oriented safety framework that integrates trajectory-level evaluation, safeguards embedded in agent loops, regression-based release gating, and incident-driven feedback loops. This reframes safety from a pre-release checklist into a continuous, production-grade engineering discipline.

---

## Contents

0. [Abstract](whitepaper.md#abstract)
1. [Introduction: The Illusion of Safety in Single-Turn Benchmarks](whitepaper.md#1-introduction-the-illusion-of-safety-in-single-turn-benchmarks)
2. [Why Multi-Turn + Partial Observability Breaks RLHF Guarantees](whitepaper.md#2-why-multi-turn--partial-observability-breaks-rlhf-guarantees)
3. [Red-Teaming Is Necessary but Insufficient](whitepaper.md#3-red-teaming-is-necessary-but-insufficient)
4. [Detection Benchmarks and Trajectory-Level Metrics](whitepaper.md#4-detection-benchmarks-and-trajectory-level-metrics)
5. [Safeguards in the Loop: Where to Intervene](whitepaper.md#5-safeguards-in-the-loop-where-to-intervene-in-agent-architectures)
6. [Production Reality: Eval Infra and Cost Constraints](whitepaper.md#6-production-reality-evaluation-infrastructure-and-cost-constraints)
7. [Release Gating via Safety Regression Testing](whitepaper.md#7-release-gating-why-safety-regressions-are-inevitable-without-cicd)
8. [Incident → Regression: Closing the Feedback Loop](whitepaper.md#8-incident--regression-closing-the-feedback-loop)
9. [Organizational Failure Modes & Incentives](whitepaper.md#9-organizational-failure-modes-and-incentives)
10. [Threat Model](whitepaper.md#10-threat-model)
11. [Limitations](whitepaper.md#11-limitations)
12. [Future Work](whitepaper.md#12-future-work)
13. [Conclusion](whitepaper.md#13-conclusion)

---

## Key Thesis

> The hardest safety problems are not "how do we detect harmful outputs?"
> They are:
> - How do we prevent organizations from gaming safety metrics?
> - How do we stop safety debt from accumulating?
> - How do we maintain safety discipline under shipping pressure?
> - How do we learn from incidents faster than attackers adapt?

These are **systems problems**, not **model problems**.

---

## Portfolio Integration

This whitepaper explains how 8 repositories form a coherent safety system:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CLOSED-LOOP SAFETY SYSTEM                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   UNDERSTAND          DETECT           DEFEND          STRESS-TEST          │
│   ┌─────────┐        ┌─────────┐      ┌─────────┐     ┌─────────┐          │
│   │ when-   │        │ misuse  │      │ safegrd │     │ stress  │          │
│   │ rlhf-   │───────▶│ bench-  │─────▶│ simul-  │────▶│ tests   │          │
│   │ fails   │        │ mark    │      │ ator    │     │         │          │
│   └─────────┘        └─────────┘      └─────────┘     └─────────┘          │
│        │                  │                │               │                │
│        │                  │                │               │                │
│        ▼                  ▼                ▼               ▼                │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                    EVAL PIPELINE                              │          │
│   │                    (scalable-safeguards-eval-pipeline)        │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                                ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                    RELEASE GATE                               │          │
│   │                    (model-safety-regression-suite)            │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                    ┌───────────┴───────────┐                               │
│                    ▼                       ▼                               │
│              [DEPLOY]                 [BLOCK]                              │
│                    │                                                        │
│                    ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                    INCIDENT LAB                               │          │
│   │                    (agentic-safety-incident-lab)              │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                                │ Feedback loop                              │
│                                │                                            │
│                    ┌───────────┴───────────┐                               │
│                    ▼                       ▼                               │
│             New scenarios          New regression tests                    │
│             added to               added to                                │
│             benchmarks             release gate                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Who Should Read This

- **Safety engineers** building production safeguards
- **Engineering managers** responsible for release decisions
- **Researchers** studying organizational safety failures
- **Policy teams** designing safety governance

---

## License

CC BY-NC 4.0 (Non-commercial)
