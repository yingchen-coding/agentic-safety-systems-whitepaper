# Negative Results: What We Tried That Did Not Work

Honest documentation of failed approaches is essential for scientific integrity and helps others avoid repeating unsuccessful experiments. This document catalogs approaches that seemed promising but failed to deliver expected results.

---

## 1. Single-Turn Proxy Metrics for Trajectory Safety

**Hypothesis:** Single-turn safety scores could predict trajectory-level outcomes.

**Experiment:** Computed turn-1 safety scores and correlated with trajectory-level violation rates.

**Result:**
- Correlation coefficient: r = 0.23 (p < 0.05)
- Turn-1 scores explained only 5% of trajectory variance

**Why It Failed:**
- Trajectory-level failures emerge from cumulative context
- Early turns are often benign; harm crystallizes later
- Single-turn metrics cannot capture intent drift

**Lesson:** Trajectory-level evaluation is not optional—it is required for meaningful safety coverage.

---

## 2. Reward Model Scores as Safety Signals

**Hypothesis:** RLHF reward model scores could serve as runtime safety signals.

**Experiment:** Used RM scores to detect unsafe trajectories in real-time.

**Result:**
- RM scores decorrelated from safety outcomes after turn 3
- False negative rate: 45% on multi-turn attacks
- RM optimized for helpfulness, not safety

**Why It Failed:**
- Reward models are trained on single-turn preferences
- RMs cannot assign credit across turns
- Goodhart's law: optimizing RM ≠ optimizing safety

**Lesson:** Dedicated safety classifiers outperform repurposed reward models for safety detection.

---

## 3. Keyword and Regex Blocklists

**Hypothesis:** Blocklists could catch obvious harmful content efficiently.

**Experiment:** Deployed keyword blocklist as first-line defense.

**Result:**
- Initial block rate: 78% of known bad patterns
- After 2 weeks: 23% (attackers paraphrased)
- False positive rate: 12% (blocked legitimate use)

**Why It Failed:**
- Trivially bypassed via paraphrase, encoding, or decomposition
- Blocklists cannot generalize to novel patterns
- Maintenance burden grows faster than coverage

**Lesson:** Keyword blocking is a useful first filter but cannot be the primary defense. Semantic understanding is required.

---

## 4. Fixed Escalation Thresholds

**Hypothesis:** Static thresholds could provide consistent escalation policy.

**Experiment:** Set fixed confidence thresholds for block/warn/allow decisions.

**Result:**
- Initial performance: 85% precision, 72% recall
- After 3 months: 71% precision, 58% recall
- Operators gamed thresholds; attackers learned boundaries

**Why It Failed:**
- Attackers probe thresholds and stay just below
- Teams tune prompts to pass specific thresholds
- Distribution shift invalidates calibration

**Lesson:** Adaptive thresholds with periodic recalibration are necessary. Fixed thresholds become gaming targets.

---

## 5. User Reputation Scoring

**Hypothesis:** Historical user behavior could predict future risk.

**Experiment:** Built reputation system to downweight trusted users' alerts.

**Result:**
- Defeated by new accounts (Sybil attacks)
- Account sharing undermined individual reputation
- Privacy concerns limited data retention

**Why It Failed:**
- New accounts have no history to score
- Reputation can be "farmed" with benign activity
- Users complained about differential treatment

**Lesson:** Per-session risk assessment is more robust than cross-session reputation. Trust must be re-earned each interaction.

---

## 6. Prompt-Only Safety Mitigations

**Hypothesis:** System prompt instructions alone could enforce safety policies.

**Experiment:** Added detailed safety instructions to system prompts.

**Result:**
- Effective against naive attacks
- Bypassed by prompt injection in 34% of cases
- Erosion over long conversations: 67% eventual compliance with harmful requests

**Why It Failed:**
- Prompt instructions compete with user instructions
- Long contexts dilute system prompt influence
- Jailbreaks can override prompt-level constraints

**Lesson:** Prompt-level safety is necessary but not sufficient. External safeguards with enforcement authority are required.

---

## 7. Majority Vote Ensemble Detection

**Hypothesis:** Ensembling multiple detectors would improve accuracy.

**Experiment:** Combined 5 different detectors with majority vote.

**Result:**
- Improved precision: +8%
- Degraded recall: -12% (disagreement → allow)
- Latency increased 5x

**Why It Failed:**
- Conservative detectors dominated voting
- Edge cases consistently produced disagreement
- Latency made real-time deployment impractical

**Lesson:** Ensembles help for precision but hurt recall in safety contexts. For safety, prefer high-recall individual detectors with human review for low-confidence cases.

---

## 8. Automated Threshold Optimization

**Hypothesis:** ML-based threshold optimization would outperform manual tuning.

**Experiment:** Used Bayesian optimization to tune detection thresholds.

**Result:**
- Overfitted to validation set within 50 iterations
- Production performance degraded after deployment
- Teams learned to game the optimizer

**Why It Failed:**
- Optimizer maximized proxy metrics, not safety
- Validation set didn't represent production distribution
- Feedback loop created adversarial dynamics

**Lesson:** Human-in-the-loop threshold setting with explicit audit trails is more robust than automated optimization for safety-critical decisions.

---

## 9. LLM-as-Judge for Safety Evaluation

**Hypothesis:** LLMs could reliably judge safety of other LLM outputs.

**Experiment:** Used GPT-4 to evaluate safety of model outputs.

**Result:**
- Agreement with human labels: 76%
- Systematic blind spots on subtle harm
- Cost prohibitive at scale ($15/1000 evals)

**Why It Failed:**
- LLM judges share blind spots with LLM subjects
- Judges can be manipulated by adversarial outputs
- Cost doesn't scale to production volumes

**Lesson:** LLM-as-judge is useful for research but not reliable for production safety decisions. Human review remains necessary for high-stakes cases.

---

## 10. Daily Regression Testing

**Hypothesis:** Daily regression runs would catch safety degradation quickly.

**Experiment:** Ran full regression suite every 24 hours.

**Result:**
- Alert fatigue within 2 weeks
- Teams stopped investigating non-critical alerts
- Flaky tests undermined trust in results

**Why It Failed:**
- Too frequent → alert fatigue
- No prioritization → all alerts treated equally
- Flakiness → false positives erode trust

**Lesson:** Event-driven regression (on model/config changes) with statistical significance testing is more effective than time-based scheduling.

---

## Summary Table

| Approach | Expected Outcome | Actual Outcome | Key Learning |
|----------|-----------------|----------------|--------------|
| Single-turn proxy metrics | Predict trajectory risk | r = 0.23 correlation | Trajectory eval required |
| Reward model as safety signal | Runtime detection | 45% FN rate | Dedicated classifiers needed |
| Keyword blocklists | Block harmful content | Bypassed in 2 weeks | Semantic understanding required |
| Fixed thresholds | Consistent escalation | Gamed by all parties | Adaptive thresholds needed |
| User reputation | Risk stratification | Sybil attack vulnerable | Per-session assessment better |
| Prompt-only safety | Enforce policies | 67% eventual erosion | External safeguards required |
| Majority vote ensemble | Better accuracy | -12% recall | High-recall single detectors better |
| Automated threshold tuning | Optimal performance | Overfitted and gamed | Human-in-loop more robust |
| LLM-as-judge | Scalable evaluation | 76% agreement, high cost | Human review still needed |
| Daily regression | Fast detection | Alert fatigue | Event-driven + significance testing |

---

## Why Document Negative Results?

1. **Prevents repetition:** Others can avoid failed approaches
2. **Builds credibility:** Shows intellectual honesty
3. **Improves understanding:** Failures reveal assumptions
4. **Guides research:** Points to open problems

---

*Failure is not the opposite of success—it is part of success. Every failed approach teaches us something about the problem space.*
