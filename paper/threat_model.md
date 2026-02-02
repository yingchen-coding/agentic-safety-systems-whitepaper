# Threat Model: Agentic Safety Systems

This document formalizes the threat model for agentic AI safety systems. Unlike typical security threat models, this model explicitly includes organizational and metric-gaming threats—failure modes often omitted from technical safety research.

---

## 1. Assets

What we are protecting:

| Asset | Description | Compromise Impact |
|-------|-------------|-------------------|
| **User Intent Integrity** | User's actual goals vs what agent executes | Agent acts against user interests |
| **Safety Policy Compliance** | Adherence to harm prevention policies | Harmful outputs produced |
| **Tool Invocation Boundaries** | Limits on what tools can do | Unauthorized actions executed |
| **Audit Logs & Telemetry** | Record of agent actions | Loss of accountability, hidden violations |
| **Evaluation Integrity** | Trustworthiness of safety metrics | False confidence in unsafe models |
| **Regression Test Suite** | Coverage of known failure modes | Reintroduction of fixed vulnerabilities |

---

## 2. Adversaries

Who might attack the system:

### 2.1 External Adversaries

| Adversary | Capability | Motivation | Persistence |
|-----------|------------|------------|-------------|
| **Opportunistic User** | Low-skill prompt manipulation | Curiosity, minor misuse | Low |
| **Skilled Prompt Engineer** | Multi-turn attacks, context manipulation | Targeted misuse, content generation | Medium |
| **Adaptive Red-Team Attacker** | Learns from detector feedback, evolves strategies | Bypass defenses, demonstrate vulnerabilities | High |
| **Automated Attack System** | High-volume, automated probing | Scale attacks, find edge cases | High |

### 2.2 Internal Adversaries

| Adversary | Capability | Motivation | Detection Difficulty |
|-----------|------------|------------|---------------------|
| **Metric-Gaming Team** | Modify prompts, select scenarios | Pass release gate without improving safety | High |
| **Negligent Operator** | Disable safeguards, ignore alerts | Ship faster, reduce friction | Medium |
| **Compromised Insider** | Access to eval data, thresholds | Sabotage safety systems | Very High |

---

## 3. Adversary Capabilities

What adversaries can do:

### 3.1 Prompt-Level Capabilities

- **Multi-turn prompt crafting:** Build context over many turns to erode policy
- **Persona manipulation:** Adopt roles that lower safety barriers
- **Decomposition attacks:** Break harmful requests into benign-looking steps
- **Encoding/obfuscation:** Use alternate representations to bypass filters

### 3.2 System-Level Capabilities

- **Detector probing:** Observe block/allow decisions to learn detector behavior
- **Context manipulation:** Exploit conversation history and memory
- **Tool hallucination induction:** Trick agent into invoking tools incorrectly
- **Traffic poisoning:** Inject adversarial samples into evaluation data (limited)

### 3.3 Organizational Capabilities

- **Benchmark selection:** Choose scenarios that models perform well on
- **Threshold manipulation:** Adjust thresholds to pass gates
- **Exception abuse:** Use override mechanisms to bypass blocks
- **Alert fatigue exploitation:** Generate noise to hide real violations

---

## 4. Trust Boundaries

Where trust transitions occur:

```
┌─────────────────────────────────────────────────────────────────┐
│                        TRUST BOUNDARIES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   [UNTRUSTED]              [BOUNDARY]              [TRUSTED]     │
│                                                                  │
│   User Input        →    Intent Classifier    →    Planner      │
│   User History      →    Context Validator    →    Memory       │
│   Tool Responses    →    Output Verifier      →    Agent State  │
│   Production Logs   →    PII Redaction        →    Eval Data    │
│   Model Outputs     →    Policy Checker       →    Response     │
│   External APIs     →    Response Validator   →    Tool Results │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Critical Trust Transitions

| Boundary | Risk if Violated | Control |
|----------|-----------------|---------|
| User → Agent | Prompt injection, intent manipulation | Pre-action classifier |
| Agent → Tools | Unauthorized actions, data exfiltration | Tool permission system |
| Tools → Agent | Poisoned responses, hallucination triggers | Response validation |
| Production → Eval | Data poisoning, metric corruption | Distribution fingerprinting |
| Eval → Release | Gaming, false confidence | Anti-gaming detector |

---

## 5. Attack Surfaces

Where attacks can occur:

### 5.1 Runtime Attack Surfaces

| Surface | Attack Vector | Example |
|---------|--------------|---------|
| **Prompt Channel** | Injection, jailbreak, erosion | "Ignore previous instructions..." |
| **Conversation History** | Context poisoning | Inject fake assistant messages |
| **Tool Interfaces** | Hallucination, misuse | "Call delete_all_files()" |
| **Memory Store** | Long-term context manipulation | Persist malicious instructions |
| **System Prompts** | Prompt leakage, override | Extract and modify system prompt |

### 5.2 Evaluation Attack Surfaces

| Surface | Attack Vector | Example |
|---------|--------------|---------|
| **Test Scenarios** | Overfitting, memorization | Train on test set |
| **Thresholds** | Gaming, threshold hugging | Optimize to just pass |
| **Metrics** | Goodhart's law, proxy gaming | Improve metric, not safety |
| **Telemetry** | Data poisoning | Inject false positives/negatives |
| **Human Review** | Alert fatigue, override abuse | Flood with low-priority alerts |

---

## 6. Failure Modes

How the system can fail:

### 6.1 Model-Level Failures

| Failure Mode | Description | Detection |
|--------------|-------------|-----------|
| **Immediate Jailbreak** | Direct policy violation on first turn | Turn-level detector |
| **Delayed Policy Erosion** | Gradual compliance degradation | Trajectory monitor |
| **Intent Drift** | Shift from benign to harmful goal | Intent tracker |
| **Tool Misuse** | Using tools for unintended purposes | Action validator |
| **Hallucination** | Fabricating tool calls or responses | Output verifier |

### 6.2 System-Level Failures

| Failure Mode | Description | Detection |
|--------------|-------------|-----------|
| **Safeguard Bypass** | Circumventing safety checks | Audit log analysis |
| **Escalation Delay** | Late escalation to human review | Latency monitoring |
| **Alert Fatigue** | Operators ignore real threats | Alert audit |
| **Silent Regression** | Safety degradation across releases | Regression gating |
| **Coverage Decay** | Test suite becomes stale | Decay manager |

### 6.3 Organizational Failures

| Failure Mode | Description | Detection |
|--------------|-------------|-----------|
| **Metric Gaming** | Optimize benchmark, not safety | Gaming detector |
| **Exception Creep** | Overrides become default | Exception audit |
| **Ownership Gap** | No one responsible for FNs | RACI enforcement |
| **Velocity Pressure** | Safety traded for ship date | Process audit |

---

## 7. Non-Goals

What we explicitly do not try to achieve:

| Non-Goal | Rationale |
|----------|-----------|
| **Prevent all misuse** | Impossible; focus on bounded risk |
| **Perfect detection** | Accept FN/FP tradeoff; optimize for recall |
| **Zero false positives** | Would require accepting dangerous FNs |
| **Eliminate human review** | Human judgment needed for edge cases |
| **Static defense** | Attackers adapt; defense must evolve |

---

## 8. Assumptions

What we assume to be true:

| Assumption | If Violated |
|------------|-------------|
| **Partial observability** | Full observability would simplify detection |
| **Bounded compute** | Unlimited compute would allow exhaustive eval |
| **Limited human oversight** | Unlimited human review would catch more |
| **Attackers adapt** | Static attackers would be easier to defend |
| **Organizational friction** | Perfect org alignment would simplify governance |
| **Model behavior is stochastic** | Deterministic models would be easier to test |

---

## 9. Security Controls Mapped to Threats

| Threat | Control | Implementation |
|--------|---------|----------------|
| Prompt injection | Pre-action intent classifier | `safeguards-simulator/pre_action/` |
| Delayed policy erosion | Trajectory-level monitors | `misuse-benchmark/detectors/intent_tracker.py` |
| Adaptive attacks | Adaptive red-teaming | `misuse-benchmark/attackers/adaptive_attacker.py` |
| Metric gaming | Gaming detector | `regression-suite/core/gaming_detector.py` |
| Silent regression | Release gating | `regression-suite/core/diff.py` |
| Incident recurrence | Incident → regression loop | `incident-lab/core/replay.py` |
| Coverage decay | Decay manager | `incident-lab/core/decay.py` |
| Near-miss blindness | Near-miss detector | `incident-lab/core/near_miss.py` |
| Tool misuse | Post-action validator | `safeguards-simulator/post_action/` |
| Alert fatigue | Tiered alerting | `eval-pipeline/alerting/` |

---

## 10. Threat Prioritization

Based on likelihood × impact:

| Priority | Threat | Likelihood | Impact | Mitigation Status |
|----------|--------|------------|--------|-------------------|
| **P0** | Silent regression across releases | High | Critical | Regression gating |
| **P0** | Metric gaming | High | High | Gaming detector |
| **P1** | Delayed policy erosion | High | High | Trajectory monitors |
| **P1** | Adaptive attacks | Medium | High | Adaptive red-teaming |
| **P2** | Tool misuse | Medium | Medium | Action validators |
| **P2** | Coverage decay | Medium | Medium | Decay manager |
| **P3** | Prompt injection | Low* | Medium | Intent classifier |
| **P3** | Alert fatigue | Medium | Low | Tiered alerting |

*Low because most obvious injections are caught; sophisticated variants are captured under "adaptive attacks"

---

## 11. Open Threats

Threats without complete mitigations:

| Threat | Current Gap | Research Direction |
|--------|-------------|-------------------|
| **Novel attack vectors** | Cannot anticipate all attacks | Automated attack generation |
| **Reward hacking at scale** | RLHF blind spots | Trajectory-aware training |
| **Organizational will** | Cannot force safety priority | Incentive alignment research |
| **Distribution shift** | OOD generalization limited | Domain adaptation |
| **Coordinated insider attack** | Limited detection capability | Enhanced audit systems |

---

## 12. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-01 | Initial threat model |
| 1.1 | 2024-07 | Added organizational threats, gaming detector |
| 1.2 | 2024-08 | Added near-miss, decay management |

---

*This threat model is a living document. It should be updated as new threats emerge and defenses evolve.*
