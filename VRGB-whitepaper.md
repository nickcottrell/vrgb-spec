# VRGB: Stable Latent Coordinates for Semantic Preference Encoding

**Nicholas Cottrell**
**Version 2.7 — April 2026**

---

## Table of Contents

1. [Abstract](#abstract)
2. [Introduction: The Architecture That Matters](#1-introduction-the-architecture-that-matters)
   - [1.1 A Pattern, Not a Model](#11-a-pattern-not-a-model)
   - [1.2 Beyond Learned Latents](#12-beyond-learned-latents)
   - [1.3 VRGB as Instantiation and Proof](#13-vrgb-as-instantiation-and-proof)
3. [Abstract Stable Latent / Mutable Interpretation Pattern](#2-abstract-stable-latent--mutable-interpretation-pattern)
   - [2.1 Core Tuple](#21-core-tuple)
4. [VRGB as an Instantiation of SLI](#3-vrgb-as-an-instantiation-of-sli)
   - [3.1 Latent Space: Colorspace Coordinates](#31-latent-space-colorspace-coordinates)
   - [3.2 Colorspace Transforms](#32-colorspace-transforms)
   - [3.3 Preference Domains and Schemas](#33-preference-domains-and-schemas)
5. [Interpretation and Confidence](#4-interpretation-and-confidence)
   - [4.1 Interpretation Function](#41-interpretation-function)
   - [4.2 Confidence Thresholds](#42-confidence-thresholds)
6. [System Architecture and Invariants](#5-system-architecture-and-invariants)
   - [5.1 Layered Architecture](#51-layered-architecture)
   - [5.2 Design Invariants](#52-design-invariants)
7. [Geometry, Capacity, and Limitations](#6-geometry-capacity-and-limitations)
   - [6.1 Colorspace as Manifold](#61-colorspace-as-manifold)
   - [6.2 Information Capacity](#62-information-capacity)
8. [Cross-Domain Interpretation](#7-cross-domain-interpretation)
   - [7.1 Shared Latent, Multiple Schemas](#71-shared-latent-multiple-schemas)
   - [7.2 Geometric Regions and Analytics](#72-geometric-regions-and-analytics)
9. [Security and Auditability](#8-security-and-auditability)
   - [8.1 Semantic Privacy Boundary](#81-semantic-privacy-boundary)
   - [8.2 Schema Hashing and Merkle-style Audit](#82-schema-hashing-and-merkle-style-audit)
   - [8.3 Interpretation Audit Record](#83-interpretation-audit-record)
10. [Geometric Security Model](#9-geometric-security-model)
    - [9.1 Claims](#91-claims)
    - [9.2 Threat Model](#92-threat-model)
    - [9.3 Accountable Privacy](#93-accountable-privacy)
    - [9.4 Four Manifestations of Accountable Privacy](#94-four-manifestations-of-accountable-privacy)
    - [9.5 Distributed Addressing: Why Not Tor](#95-distributed-addressing-why-not-tor)
    - [9.6 Geometric Impossibility as Design Constraint](#96-geometric-impossibility-as-design-constraint)
11. [Operating Doctrine](#10-operating-doctrine)
    - [10.1 Stance](#101-stance)
    - [10.2 Policy](#102-policy)
    - [10.3 The Pattern](#103-the-pattern)
    - [10.4 Audience / Not For](#104-audience--not-for)
    - [10.5 Observers Welcome](#105-observers-welcome)
    - [10.6 Propagation](#106-propagation)
12. [Implementation Sketch](#11-implementation-sketch)
    - [11.1 Core API Shape](#111-core-api-shape)
    - [11.2 Schema Design Guidelines](#112-schema-design-guidelines)
13. [Related Work](#12-related-work)
14. [Conclusion: What Becomes Possible](#13-conclusion-what-becomes-possible)
    - [13.1 The Pattern Is Portable](#131-the-pattern-is-portable)
    - [13.2 Hybrid Architectures](#132-hybrid-architectures)
    - [13.3 Beyond Preferences](#133-beyond-preferences)
    - [13.4 The Cost of Interpretability](#134-the-cost-of-interpretability)
    - [13.5 What We Hope You Take Away](#135-what-we-hope-you-take-away)
15. [References](#references)
16. [Appendix A: Complete Schema Example](#appendix-a-complete-schema-example)

---

## Abstract

VRGB is a system for encoding preferences as fixed 3-dimensional color coordinates while externalizing semantics into versioned schemas. It follows the same architectural separation used by latent diffusion models: a **stable latent space**, a **mutable family of interpreters**, and **quality metrics** over interpretations. Unlike image diffusion, VRGB is **deterministic**, does not perform stochastic denoising, and does not learn its latent geometry. Instead, it adopts established colorspaces (RGB, HSL, LAB, etc.) as low-dimensional coordinate systems and treats schemas as explicit, human-authored mappings from color coordinates to domain-specific parameters.

This paper formalizes the VRGB architecture as an instance of a general *stable latent / mutable interpretation* pattern, defines its invariants and limitations, and outlines implementation and audit considerations. We emphasize both the **interpretability** and the **capacity constraints** of 3D colorspaces, positioning VRGB as a complementary alternative to high-dimensional embeddings in settings where coarse but stable preference encoding is desirable.

---

## 1. Introduction: The Architecture That Matters

### 1.1 A Pattern, Not a Model

When Stable Diffusion transformed generative AI, the breakthrough wasn't the denoising algorithm or the U-Net architecture. It was an architectural insight: **separate what doesn't change from what must evolve**.

The pattern is simple:

```
Fixed Latent Space
    ↓
Versioned Interpreter
    ↓
Generated Output
```

This separation enables something profound: you can improve, replace, or compose interpreters without touching stored coordinates. The same latent vector can be decoded by SD 1.5, SD 2.0, or SDXL. Better models emerge, but the addressing system remains stable.

We observe that **this pattern is not specific to images**. It works wherever:
- There exists a meaningful low-dimensional geometric structure
- Semantics must evolve faster than data can migrate
- Multiple interpreters may need to coexist
- Interpretations must be auditable and reversible

### 1.2 Beyond Learned Latents

Most implementations of this pattern use *learned* latent spaces - VAEs discover useful representations through gradient descent on reconstruction loss. This works brilliantly for complex domains like images or language where the "right" geometry is non-obvious.

But many semantic domains already have natural geometric structures. Color perception has been studied for a century; colorspaces like LAB approximate perceptual uniformity [Wyszecki & Stiles, 2000]. These spaces are:
- Low-dimensional by design (3D)
- Geometrically well-understood
- Standardized and interoperable
- Visually interpretable by humans

**What if we could use these existing geometries as latent spaces for semantic preferences?**

The result would combine the architectural benefits of stable diffusion (immutable coordinates, mutable interpreters) with the interpretability benefits of hand-designed spaces (visualizable, explainable, auditable).

### 1.3 VRGB as Instantiation and Proof

VRGB instantiates the stable latent / mutable interpretation pattern using colorspaces as addressing systems for semantic preferences. Where image diffusion stores 512-dimensional vectors and decodes them stochastically, VRGB stores 3-dimensional hex colors and decodes them deterministically through versioned schemas.

**The claim is not that colorspaces are optimal for all semantics.** They're not. 24 bits of address space imposes real constraints on expressiveness.

**The claim is that the architectural pattern generalizes beyond learned latents.** For domains with coarse, stable preference structures - clinical severity, dietary preferences, urgency levels, style profiles - a 3D geometric space can be sufficient. And when it is, the interpretability gains are substantial.

This paper:
1. Formalizes the abstract stable latent / mutable interpretation pattern
2. **Exhibits VRGB as a concrete instantiation** using colorspace geometry
3. Defines the invariants, metrics, and limitations that make the system work
4. Demonstrates how the same coordinate can be interpreted across domains
5. **Sketches how approximate computation can trade accuracy for scalability**

If VRGB succeeds, it suggests that **many semantic systems could benefit from explicit geometric structure** rather than defaulting to high-dimensional embeddings. Not everything needs to be learned; sometimes the geometry is already there, waiting to be used.

---

## 2. Abstract Stable Latent / Mutable Interpretation Pattern

We first define a general pattern that VRGB instantiates.

### 2.1 Core Tuple

A *stable latent / mutable interpretation system* `SLI` is defined as:

```
SLI = (L, D, I, M)
```

where:

* `L`: latent space, a fixed geometric set.
* `D`: a family of interpreters `{D_v}` indexed by version or configuration.
* `I`: interpretation function mapping `(latent, interpreter)` to outputs in a domain-specific space.
* `M`: metrics assessing the quality or confidence of an interpretation.

We require:

* **Latent stability (design invariant)**:
  Latents in `L` are treated as immutable identifiers once stored; changes in business logic are handled by updating `D`, not `L`.

* **Interpreter versioning**:
  Interpreters `{D_v}` can be added, deprecated, or composed, while `L` remains unchanged.

* **Deterministic interpretation**:
  For a given `ℓ ∈ L` and `D_v`, `I(ℓ, D_v)` is deterministic (stochastic models may be wrapped, but the interface is deterministic).

* **Metric availability**:
  `M` provides at least one scalar score per interpretation, e.g., confidence or coverage, to support gating and audit.

Latent diffusion models satisfy this pattern with:

* `L`: learned latent vectors (e.g., ℝ^512).
* `D`: U-Net + conditioning stacks, versioned models.
* `I`: decode latent to image.
* `M`: FID, perceptual metrics, classifier scores.

VRGB will satisfy it with a 3D colorspace as `L` and schemas as `D`.

---

## 3. VRGB as an Instantiation of SLI

### 3.1 Latent Space: Colorspace Coordinates

VRGB's latent space `L_color` is defined via sRGB:

* Canonical representation of a coordinate `C`:

  ```
  C = #RRGGBB
  R,G,B ∈ {0,…,255}
  ```
* Discrete latent space size:

  ```
  |L_color| = 256³ = 16,777,216
  ```
* Continuous approximation:

  ```
  L_color ⊂ [0,1]³
  ```

Coordinates are stored in the database as hex strings and treated as immutable identifiers.

### 3.2 Colorspace Transforms

VRGB uses **transform functions** between sRGB and other colorspaces:

* For a chosen colorspace `T` (RGB, HSL, HSV, LAB, XYZ, etc.), define:

  ```
  decode_T: C_sRGB → (d₁, d₂, d₃, meta_T)
  encode_T: (d₁, d₂, d₃) → (C_sRGB', meta_T')
  ```

  where:

  * `(d₁, d₂, d₃)` are coordinates in `T`,
  * `meta_T` includes information about clipping, out-of-gamut adjustments, etc.

Important constraints:

* RGB↔HSL / RGB↔HSV are invertible on sRGB.
* RGB↔LAB / RGB↔XYZ are **gamut-dependent** and generally not bijective:

  * Some LAB points do not map to any valid sRGB color.
  * Implementations must specify:

    * reference white (e.g., D65),
    * adaptation method (e.g., Bradford),
    * clipping policy (e.g., clamp to nearest in-gamut point).

VRGB therefore treats transforms as:

* **Deterministic** functions with **explicitly defined clipping behavior**, not as perfect mathematical inverses.

### 3.3 Preference Domains and Schemas

A **preference domain** captures a semantic category and its chosen transform:

```
D = (name, T, lineage_id)
```

* `name`: human-readable identifier (e.g., `"blood_panel_analysis"`).
* `T`: chosen colorspace transform for interpretation.
* `lineage_id`: identifies a family of schemas that share the same transform `T`.

Schemas are the interpreters `{D_v}` in the SLI pattern:

```json
{
  "domain": "blood_panel_analysis",
  "lineage_id": "blood_panel_lab",
  "schema_version": "v2.1",
  "transform": {
    "colorspace": "lab",
    "reference_white": "D65",
    "adaptation": "Bradford",
    "gamma": "sRGB",
    "clipping_policy": "perceptual_clamp"
  },
  "dimensions": {
    "L": {
      "param": "clinical_severity",
      "label": "Clinical Severity",
      "range": [0, 100]
    },
    "a": {
      "param": "urgency_level",
      "label": "Urgency Level",
      "range": [-128, 127]
    },
    "b": {
      "param": "temporal_relevance",
      "label": "Temporal Relevance",
      "range": [-128, 127]
    }
  },
  "anchors": [
    {
      "id": "routine_screening",
      "latent": "#707070",
      "decoded_lab": {"L": 15, "a": -60, "b": -10}
    },
    {
      "id": "acute_concern",
      "latent": "#FF3300",
      "decoded_lab": {"L": 90, "a": 100, "b": 60}
    }
  ],
  "quality_config": {
    "min_confidence": 0.85,
    "warning_threshold": 0.75
  },
  "audit_mode": true,
  "schema_hash": "sha256-…"
}
```

**Lineage invariant**:
All schemas sharing the same `lineage_id` must use the same transform `T`. If the transform changes (e.g., from HSL to LAB), a new lineage is created.

---

## 4. Interpretation and Confidence

### 4.1 Interpretation Function

Given:

* a latent coordinate `C ∈ L_color`,
* a schema `S` with transform `T`,

VRGB defines an interpretation function:

```
I(C, S) → (params, ρ_total)
```

Implementation pattern:

1. **Decode latent coordinate**:

   ```
   (d₁, d₂, d₃, meta_T) = decode_T(C_sRGB)
   ```

2. **Map dimensions to semantic parameters** using schema ranges and any additional mapping logic (e.g., linear scaling, piecewise mapping):

   ```python
   params = {
       dim_name.param: map_to_range(d_i, dim_name.range)
       for dim_name in [L, a, b]
   }
   ```

3. **Compute confidence score** `ρ_total` via a *cascading heuristic*:

   * `ρ_transform`: quality of transform (e.g., in-gamut vs heavy clipping).
   * `ρ_schema`: coverage (e.g., how well anchors and dimension ranges cover this region).
   * `ρ_proximity`: distance from nearest anchor(s).

   Example heuristic:

   ```
   ρ_total = ρ_transform × ρ_schema × ρ_proximity
   ```

This is **not** statistically calibrated uncertainty; it is a structured heuristic that can be tuned per domain.

### 4.2 Confidence Thresholds

Schemas can define thresholds:

* `min_confidence`: below this, results are flagged or rejected.
* `warning_threshold`: range where results remain usable but should be surfaced with caveats.

Suggested taxonomy:

* `ρ_total ≥ 0.90`: high-confidence interpretation.
* `0.75 ≤ ρ_total < 0.90`: acceptable but warn for critical decisions.
* `ρ_total < 0.75`: low-confidence; suggest manual review or alternative handling.

---

## 5. System Architecture and Invariants

### 5.1 Layered Architecture

Conceptually, VRGB is organized into four layers:

1. **Latent Storage Layer**

   * Stores `C = #RRGGBB` per user/domain or per key.
   * Values are treated as immutable once written.

2. **Diffusion / Interpretation Layer**

   * Loads schemas.
   * Applies decode + semantic mapping + confidence calculation.

3. **Application Logic Layer**

   * Implements product logic using `(params, ρ_total)`.
   * Enforces thresholds, routes edge cases, logs audit records.

4. **Governance & Audit Layer**

   * Tracks schema versions, hashes, lineage.
   * Ensures regulatory or internal compliance.

### 5.2 Design Invariants

VRGB enforces the following design-level invariants:

* **I1: Latent Immutability**
  Once stored, a latent coordinate is not mutated; semantic evolution is handled by new schemas or schema versions.

* **I2: Transform Consistency per Lineage**
  A lineage `lineage_id` must always use the same transform `T`. Transform changes spawn a new lineage.

* **I3: Monotonic Semantic Refinement**
  Within a lineage, schema versions should refine or extend semantics (e.g., more anchors, refined ranges), not reverse or substantially reassign dimension meaning. Breaking changes require a new lineage.

* **I4: Deterministic Interpretation**
  For a fixed `(C, S)`, `I` must return the same `(params, ρ_total)`.

These are governance policies rather than mathematical theorems, but they are essential to VRGB's stability story.

---

## 6. Geometry, Capacity, and Limitations

### 6.1 Colorspace as Manifold

Colorspaces can be viewed as low-dimensional manifolds with metrics:

* LAB: often modeled with Euclidean distance `ds² = dL² + da² + db²` as an approximation to perceptual difference.
* HSL: cylindrical coordinates with periodic hue; distance metrics must account for wrap-around.

VRGB does not rely on exact perceptual uniformity; it uses these geometries as *practical* semantic spaces:

* LAB: good for severity/urgency-like semantics with more uniform distances.
* HSL: good for categorical semantics (hue) plus intensity (saturation/lightness).
* RGB: simple orthogonal axes, non-uniform perceptually but trivial to implement.

### 6.2 Information Capacity

The address space of sRGB hex has:

* `|L_color| = 256³` discrete points.
* Maximum raw bit capacity:

  ```
  log₂(256³) = 24 bits
  ```

Practical implications:

* VRGB can meaningfully represent on the order of 10³–10⁴ distinct **semantic regions** under realistic constraints; beyond that, regions become dense and difficult to interpret or govern.
* High-dimensional semantics (e.g., full language meaning) **should not** be compressed into a single VRGB coordinate without external structure.

VRGB is therefore positioned for:

* Coarse categorical preferences.
* Severity ladders.
* Style or profile sliders.
* Routing decisions with relatively few modes.

---

## 7. Cross-Domain Interpretation

### 7.1 Shared Latent, Multiple Schemas

The most powerful consequence of VRGB's architecture is that the same latent coordinate can be interpreted in multiple domains:

```
C = #FF3300

Medical schema S_med:
  I(C, S_med) → {severity: "high", urgency: "immediate", ρ_total: 0.92}

Dietary schema S_diet:
  I(C, S_diet) → {flavor_intensity: "strong", cuisine_style: "Mediterranean", ρ_total: 0.80}

Fitness schema S_fit:
  I(C, S_fit) → {effort: "maximum", duration: "short", ρ_total: 0.87}
```

VRGB itself does **not** assert that these interpretations are ontologically linked. Any cross-domain correlation must be discovered empirically (e.g., via analytics) or designed intentionally (e.g., by aligning anchor placement across schemas).

### 7.2 Geometric Regions and Analytics

Collections of coordinates:

```
R = {C₁, C₂, …}
```

allow:

* Clustering in decoded space (e.g., k-means in LAB).
* Anomaly detection (e.g., large distance from any anchor).
* Multi-user aggregation (e.g., average color per cohort, then interpret via schema).

Such analytics should always be conditioned on:

* The transform `T`.
* The domain-specific schema `S`.
* The confidence scores `ρ_total`.

---

## 8. Security and Auditability

### 8.1 Semantic Privacy Boundary

VRGB separates:

* **Public or shareable:**

  * raw hex coordinates
  * colorspace transforms (as math)
  * distance/interpolation operations

* **Private or access-controlled:**

  * schemas (semantic mappings)
  * anchors and their semantic labels
  * confidence thresholds

If schemas and anchors are treated as internal configuration, then an external party observing only hex values and public transforms cannot reconstruct semantic interpretations.

### 8.2 Schema Hashing and Merkle-style Audit

Schemas can be hashed as:

* Simple mode:

  ```
  schema_hash = SHA256(entire_schema)
  ```

* Hierarchical mode (Merkle-style):

  ```
  dimensions_hash = H(H(dim1) || H(dim2) || H(dim3))
  anchors_hash    = H(H(anchor1) || …)
  metadata_hash   = H(…)
  root_hash       = H(dimensions_hash || anchors_hash || metadata_hash)
  ```

This allows:

* Fine-grained diffing between schema versions.
* Verifiable audit records showing *which* schema version produced a given interpretation.

### 8.3 Interpretation Audit Record

An interpretation event can be logged as:

```json
{
  "interpretation_id": "uuid-123",
  "timestamp": "2025-11-20T14:30:00Z",
  "latent_coordinate": "#FF3300",
  "schema_ref": {
    "domain": "blood_panel_analysis",
    "lineage_id": "blood_panel_lab",
    "schema_version": "v2.1",
    "schema_hash": "sha256-abc123..."
  },
  "output": {
    "severity": 87,
    "urgency": 92,
    "temporal": 78
  },
  "quality": {
    "ρ_transform": 0.98,
    "ρ_schema": 0.95,
    "ρ_proximity": 0.99,
    "ρ_total": 0.92
  }
}
```

This structure is sufficient to reconstruct the *exact* interpretation given a preserved copy of the schema referenced by `schema_hash`.

---

## 9. Geometric Security Model

Section 8 establishes the mechanical primitives: what is public, what is private, how schemas are hashed, how interpretations are audited. This section takes the next step. It states the security model VRGB claims to provide, defines the threat model it addresses, and argues that the resulting guarantees are *derived from geometry* rather than assigned by policy.

### 9.1 Claims

This section defends four claims:

1. **Accountable privacy is a distinct architectural category.** It is not a weaker form of anonymity and not a stronger form of transparency. It is a first-class design primitive with its own guarantees.
2. **Security properties are derivable from geometry, not assigned by policy.** In VRGB, the invariants that matter for security fall out of the math of the colorspace and the schema contract. They are not access control lists wrapped around a trust boundary.
3. **VRGB is a privacy primitive, not a display system.** Color rendering is one application of colorspace-as-substrate. Preference addressing, conformance proofs, and distributed identifiers are others. The substrate is load-bearing for privacy even when nothing on a screen changes color.
4. **Spectral binding makes surveillance economically irrational.** Not impossible — uneconomical. A system where every address is discovered rather than assigned, and where interpretation requires the correct schema, raises the cost of undirected surveillance past the point where it pays.

### 9.2 Threat Model

VRGB is designed to resist:

* **Centralized surveillance of preference and behavior addresses.** No single authority owns the address space; no central registry can be subpoenaed or compromised to reveal who holds which coordinate.
* **Forged or silently-rewritten interpretations.** Schemas are content-addressed (§8.2). A schema rewrite changes its hash; audit records (§8.3) pin the interpretation to a specific schema version.
* **Schema-level semantic drift.** Because lineage and hash are recorded at interpretation time, a later change to a schema cannot retroactively alter the meaning of a prior output.
* **Unaccountable publication.** Interpretations that affect others carry a provenance chain. A claim with no chain is recognizable as such.

VRGB does **not** claim to defend against:

* Anonymity from consensual disclosure. Users who publish their own coordinates are no longer private with respect to those coordinates.
* Side-channel attacks on the execution environment (memory, cache, network timing).
* Social-engineering attacks on schema authors or key holders.
* Key-custody failures outside the protocol.

A useful shorthand: **policy is a fence; geometry is a law of physics.** Fences can be walked around, torn down, or redefined. Geometric invariants cannot be waived by a trusted operator, because there is no trusted operator in the protocol.

### 9.3 Accountable Privacy

> **Accountable Privacy** — the structural separation of *content* from *identity* and *behavior* from *surveillance*, such that individuals control what is observed about them by default, but cannot use that control to evade legitimate accountability for actions that affect others.

The load-bearing word is *structural*. Accountable privacy is not:

* **Policy-based** (a promise by the operator to behave well), or
* **Platform-based** (trust in a particular provider's integrity),

but baked into the math of the system such that the operator cannot unilaterally revoke it, and the user cannot unilaterally escape it.

The contrast with anonymity is precise:

| Property | Anonymity | Accountable Privacy |
|---|---|---|
| Default posture | "No one can ever know." | "Observed only by default; disclosable under defined conditions." |
| Enforcement | Path obfuscation | Cryptographic and geometric contracts |
| Audit surface | None (by design) | Present, but consent-gated |
| Failure mode | Exposure breaks the model | Disclosure follows the contract |

Anonymity and accountable privacy are not points on a spectrum. They are different architectures with different threat models. Bad actors need **unaccountable anonymity** — a posture in which actions leave no recoverable trace. VRGB provides **accountable privacy** — a posture in which actions leave traces that only the right parties, under the right conditions, can read. The first is a shield for harm; the second is a substrate for trust.

### 9.4 Four Manifestations of Accountable Privacy

Accountable privacy is an abstract posture. It manifests concretely through at least four mechanisms, each of which can be composed with VRGB's colorspace substrate:

1. **Cryptographic commitment without disclosure.** A user publishes a hash commitment to an action; the action's content stays sealed. If accountability is later required, the commitment proves the action cannot have been retroactively rewritten. Privacy is preserved unless behavior triggers review.
2. **Selective disclosure via zero-knowledge proofs.** A user proves a predicate ("over 21," "paid taxes," "author of this work") without revealing the underlying data. The proof transmits exactly the accountability signal needed — no more.
3. **Geometric conformance proofs (VRGB-native).** A user proves that a behavior sits within an accepted region of phase space without revealing its exact coordinates. *"I am operating within the green band"* is verifiable; the specific shade is not exposed. This is the VRGB-native form, and the reason the colorspace substrate matters for privacy rather than merely for display: the substrate lets the proof be a shape-membership claim rather than a value disclosure.
4. **Consent-gated access with mandatory audit trail.** Data is encrypted to the user's key. Any access — including by the user themselves at a later date, or by law enforcement under a disclosed warrant — leaves a tamper-evident log. Privacy is the default; access is possible but never invisible. The watcher is watched.

These are not mutually exclusive. A mature accountable-privacy deployment composes at least (3) and (4), and frequently all four.

### 9.5 Distributed Addressing: Why Not Tor

Tor is the nearest existing analogue, and the comparison is instructive.

* **Onion routing** achieves privacy of *path*: no single node sees both source and destination.
* **VRGB** achieves privacy of *identifier*: no single authority owns the address space.

Both are structural rather than policy-based. Both degrade gracefully rather than failing catastrophically. The distinction is what each removes:

* Tor removes the observer's ability to know *who is talking to whom*.
* VRGB removes the system's need for a central *who* to exist at all.

A VRGB address is not anonymous — it carries a cryptographic lineage through chip signatures, vault provenance, and the keeper chain. But it is not *centrally resolvable*. The address space is discovered by spectral binding, not assigned by a registry. This is accountable privacy at the identifier layer: every address has a verifiable history, and no party can top-down surveil the space.

### 9.6 Geometric Impossibility as Design Constraint

"Geometrically impossible misuse" is a concrete design constraint, not an aspiration. It means the misuse path does not exist in the topology of the system — not that it is forbidden by policy, and not that it is merely expensive. A policy can be waived; a cost can be paid; a geometry cannot be negotiated with.

In VRGB:

* Addresses are discovered via spectral binding, not assigned by a registry. There is no "issue a new address" API call, and therefore no authority to compromise for forgery.
* Interpretation requires the correct schema. Without the schema, a hex coordinate is an uninterpretable three-tuple. Surveillance that captures raw coordinates captures nothing of semantic value.
* Every step of the audit chain is verifiable against the geometry. No node in the protocol requires the reader to *trust* any party; every claim is checkable.

The inverse posture — start with access control lists and add cryptography to protect the lists — remains the industry default. VRGB starts with a shape. The access controls that matter fall out of which operations the shape permits.

---

## 10. Operating Doctrine

Section 9 is the contract: *what the system does, and what a researcher is entitled to rely on.* This section is the doctrine: *the stance behind the contract, the rules that follow from the stance, and the pattern other projects can fork.* Contract without doctrine is brittle under governance pressure; doctrine without contract is a press release. Both belong in the same artifact.

### 10.1 Stance

Privacy is not secrecy. Accountable privacy is the default posture of a system that respects both the individual and the collective. A well-designed substrate makes that posture the path of least resistance — users do not have to choose it consciously, and operators cannot unilaterally override it.

Three commitments flow from the stance:

1. **Transparency about architecture, privacy about content.** The system's invariants, schemas (as hashes), and audit records are public. The payloads that ride on them are not, except under the user's contract.
2. **Legibility over endorsement.** The project publishes its current state so that oversight communities, researchers, and peers can *observe* the work without being *involved* with it. Being watched is a feature of the design, not a concession to it.
3. **Altruistic closure of exploits.** Every closed vulnerability becomes a published lesson. Security improvements are artifacts of the commons — not because the project is generous, but because the stance requires it.

### 10.2 Policy

The stance becomes operational through rules such as:

* No central identity registry.
* No single point of de-anonymization.
* Disclosure thresholds require multi-party consent, cryptographic rather than social.
* Schemas and interpretations are content-addressed; silent rewrites are detectable by anyone, not just the project.
* Every rule traces back to a stance commitment. Rules that do not are candidates for removal.

### 10.3 The Pattern

The doctrine is intended to be forkable. The reusable shape is:

```
stance → threat model → architectural constraints → cryptographic primitives → published contract
```

Each layer constrains the next, but none directly implements the layer above:

* The **stance** does not execute code. It rules out classes of implementation.
* The **threat model** does not enforce anything. It declares which attacks the contract is designed to resist.
* The **architectural constraints** do not perform cryptography. They constrain which primitives are acceptable.
* The **cryptographic primitives** do not declare policy. They realize the constraints at the byte level.
* The **published contract** does not run. It states what the running system owes its observers.

A project that builds this ladder, in this order, is running compatible doctrine regardless of domain.

### 10.4 Audience / Not For

This doctrine is for operators building verifiable-trust infrastructure — systems where users need to be *legibly private* rather than *invisibly hidden*.

It is explicitly **not** for:

* **Anonymity-seekers.** If the goal is "no one can ever know," the audit trail and cryptographic lineage will be in the way. That is correct by design.
* **Performance-seekers.** The substrate prioritizes interpretability and geometric invariants over throughput. Low-latency workloads will find the contract expensive.
* **Those seeking escape from accountability.** Actions in accountable-privacy systems remain traceable by the right parties under the right conditions. That is the whole point of the architecture.

If any of the above is the user's goal, the architecture will frustrate them on purpose. This is not a defect to be engineered around in a later release.

### 10.5 Observers Welcome

The project is designed to be watched. Oversight communities — AI-safety organizations, alignment researchers, infrastructure peers, civil-society auditors — are invited to observe without obligation to engage. The legibility surface consists of:

* The published specification (this document) and its provenance.
* A public changelog tracking architectural decisions.
* Tagged commits and posts for RSS-level tracking (`#vrgb-doctrine`, `#geometric-security`).
* Open issues reflecting current threat-model assumptions and known limitations.

Responses from observers are not expected. The posture is *observable under supervision*, not *seeking endorsement*.

### 10.6 Propagation

The doctrine is branded only incidentally. Compatibility is architectural:

A project running compatible doctrine is one that (a) publishes an analogous stance, (b) derives its security properties from geometry rather than policy, (c) defines accountable privacy for its own domain, and (d) invites observation on the same terms. Label agreement is not required; architectural agreement is.

The goal is propagation of the pattern — not of a term, a repository, or a name. A field of projects running compatible doctrine constitutes distributed oversight by construction. No central body has to exist; the legibility is the oversight.

---

## 11. Implementation Sketch

### 11.1 Core API Shape

A minimal Python-like interface:

```python
class ColorspaceLatent:
    def __init__(self, transform_config):
        self.transform = build_transform(transform_config)

    def decode(self, hex_coord, fidelity="exact"):
        # returns (d1, d2, d3, meta)
        ...

    def distance(self, c1, c2):
        # distance in chosen colorspace
        ...

    def interpolate(self, c1, c2, alpha):
        # returns new hex_coord
        ...

class VRGBInterpreter:
    def __init__(self, schema_json):
        self.schema = load_schema(schema_json)
        self.latent_space = ColorspaceLatent(self.schema["transform"])

    def interpret(self, hex_coord, fidelity="exact"):
        decoded, ρ_transform = self.latent_space.decode(hex_coord, fidelity)
        params = map_decoded_to_params(decoded, self.schema["dimensions"])
        ρ_schema = compute_schema_confidence(decoded, self.schema)
        ρ_proximity = compute_anchor_proximity(decoded, self.schema["anchors"])
        ρ_total = ρ_transform * ρ_schema * ρ_proximity
        return params, ρ_total

    def traverse(self, start_coord, end_coord, steps=10):
        path = self.latent_space.interpolate(start_coord, end_coord, steps)
        return [self.interpret(c) for c in path]

    def nearest(self, target_coord, candidates, k=10):
        dists = [(c, self.latent_space.distance(target_coord, c)) for c in candidates]
        return sorted(dists, key=lambda x: x[1])[:k]
```

### 11.2 Schema Design Guidelines

To keep VRGB interpretable and stable:

* Choose `T` such that its geometry reflects the semantic structure (e.g., LAB for severity-like notions).
* Define **3–7 anchor presets** per schema as semantic landmarks.
* Ensure dimension definitions are **broad and stable** (e.g., "overall severity", not "2025 risk score v3").
* Use conservative, monotonic evolutions:

  * Add anchors where confidence is low.
  * Refine ranges without flipping dimension meaning.

---

## 12. Related Work

**Latent diffusion models [Rombach et al., 2022]** separate learned latent spaces from generative decoders. VRGB borrows the architectural pattern (stable latents, mutable models, quality metrics) but uses hand-designed geometric spaces rather than learned representations.

**Vector embeddings [Mikolov et al., 2013]** encode semantics in high-dimensional spaces learned from data. VRGB trades capacity for interpretability, using only 3 dimensions but gaining visual and geometric clarity.

**Color theory and perceptual spaces [Wyszecki & Stiles, 2000]** provide approximately uniform perceptual distances. VRGB adopts these as convenient semantic geometries but does not depend on perfect perceptual uniformity.

**Explainable and auditable systems** increasingly require interpretable representations and complete decision trails. VRGB's strict separation of storage, interpretation, and metrics aligns with these goals, particularly in regulated domains (medical, financial) where reconstruction of decision context matters.

---

## 13. Conclusion: What Becomes Possible

### 13.1 The Pattern Is Portable

VRGB demonstrates that the stable latent / mutable interpretation pattern works beyond learned representations. This suggests a broader principle: **when natural geometric structures exist, they are candidates for latent spaces**, rather than defaulting to learned embeddings.

Many semantic domains have intuitive low-dimensional structures:
- **Audio**: pitch × timbre × loudness
- **Temporal**: urgency × duration × frequency
- **Haptic**: pressure × texture × temperature
- **Social**: intimacy × formality × urgency

Each of these could be encoded in 3D coordinate systems, interpreted through versioned schemas, and evolved without data migration. The question is not whether the geometry is perfect, but whether it is *useful enough* that the interpretability gains outweigh the capacity constraints.

### 13.2 Hybrid Architectures

VRGB need not replace high-dimensional embeddings. It can complement them:

**Coarse routing via VRGB, fine-tuning via embeddings:**
```
User preference: #FF3300 → "high urgency medical context"
  ↓
Route to specialized medical embedding model
  ↓
Fine-grained clinical predictions
```

**Human-interpretable override layer:**
```
Embedding-based recommendation: [0.23, -0.87, ...]
  ↓
Convert to nearest VRGB coordinate: #B84F3A
  ↓
User adjusts via sliders in interpretable space
  ↓
Convert back to embedding for execution
```

The geometric interpretability of VRGB makes it useful as an interface layer even when more powerful representations exist underneath.

### 13.3 Beyond Preferences

The pattern applies wherever:
- **Semantics must evolve** faster than data can migrate
- **Multiple models** need to coexist (A/B testing, gradual rollouts)
- **Interpretations must be auditable** (regulatory compliance)
- **Cross-domain reuse** creates value

Potential applications:
- **Configuration management**: System settings as color coordinates, interpreted differently per deployment context
- **Content moderation**: Severity coordinates stable across evolving policy definitions
- **Recommendation systems**: User taste profiles as traversable geometric spaces
- **Routing and orchestration**: Service selection based on interpretable multi-dimensional preferences

### 13.4 The Cost of Interpretability

VRGB's 24-bit capacity is a real constraint. It cannot encode rich semantic nuance. It cannot replace transformers for language understanding or ViT for vision. It will never be the right tool for learned, emergent representations.

But for stable, coarse-grained semantic categories—the kind that humans think about and discuss explicitly—geometric structure may be enough. And when it is, the gains in interpretability, auditability, and governance may justify the capacity trade-off.

### 13.5 What We Hope You Take Away

If you remember one thing from this paper, let it be this: **the stable diffusion pattern is not about images or noise schedules. It's about separating what changes from what doesn't.**

VRGB **demonstrates** this pattern works with:
- Hand-designed geometry instead of learned latents
- Deterministic schemas instead of stochastic denoising
- 3D colorspaces instead of 512D vectors

If a 3D geometric space can serve as a useful semantic addressing system, what else might work? What other domains have natural structures waiting to be used as latent spaces? What other rigid hierarchies might be replaced with versioned interpreters over stable coordinates?

The future may not belong exclusively to learned representations. There might be room for systems that are **designed to be understood**, where the geometry means something before any data is seen, where evolution happens through explicit versioning rather than gradient descent.

VRGB is one proof of concept. We suspect there are others.

---

## References

[1] Rombach, R., et al. (2022). "High-Resolution Image Synthesis with Latent Diffusion Models." *CVPR 2022*.

[2] Wyszecki, G., & Stiles, W. S. (2000). *Color Science: Concepts and Methods*. John Wiley & Sons.

[3] Mikolov, T., et al. (2013). "Efficient Estimation of Word Representations in Vector Space." *arXiv:1301.3781*.

[4] CIE (2004). *Colorimetry*. CIE Publication 15:2004.

[5] Schneier, B. (2015). *Applied Cryptography*. John Wiley & Sons.

[6] Kingma, D. P., & Welling, M. (2013). "Auto-Encoding Variational Bayes." *arXiv:1312.6114*.

---

## Appendix A: Complete Schema Example

```json
{
  "domain": "blood_panel_analysis",
  "lineage_id": "blood_panel_lab",
  "schema_version": "v2.6",
  "created": "2025-11-20",
  "schema_hash": "sha256-abc123...",

  "transform": {
    "colorspace": "lab",
    "reference_white": "D65",
    "adaptation": "Bradford",
    "gamma": "sRGB",
    "clipping_policy": "perceptual_clamp"
  },

  "dimensions": {
    "L": {
      "param": "clinical_severity",
      "label": "Clinical Severity",
      "range": [0, 100],
      "description": "Overall clinical concern level"
    },
    "a": {
      "param": "urgency_level",
      "label": "Urgency Level",
      "range": [-128, 127],
      "description": "Time-sensitivity axis"
    },
    "b": {
      "param": "temporal_relevance",
      "label": "Temporal Relevance",
      "range": [-128, 127],
      "description": "Historical vs trending axis"
    }
  },

  "anchors": [
    {
      "id": "routine_screening",
      "label": "Routine Annual Screening",
      "latent": "#707070",
      "decoded_lab": {"L": 15, "a": -60, "b": -10}
    },
    {
      "id": "monitoring_chronic",
      "label": "Chronic Condition Monitoring",
      "latent": "#B88040",
      "decoded_lab": {"L": 50, "a": 20, "b": 40}
    },
    {
      "id": "acute_concern",
      "label": "Acute Clinical Concern",
      "latent": "#FF3300",
      "decoded_lab": {"L": 90, "a": 100, "b": 60}
    }
  ],

  "quality_config": {
    "min_confidence": 0.85,
    "warning_threshold": 0.75
  },

  "audit_mode": true
}
```

---

## Copyright and License

Copyright © 2025 Nicholas Cottrell

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

---

**Document Version**: 2.6
**Last Updated**: November 21, 2025
**Document Hash**: `sha256-[to-be-computed]`
