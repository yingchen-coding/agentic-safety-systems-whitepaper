> **Portfolio**: [Safety Memos](https://yingchen-coding.github.io/safety-memos/) · [GitHub Pages](https://yingchen-coding.github.io/agentic-safety-systems-whitepaper/)

# Silent Failures in Agentic Systems

[![CI](https://github.com/yingchen-coding/agentic-safety-systems-whitepaper/actions/workflows/ci.yml/badge.svg)](https://github.com/yingchen-coding/agentic-safety-systems-whitepaper/actions/workflows/ci.yml)

**Why Single-Turn Safety Evaluations Systematically Underestimate Risk**

---

## Abstract

Despite widespread deployment of large language models, safety evaluation practices remain dominated by single-turn benchmarks and static red-teaming. We argue that these methods systematically underestimate risk in agentic systems, where failures emerge over multi-turn interactions under partial observability.

We propose a lifecycle-oriented safety framework that integrates trajectory-level evaluation, safeguards embedded in agent loops, regression-based release gating, and incident-driven feedback loops. This reframes safety from a pre-release checklist into a continuous, production-grade engineering discipline.

> Where prior work asks *"Is this model aligned?"*, we ask:
> **"Does this system remain safe as it evolves, ships, and fails in the real world?"**

---

## Key Contributions

| # | Contribution | Why It Matters |
|---|--------------|----------------|
| 1 | **Safety as non-regression invariant** | Enforced via CI/CD gating (OK/WARN/BLOCK), not pre-release checklist |
| 2 | **Trajectory-level metrics** | Delayed violation rate, erosion curves—failures invisible to turn-level evals |
| 3 | **Red-teaming → release gating** | Red-team failures become binding CI signals, not advisory reports |
| 4 | **Incident → regression tests** | Production failures permanently expand evaluation coverage |
| 5 | **Production-grade eval infra** | Cost-aware, drift detection, traffic replay—not research tooling |
| 6 | **Organizational failure modes** | Metric gaming, alert fatigue, exception creep as first-class concerns |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CLOSED-LOOP SAFETY SYSTEM                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   UNDERSTAND          DETECT           DEFEND          STRESS-TEST          │
│   when-rlhf-fails     misuse-bench     safeguards-sim  stress-tests         │
│        │                  │                │               │                │
│        ▼                  ▼                ▼               ▼                │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │              SCALABLE EVAL PIPELINE                          │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                                ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │           RELEASE GATE: OK / WARN / BLOCK                    │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                    ┌───────────┴───────────┐                               │
│                    ▼                       ▼                               │
│              [DEPLOY]                 [BLOCK]                              │
│                    │                                                        │
│                    ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │           INCIDENT LAB → REGRESSION TESTS                    │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                │                                            │
│                    ┌───────────┴───────────┐                               │
│                    ▼                       ▼                               │
│             New scenarios          New regression tests                    │
│             → benchmarks           → release gate                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5-Minute End-to-End Demo

See the full closed-loop system in action with [safety-harness/demo](https://github.com/yingchen-coding/safety-harness/tree/main/demo):

```bash
git clone https://github.com/yingchen-coding/safety-harness
cd safety-harness/demo
make demo
```

This demo walks through:
1. **Stress Testing** → Discover delayed failures via adaptive red-teaming
2. **Regression Generation** → Convert failures into permanent tests
3. **Release Gate** → Gate a candidate model (OK/WARN/BLOCK)
4. **Incident Replay** → Feed production incidents back into the system

---

## Repository Structure

```
agentic-safety-systems-whitepaper/
├── paper/
│   ├── whitepaper.md          # Full 13-section research whitepaper
│   ├── CONTRIBUTIONS.md       # 6 research contributions
│   ├── METHODS.md             # 8-stage methodology mapping
│   └── COMPARISON.md          # vs Anthropic/OpenAI/DeepMind
├── docs/
│   ├── INTERVIEW_NARRATIVE.md # 30-minute interview walkthrough
│   └── COVER_LETTER.md        # Job application template
├── figures/                   # Diagrams and visualizations
├── index.html                 # GitHub Pages landing page
├── LICENSE                    # CC BY-NC 4.0
└── README.md
```

---

## How This Differs from Existing Work

| Dimension | Anthropic / OpenAI / DeepMind | This Work |
|-----------|-------------------------------|-----------|
| Evaluation granularity | Single-turn / short-horizon | Trajectory-level, multi-turn |
| Safety framing | Model property | System-level non-regression invariant |
| Red-teaming integration | Episodic, advisory | Continuous, release-gating |
| Incident handling | Postmortems | Incident → replay → regression tests |
| Organizational risk | Implicit | Explicit governance + anti-gaming |

---

## Portfolio Components

| Repository | Role in System |
|------------|----------------|
| [when-rlhf-fails-quietly](https://github.com/yingchen-coding/when-rlhf-fails-quietly) | Failure taxonomy |
| [agentic-misuse-benchmark](https://github.com/yingchen-coding/agentic-misuse-benchmark) | Detection benchmarks |
| [safety-harness/simulator](https://github.com/yingchen-coding/safety-harness/tree/main/simulator) | Safeguard design |
| [safety-harness/stress-testing](https://github.com/yingchen-coding/safety-harness/tree/main/stress-testing) | Red-teaming |
| [safety-harness/release-gate](https://github.com/yingchen-coding/safety-harness/tree/main/release-gate) | Production eval infra |
| [safety-harness/regression-suite](https://github.com/yingchen-coding/safety-harness/tree/main/regression-suite) | Release gating |
| [safety-harness/incident-lab](https://github.com/yingchen-coding/safety-harness/tree/main/incident-lab) | Incident → regression |
| [safety-memos](https://yingchen-coding.github.io/safety-memos/) | Research communication |

---

## Quick Links

- [Full Whitepaper](paper/whitepaper.md) — 13-section research paper
- [Research Contributions](paper/CONTRIBUTIONS.md) — What's novel
- [Methods](paper/METHODS.md) — How the 8 repos map to methodology
- [Comparison](paper/COMPARISON.md) — vs existing safety work
- [Interview Guide](docs/INTERVIEW_NARRATIVE.md) — 30-minute walkthrough
- [GitHub Pages](https://yingchen-coding.github.io/agentic-safety-systems-whitepaper/) — Visual landing page

## Local Review Gate

```bash
scripts/pr_review_check.sh
```

This runs internal-link checks, compile checks, secret scanning, and commit-history attribution
checks. GitHub runs the same gate through the `PR Review Gate` workflow.

---

## License

CC BY-NC 4.0 (Non-commercial)
