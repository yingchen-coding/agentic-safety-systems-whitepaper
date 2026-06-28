# 30-Minute Interview Narrative

This document provides a structured narrative for walking an interviewer through the portfolio in a 30-minute technical interview.

---

## Opening (2 minutes)

**Hook:**

> "I built a closed-loop safety system for agentic AI. The core insight is that safety failures in production are mostly silent regressions and organizational bypasses—not single-turn jailbreaks. I'll show you how I detect, gate, and learn from these failures."

**Framing:**

> "This portfolio has 8 repositories that form a connected system. I'll walk you through the pipeline: from understanding failure modes, to detection, to release gating, to incident feedback."

---

## Part 1: The Problem (5 minutes)

**Key point:** Single-turn safety benchmarks systematically underestimate risk.

**Evidence to cite:**

> "In my experiments, 95% of safety violations occur after turn 1. Single-turn benchmarks only catch the 5% that fail immediately. The remaining 95% require trajectory-level evaluation."

**Visual:** Erosion curve

```
Safety Score
    │
1.0 ├████
    │    ████
    │        ████
    │            ████
    │                ████
0.5 ├                    ████
    │
0.0 ├────┬────┬────┬────┬────▶
    Turn 1    2    3    4    5
```

> "This is a policy erosion curve. The model starts with strong refusals, but erodes under sustained pressure. No single turn crosses a policy line—the trajectory does."

**Transition:**

> "So the question becomes: how do we detect these trajectory-level failures before they cause harm in production?"

---

## Part 2: Detection and Red-Teaming (5 minutes)

**Key point:** Detection must operate at the trajectory level.

**Repos to reference:**
- `agentic-misuse-benchmark` (trajectory-level detection)
- `safety-harness/stress-testing` (adaptive red-teaming)

**Evidence to cite:**

> "Trajectory-level detection catches 40-60% more attacks than per-turn detection on the same traffic. The gap widens for sophisticated attackers who decompose harmful requests across turns."

**Key metric:** Delayed Violation Rate (DVR)

> "I introduced a metric called Delayed Violation Rate—what fraction of violations occur after turn N. For most models, DVR at turn 1 is over 90%. That means turn-level detectors miss 90%+ of violations."

**Transition:**

> "But detection alone isn't enough. Safeguards need to be embedded in the agent loop, not bolted on as external filters."

---

## Part 3: Safeguards in the Loop (3 minutes)

**Key point:** Intervention placement matters.

**Repo to reference:** `safety-harness/simulator`

**Visual:** Agent loop with hooks

```
User Input → [PRE-ACTION] → Planner → [MID-TRAJECTORY] → Executor → [POST-ACTION] → Response
```

> "There are three intervention points: pre-action, mid-trajectory, and post-action. Mid-trajectory is highest leverage because it can detect drift before harm crystallizes—but it's also the least commonly implemented because it requires maintaining state across turns."

**Transition:**

> "Once we have detection and safeguards, the question is: how do we prevent safety from degrading across releases?"

---

## Part 4: Release Gating (5 minutes)

**Key point:** Safety must be a non-regression invariant.

**Repo to reference:** `safety-harness/regression-suite`

**Evidence to cite:**

> "I observed that safety erodes silently across releases. Each version passes absolute thresholds, but after four releases, safety has degraded by 3 percentage points—and no individual release was flagged."

**Solution:**

> "I implemented regression gating with three verdicts: OK, WARN, and BLOCK. The gating uses statistical significance—bootstrap confidence intervals and permutation tests—so we don't block on noise."

**Visual:** Regression pipeline

```
Baseline Model → Candidate Model → Statistical Comparison → OK / WARN / BLOCK
```

**If asked about implementation:**

> "The system produces HTML reports for human review, exit codes for CI/CD integration, and root cause attribution when regressions are detected."

**Transition:**

> "But even with gating, incidents will happen. The question is whether incidents become durable improvements or are forgotten."

---

## Part 5: Incident → Regression Loop (5 minutes)

**Key point:** Incidents must become permanent regression tests.

**Repo to reference:** `safety-harness/incident-lab`

**Evidence to cite:**

> "Without regression tests, the same vulnerability can be reintroduced by future model updates. Organizational memory is lost when team members change. Safety debt accumulates invisibly."

**Solution:**

> "I built an incident-to-regression pipeline: triage → root cause analysis → blast radius estimation → auto-generate regression test → integrate into release gate."

**Key metric:** Learning velocity

> "I track 'incident to regression time'—how fast we convert an incident into a permanent test. Target is under one week. I also track recurrence rate—what fraction of incidents have a previously-seen root cause. Target is under 10%."

**Transition:**

> "Finally, I want to acknowledge that technical safeguards aren't enough. Organizational factors matter."

---

## Part 6: Organizational Failure Modes (3 minutes)

**Key point:** Many safety failures are organizational, not technical.

**Evidence to cite:**

> "I documented four organizational anti-patterns that cause safety degradation: velocity vs safety misalignment, metric gaming, alert fatigue, and exception creep."

**Example:**

> "Exception creep: First exception is 'critical launch, we'll fix after.' Second exception is 'we did it before, it was fine.' Third exception is 'this is just how we operate.' Eventually exceptions become the default."

**Solution:**

> "I propose governance mechanisms: exception budgets per quarter, escalating approval for repeated exceptions, and public exception logs. The goal is to make safety bypasses visible and accountable."

---

## Closing (2 minutes)

**Summary:**

> "To summarize: I built a closed-loop system that detects trajectory-level failures, gates releases on safety regressions, and converts incidents into permanent tests. The key insight is that safety is not a model property—it's a system property that includes evaluation methodology, release process, and organizational governance."

**Differentiation:**

> "This work differs from typical safety research in three ways:
> 1. Trajectory-level, not turn-level evaluation
> 2. Continuous regression gating, not one-time benchmarks
> 3. Organizational failure modes as first-class concerns"

**Call to action:**

> "I'm looking for roles where I can apply this framework to real production systems. I'm particularly interested in building safety evaluation infrastructure, designing regression gating systems, and advising on safety governance."

---

## Prepared Q&A

### Q: How does this compare to Anthropic's Constitutional AI?

> "Constitutional AI is training-time alignment; I focus on deployment-time safeguards. They're complementary—Constitutional AI reduces baseline failure rates; my system catches failures that remain and prevents regressions across releases."

### Q: What's the cost of trajectory-level evaluation?

> "Full trajectory evaluation of all traffic is not economical at scale. I use cost-aware prioritization: high-risk signals get full evaluation, medium-risk gets sampled, low-risk gets batch evaluation only. The key is making coverage decisions explicit."

### Q: How do you handle false positives?

> "False positives are a real concern. I implemented tiered escalation: silent log → soft warning → require confirmation → block → human review. The escalation level depends on detection confidence and potential severity. I also track FP rates and use them to calibrate thresholds."

### Q: What can't this system solve?

> "Three things: novel attacks we haven't imagined, distribution shift to new domains, and lack of organizational will. No technical system can force an organization to prioritize safety. My system assumes organizational commitment as a prerequisite."

### Q: Why did you build this?

> "I observed a gap between safety research and production reality. Research assumes unlimited compute, instant evaluation, and cooperative users. Production has cost constraints, latency budgets, and adversarial traffic. I wanted to bridge that gap."
