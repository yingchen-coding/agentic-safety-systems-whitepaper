# Comparison with Existing Safety Work

This document positions our work relative to safety research from leading AI labs and academia.

---

## Anthropic

### Constitutional AI (Bai et al., 2022)

**Their focus:** Training-time alignment via principle-based self-critique and revision.

**Our focus:** Deployment-time safeguards and regression detection.

**Relationship:** Complementary. Constitutional AI reduces baseline failure rates; our system catches failures that remain post-training and prevents regressions across releases.

| Aspect | Constitutional AI | Our Work |
|--------|------------------|----------|
| Stage | Training | Deployment |
| Mechanism | Self-critique | External safeguards |
| Evaluation | Per-model | Cross-release regression |
| Failure mode addressed | Misalignment | Silent regression |

### Sleeper Agents (Hubinger et al., 2024)

**Their focus:** Demonstrating that deceptive behavior can persist through safety training.

**Our focus:** Detecting behavioral drift and erosion in deployed systems.

**Relationship:** Our trajectory-level monitoring could detect the kind of delayed, context-dependent failures that sleeper agents exhibit.

---

## OpenAI

### Preparedness Framework (OpenAI, 2023)

**Their focus:** Pre-deployment capability assessment for dangerous capabilities (bio, cyber, persuasion, autonomy).

**Our focus:** Post-deployment regression detection and incident-driven learning.

**Relationship:** Preparedness is primarily pre-release; we extend safety governance into production with continuous monitoring and regression gating.

| Aspect | Preparedness | Our Work |
|--------|-------------|----------|
| Timing | Pre-release | Continuous |
| Scope | Capability elicitation | Behavioral regression |
| Output | Risk assessment | Release gate verdict |

### Red-Teaming Language Models (Perez et al., 2022)

**Their focus:** Automated red-teaming to discover model vulnerabilities.

**Our focus:** Integrating red-teaming into CI/CD as a binding release constraint.

**Relationship:** We operationalize red-teaming findings as regression tests rather than one-off reports.

---

## Google DeepMind

### Evaluating Social and Ethical Risks (Weidinger et al., 2023)

**Their focus:** Taxonomy of social and ethical harms from language models.

**Our focus:** Operational detection and gating of harms in deployment.

**Relationship:** We implement runtime detection for the harm categories they taxonomize.

### Scalable Agent Alignment via Reward Modeling (Stiennon et al., 2020)

**Their focus:** Training reward models from human preferences.

**Our focus:** Detecting when reward model signals fail in multi-turn contexts.

**Relationship:** We document reward model blind spots as a failure mode and propose trajectory-level alternatives.

---

## Academic Research

### HarmBench (Mazeika et al., 2024)

**Their focus:** Comprehensive single-turn jailbreak benchmark.

**Our focus:** Trajectory-level evaluation that surfaces delayed failures.

**Relationship:** HarmBench measures turn-level robustness; we measure erosion over trajectories.

| Aspect | HarmBench | Our Work |
|--------|-----------|----------|
| Evaluation unit | Single turn | Trajectory |
| Metric | Attack success rate | Delayed violation rate |
| Failure mode | Immediate jailbreak | Policy erosion |

### UIUC Red-Teaming (Zou et al., 2023)

**Their focus:** Adversarial suffix attacks demonstrating universal jailbreaks.

**Our focus:** Adaptive multi-turn attacks that simulate realistic adversaries.

**Relationship:** Their attacks are single-turn; we extend to multi-turn erosion strategies.

### Stanford HELM (Liang et al., 2022)

**Their focus:** Holistic evaluation across multiple dimensions.

**Our focus:** Longitudinal regression detection across releases.

**Relationship:** HELM provides point-in-time snapshots; we track trends over time.

---

## Summary: What Makes Our Work Different

| Dimension | Typical Safety Research | Our Work |
|-----------|------------------------|----------|
| Evaluation timing | Pre-release | Continuous |
| Evaluation unit | Single turn | Trajectory |
| Output | Report / score | Release gate verdict |
| Incident handling | Ad-hoc | Systematic → regression test |
| Organizational factors | Out of scope | First-class concern |

### Our Unique Contributions

1. **Safety as non-regression invariant:** We enforce safety longitudinally, not just at release time.

2. **Trajectory-level metrics:** We measure failure modes invisible to turn-level evaluation.

3. **Release gating integration:** Red-teaming becomes a binding CI/CD signal.

4. **Incident-to-test pipeline:** Production failures become permanent regression tests.

5. **Organizational failure modeling:** We treat governance and incentives as safety-relevant.

---

## Positioning Statement

> We do not claim to solve alignment. We assume partial alignment and focus on detecting when alignment properties degrade in deployment.
>
> We do not claim to prevent all attacks. We focus on bounded risk, fast detection, and continuous learning.
>
> We do not claim organizational problems are easy. We document failure modes and propose governance mechanisms.

Our work is complementary to training-time alignment research. We operate in the space between model release and production incidents—a space that is under-addressed in current safety literature.
