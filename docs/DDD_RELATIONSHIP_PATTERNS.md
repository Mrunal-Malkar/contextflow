# DDD Context Map Relationship Patterns: Validity Matrix

This document captures the valid and invalid combinations of DDD strategic patterns on context map relationships, based on the canonical definitions from Eric Evans' [DDD Reference](https://www.domainlanguage.com/wp-content/uploads/2016/05/DDD_Reference_2015-03.pdf) (pp. 32-34), Vaughn Vernon's "Implementing Domain-Driven Design," and the [ddd-crew/context-mapping](https://github.com/ddd-crew/context-mapping) reference.

## Background: How Patterns Actually Work

The ddd-crew classification organizes context mapping along two dimensions:

1. **Team relationship** (the organizational dynamic): Mutually Dependent, Upstream/Downstream, or Free
2. **Context map patterns** (the technical/governance strategy): the specific patterns teams adopt within that relationship

ContextFlow currently models this as a **single pattern per relationship**, which conflates these concerns. This document informs the redesign.

## Relationship Categories

The top-level split is the team relationship, which determines which patterns are applicable.

### Upstream-Downstream (power differential)

One context depends on another. The upstream can succeed independently; the downstream is affected by upstream decisions.

Evans presents three **mutually exclusive** responses to this power differential (DDD Reference pp. 32-34). They form a progression based on how much influence and autonomy the downstream team has:

| Pattern | Description | When to use |
|---------|-------------|-------------|
| **Customer-Supplier** | The *collaborative* response. Downstream priorities factor into upstream planning. Teams negotiate and jointly develop acceptance tests. | When the upstream team is willing to accommodate downstream needs. The best-case U/D scenario. |
| **Conformist** | The *capitulation* response. Downstream "slavishly adheres to the model of the upstream team." No translation, no negotiating power. | When "the upstream has no motivation to provide for the downstream team's needs" (Evans). The downstream is helpless and learns to live with what it's given. |
| **Anti-Corruption Layer** | The *defensive* response. Downstream creates an isolating translation layer. | When "control or communication is not adequate to pull off a shared kernel, partner or customer/supplier relationship" (Evans). ACL is what you reach for when Customer-Supplier has failed or isn't possible. |

These three are **not combinable**:
- Customer-Supplier + ACL: if the upstream accommodates you, you don't need to defend against their model
- Customer-Supplier + Conformist: if they're accommodating you, you're not "slavishly adhering"
- ACL + Conformist: translate vs. adopt as-is are contradictory

**Separately**, the upstream may also adopt exposure patterns that describe *how* it makes its model available. These overlay any of the three relationship patterns above:

| Pattern | Description |
|---------|-------------|
| **Open Host Service (OHS)** | Upstream provides a well-defined, general-purpose protocol/API for multiple consumers |
| **Published Language (PL)** | Upstream uses a well-documented, often standardized interchange format. Frequently paired with OHS. |

OHS and PL describe the upstream's exposure mechanism, not the team dynamic. An upstream could provide OHS+PL and the downstream might still use an ACL (because the domains diverge enough to need translation), or might conform (because the published model fits well enough).

### Mutually Dependent (shared power)

No upstream/downstream distinction. Teams coordinate as equals.

| Pattern | Description |
|---------|-------------|
| **Partnership** | Mutual dependency; teams coordinate closely, succeed or fail together. Joint interface evolution. |
| **Shared Kernel** | Teams explicitly share a subset of the domain model with joint ownership and special change management. |

These patterns are **incompatible with upstream-downstream patterns**. Adding OHS/ACL/Conformist/Customer-Supplier implies a power asymmetry that contradicts the mutual relationship.

### Free (no relationship)

| Pattern | Description |
|---------|-------------|
| **Separate Ways** | Deliberate decision not to integrate. No connection exists between the contexts. |

This is the **absence** of a relationship, not a type of relationship (see "The Separate Ways Problem" below).

### Other

| Pattern | Description |
|---------|-------------|
| **Big Ball of Mud** | Demarcation of a poor-quality system. Used to mark a context whose internals are tangled, signaling that other contexts should protect themselves from its model. |

Big Ball of Mud is a property of a single context, not a relationship pattern. It often implies downstream contexts should use ACL when integrating with it.

## Valid Combinations (Upstream-Downstream)

The relationship pattern (Customer-Supplier, Conformist, or ACL) combines with optional upstream exposure patterns:

| Relationship Pattern | Upstream Exposure | Valid? | Notes |
|---------------------|------------------|--------|-------|
| Customer-Supplier | OHS | Yes | Collaborative governance with a clean API |
| Customer-Supplier | OHS + PL | Yes | Collaborative governance with formal API and standardized format |
| Customer-Supplier | PL | Yes | Collaborative governance with standardized format |
| Customer-Supplier | *(none)* | Yes | Collaborative governance, upstream exposes raw model |
| Conformist | OHS | Yes | Downstream adopts upstream's clean API as-is |
| Conformist | OHS + PL | Yes | Downstream adopts upstream's formal API/format as-is |
| Conformist | PL | Yes | Downstream adopts standard format as-is (e.g., HL7, SWIFT) |
| Conformist | *(none)* | Yes | Classic "big upstream, small downstream" (e.g., conforming to an ERP) |
| ACL | OHS | Yes | Upstream provides clean API, downstream still translates to protect its model |
| ACL | OHS + PL | Yes | Even with a great API and standard format, downstream domain needs isolation |
| ACL | PL | Yes | Standard format exists, downstream translates it |
| ACL | *(none)* | Yes | Upstream exposes raw model, downstream defends itself |

## Invalid Combinations

| Combination | Why Invalid |
|-------------|------------|
| **Customer-Supplier + ACL** | If the upstream accommodates your needs (C/S), you don't need to defend against their model (ACL). These represent opposite ends of the collaboration spectrum. |
| **Customer-Supplier + Conformist** | If the upstream accommodates your needs (C/S), you're not helplessly capitulating (Conformist). |
| **ACL + Conformist** | Mutually exclusive: either you translate (ACL) or you adopt as-is (Conformist). |
| **OHS on downstream side** | OHS describes how an upstream exposes services. A downstream doesn't "host" a service for its upstream. |
| **PL on downstream side** | Published Language describes what the upstream publishes. The downstream either conforms to it or translates it. |
| **Partnership + any U/D pattern** | Partnership is symmetric. Adding OHS/ACL/Conformist/Customer-Supplier implies power asymmetry. |
| **Shared Kernel + any U/D pattern** | Same reasoning. Shared Kernel is joint ownership; there's no upstream or downstream. |
| **Separate Ways + any pattern** | Separate Ways means no integration. Can't simultaneously have no integration and an integration pattern. |

## The "Separate Ways" Problem

Separate Ways is unique: it's not a relationship pattern, it's the **absence** of a relationship. Drawing a connection line between two contexts labeled "Separate Ways" is semantically wrong. The whole point is that these contexts have decided not to integrate.

### Options for the App

1. **Remove Separate Ways from the relationship dropdown entirely.** Represent it as the default state: two contexts with no line between them. Optionally allow users to annotate this decision (e.g., a note on a context saying "deliberately not integrated with X").

2. **Show it as a dashed/faded non-connection.** If users want to explicitly document the decision, render it as a subtle visual indicator (dashed line, very faint, or X-mark) that clearly reads as "no integration" rather than as a connection.

3. **Move it to context-level metadata.** A context could have a "Separate Ways with: [list]" field, making the deliberate decision visible without implying a connection.

## Implications for ContextFlow's Data Model

The current single-pattern model can't express combinations like "ACL with OHS upstream." To properly support DDD Context Maps, the relationship model needs:

```
Relationship {
  id: string
  sourceContextId: string   // upstream
  targetContextId: string   // downstream

  // The team dynamic (mutually exclusive choice)
  pattern: 'customer-supplier' | 'conformist' | 'anti-corruption-layer'
         | 'partnership' | 'shared-kernel'

  // Upstream exposure (only when pattern is C/S, Conformist, or ACL)
  upstreamExposure?: ('open-host-service' | 'published-language')[]
}
```

This model:
- Makes the five relationship patterns a single mutually exclusive choice
- Allows OHS and PL as optional upstream exposure overlays on any U/D pattern
- Prevents invalid states (no upstream exposure on Partnership or Shared Kernel)
- Removes Separate Ways from relationship types (it's the absence of one)
- Is simpler than the previous three-field model (one choice + optional overlay, not three independent axes)

### Validation Rules

1. If `pattern` is `partnership` or `shared-kernel`: `upstreamExposure` must be empty
2. `upstreamExposure` can contain OHS, PL, or both (they're complementary)
3. OHS and PL are only meaningful when the relationship has an upstream-downstream dynamic

## References

- Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Chapter 14: Maintaining Model Integrity.
- Evans, Eric. [DDD Reference](https://www.domainlanguage.com/wp-content/uploads/2016/05/DDD_Reference_2015-03.pdf). pp. 32-34 (Customer-Supplier, Conformist, ACL).
- Vernon, Vaughn. "Implementing Domain-Driven Design." Chapter 3: Context Maps.
- Brandolini, Alberto. Various talks and writings on Context Mapping.
- [ddd-crew/context-mapping](https://github.com/ddd-crew/context-mapping) - Community reference for context mapping patterns.
