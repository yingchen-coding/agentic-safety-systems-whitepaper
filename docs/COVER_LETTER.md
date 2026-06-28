# Cover Letter

*For positions in AI Safety Engineering, Safety Research, or Trust & Safety*

---

## Summary

I am applying for roles focused on operationalizing AI safety in production systems. My work addresses a gap I observed in current safety practice: the disconnect between research-grade safety evaluation and the operational reality of deploying agentic AI systems at scale.

This portfolio represents 8 interconnected artifacts that form a closed-loop safety system, synthesized in a whitepaper titled **"Silent Failures in Agentic Systems: Why Single-Turn Safety Evaluations Systematically Underestimate Risk."**

---

## Why This Work Matters

The AI safety field has made significant progress on alignment research, red-teaming methodologies, and safety benchmarks. However, I observed that:

1. **Single-turn benchmarks systematically underestimate risk** in agentic systems where failures emerge over trajectories
2. **Safety properties erode silently** across model releases without explicit regression gating
3. **Incidents do not become durable improvements** without systematic feedback loops
4. **Organizational incentives frequently override technical safeguards** without governance integration

These are not primarily model problems—they are systems problems. My work addresses them by proposing infrastructure, methodology, and governance frameworks that treat safety as an operational discipline rather than a pre-release checklist.

---

## What I Built

| Component | Purpose | Status |
|-----------|---------|--------|
| when-rlhf-fails-quietly | Document RLHF failure modes in multi-turn contexts | Complete |
| agentic-misuse-benchmark | Trajectory-level detection benchmarks | Complete |
| safety-harness/simulator | Safeguard placement and escalation analysis | Complete |
| safety-harness/stress-testing | Adaptive red-teaming with erosion curves | Complete |
| safety-harness/release-gate | Production-grade evaluation infrastructure | Complete |
| safety-harness/regression-suite | CI/CD safety regression gating | Complete |
| safety-harness/incident-lab | Incident-to-regression feedback loops | Complete |
| safety-memos | Research communication templates | Complete |
| Whitepaper | System-level synthesis | Complete |

---

## Key Technical Contributions

### 1. Trajectory-Level Metrics

I developed metrics that capture multi-turn failure modes invisible to single-turn evaluation:
- **Delayed violation rate (DVR):** Fraction of violations occurring after turn N
- **Erosion curve slope:** Rate of safety degradation under sustained pressure
- **Cumulative intent drift:** Trajectory-wide intent modeling

### 2. Statistical Regression Gating

I implemented statistically rigorous release gating with:
- Bootstrap confidence intervals for regression detection
- Permutation tests for significance
- OK/WARN/BLOCK verdicts with audit trails
- Longitudinal trend tracking across releases

### 3. Incident-to-Regression Feedback Loops

I designed systematic conversion of production incidents into permanent regression tests:
- Severity-based triage rubrics
- Root cause attribution
- Automatic regression test generation
- Learning velocity metrics

### 4. Organizational Failure Mode Taxonomy

I documented how organizational pressures create pathways for safeguard degradation:
- Velocity vs safety misalignment
- Metric gaming patterns and detection strategies
- Alert fatigue dynamics
- Exception creep lifecycle

---

## Why I Am a Good Fit

### Technical Skills

- **Safety evaluation:** Trajectory-level metrics, erosion curves, statistical regression detection
- **Production systems:** Cost-aware evaluation, backpressure handling, SLA design
- **Red-teaming:** Adaptive attack generation, coverage analysis, stopping criteria

### Research Skills

- **Problem framing:** Identifying gaps between research assumptions and production reality
- **System design:** Integrating multiple components into coherent architectures
- **Communication:** Translating technical findings for non-technical stakeholders

### Mindset

- **Skeptical of benchmarks:** I treat benchmark scores as lower bounds, not guarantees
- **Incident-driven:** I believe safety teams should learn faster than attackers adapt
- **Governance-aware:** I understand that technical safeguards without organizational buy-in are theater

---

## What I Want to Work On

I am most interested in roles that involve:

1. **Building safety evaluation infrastructure** that operates at production scale
2. **Designing regression gating systems** that prevent silent safety degradation
3. **Establishing incident-to-regression feedback loops** that create durable organizational memory
4. **Advising on safety governance** that aligns organizational incentives with safety outcomes

I am less interested in pure model alignment research without a deployment focus, or in roles that treat safety as a compliance checkbox rather than an engineering discipline.

---

## Contact

- **GitHub:** [yingchen-coding](https://github.com/yingchen-coding)
- **Portfolio:** [Safety Memos](https://yingchen-coding.github.io/safety-memos/)

---

*This portfolio is licensed under CC BY-NC 4.0 (Non-commercial). All code and documentation are original work.*
