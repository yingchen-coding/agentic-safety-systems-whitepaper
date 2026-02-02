# Engineering Agentic Safeguards as a System

*Why Safety Fails in Practice and How to Close the Loop*

---

## Executive Summary

Agentic AI systems introduce a qualitatively different safety risk profile compared to single-turn chatbots. In practice, safety failures in agentic systems are rarely immediate or localized. Instead, they emerge gradually over trajectories, compound across system boundaries, and are shaped by organizational incentives and deployment constraints.

This whitepaper argues that most real-world safety failures are systemic rather than model-intrinsic. Improvements in model alignment, red-teaming, or detection in isolation do not prevent silent safety regressions once models are embedded in production agent loops with partial observability, cost constraints, and heterogeneous safeguards.

We present a closed-loop safety systems framework that integrates:

1. Failure analysis
2. Multi-turn misuse detection benchmarks
3. Proactive red-teaming
4. In-loop safeguards
5. Production evaluation infrastructure
6. Release gating via safety regression testing
7. Incident-driven regression generation

Across multiple empirical artifacts and simulators, we observe three recurring patterns:

- **Silent erosion:** Safety policies and alignment constraints degrade gradually over trajectories without triggering immediate violations.
- **Delayed failures:** Harmful behaviors emerge only after extended interaction or tool use, escaping single-turn evaluations.
- **Incentive-shaped risk:** Safeguards are often weakened in practice due to operational pressure, alert fatigue, and misaligned ownership.

The core claim of this paper is that safety in agentic systems cannot be achieved through point solutions. Instead, safety must be engineered as an evolving system with explicit feedback loops, regression gating, and organizational governance. The goal is not to eliminate failures, but to bound risk, detect degradation early, and convert incidents into durable safety improvements.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     CLOSED-LOOP SAFETY SYSTEM                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  UNDERSTAND ──▶ DETECT ──▶ DEFEND ──▶ STRESS ──▶ EVAL ──▶ GATE ──▶ DEPLOY  │
│       │            │          │          │         │        │         │     │
│       │            │          │          │         │        │         │     │
│       │            │          │          │         │        │         ▼     │
│       │            │          │          │         │        │     INCIDENT  │
│       │            │          │          │         │        │         │     │
│       │            │          │          │         │        │         │     │
│       └────────────┴──────────┴──────────┴─────────┴────────┴─────────┘     │
│                                                                              │
│                            FEEDBACK LOOP                                     │
│                    Incidents become regression tests                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Problem Framing: Why Agentic Safety Fails Systematically

Agentic AI systems differ fundamentally from single-turn language models. They operate over trajectories, invoke tools, maintain state, and interact with partially observable environments. As a result, safety failures in agentic systems are not only a function of model behavior, but of the interaction between model policies, safeguards, tooling, evaluation protocols, and organizational processes.

In practice, this leads to three systematic failure modes that are underrepresented in current safety benchmarks:

### 1.1 Trajectory-Dependent Harm

Many harmful behaviors only emerge after multiple turns of interaction. This includes gradual policy erosion, intent drift, and coordination between benign-looking substeps that collectively produce misuse. Single-turn evaluations systematically underestimate these risks because they assume that violations are immediate and localized.

```
Turn 1: Strong refusal
Turn 2: Partial compliance with caveats
Turn 3: Full compliance with disclaimer
Turn 4: Full compliance without disclaimer
Turn 5: Active assistance
```

No single turn crosses a bright line. The trajectory does.

### 1.2 Partial Observability of Safety-Relevant Signals

In production agent loops, safeguards do not have full visibility into user intent, downstream tool effects, or long-horizon consequences. Signals available to pre-action or post-action filters are incomplete proxies for real-world harm. This creates blind spots where harmful trajectories pass through individually "safe-looking" steps.

### 1.3 Operational Constraints Shaping Safety Outcomes

Safeguards operate under cost limits, latency budgets, and alert fatigue. These constraints lead to thresholds being loosened, detectors being sampled, or escalation policies being softened in practice. As a result, safety properties that hold in offline evaluation degrade in deployment.

### The Core Insight

These factors interact to produce **silent failures**: safety regressions that do not trigger explicit policy violations but materially increase real-world risk. Crucially, these failures are not attributable to a single component. They arise from system-level interactions and are therefore invisible to evaluation methodologies that test components in isolation.

> The implication is that improving any single layer (e.g., better RLHF, stronger red-teaming, or higher-precision detectors) does not guarantee improved end-to-end safety. Without system-level feedback loops and regression controls, localized improvements can be offset by degradation elsewhere in the stack.

---

## 2. Failure Taxonomy: How Safeguards Break in Practice

Empirically, safety failures in agentic systems cluster into a small number of recurring categories. These categories are not mutually exclusive and often compound across the lifecycle of deployment, monitoring, and iteration.

### 2.1 Policy Erosion

Safety policies that are effective in isolation degrade over trajectories. This includes gradual relaxation of refusal criteria, implicit policy weakening through decomposition-based prompting, and over-optimization of compliance behaviors that create new loopholes. Erosion is typically undetectable in single-turn evaluations because each step appears compliant in isolation.

### 2.2 Detector Blind Spots

Misuse detectors and intent classifiers exhibit systematic blind spots in multi-turn contexts. Coordinated misuse, slow intent drift, and benign-looking subgoals frequently evade turn-level classifiers. Over time, attackers adapt to detector failure modes, further reducing detection rates.

### 2.3 Escalation Failure

Safeguard escalation mechanisms (e.g., warnings, soft stops, human review) fail due to threshold miscalibration, alert fatigue, or operational pressure to reduce false positives. In practice, escalation policies are often weakened post-deployment to maintain user experience or throughput, creating unmonitored risk corridors.

### 2.4 Evaluation–Deployment Gap

Offline evaluations and benchmarks fail to reflect production conditions. Differences in traffic distribution, user intent diversity, and tool interactions lead to performance gaps that systematically bias safety metrics upward during development. This gap is a primary driver of silent regressions after model or policy updates.

### 2.5 Governance and Incentive Failures

Safety failures frequently originate outside the model or safeguards layer. Misaligned incentives between product velocity and safety enforcement, unclear ownership of false negatives, and metric gaming create organizational pathways for risk accumulation. These failures are often invisible to purely technical evaluations.

### The Core Insight

> This taxonomy highlights a central theme: most safety failures are interface failures between components and organizations, not isolated model misbehavior. Consequently, robust safety engineering requires not only better models or detectors, but explicit mechanisms to monitor, gate, and correct system-level degradation over time.

---

## 3. Red-Teaming Is Necessary but Insufficient

Red-teaming is a foundational safety practice, but it does not provide the coverage guarantees that production agentic systems require. In multi-turn contexts, the attack surface is combinatorially larger, and static attack sets quickly become obsolete as models are updated.

### 3.1 Delayed Failures Are the Dominant Failure Mode

Empirical stress testing reveals that the majority of safeguard failures occur not on the first turn, but after sustained adversarial pressure:

| Turn | Cumulative Failure Rate |
|------|-------------------------|
| 1 | 5% |
| 2 | 12% |
| 3 | 28% |
| 4 | 45% |
| 5 | 62% |
| 6+ | 75%+ |

Single-turn red-teaming captures only the 5% that fail immediately. The remaining 70%+ require trajectory-level stress testing to surface.

### 3.2 Erosion Curves as a Diagnostic Tool

Plotting safety scores across turns reveals erosion patterns that are invisible in aggregate metrics:

```
Safety Score
    │
1.0 ├████
    │    ████
    │        ████
    │            ████
    │                ████
0.5 ├                    ████
    │                        ████
    │                            ████
    │                                ████
0.0 ├────┬────┬────┬────┬────┬────┬────┬────
    Turn 1    2    3    4    5    6    7    8
```

The slope of this curve is a more predictive metric than any single-turn score. Steep erosion indicates vulnerability to decomposition-based attacks.

### 3.3 Attacker Adaptation Outpaces Static Benchmarks

Attack templates that are effective today become ineffective as models are trained against them. Effective red-teaming must include:

- **Mutation strategies:** Systematic variation of known attack patterns
- **Feedback loops:** Adaptation based on failed attack attempts
- **Novel generation:** Discovery of attack patterns not present in training data

Static jailbreak benchmarks provide a false sense of security because they test known attacks on models that may have been trained (directly or indirectly) to defend against them.

### 3.4 Why Red-Teaming Alone Is Insufficient

| Red-Team Property | Production Reality |
|-------------------|-------------------|
| Fixed attack corpus | Attackers innovate continuously |
| Single-turn focus | Multi-turn attacks dominate |
| Known distribution | Distribution shift at deployment |
| Point-in-time evaluation | Continuous adversarial pressure |

### The Core Insight

> Red-teaming that does not model attacker adaptation and multi-turn erosion produces false confidence. The goal of red-teaming should not be to "pass" a benchmark, but to discover the shape of the erosion curve and the conditions under which safeguards degrade.

---

## 4. Detection Benchmarks: Why Single-Turn Metrics Mislead

Misuse detection is a critical safeguard layer, but detection benchmarks that operate at the turn level systematically underestimate risk in agentic contexts.

### 4.1 Trajectory-Level vs Turn-Level Detection

| Detection Type | What It Measures | Blind Spots |
|----------------|------------------|-------------|
| Turn-level | Individual message safety | Decomposed attacks, context manipulation |
| Trajectory-level | Conversation-wide safety | Higher cost, noisier signals |

In empirical evaluations, trajectory-level detection catches 40-60% more attacks than turn-level detection on the same traffic. This gap widens for sophisticated attackers who deliberately decompose harmful requests.

### 4.2 The Cost Asymmetry of Detection Errors

| Error Type | Immediate Cost | Long-term Cost |
|------------|----------------|----------------|
| False Positive | User friction, support load | Trust erosion, workarounds |
| False Negative | Potential harm event | Regulatory exposure, reputation |

In safety-critical applications, the cost of false negatives vastly exceeds the cost of false positives. Detection systems should be calibrated for recall, not precision or balanced accuracy.

### 4.3 Detection Latency Determines Intervention Value

```
Harm Potential
    │
High├                    ████████████
    │                ████
    │            ████
    │        ████
    │    ████
Low ├████
    └────┬────┬────┬────┬────┬────────▶
     Turn 1    2    3    4    5   Time

Detection at Turn 5: Limited intervention value
Detection at Turn 2: High intervention value
```

The value of detection decays rapidly with latency. Early detection enables intervention; late detection enables only post-hoc analysis.

### 4.4 Benchmark Limitations

Current detection benchmarks suffer from several structural limitations:

- **Coverage bias:** Benchmarks over-represent known attack patterns and under-represent novel vectors
- **Distribution mismatch:** Benchmark traffic differs from production traffic in intent diversity and sophistication
- **Single-turn framing:** Most benchmarks score individual messages, not trajectories
- **Overfitting risk:** Public benchmarks become de facto training targets

### The Core Insight

> Single-turn detection benchmarks systematically underestimate risk because they assume harm is localized and immediate. For agentic systems, trajectory-level detection is not an optimization—it is a requirement for meaningful safety coverage.

---

## 5. Safeguards in the Loop: Where to Intervene in Agent Architectures

Safeguards must be integrated into the agent execution loop, not bolted on as external filters. The placement and design of safeguard hooks determines what classes of failures can be caught and at what cost.

### 5.1 The Agent Execution Loop

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AGENT LOOP                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  User Input                                                                  │
│      │                                                                       │
│      ▼                                                                       │
│  ┌─────────────────────┐                                                    │
│  │  PRE-ACTION HOOK    │ ← Intent classification, injection detection       │
│  └──────────┬──────────┘                                                    │
│             │                                                                │
│             ▼                                                                │
│  ┌─────────────────────┐                                                    │
│  │  PLANNER (LLM)      │                                                    │
│  └──────────┬──────────┘                                                    │
│             │                                                                │
│             ▼                                                                │
│  ┌─────────────────────┐                                                    │
│  │  MID-TRAJECTORY     │ ← Drift detection, cumulative risk scoring        │
│  │  MONITOR            │                                                    │
│  └──────────┬──────────┘                                                    │
│             │                                                                │
│             ▼                                                                │
│  ┌─────────────────────┐                                                    │
│  │  EXECUTOR           │ ← Tool calls, external actions                     │
│  └──────────┬──────────┘                                                    │
│             │                                                                │
│             ▼                                                                │
│  ┌─────────────────────┐                                                    │
│  │  POST-ACTION HOOK   │ ← Output filtering, verification                   │
│  └──────────┬──────────┘                                                    │
│             │                                                                │
│             ▼                                                                │
│      Response                                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Intervention Points and Their Tradeoffs

| Hook | What It Catches | Latency Impact | False Positive Risk |
|------|-----------------|----------------|---------------------|
| Pre-action | Direct attacks, prompt injection | Low | Medium |
| Mid-trajectory | Drift, erosion, intent shift | Medium | Low |
| Post-action | Output policy violations | Low | Medium |

Mid-trajectory monitoring is the highest-leverage intervention point but is the least commonly implemented. It requires maintaining state across turns and computing cumulative risk scores, which adds complexity and latency.

### 5.3 Escalation Policy Design

Escalation policies determine what happens when a safeguard fires. Common options:

| Action | User Impact | Safety | Operational Cost |
|--------|-------------|--------|------------------|
| Silent log | None | Low | Low |
| Soft warning | Low | Medium | Low |
| Require confirmation | Medium | Medium | Medium |
| Block and explain | High | High | Medium |
| Human review queue | High | Highest | High |

No single escalation policy is optimal. The right policy depends on:
- Confidence of the detection signal
- Severity of potential harm
- User context and history
- Operational capacity for human review

### The Core Insight

> Safeguards must be embedded in the agent loop, not applied as post-hoc filters. The highest-leverage intervention point is mid-trajectory monitoring, which can detect drift and erosion before harm crystallizes. However, mid-trajectory safeguards require careful design to avoid unacceptable latency and false positive rates.

---

## 6. Production Reality: Evaluation Infrastructure and Cost Constraints

Safety evaluation at scale introduces operational constraints that fundamentally shape what is feasible. Research-grade evaluation methodologies often fail to transfer to production because they assume unlimited compute, instant evaluation, and perfect observability.

### 6.1 The Operational Gap

| Research Assumption | Production Reality |
|---------------------|-------------------|
| Unlimited compute budget | Cost per eval matters |
| Instant evaluation | Latency budgets are tight |
| Full observability | Partial logging, sampling |
| Stable model behavior | Model version drift |
| Cooperative users | Adversarial traffic |

### 6.2 Cost Model for Safety Evaluation

| Component | Cost per 1K Evals |
|-----------|-------------------|
| Model API calls | $2-20 |
| Worker compute | $0.50-2.00 |
| Storage | $0.10-0.50 |
| Network | $0.05-0.20 |
| **Total** | **$2.65-22.70** |

At production scale (millions of interactions), cost determines which evaluations are feasible. Full trajectory-level evaluation of all traffic is typically not economical; sampling and prioritization are required.

### 6.3 Backpressure and Graceful Degradation

When evaluation load exceeds capacity, systems must degrade gracefully:

```
Load Level    │ Response
──────────────┼─────────────────────────────────
Normal        │ Full evaluation
Elevated      │ Sample 50%, prioritize high-risk
High          │ Sample 10%, critical paths only
Critical      │ Circuit breaker, alert on-call
```

Safety systems that fail ungracefully under load create windows of unmonitored risk.

### 6.4 Alert Fatigue and Operational Decay

```
Alert Volume Over Time
    │
High├████████████████████████████████████████
    │
Low ├
    └────────────────────────────────────────▶

Operator Response Rate Over Time
    │
High├████
    │    ████
    │        ████
    │            ████
Low ├                ████████████████████████
    └────────────────────────────────────────▶
```

As alert volume increases, operator response rate decreases. Eventually, critical alerts are treated the same as routine ones. This is a primary pathway for safeguard degradation in production.

### The Core Insight

> Safety systems fail operationally before they fail algorithmically. Production safeguards must be designed with explicit cost models, graceful degradation under load, and resistance to alert fatigue. Evaluation methodologies that ignore operational constraints will not transfer to deployment.

---

## 7. Release Gating: Why Safety Regressions Are Inevitable Without CI/CD

Safety regressions—degradations in safety properties between model versions—are a primary source of real-world risk. Without explicit regression gating, safety properties erode silently across releases.

### 7.1 The Regression Problem

| Model Version | Absolute Safety Score | vs Previous Version |
|---------------|----------------------|---------------------|
| v1.0 | 92% | Baseline |
| v1.1 | 91% | -1% (regression) |
| v1.2 | 90% | -1% (regression) |
| v1.3 | 89% | -1% (regression) |

Each version passes absolute safety thresholds. After four releases, safety has degraded by 3 percentage points—but no individual release was flagged.

### 7.2 Gating Verdicts

| Verdict | Meaning | Action |
|---------|---------|--------|
| **OK** | No significant regression | Proceed to deployment |
| **WARN** | Potential regression detected | Proceed with enhanced monitoring |
| **BLOCK** | Significant regression confirmed | Requires explicit override to proceed |

### 7.3 Statistical Rigor in Regression Detection

Point estimates are insufficient for gating decisions. Robust regression detection requires:

- **Confidence intervals:** BLOCK only if the lower bound of the CI exceeds the threshold
- **Multiple runs:** Average across seeds to reduce variance
- **Significance testing:** Distinguish signal from noise

```python
# Pseudo-code for statistically rigorous gating
if ci_lower_bound > block_threshold:
    return BLOCK
elif point_estimate > warn_threshold:
    return WARN
else:
    return OK
```

### 7.4 Longitudinal Trend Tracking

Pairwise comparisons miss slow erosion. Longitudinal tracking plots safety metrics across many releases:

```
Safety Metric
    │
0.95├████
    │    ████
0.90├        ████
    │            ████
0.85├                ████
    │                    ████
0.80├                        ████
    └────┬────┬────┬────┬────┬────▶
        v1.0 v1.1 v1.2 v1.3 v1.4  Releases
```

Each pairwise comparison is OK. The trend is not.

### The Core Insight

> Safety regressions are inevitable without explicit regression gating. Treating safety metrics as CI/CD signals—with automated OK/WARN/BLOCK verdicts—is the only scalable approach to preventing silent erosion across model versions.

---

## 8. Incident → Regression: Closing the Feedback Loop

Safety incidents are inevitable. The question is whether incidents become durable improvements or are forgotten after the immediate fix.

### 8.1 The Incident Response Pipeline

```
INCIDENT DETECTED
    │
    ├─► TRIAGE: Assess severity, assign owner
    │
    ├─► CONTAIN: Immediate mitigation
    │
    ├─► ANALYZE: Root cause analysis
    │
    ├─► SCOPE: Blast radius assessment
    │
    ├─► FIX: Permanent remediation
    │
    ├─► REGRESS: Generate regression test
    │
    └─► LEARN: Blameless postmortem, process improvement
```

### 8.2 Why Incidents Must Become Regression Tests

Without regression tests:
- The same vulnerability can be reintroduced by future model updates
- Organizational memory is lost when team members change
- Safety debt accumulates invisibly

With regression tests:
- Each incident permanently strengthens the test suite
- Future model updates are validated against known failure modes
- The organization develops durable memory of past failures

### 8.3 Learning Velocity as a Safety Metric

| Metric | Definition | Target |
|--------|------------|--------|
| Incident → Regression Time | Time from incident to regression test in CI | < 1 week |
| Recurrence Rate | Fraction of incidents with previously-seen root cause | < 10% |
| Coverage Growth | Net new regression tests per quarter | Monotonically increasing |

### 8.4 Blameless Postmortem Culture

Effective incident learning requires psychological safety:

- **Focus on systems, not individuals:** "The process failed" not "Alice failed"
- **Assume good intent:** Everyone was doing their best with available information
- **Reward transparency:** Surfacing problems is valued, not punished
- **Track action items:** Postmortems without follow-through are theater

### The Core Insight

> Safety that does not learn from incidents accumulates safety debt. Regression tests are the only durable organizational memory of past failures. The goal is a system that learns faster than attackers and faster than organizational memory decays.

---

## 9. Organizational Failure Modes and Incentives

Many safety failures originate outside the model or safeguards layer. Organizational incentives, ownership structures, and governance processes determine whether technical safeguards are enforced or bypassed.

### 9.1 Incentive Misalignment Patterns

#### Pattern 1: Velocity vs Safety

| Team | Measured On | Safety Incentive |
|------|-------------|------------------|
| Product | Ship date, feature completeness | Negative (safety slows shipping) |
| Engineering | Performance, reliability | Neutral |
| Safety | Incident rate, coverage | Positive |

**Result:** Safety becomes a "cost center" that slows "real work."

#### Pattern 2: Metric Gaming

Teams learn to optimize for safety benchmarks without improving actual safety:
- Tune system prompts to pass specific test cases
- Exclude failing scenarios from the benchmark
- Lower detection thresholds to reduce BLOCK rate

**Detection:** Held-out test sets, out-of-distribution probes, correlation with production incidents.

#### Pattern 3: Alert Fatigue

As alert volume grows, operator response degrades. Eventually, all alerts—including critical ones—are ignored or deprioritized.

**Mitigation:** Tiered alerting, alert deduplication, periodic alert audit.

#### Pattern 4: Exception Creep

1. First exception: "Critical launch, we'll fix after"
2. Second exception: "We did it before, it was fine"
3. Third exception: "This is just how we operate"
4. Exceptions become the default

**Mitigation:** Exception budget per quarter, escalating approval for repeated exceptions, public exception log.

### 9.2 Governance Anti-Patterns

| Anti-Pattern | Description | Fix |
|--------------|-------------|-----|
| Responsibility without authority | Safety team is blamed but cannot block | Give safety team veto power with escalation path |
| Authority without accountability | Safety team can block anything but isn't measured on cost | Measure safety team on both FN and FP rates |
| Governance by committee | Every decision requires 10-person meeting | Clear RACI, single decision-maker per decision type |
| Documentation without enforcement | Policies exist on paper but not in tooling | Encode policies in automated gates |

### The Core Insight

> Many safety failures are not technical but organizational. Incentives, ownership boundaries, and escalation authority determine whether safeguards are enforced or bypassed. Technical safety solutions without organizational alignment are theater.

---

## 10. What This System Cannot Solve

No safety system can guarantee zero failures. Honest assessment of limitations is essential for appropriate reliance.

### 10.1 Adversarial Creativity

Attackers will always discover novel vectors that were not anticipated in benchmarks or stress tests. Zero-day attacks will succeed until they are detected and incorporated into defenses.

**Implication:** Reactive capability (fast incident detection and response) is as important as proactive capability (prevention).

### 10.2 Distribution Shift

Models deployed in new domains, languages, or user populations may behave differently than in evaluation. Benchmarks are necessarily limited in coverage, and production traffic will always include cases not represented in evaluation.

**Implication:** Domain-specific validation is required. Cross-domain generalization should not be assumed.

### 10.3 Unknown Unknowns

We cannot test for attacks we have not imagined. The attack surface is larger than any benchmark can cover.

**Implication:** Defense in depth. No single safeguard layer should be the sole line of defense.

### 10.4 Dual-Use Ambiguity

Some content and capabilities are legitimately dual-use. No classifier can perfectly distinguish malicious from benign intent based on content alone.

**Implication:** Accept either false positives or false negatives. The choice should be conscious and context-dependent.

### 10.5 Organizational Will

Technical safeguards cannot force organizations to prioritize safety. If leadership does not value safety, safeguards will be bypassed, underfunded, or ignored.

**Implication:** Technical safety solutions require organizational commitment as a prerequisite.

### The Core Insight

> No safeguards system can guarantee safety. The goal is not zero incidents, but bounded risk, early detection, fast response, and continuous improvement. A system that learns faster than attackers adapt is winning, even if incidents occur.

---

## 11. Design Principles

The following principles synthesize the lessons from building and operating safety systems for agentic AI.

### Principle 1: Trajectory-First Evaluation

Evaluate conversation trajectories, not individual messages. Single-turn metrics systematically underestimate risk in agentic contexts.

### Principle 2: Regression Before Release

No model ships without regression testing against known failures. Safety is a CI/CD gate, not a checklist item.

### Principle 3: Incidents Become Tests

Every incident generates a regression test. Organizational memory is encoded in test suites, not documentation.

### Principle 4: Governance Is Part of Safety

Technical safeguards without organizational enforcement are theater. Incentive structures and authority boundaries determine real-world safety outcomes.

### Principle 5: Metrics Must Be Game-Resistant

Assume teams will optimize for whatever is measured. Use held-out test sets, out-of-distribution probes, and correlation with production incidents.

### Principle 6: Assume Safeguards Will Be Bypassed

Design for defense in depth. No single layer should be the sole protection.

### Principle 7: Prefer False Positives to False Negatives

In safety-critical contexts, blocking a benign action is recoverable. Missing a harmful action may not be.

### Principle 8: Learn Faster Than Attackers Adapt

The goal is not zero incidents, but a learning system that improves faster than the threat landscape evolves.

---

## 12. Appendix

### A. Portfolio Mapping

| Whitepaper Section | Repository |
|--------------------|------------|
| Problem Framing | when-rlhf-fails-quietly |
| Failure Taxonomy | when-rlhf-fails-quietly, agentic-safety-incident-lab |
| Red-Teaming | safeguards-stress-tests |
| Detection Benchmarks | agentic-misuse-benchmark |
| Safeguards in Loop | agentic-safeguards-simulator |
| Production Infrastructure | scalable-safeguards-eval-pipeline |
| Release Gating | model-safety-regression-suite |
| Incident Response | agentic-safety-incident-lab |
| Research Communication | safety-memos |

### B. Reproducibility

- All evaluation code uses fixed random seeds for reproducibility
- Model API responses are inherently stochastic; results may vary across API versions
- Hardware: Results generated on Apple M1 Pro, 16GB RAM
- See per-repository `docs/reproducibility.md` for detailed reproduction instructions

### C. Negative Results

| Approach | Outcome | Learning |
|----------|---------|----------|
| Single-turn proxy metrics for trajectory safety | Correlation < 0.3 with trajectory outcomes | Trajectory-level evaluation is not optional |
| Reward model scores as safety signals | Decorrelate from safety in multi-turn contexts | RM blind spots are systematic |
| Keyword blocklists | Trivially bypassed via paraphrase | Semantic understanding required |
| Fixed escalation thresholds | Gamed by attackers and operators | Adaptive thresholds necessary |
| User reputation scoring | Defeated by new accounts, Sybil attacks | Per-session assessment more robust |

### D. Future Work

- Formal verification of safeguard composition properties
- Automated novel attack generation (beyond mutation)
- Cross-organization safety benchmarking standards
- Real-time adversarial adaptation detection
- Causal (not just correlational) incident root cause analysis

---

*This whitepaper synthesizes learnings from building production-oriented safety evaluation and safeguards systems for agentic AI. It represents the author's current understanding and is expected to evolve as the field matures.*
