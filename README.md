#Celestial Chrorus

# Weighted Trait Gestalt NPCs  
*A composable personality-vector model for game agents*

**Status:** Design Note / White Paper (Concise)  
**Scope:** Fictional NPC behavior generation for games (dialogue, choices, reactions, quest hooks).  
**Non-goal:** Clinical psychology, diagnosis, or real-world behavioral inference.

---

## 1. Summary

This document proposes a **Weighted Trait Gestalt** model for non-player characters (NPCs).  
Each NPC is represented as a **vector of weighted traits** across multiple **trait bands** (e.g., prosocial stance, cognitive style, temperament under stress). The NPC’s moment-to-moment behavior emerges from:

1. **Stable Trait Vector** (the character’s long-term disposition)  
2. **Current State** (needs, mood, fatigue, threat level, context flags)  
3. **Context Gates** (which traits are relevant/active in the current situation)  
4. **Relationship Memory** (history with the player and other agents)

This yields consistent personalities that still adapt to circumstances, producing believable variability without scripting every edge case.

---

## 2. Core Idea: NPC as a Personality Vector

Define an NPC as a set of trait weights:

- Traits are **tags** (e.g., *Caregiver, Diplomat, Analyst, Stoic, Boundary Keeper*).
- Weights are **continuous** (e.g., 0.0–1.0 or 0–100).
- NPCs are **gestalts**: not “one archetype,” but a stable mixture that expresses differently depending on context.

**Key property:**  
> Personality is stable; behavior is personality modulated by state and context.

---

## 3. Trait Bands (Sub-vectors)

Traits are organized into **bands** so that weights remain comparable “units.”  
Bands may include:

- **Prosocial / Cooperative** (social stance)
- **Explorer / Creator / Play** (novelty and agency expression)
- **Cognitive / Epistemic Styles** (belief-formation and planning)
- **Social Power & Status Styles** (coordination, influence, hierarchy)
- **Temperament & Regulation** (stress response and self-control)
- **Attachment / Relationship Patterns** (bonding and distancing dynamics)
- **Identity / Meaning Orientation** (value/mission posture)
- **Shadow Patterns** (maladaptive but non-forensic patterns)
- **Forensic / Harm-Risk** (rare; internal controls)
- **Erotic / NSFW** (optional; gated by higher-level policy systems)

**Implementation note:** Some bands are suitable for player-facing descriptions; others (e.g., forensic) are best kept internal.

---

## 4. Weighting Rules (Normalization & Sparsity)

To avoid contradictory “everything NPCs,” enforce structure:

### 4.1 Within-band normalization
For each band `B`, weights sum to 1.0 (or 100).  
This preserves interpretability:

- `Σ w_i (in band B) = 1.0`

### 4.2 Sparsity / Top-k activation (optional but recommended)
Only the top `k` traits per band are considered “active,” the rest are near-zero.  
This prevents blended mush and improves readability.

---

## 5. Behavior Generation Model

At runtime, behaviors are selected by scoring candidate actions using trait weights, context gates, and state.

### 5.1 Notation
- `w(t)` = stable weight of trait `t`
- `g(t, C)` = context gate for trait `t` given context `C` (0..1)
- `s(t, S)` = state modulation for trait `t` given state `S` (0..1 or multiplier)
- `r(t, R)` = relationship modulation given relationship memory `R`
- `u(a)` = base utility or suitability of action `a`

### 5.2 Effective activation
For each trait `t`:

`A(t) = w(t) * g(t, C) * s(t, S) * r(t, R)`

### 5.3 Action scoring (conceptual)
For action `a`, define a trait affinity map `affinity(a, t)` (positive or negative).

`Score(a) = u(a) + Σ_t [ A(t) * affinity(a, t) ] + noise`

Then:
- pick argmax action (deterministic), or
- sample from softmax of scores (stochastic).

This separates **what the NPC is** (weights) from **what the NPC does now** (scores under gates/state).

---

## 6. Context Gating (Preventing Trait Bleed)

Not all traits should influence all situations.  
**Gates** activate relevant traits in relevant contexts:

Examples:
- *Diplomat* gate rises in negotiation/mediation scenes.
- *Protector* gate rises when allies are threatened.
- *Archivist* gate rises during lore retrieval or evidence evaluation.

Gating can be:
- rule-based (scene tags),
- learned (classifier over context),
- or hybrid.

---

## 7. Trait Interactions (Synergy & Inhibition)

Distinct “character signatures” often come from trait combinations.

Two mechanisms:

### 7.1 Synergy matrix (lightweight)
Define pairwise bonuses/penalties:

`Synergy(t_i, t_j) ∈ [-1, +1]`

Applied either to:
- trait activation (modifying `A(t)`), or
- action scoring (modifying `affinity`).

### 7.2 Threshold bundles (authorable)
If specific thresholds are met, enable a style bundle:

Example bundle:
- If *(Skeptic > 0.2 AND Teacher > 0.2 AND Mediator > 0.15)*  
  then enable “Evidence-Based De-escalator” dialogue patterns.

---

## 8. Relationship Memory (Consistency Across Time)

NPCs track relationship signals with the player/factions:
- trust, respect, fear, debt, intimacy, betrayal, gratitude, etc.

Relationship memory affects expression:
- increases/decreases `r(t, R)` for traits like *Loyalist, Boundary Keeper, People-Pleaser, Rebel*.
- influences gates (hostile contexts activate different traits than friendly ones).

---

## 9. Example: One Vector, Many Expressions

An NPC can have stable weights like:

- Prosocial: *Caregiver, Protector, Teacher, Mediator*
- Cognitive: *Systems Thinker, Strategist, Skeptic*
- Temperament: *Stoic, Resilient Grinder, Anxious Watcher*
- Relationship: *Boundary Keeper, Secure Partner, Withdrawn Observer*
- Meaning: *Duty-Bound, Mission Person, Rationalist*

Then behavior shifts with context:
- Peaceful town → Teacher/Mediator express; Protector stays latent.
- Crisis → Protector/Duty-Bound gates spike; command behaviors rise.
- Deception detected → Skeptic/Boundary Keeper activate; trust penalties apply.

Same personality. Different scene. Believable variance.

---

## 10. Content & Safety Gating (High-Level)

Some trait bands may be **policy gated** at the account/game mode level:
- NSFW archetypes only evaluated when enabled by explicit settings and consent rules.
- Forensic/harm-risk traits used as internal control signals and never surfaced directly.

This model treats gating as an upstream responsibility; the trait system remains generic.

---

## 11. Benefits

- **Composability:** Mix-and-match traits to create many distinct NPCs.
- **Consistency with flexibility:** Stable personality + situational expression.
- **Authorability:** Designers can tune weights, gates, and synergies without scripting every branch.
- **Scalability:** Works for single NPCs, factions, and population-level variation.
- **Interpretability:** Weights remain debuggable and legible.

---

## 12. Known Risks / Failure Modes

- **Trait soup:** too many nonzero weights → bland, incoherent output.  
  *Mitigation:* normalization + sparsity.
- **Over-gating:** NPC feels inconsistent across scenes.  
  *Mitigation:* keep some “always-on” baseline traits.
- **Synergy explosions:** strong pairwise bonuses create unstable behavior.  
  *Mitigation:* cap synergy contributions and test edge cases.
- **Player exploitability:** predictable traits can be gamed.  
  *Mitigation:* stochastic sampling + hidden state + relationship memory.

---

## 13. Minimal Implementation Checklist

1. Define trait bands and trait vocabulary.
2. Choose weight scale and apply normalization per band.
3. Implement context tagging and gating functions.
4. Implement state variables and state modulation.
5. Add relationship memory signals and modulation.
6. Define candidate actions and trait affinities.
7. Add optional synergy/bundles.
8. Choose deterministic vs probabilistic action selection.
9. Test for consistency, variety, and exploit resistance.

---

## 14. Conclusion

Weighted Trait Gestalt NPCs provide a practical middle ground between rigid scripting and opaque end-to-end generation. The model yields coherent personalities that adapt to context through gating and state modulation, supports author-driven tuning, and scales cleanly from individual characters to societies.

---
