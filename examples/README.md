# VRGB Schema Examples

This directory contains complete, production-ready VRGB schema examples demonstrating different use cases and colorspace transforms.

---

## Available Examples

### 1. Blood Panel Analysis (`blood-panel-schema.json`)
**Domain**: Medical urgency and clinical severity
**Transform**: LAB (perceptually uniform for severity metrics)
**Use Case**: Interpreting blood test results for urgency triage

**Key Features**:
- 3D severity/urgency/temporal mapping
- High confidence thresholds (medical context)
- Comprehensive anchor coverage
- Audit mode enabled

**Sample Interpretation**:
```
#FF3300 → {
  clinical_severity: 87,
  urgency_level: 92,
  temporal_relevance: 78,
  confidence: 0.92
}
```

---

### 2. Recipe Recommendation (`recipe-recommendation-schema.json`)
**Domain**: Dietary preferences and cuisine style
**Transform**: HSL (categorical hue for cuisine types, saturation for intensity)
**Use Case**: Personalizing recipe suggestions based on user taste preferences

**Key Features**:
- Hue dimension maps to cuisine categories
- Saturation represents flavor intensity
- Lightness represents dietary restrictions
- Moderate confidence thresholds (recommendation context)

**Sample Interpretation**:
```
#FF3300 → {
  cuisine_category: "Mediterranean",
  flavor_intensity: 8.5,
  dietary_preference: "Bold flavors",
  confidence: 0.85
}
```

---

### 3. Meeting Urgency (`meeting-urgency-schema.json`)
**Domain**: Calendar prioritization and meeting scheduling
**Transform**: LAB (severity-like urgency metrics)
**Use Case**: Routing meeting requests and calendar prioritization

**Key Features**:
- Priority/urgency/duration axes
- Business hour vs. off-hour handling
- Integration with calendar systems
- Balanced confidence thresholds

**Sample Interpretation**:
```
#FF3300 → {
  priority_level: 95,
  urgency_score: 90,
  duration_preference: "short",
  confidence: 0.88
}
```

---

## Schema Structure Reference

All schemas follow this structure:

```json
{
  "domain": "string (semantic domain identifier)",
  "lineage_id": "string (transform family identifier)",
  "schema_version": "string (semantic version)",
  "created": "date (ISO 8601)",
  "schema_hash": "string (SHA-256 hash for audit)",

  "transform": {
    "colorspace": "lab|hsl|hsv|rgb",
    "reference_white": "D65|D50|...",
    "adaptation": "Bradford|VonKries|...",
    "gamma": "sRGB|linear|...",
    "clipping_policy": "perceptual_clamp|hard_clip|flag_and_warn"
  },

  "dimensions": {
    "L|H|R": {
      "param": "string (parameter name)",
      "label": "string (human-readable label)",
      "range": [min, max],
      "description": "string (semantic meaning)"
    },
    "a|S|G": { ... },
    "b|L|B": { ... }
  },

  "anchors": [
    {
      "id": "string (unique identifier)",
      "label": "string (semantic description)",
      "latent": "#RRGGBB (hex coordinate)",
      "decoded_*": { "dimension": value }
    }
  ],

  "quality_config": {
    "min_confidence": 0.0-1.0,
    "warning_threshold": 0.0-1.0
  },

  "audit_mode": boolean
}
```

---

## Usage Examples

### Loading and Using a Schema

```python
import json
from vrgb import VRGBInterpreter

# Load schema
with open("blood-panel-schema.json") as f:
    schema_json = json.load(f)

# Create interpreter
interpreter = VRGBInterpreter(schema_json)

# Interpret a coordinate
params, confidence = interpreter.interpret("#FF3300")

print(f"Severity: {params['clinical_severity']}")
print(f"Urgency: {params['urgency_level']}")
print(f"Confidence: {confidence}")

# Check confidence threshold
if confidence < schema_json["quality_config"]["min_confidence"]:
    print("⚠️  Low confidence - manual review recommended")
```

### Exploring the Semantic Space

```python
# Traverse between two anchors
start = "#707070"  # Routine screening
end = "#FF3300"    # Acute concern

trajectory = interpreter.traverse(start, end, steps=10)

for i, (params, confidence) in enumerate(trajectory):
    print(f"Step {i}: severity={params['clinical_severity']:.1f}, ρ={confidence:.2f}")
```

### Finding Nearest Anchors

```python
# Given a coordinate, find closest semantic landmarks
coordinate = "#DD4422"
anchors = schema_json["anchors"]
anchor_coords = [a["latent"] for a in anchors]

nearest = interpreter.nearest(coordinate, anchor_coords, k=3)

print("Nearest anchors:")
for coord, distance in nearest:
    anchor = next(a for a in anchors if a["latent"] == coord)
    print(f"  {anchor['label']}: distance={distance:.2f}")
```

---

## Design Patterns

### Pattern 1: Severity Scales (LAB)

Use LAB transform for continuous severity/urgency metrics:
- **L dimension**: Overall severity (0-100)
- **a dimension**: Urgency axis (past ← 0 → future)
- **b dimension**: Secondary axis (cold ← 0 → hot)

**Examples**: Medical triage, content moderation, risk assessment

### Pattern 2: Categorical Preferences (HSL)

Use HSL transform for categorical + intensity preferences:
- **H dimension**: Category (hue wraps: 0° = 360°)
- **S dimension**: Intensity/strength (0-100%)
- **L dimension**: Constraint axis (dark ← 50% → light)

**Examples**: Cuisine preferences, music genres, design styles

### Pattern 3: Orthogonal Axes (RGB)

Use RGB transform for independent orthogonal preferences:
- **R dimension**: Independent axis 1
- **G dimension**: Independent axis 2
- **B dimension**: Independent axis 3

**Examples**: Multi-dimensional sliders, feature toggles, resource allocation

---

## Anchor Placement Guidelines

### Minimum Coverage
- **2-3 anchors**: Sparse coverage, suitable for coarse categories
- **4-6 anchors**: Good coverage for most use cases
- **7-10 anchors**: Dense coverage for critical domains (medical, financial)

### Spatial Distribution
Place anchors to:
1. Cover extreme corners of the semantic space
2. Mark decision boundaries (e.g., moderate → severe threshold)
3. Represent common or important semantic regions
4. Ensure no large gaps in coverage (affects confidence)

### Example: Medical Urgency

```
Corner anchors:
  #707070 → "Routine screening" (low severity, low urgency)
  #FF3300 → "Acute concern" (high severity, high urgency)

Boundary anchor:
  #B88040 → "Monitoring" (moderate severity, moderate urgency)

Common region:
  #88AA88 → "Wellness check" (low severity, medium urgency)
```

---

## Confidence Tuning

Confidence thresholds depend on domain risk:

| Domain | Min Confidence | Warning Threshold | Rationale |
|--------|---------------|-------------------|-----------|
| Medical | 0.85-0.95 | 0.75-0.85 | High stakes, safety-critical |
| Financial | 0.80-0.90 | 0.70-0.80 | Regulatory compliance |
| Recommendations | 0.70-0.80 | 0.60-0.70 | Low stakes, user experience |
| Internal tools | 0.65-0.75 | 0.55-0.65 | Development/testing context |

Adjust based on empirical testing and domain requirements.

---

## Validation Checklist

Before deploying a schema, verify:

- [ ] `domain` and `lineage_id` are unique and descriptive
- [ ] `schema_version` follows semantic versioning
- [ ] `transform.colorspace` matches semantic structure (LAB for severity, HSL for categories)
- [ ] All `transform` parameters are specified (reference_white, clipping_policy, etc.)
- [ ] Each dimension has `param`, `label`, `range`, and `description`
- [ ] Dimension ranges are appropriate for the colorspace (e.g., LAB a/b: [-128, 127])
- [ ] At least 3 anchors are defined
- [ ] Anchors cover semantic space extremes
- [ ] Anchor `decoded_*` values match actual transform decoding (test this!)
- [ ] `quality_config` thresholds are appropriate for domain risk
- [ ] `schema_hash` is computed and stored
- [ ] `audit_mode` is enabled for production schemas

---

## Common Pitfalls

### 1. Mismatched Colorspace Semantics
**Problem**: Using RGB for severity scales (non-uniform perceptually)
**Solution**: Use LAB for continuous metrics, HSL for categorical

### 2. Sparse Anchor Coverage
**Problem**: Only 2 anchors, large regions have low confidence
**Solution**: Add anchors at decision boundaries and common regions

### 3. Inconsistent Lineage
**Problem**: Changing transform within a lineage (breaking geometric consistency)
**Solution**: Create new lineage for transform changes

### 4. Overly Specific Dimension Labels
**Problem**: `param: "2025_q4_risk_score"` (not future-proof)
**Solution**: `param: "risk_score"` (broad, stable)

### 5. Ignoring Confidence Scores
**Problem**: Using low-confidence interpretations without warnings
**Solution**: Always check `confidence` against thresholds, flag or reject low scores

---

## Extending These Examples

To create your own schema:

1. **Choose domain** and semantic axes (what do you want to encode?)
2. **Select colorspace** (LAB for severity, HSL for categories, RGB for orthogonal)
3. **Define dimensions** (map colorspace axes to semantic parameters)
4. **Place anchors** (identify 4-6 key semantic regions)
5. **Set thresholds** (confidence levels based on domain risk)
6. **Validate** (test interpretation with diverse coordinates)
7. **Iterate** (refine based on real-world usage)

---

## Additional Resources

- **[White Paper](../VRGB-whitepaper.md)**: Formal specification and theory
- **[Architecture Overview](../docs/architecture-overview.md)**: Visual explanations
- **[FAQ](../docs/faq.md)**: Common questions and design patterns

---

**Last Updated**: November 21, 2025
