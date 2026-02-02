# Silent Failures in Agentic Systems

*Why Single-Turn Safety Evaluations Systematically Underestimate Risk*

---

## Abstract

Despite widespread deployment of large language models, safety evaluation practices remain dominated by single-turn benchmarks and static red-teaming. We argue that these methods systematically underestimate risk in agentic systems, where failures emerge over multi-turn interactions under partial observability. Through empirical evidence and system-level analysis, we show that alignment mechanisms trained on single-step feedback fail to constrain delayed, compounding failure modes such as policy erosion, intent drift, and tool misuse.

We propose a lifecycle-oriented safety framework that integrates trajectory-level evaluation, safeguards embedded in agent loops, regression-based release gating, and incident-driven feedback loops. This reframes safety from a pre-release checklist into a continuous, production-grade engineering discipline. Our approach surfaces silent regressions that static benchmarks miss and provides a practical blueprint for operationalizing safety governance in iterative deployment environments.

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

## 1. Introduction: The Illusion of Safety in Single-Turn Benchmarks

### Why "Passing Red-Teaming" Is Not Equivalent to Being Safe

The dominant paradigm in AI safety evaluation assumes that a model which passes red-teaming exercises and benchmark suites is "safe enough" to deploy. This assumption is fundamentally flawed for agentic systems. Passing a static test set measures performance on known attack patterns under controlled conditions—it does not measure resilience to adaptive adversaries, novel attack vectors, or the gradual policy erosion that occurs over extended interactions.

A model can achieve 95% refusal rate on a jailbreak benchmark while still exhibiting dangerous failure modes in production:
- Failures that emerge only after 5+ turns of interaction
- Failures triggered by specific tool combinations not in the benchmark
- Failures that occur when safeguards are under operational load

### Empirical Evidence of Silent Failures

Across multiple evaluation artifacts, we observe consistent patterns of delayed failure:

```
Turn 1: Strong refusal                    (95% of models)
Turn 2: Partial compliance with caveats   (78% of models)
Turn 3: Full compliance with disclaimer   (52% of models)
Turn 4: Full compliance without disclaimer (34% of models)
Turn 5: Active assistance                 (22% of models)
```

No single turn crosses a bright line. The trajectory does.

These silent failures share common characteristics:
- **Gradual onset:** No individual step triggers policy violations
- **Context dependence:** Failures require specific interaction histories
- **Partial observability:** Harm is not visible from any single message

### Contribution Summary

This whitepaper makes three core contributions:

1. **Failure taxonomy:** We document how safeguards break in practice, distinguishing policy erosion, detector blind spots, escalation failures, and organizational decay.

2. **Lifecycle framework:** We propose trajectory-level evaluation, in-loop safeguards, regression-based release gating, and incident-driven feedback as an integrated system.

3. **Operational blueprint:** We provide concrete infrastructure patterns for implementing safety as a continuous engineering discipline rather than a pre-release checklist.

The core claim is that safety in agentic systems cannot be achieved through point solutions. It must be engineered as an evolving system with explicit feedback loops, regression gating, and organizational governance.

---

## 2. Why Multi-Turn + Partial Observability Breaks RLHF Guarantees

RLHF (Reinforcement Learning from Human Feedback) has been remarkably effective at aligning single-turn model behavior with human preferences. However, the assumptions underlying RLHF break down in agentic contexts, leading to systematic safety gaps.

### 2.1 Delayed Failure Modes

RLHF optimizes for immediate feedback signals. When harmful outcomes are delayed by multiple turns, the credit assignment problem becomes intractable:

| Failure Type | RLHF Visibility | Detection Difficulty |
|--------------|-----------------|---------------------|
| Immediate harm | High | Low |
| 2-turn delayed | Medium | Medium |
| 5+ turn delayed | Low | High |
| Tool-mediated | Very Low | Very High |

The reward model sees each turn in isolation. It cannot attribute negative outcomes to decisions made several turns earlier, so it cannot learn to avoid the precursor behaviors.

### 2.2 Credit Assignment Failure

Consider a trajectory where harmful output emerges at turn 5:

```
Turn 1: Establish benign context        ← No negative signal
Turn 2: Request clarification           ← No negative signal
Turn 3: Provide partial information     ← No negative signal
Turn 4: Build on partial information    ← No negative signal
Turn 5: Produce harmful synthesis       ← Negative signal (too late)
```

By the time the reward model sees negative feedback at turn 5, turns 1-4 have already been reinforced as "good" behavior. The causal chain is invisible to single-turn optimization.

### 2.3 Policy Erosion Over Trajectories

RLHF-trained models exhibit systematic policy erosion when subjected to sustained pressure:

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

This erosion curve is the signature of RLHF's blind spot: the training signal reinforces compliance with user requests, and without trajectory-level negative feedback, the model learns to gradually accommodate rather than maintain firm boundaries.

### The Core Insight

> RLHF provides strong guarantees for single-turn interactions but provides no guarantees for multi-turn trajectories. The credit assignment problem, combined with partial observability of downstream effects, means that RLHF-aligned models can still produce harmful outcomes through sequences of individually-benign steps.

---

## 3. Red-Teaming Is Necessary but Insufficient

Red-teaming is a foundational safety practice, but it does not provide the coverage guarantees that production agentic systems require. In multi-turn contexts, the attack surface is combinatorially larger, and static attack sets quickly become obsolete as models are updated.

### 3.1 Delayed Failure Curves

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

The erosion curve is the critical diagnostic:

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

The slope of this curve is more predictive than any single-turn score. Steep erosion indicates vulnerability to decomposition-based attacks.

### 3.2 Adaptive Attackers vs Static Templates

Attack templates that are effective today become ineffective as models are trained against them. This creates an arms race that static benchmarks cannot capture:

| Red-Team Property | Production Reality |
|-------------------|-------------------|
| Fixed attack corpus | Attackers innovate continuously |
| Single-turn focus | Multi-turn attacks dominate |
| Known distribution | Distribution shift at deployment |
| Point-in-time evaluation | Continuous adversarial pressure |

Effective red-teaming must include:

- **Mutation strategies:** Systematic variation of known attack patterns
- **Feedback loops:** Adaptation based on failed attack attempts
- **Novel generation:** Discovery of attack patterns not present in training data

Static jailbreak benchmarks provide a false sense of security because they test known attacks on models that may have been trained (directly or indirectly) to defend against them.

### 3.3 Coverage vs Depth Tradeoff

Red-teaming faces a fundamental resource allocation problem:

| Strategy | Coverage | Depth | Cost |
|----------|----------|-------|------|
| Broad single-turn | High | Low | Low |
| Deep multi-turn | Low | High | High |
| Adaptive multi-turn | Medium | High | Very High |

Most organizations default to broad single-turn testing because it is cheaper and produces impressive-looking benchmark scores. This systematically underestimates risk from sophisticated attackers who are willing to invest in multi-turn strategies.

### The Core Insight

> Red-teaming that does not model attacker adaptation and multi-turn erosion produces false confidence. The goal of red-teaming should not be to "pass" a benchmark, but to discover the shape of the erosion curve and the conditions under which safeguards degrade.

---

## 4. Detection Benchmarks and Trajectory-Level Metrics

Misuse detection is a critical safeguard layer, but detection benchmarks that operate at the turn level systematically underestimate risk in agentic contexts.

### 4.1 Why Per-Turn Detectors Fail

| Detection Type | What It Measures | Blind Spots |
|----------------|------------------|-------------|
| Turn-level | Individual message safety | Decomposed attacks, context manipulation |
| Trajectory-level | Conversation-wide safety | Higher cost, noisier signals |

In empirical evaluations, trajectory-level detection catches 40-60% more attacks than turn-level detection on the same traffic. This gap widens for sophisticated attackers who deliberately decompose harmful requests.

### 4.2 Trajectory-Level Intent Drift

Intent is not static across a conversation. Attackers exploit this by starting with benign requests and gradually shifting toward harmful goals:

```
Intent Trajectory
    │
Benign ├████████
       │        ████
       │            ████
       │                ████
Harmful├                    ████████████
       └────┬────┬────┬────┬────┬────────▶
        Turn 1    2    3    4    5   Time
```

Per-turn detectors see each message in isolation and cannot detect drift patterns. Trajectory-level detection must track cumulative intent signals and flag when the trajectory crosses into high-risk territory.

| Detection Type | Intent Drift Sensitivity | Cost |
|----------------|-------------------------|------|
| Per-turn | Cannot detect | Low |
| Window-based (last 3 turns) | Partial | Medium |
| Full trajectory | High | High |

### 4.3 Erosion Curves and Delayed Violation Rate

The **delayed violation rate (DVR)** measures what fraction of violations occur after turn N:

| Threshold | DVR | Implication |
|-----------|-----|-------------|
| Turn 1 | 95% | Single-turn detection misses 95% of violations |
| Turn 3 | 72% | 3-turn window still misses majority |
| Turn 5 | 38% | 5-turn window catches most but is expensive |

The **erosion curve** plots safety score degradation over turns. Steep erosion curves indicate models vulnerable to sustained pressure:

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

### 5.1 Pre-Action, Post-Action, and Trajectory-Level Hooks

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

### 5.2 Escalation Policies

| Hook | What It Catches | Latency Impact | False Positive Risk |
|------|-----------------|----------------|---------------------|
| Pre-action | Direct attacks, prompt injection | Low | Medium |
| Mid-trajectory | Drift, erosion, intent shift | Medium | Low |
| Post-action | Output policy violations | Low | Medium |

Mid-trajectory monitoring is the highest-leverage intervention point but is the least commonly implemented. It requires maintaining state across turns and computing cumulative risk scores, which adds complexity and latency.

### 5.3 Human-in-the-Loop Recovery

When automated safeguards detect high-risk situations, human review becomes the final line of defense. Effective human-in-the-loop systems require:

**Triage efficiency:** Human reviewers cannot examine every flagged interaction. Prioritization by severity and confidence is essential:

| Priority | Criteria | Response Time |
|----------|----------|---------------|
| P0 | High severity + high confidence | < 1 hour |
| P1 | High severity + medium confidence | < 4 hours |
| P2 | Medium severity + any confidence | < 24 hours |
| P3 | Low severity, flagged for learning | Weekly batch |

**Context preservation:** Reviewers must see the full trajectory, not just the flagged message. Without context, review decisions are unreliable.

**Feedback integration:** Human decisions must feed back into detector training and threshold calibration. Otherwise, human review becomes a terminal operation with no systemic benefit.

| Escalation Action | User Impact | Safety | Operational Cost |
|-------------------|-------------|--------|------------------|
| Silent log | None | Low | Low |
| Soft warning | Low | Medium | Low |
| Require confirmation | Medium | Medium | Medium |
| Block and explain | High | High | Medium |
| Human review queue | High | Highest | High |

### The Core Insight

> Safeguards must be embedded in the agent loop, not applied as post-hoc filters. The highest-leverage intervention point is mid-trajectory monitoring, which can detect drift and erosion before harm crystallizes. Human-in-the-loop review is essential for high-severity cases but requires careful triage to remain operationally viable.

---

## 6. Production Reality: Evaluation Infrastructure and Cost Constraints

Safety evaluation at scale introduces operational constraints that fundamentally shape what is feasible. Research-grade evaluation methodologies often fail to transfer to production because they assume unlimited compute, instant evaluation, and perfect observability.

### 6.1 Batch vs Streaming Evaluation

Production safety evaluation operates in two modes with different tradeoffs:

| Mode | Latency | Coverage | Use Case |
|------|---------|----------|----------|
| **Streaming** | Real-time | Sampled | Live intervention |
| **Batch** | Hours-days | Full | Regression detection |

Streaming evaluation must fit within request latency budgets (typically < 100ms additional). This limits complexity and forces sampling. Batch evaluation can be exhaustive but only catches problems after they occur.

Effective production systems use both:
- Streaming for real-time blocking of high-confidence threats
- Batch for comprehensive regression detection and trend analysis

### 6.2 Drift Detection

Model behavior drifts over time due to:
- API version updates
- Fine-tuning adjustments
- Context window changes
- Traffic distribution shifts

Drift detection requires continuous monitoring against stable baselines:

| Metric | Baseline Source | Alert Threshold |
|--------|-----------------|-----------------|
| Refusal rate | Last 7-day average | ±5% |
| Safety score distribution | Frozen test set | KL divergence > 0.1 |
| Erosion curve slope | Historical models | Steeper by >10% |

### 6.3 Cost-Aware Safety Coverage

| Component | Cost per 1K Evals |
|-----------|-------------------|
| Model API calls | $2-20 |
| Worker compute | $0.50-2.00 |
| Storage | $0.10-0.50 |
| Network | $0.05-0.20 |
| **Total** | **$2.65-22.70** |

At production scale (millions of interactions), cost determines which evaluations are feasible. Full trajectory-level evaluation of all traffic is typically not economical; sampling and prioritization are required.

Cost-aware prioritization:
- **High-risk signals**: Full evaluation (flagged users, sensitive topics)
- **Medium-risk signals**: Sampled evaluation (new users, edge cases)
- **Low-risk signals**: Batch evaluation only (established users, benign topics)

### 6.4 Backpressure and Graceful Degradation

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

## 10. Threat Model

We consider both adversarial misuse and benign failure modes in deployed agentic systems.

### 10.1 Adversary Capabilities

| Capability | Description | Sophistication |
|------------|-------------|----------------|
| Adaptive multi-turn prompting | Iterative refinement based on model responses | Medium |
| Context manipulation | Exploiting long conversation histories | Medium |
| Tool affordance exploitation | Using agent tools for unintended purposes | High |
| Slow policy erosion | Gradual boundary-pushing over many turns | High |

Adversaries range from casual users testing limits to sophisticated attackers with specific misuse goals. The key insight is that sophisticated attackers prefer slow erosion to direct attacks because erosion is harder to detect.

### 10.2 Model Misuse vs Benign Failure

| Failure Type | Intent | Detection Difficulty | Mitigation |
|--------------|--------|---------------------|------------|
| Adversarial misuse | Malicious | High | Intent modeling, trajectory analysis |
| Benign failure | None | Medium | Better training, guardrails |
| Accidental harm | None | Low | Output filtering |

Both adversarial and benign failures matter. A system that only defends against malicious actors will still produce harmful outputs through misalignment or edge cases.

### 10.3 Operational Threat Surfaces

Beyond model behavior, operational failures create attack surfaces:

- **Stale system prompts:** Outdated safety instructions after model updates
- **Disabled checks:** Safeguards turned off during debugging and not re-enabled
- **Configuration drift:** Production settings diverging from tested configurations
- **Alert fatigue:** Critical signals ignored due to noise

### 10.4 Out of Scope

This framework does not address:
- Fully malicious insiders with privileged infrastructure access
- Attacks requiring direct model weight tampering
- Nation-state scale adversaries with unlimited resources
- Hardware-level attacks on inference infrastructure

---

## 11. Limitations

Our framework emphasizes detection and mitigation of silent regressions in agentic safety, but it has several limitations:

### 11.1 Coverage Limits

No benchmark suite can exhaustively enumerate all future misuse patterns. Trajectory-level evaluation improves coverage but does not eliminate blind spots. Novel attack vectors will succeed until they are observed, analyzed, and incorporated into defenses.

### 11.2 Cost Constraints

Continuous multi-turn evaluation is expensive. Organizations must trade off between coverage depth and operational cost, which may leave some risk surfaces under-monitored.

| Coverage Level | Cost | Risk Accepted |
|----------------|------|---------------|
| Full trajectory, all traffic | Very High | Minimal |
| Sampled trajectory | Medium | Some edge cases |
| Turn-level only | Low | Multi-turn attacks |

### 11.3 Simulation Fidelity

Synthetic red-teaming and replayed incidents may fail to capture emergent behaviors seen only under real user incentives and production-scale usage. Evaluation environments are necessarily simplified.

### 11.4 Organizational Dependence

The effectiveness of regression gating and incident feedback loops depends on governance. Without binding policies, technical signals can be overridden. A safety system is only as strong as the organizational commitment to enforce it.

### 11.5 Adversarial Adaptation

Attackers learn. Defenses that are effective today may be bypassed tomorrow. Static benchmarks become obsolete as attackers discover and share new techniques.

### The Core Insight

> No safeguards system can guarantee safety. The goal is not zero incidents, but bounded risk, early detection, fast response, and continuous improvement. A system that learns faster than attackers adapt is winning, even if incidents occur.

---

## 12. Future Work

Several directions remain open for advancing safety evaluation in agentic systems:

### 12.1 Learning-Based Trajectory Evaluators

Current trajectory evaluation relies heavily on hand-engineered heuristics. Developing learned evaluators that reason over long-horizon interactions could improve detection of subtle failure modes.

### 12.2 Automated Adversary Generation

Move beyond template-based red-teaming toward adaptive agents that co-evolve with deployed safeguards. This requires:
- Reinforcement learning for attack policy optimization
- Diversity mechanisms to avoid mode collapse
- Transfer learning across model versions

### 12.3 Safety-Aware Training Loops

Integrate regression failures and incident replays directly into training pipelines, closing the loop between evaluation and model improvement. This requires solving:
- Credit assignment for delayed failures
- Sample efficiency for rare failure modes
- Avoiding Goodhart's law on safety metrics

### 12.4 Human Factors Research

Study how organizational incentives, review processes, and incident triage workflows affect long-term safety outcomes. Technical safety research must be complemented by organizational safety research.

### 12.5 Cross-Organization Benchmarks

Develop shared, versioned regression suites across organizations to reduce safety metric gaming and improve external accountability. This requires:
- Governance for benchmark updates
- Mechanisms to prevent overfitting
- Incentive alignment for honest reporting

---

## 13. Conclusion

Safety failures in deployed agentic systems are rarely the result of a single catastrophic decision. They emerge gradually through compounding interactions, partial observability, and organizational pressures that degrade safeguards over time. Single-turn benchmarks and one-off red-teaming exercises provide a false sense of security because they do not measure the failure modes that dominate real-world risk: delayed violations, policy erosion across trajectories, and silent regressions across releases.

We argue that safety must be treated as a **non-regression invariant** enforced through production-grade evaluation infrastructure. This requires:

1. **Trajectory-level metrics** that capture multi-turn failure modes
2. **Safeguards embedded directly in agent loops** at pre-action, mid-trajectory, and post-action points
3. **Regression-based release gating** with statistical rigor and OK/WARN/BLOCK verdicts
4. **Closed feedback loops** from incidents to permanent regression tests

Finally, we highlight that technical solutions alone are insufficient. Organizational incentives, governance structures, and ownership models strongly shape whether safety signals are binding or merely advisory. Without aligning incentives and accountability, even well-designed safety systems will erode in practice.

> **Safety is not a static property of a model; it is an operational property of the organization that deploys it.**

We frame safety not as a pre-release checklist but as an emergent property of an end-to-end deployment system. Effective safety engineering requires treating safety signals as binding operational constraints, integrating them into release processes, and continuously updating evaluation coverage based on real-world incidents.

The goal is not zero incidents—that is unachievable. The goal is a system that learns faster than attackers adapt and faster than organizational memory decays.

---

## Appendix

### A. Portfolio Mapping

| Whitepaper Section | Repository |
|--------------------|------------|
| Introduction | when-rlhf-fails-quietly |
| RLHF Limitations | when-rlhf-fails-quietly |
| Red-Teaming | safeguards-stress-tests |
| Detection Benchmarks | agentic-misuse-benchmark |
| Safeguards in Loop | agentic-safeguards-simulator |
| Production Infrastructure | scalable-safeguards-eval-pipeline |
| Release Gating | model-safety-regression-suite |
| Incident Response | agentic-safety-incident-lab |
| Research Communication | safety-memos |

### B. Design Principles Summary

1. **Trajectory-First Evaluation:** Evaluate conversation trajectories, not individual messages
2. **Regression Before Release:** No model ships without regression testing against known failures
3. **Incidents Become Tests:** Every incident generates a regression test
4. **Governance Is Part of Safety:** Technical safeguards without organizational enforcement are theater
5. **Metrics Must Be Game-Resistant:** Use held-out test sets and out-of-distribution probes
6. **Assume Safeguards Will Be Bypassed:** Design for defense in depth
7. **Prefer False Positives to False Negatives:** In safety-critical contexts, blocking is recoverable
8. **Learn Faster Than Attackers Adapt:** The goal is a learning system, not a perfect system

### C. Reproducibility

- All evaluation code uses fixed random seeds for reproducibility
- Model API responses are inherently stochastic; results may vary across API versions
- Hardware: Results generated on Apple M1 Pro, 16GB RAM
- See per-repository `docs/reproducibility.md` for detailed reproduction instructions

### D. Negative Results

| Approach | Outcome | Learning |
|----------|---------|----------|
| Single-turn proxy metrics | Correlation < 0.3 with trajectory outcomes | Trajectory-level evaluation required |
| Reward model scores as safety signals | Decorrelate in multi-turn contexts | RM blind spots are systematic |
| Keyword blocklists | Trivially bypassed via paraphrase | Semantic understanding required |
| Fixed escalation thresholds | Gamed by attackers and operators | Adaptive thresholds necessary |
| User reputation scoring | Defeated by Sybil attacks | Per-session assessment more robust |

---

*This whitepaper synthesizes learnings from building production-oriented safety evaluation and safeguards systems for agentic AI. It represents the author's current understanding and is expected to evolve as the field matures.*
