# Architecture Overview

A simplified, visual guide to understanding VRGB's design.

---

## The Big Picture

VRGB separates **what doesn't change** (color coordinates) from **what must evolve** (semantic interpretation):

```
┌─────────────────────────────────────────────────┐
│  User Preference: #FF3300 (stored forever)      │
└────────────────────┬────────────────────────────┘
                     │
                     ├──→ Medical Schema v2.1
                     │    └→ {severity: 87, urgency: 92}
                     │
                     ├──→ Dietary Schema v1.5
                     │    └→ {intensity: "strong", style: "bold"}
                     │
                     └──→ Meeting Schema v3.0
                          └→ {priority: "urgent", duration: "short"}
```

**Key insight**: The same coordinate can mean different things in different contexts, and those meanings can evolve independently without touching stored data.

---

## Four Layers

### 1. Storage Layer
**What**: Hex color codes (`#RRGGBB`)
**Why**: Immutable, compact (6 characters), universally understood
**Example**: `#FF3300`, `#707070`, `#B84F3A`

```
Database Table: user_preferences
┌─────────┬──────────────┬────────────┐
│ user_id │ domain       │ coordinate │
├─────────┼──────────────┼────────────┤
│ alice   │ medical      │ #FF3300    │
│ alice   │ dietary      │ #88CC44    │
│ bob     │ meeting_mgmt │ #3366FF    │
└─────────┴──────────────┴────────────┘
```

### 2. Interpretation Layer
**What**: Colorspace transforms + schema mappings
**Why**: Convert geometric coordinates to semantic parameters
**How**: `#FF3300` → LAB(90, 100, 60) → {severity: 87, urgency: 92}

```
┌──────────────┐
│   #FF3300    │  ← Hex coordinate
└──────┬───────┘
       │ decode via LAB transform
       ↓
┌──────────────┐
│ L: 90        │  ← Colorspace coordinates
│ a: 100       │
│ b: 60        │
└──────┬───────┘
       │ map via schema dimensions
       ↓
┌──────────────┐
│ severity: 87 │  ← Semantic parameters
│ urgency: 92  │
│ temporal: 78 │
└──────────────┘
```

### 3. Application Layer
**What**: Business logic using interpreted parameters
**Why**: Make decisions, route requests, personalize experiences
**Example**: If urgency > 80, notify doctor immediately

```python
params, confidence = schema.interpret("#FF3300")

if params["urgency"] > 80 and confidence > 0.85:
    notify_doctor(patient_id, params)
elif confidence < 0.75:
    flag_for_manual_review(patient_id)
```

### 4. Governance Layer
**What**: Schema versioning, audit trails, compliance
**Why**: Regulatory requirements, reproducibility, accountability

```json
{
  "interpretation_id": "uuid-123",
  "timestamp": "2025-11-20T14:30:00Z",
  "latent": "#FF3300",
  "schema_version": "v2.1",
  "schema_hash": "sha256-abc123...",
  "output": {"severity": 87, "urgency": 92},
  "confidence": 0.92
}
```

---

## Colorspace Transforms

VRGB supports multiple geometric interpretations of the same coordinate:

### RGB (Red-Green-Blue)
```
Geometry: Cartesian cube [0,255]³
Best for: Simple, orthogonal axes
Perceptually: Non-uniform (blue appears darker than green)
```

### HSL (Hue-Saturation-Lightness)
```
Geometry: Cylindrical (hue wraps at 360°)
Best for: Categorical preferences (hue) + intensity sliders
Perceptually: More intuitive than RGB for humans
```

### LAB (Lightness-a-b)
```
Geometry: Perceptually uniform (approximately)
Best for: Severity/urgency metrics, continuous scales
Perceptually: Designed to match human vision
```

**Example: Same color, different interpretations**

```
Hex:  #FF3300

RGB:  {R: 255, G: 51,  B: 0}
HSL:  {H: 12°,  S: 100%, L: 50%}
LAB:  {L: 90,   a: 100,  b: 60}
```

Each transform reveals different geometric structure suitable for different semantic domains.

---

## Schema Evolution

Schemas can evolve **without touching stored coordinates**:

```
Timeline of blood_panel_lab lineage:

v1.0 (2023)
  └─ 2 anchors, basic severity mapping

v2.0 (2024)
  └─ 3 anchors, added urgency dimension
  └─ Refined confidence thresholds

v2.1 (2025)
  └─ 5 anchors, improved coverage
  └─ Added temporal relevance
  └─ Merkle-style schema hashing

All versions interpret the SAME hex coordinates!
The coordinate #FF3300 stored in 2023 still works in 2025.
```

**Breaking changes** (e.g., switching from LAB to HSL) require a **new lineage**:

```
blood_panel_lab (LAB)  ← Original
blood_panel_hsl (HSL)  ← New lineage, different geometric interpretation
```

---

## Confidence Metrics

Every interpretation comes with a confidence score:

```
ρ_total = ρ_transform × ρ_schema × ρ_proximity

where:
  ρ_transform = quality of colorspace conversion (gamut clipping?)
  ρ_schema    = how well anchors cover this region
  ρ_proximity = distance from nearest anchor
```

**Confidence ranges**:
- `ρ ≥ 0.90`: High confidence, safe to use directly
- `0.75 ≤ ρ < 0.90`: Acceptable with warnings for critical decisions
- `ρ < 0.75`: Low confidence, flag for manual review

**Example**:

```python
# Well-calibrated coordinate near anchors
params, ρ = schema.interpret("#FF3300")
# → ρ = 0.92 (high confidence)

# Outlier coordinate far from any anchor
params, ρ = schema.interpret("#7F7F7F")
# → ρ = 0.68 (low confidence, review needed)
```

---

## Cross-Domain Interpretation

The same coordinate can be interpreted across different domains:

```
Alice's preference: #FF3300

Medical domain:
  severity: "high"
  urgency: "immediate"
  temporal: "acute"

Dietary domain:
  flavor: "bold"
  spice: "hot"
  cuisine: "Mediterranean"

Fitness domain:
  intensity: "maximum"
  duration: "short"
  recovery: "extended"
```

**Important**: VRGB does **not** claim these interpretations are causally linked. Correlations must be:
- Discovered empirically (analytics on user cohorts)
- Designed intentionally (aligned anchor placement)

---

## Anchors: Semantic Landmarks

Anchors are **labeled reference points** in colorspace that define semantic meaning:

```
Medical Schema Anchors:

#707070 → "Routine annual screening"
├─ L: 15  (low severity)
├─ a: -60 (low urgency)
└─ b: -10 (historical baseline)

#B88040 → "Chronic condition monitoring"
├─ L: 50  (moderate severity)
├─ a: 20  (moderate urgency)
└─ b: 40  (trending upward)

#FF3300 → "Acute clinical concern"
├─ L: 90  (high severity)
├─ a: 100 (maximum urgency)
└─ b: 60  (acute onset)
```

**Anchors serve three purposes**:
1. **Semantic grounding**: Define what regions of colorspace mean
2. **Confidence calibration**: Nearby coordinates have higher confidence
3. **UI/UX**: Can be surfaced as preset options in user interfaces

---

## Visual Interpretation Example

```
LAB Colorspace (simplified 2D projection)

   L (Lightness)
   ↑
100│                          ● #FF3300
   │                       ╱  (Acute concern)
   │                    ╱
 50│           ● #B88040
   │        ╱  (Monitoring)
   │     ╱
   │  ╱
  0│● #707070
   │(Routine)
   └────────────────────────────────→ a (Green-Red)
  -128        0          127
```

The distance between points in this space corresponds to semantic similarity (approximately).

---

## Data Flow: End to End

```
1. User sets preference
   └→ UI presents color picker OR semantic sliders
   └→ Store: #FF3300

2. Application needs interpretation
   └→ Load schema: blood_panel_v2.1
   └→ Decode: #FF3300 → LAB(90, 100, 60)
   └→ Map: LAB → {severity: 87, urgency: 92, temporal: 78}
   └→ Confidence: ρ = 0.92

3. Business logic executes
   └→ if urgency > 80: notify_doctor()
   └→ Log audit record

4. Analytics (optional)
   └→ Aggregate coordinates from cohort
   └→ Cluster in LAB space
   └→ Discover patterns
```

---

## When to Use VRGB

### Good Fits
- **Coarse categorical preferences** (3-10 distinct semantic regions)
- **Stable domain structure** (severity, urgency, priority)
- **Audit requirements** (medical, financial, content moderation)
- **Cross-domain correlation** (user profiles spanning contexts)
- **Interpretability requirements** (explainable decisions)

### Poor Fits
- **Fine-grained semantics** (thousands of distinct values)
- **Learned representations** (emergent structure from data)
- **High-dimensional signals** (language, vision embeddings)
- **Real-time inference** (sub-millisecond requirements)

---

## Comparison to Related Systems

| System | Dimensions | Learned? | Interpretable? | Stable? |
|--------|-----------|----------|----------------|---------|
| **VRGB** | 3D | No | Yes | Yes |
| Word2Vec | 300D | Yes | No | No |
| CLIP | 512D | Yes | Partial | No |
| Stable Diffusion | 512D | Yes | No | Yes |
| Traditional DB | 1D per field | N/A | Yes | Yes |

VRGB occupies a unique position: **low-dimensional, interpretable, and stable**.

---

## Key Takeaways

1. **Separation of concerns**: Coordinates (stable) vs. semantics (mutable)
2. **Geometric structure**: Colorspaces provide meaningful 3D geometry
3. **Deterministic interpretation**: No randomness, fully reproducible
4. **Versioned schemas**: Evolve semantics without data migration
5. **Confidence metrics**: Every interpretation has quality score
6. **Cross-domain reuse**: One coordinate, multiple interpretations
7. **Audit-friendly**: Complete provenance from storage to decision

---

## Next Steps

- **Read the [full white paper](../VRGB-whitepaper.md)** for formal definitions
- **Explore [examples](../examples/)** for complete schema implementations
- **Check the [FAQ](faq.md)** for common questions and patterns

---

**Last Updated**: November 21, 2025
