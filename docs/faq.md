# Frequently Asked Questions

Common questions about VRGB design, implementation, and use cases.

---

## General Concepts

### What is VRGB in one sentence?

VRGB stores user preferences as fixed 3D color coordinates that can be interpreted differently across contexts using versioned schemas, borrowing the stable latent / mutable interpreter pattern from Stable Diffusion.

### Why use color coordinates instead of just storing semantic values directly?

Direct semantic storage couples data to business logic. When requirements change (e.g., "urgency" definition evolves), you must migrate all stored data. VRGB stores **geometric coordinates** that are interpreted through **versioned schemas**, allowing semantic evolution without data migration.

### How is this related to Stable Diffusion?

VRGB uses the same architectural pattern:
- **Stable Diffusion**: 512D learned latent → versioned decoder → generated image
- **VRGB**: 3D color coordinate → versioned schema → semantic parameters

Both separate immutable addressing (latents) from mutable interpretation (models/schemas).

### Is VRGB a machine learning system?

No. VRGB uses **hand-designed geometry** (colorspaces), not learned representations. Schemas are explicitly authored, not trained. This trades capacity for interpretability and determinism.

---

## Design Rationale

### Why only 3 dimensions? Isn't that too limited?

Yes and no. 3D is a **deliberate constraint**:

**Advantages**:
- Human-visualizable (can be rendered as colors)
- Geometrically interpretable (distances have meaning)
- Auditable (easy to explain decisions)
- Compact storage (6-character hex codes)

**Limitations**:
- ~10³–10⁴ meaningful semantic regions (not millions)
- Not suitable for complex learned representations
- Coarse-grained preferences only

VRGB targets domains where **interpretability matters more than capacity**.

### When should I use VRGB vs. traditional embeddings?

| Use VRGB When | Use Embeddings When |
|---------------|---------------------|
| Coarse categories (severity levels, style profiles) | Fine-grained semantics (language, vision) |
| Interpretability required (audit, compliance) | Black-box acceptable |
| Semantics evolve frequently | Representations relatively stable |
| Cross-domain reuse valuable | Single-domain specialized |
| 3D geometric structure natural | High-dimensional structure needed |

**Hybrid approach**: Use VRGB for coarse routing, embeddings for fine-grained decisions.

### Why not use 4D or 5D coordinates for more capacity?

VRGB leverages **existing colorspace theory** (LAB, HSL, etc.) which is inherently 3D. Moving beyond 3D loses:
- Visual interpretability (can't render as colors)
- Standard colorspace transforms
- Perceptual uniformity research
- Human intuition about color geometry

If you need more dimensions, consider composing multiple VRGB coordinates or using traditional embeddings.

---

## Colorspaces and Transforms

### Which colorspace should I choose?

| Colorspace | Best For | Geometry | Notes |
|------------|----------|----------|-------|
| **LAB** | Severity, urgency, continuous scales | Perceptually uniform | Best for medical, priority systems |
| **HSL** | Style, categories with intensity | Cylindrical (hue wraps) | Good for aesthetic preferences |
| **RGB** | Simple orthogonal axes | Cartesian cube | Easy to implement, non-uniform |
| **HSV** | Similar to HSL | Cylindrical | Value vs. lightness |

**Rule of thumb**: If your semantics have "how much" (severity, urgency), use LAB. If they have "what kind" (style, category), use HSL.

### What happens when coordinates are out of gamut?

Some LAB coordinates don't map to valid sRGB colors (outside the RGB gamut). VRGB requires explicit **clipping policies**:

- `perceptual_clamp`: Find nearest perceptually similar in-gamut color
- `hard_clip`: Clamp each channel independently
- `flag_and_warn`: Return error, reduce confidence

This is why schemas specify `reference_white`, `adaptation`, and `clipping_policy` in their transform config.

### Can I switch colorspaces later?

**Within the same lineage**: No. All schemas in a lineage must use the same transform for geometric consistency.

**New lineage**: Yes. Create a new `lineage_id` with the new transform. Old data can be re-interpreted or migrated.

---

## Schemas and Versioning

### How often should I version my schema?

Create a new version when you:
- Add anchors to improve coverage
- Refine dimension ranges
- Adjust confidence thresholds
- Add new metadata or audit features

**Don't** create versions for:
- Cosmetic changes (label wording)
- Application logic changes (use business logic layer)

### What counts as a "breaking change" requiring a new lineage?

New lineage required when:
- Changing the colorspace transform (LAB → HSL)
- Reversing dimension semantics (low→high becomes high→low)
- Fundamentally reassigning what dimensions mean

New version sufficient when:
- Adding anchors
- Refining ranges
- Improving confidence calculation
- Extending (not changing) semantics

### How do I handle schema migration?

VRGB is designed to **avoid migration**. Old and new schemas can coexist:

```python
# Gradual rollout
if user_cohort == "beta":
    schema = load_schema("v3.0")
else:
    schema = load_schema("v2.1")

params, confidence = schema.interpret(user_coordinate)
```

When a breaking change requires new lineage:
1. Create new lineage with new transform
2. Re-interpret existing coordinates with both schemas
3. Analyze semantic drift, adjust if needed
4. Gradually migrate users to new lineage

---

## Implementation

### What's the performance overhead?

VRGB interpretation is **deterministic and fast**:
- Colorspace decode: ~1-5 μs (microseconds)
- Schema mapping: ~1-2 μs
- Confidence calculation: ~5-10 μs (depends on anchor count)

**Total**: ~10-20 μs per interpretation (50,000-100,000 ops/sec single-threaded)

Not suitable for **sub-millisecond latency** requirements, but fine for typical web/API use cases.

### Can I cache interpretations?

Yes, with caveats:

```python
# Safe to cache (coordinate + schema version → deterministic)
cache_key = f"{coordinate}:{schema_version}"
if cache_key in cache:
    return cache[cache_key]

params, confidence = schema.interpret(coordinate)
cache[cache_key] = (params, confidence)
```

**Important**: Invalidate cache when schema version changes.

### How do I validate schemas?

Define a JSON schema validator:

```python
def validate_schema(schema_json):
    # Check required fields
    assert "domain" in schema
    assert "lineage_id" in schema
    assert "transform" in schema

    # Validate transform consistency
    if schema["transform"]["colorspace"] == "lab":
        assert "reference_white" in schema["transform"]
        assert "clipping_policy" in schema["transform"]

    # Validate anchors
    for anchor in schema["anchors"]:
        assert is_valid_hex(anchor["latent"])
        # Verify decode matches declared decoded values
        decoded = decode(anchor["latent"], schema["transform"])
        assert approx_equal(decoded, anchor["decoded_lab"])

    return schema_hash(schema)
```

---

## Use Cases

### Can VRGB handle user-facing preference sliders?

Yes! This is a strong use case:

```
UI Layer:
  Severity slider [Low ─────●─── High]
  Urgency slider  [Later ──────● Immediate]

↓ Map to LAB coordinates

Storage Layer:
  Store #FF3300

↓ Interpret via schema

Application Layer:
  severity: 87, urgency: 92
```

Users interact with semantic sliders, but storage uses geometric coordinates. Schema evolution can adjust mapping without UI changes.

### How does VRGB handle multi-user analytics?

You can aggregate coordinates geometrically:

```python
# Collect coordinates from user cohort
cohort_coords = ["#FF3300", "#FF4411", "#EE2200"]

# Average in LAB space (more perceptually meaningful than RGB)
lab_coords = [decode_lab(c) for c in cohort_coords]
avg_lab = mean(lab_coords)
avg_hex = encode_lab(avg_lab)

# Interpret the average
params, confidence = schema.interpret(avg_hex)
# → Representative preferences for the cohort
```

**Warning**: Always check confidence. Averaged coordinates may land in low-confidence regions.

### Can I use VRGB for A/B testing schemas?

Absolutely! This is a core design goal:

```python
def get_schema_for_user(user_id):
    if user_id in ab_test_group_A:
        return load_schema("blood_panel_v2.1")
    else:
        return load_schema("blood_panel_v3.0_beta")

params, confidence = get_schema_for_user(user_id).interpret(coordinate)
```

Same coordinates, different interpretations. Track which schema version performs better.

### How do I handle missing or invalid coordinates?

Provide defaults and validate on read:

```python
def get_user_preference(user_id, domain, default="#808080"):
    coordinate = db.get(user_id, domain)

    if not coordinate or not is_valid_hex(coordinate):
        coordinate = default  # Neutral gray

    params, confidence = schema.interpret(coordinate)

    if confidence < schema.min_confidence:
        log_warning(f"Low confidence for user {user_id}: {confidence}")
        # Optionally fall back to defaults or flag for review

    return params, confidence
```

---

## Security and Privacy

### Are color coordinates sensitive data?

**Depends on context**. Coordinates themselves are geometric, but they encode **preferences** which may be sensitive:

- Medical urgency preferences: **PHI/sensitive**
- Dietary restrictions: **potentially sensitive**
- Meeting priorities: **business confidential**
- UI theme preferences: **not sensitive**

Treat coordinates with the same privacy/security posture as the domain they represent.

### Can someone reverse-engineer semantics from coordinates?

**Not without schemas**. VRGB's security model:

**Public/shareable**:
- Hex coordinates (#FF3300)
- Colorspace transforms (mathematical functions)

**Private/access-controlled**:
- Schemas (dimension mappings, semantic labels)
- Anchors (what coordinates mean)
- Confidence thresholds

If you control schema access, external parties see only geometric coordinates without semantic context.

### How do I audit interpretations?

Log complete audit records:

```json
{
  "interpretation_id": "uuid-123",
  "timestamp": "2025-11-20T14:30:00Z",
  "user_id": "alice",
  "coordinate": "#FF3300",
  "schema_ref": {
    "domain": "medical",
    "version": "v2.1",
    "hash": "sha256-abc123..."
  },
  "output": {"severity": 87, "urgency": 92},
  "confidence": 0.92,
  "decision": "notify_doctor"
}
```

Given the schema hash, you can reproduce the exact interpretation later (schema versioning + deterministic interpretation = full auditability).

---

## Limitations

### What are VRGB's biggest weaknesses?

1. **Capacity**: Only ~10³–10⁴ distinct semantic regions. Not suitable for fine-grained embeddings.
2. **Learning**: Cannot discover emergent patterns from data. Schemas must be hand-authored.
3. **Dimensionality**: 3D geometry doesn't fit all domains. Some semantics are inherently high-dimensional.
4. **Coordination**: Cross-domain interpretation requires deliberate anchor alignment or empirical correlation analysis.

### When should I NOT use VRGB?

Avoid VRGB for:
- **Language embeddings** (use BERT, GPT, etc.)
- **Image embeddings** (use CLIP, ResNet features, etc.)
- **Real-time inference** (<1ms latency requirements)
- **Learned representations** (when geometry should be discovered from data)
- **Fine-grained continuous parameters** (thousands of distinct values)

### Can VRGB replace my database schema?

No. VRGB is **complementary** to databases:

```
Database stores:
  - user_id (primary key)
  - vrgb_coordinate (geometric preference)
  - other_fields (structured data)

VRGB interprets:
  - vrgb_coordinate → semantic parameters
  - via versioned schemas
```

Use traditional databases for structured data. Use VRGB for **preferences and interpretable geometric encoding** within that structure.

---

## Comparisons

### How does VRGB compare to feature flags?

| Aspect | VRGB | Feature Flags |
|--------|------|---------------|
| Granularity | Continuous (3D space) | Binary (on/off) |
| Stability | Coordinates stable | Flags change |
| Versioning | Schema versions | Flag deprecation |
| Use case | User preferences | Code deployment |

**Complementary**: Use feature flags for code, VRGB for preferences.

### How does VRGB compare to configuration management?

Similar pattern, different scale:

- **Config management**: System-level settings (DB connection strings, API keys)
- **VRGB**: User-level or instance-level preferences (severity thresholds, style profiles)

VRGB adds:
- Geometric structure (distances, interpolation)
- Cross-domain interpretation
- Confidence metrics

### Is VRGB related to vector databases (Pinecone, Weaviate)?

**Different purpose**:
- **Vector DBs**: Store high-dimensional learned embeddings, optimize nearest-neighbor search
- **VRGB**: Store low-dimensional geometric coordinates, optimize interpretability and versioning

**Possible synergy**: Use vector DB for learned embeddings, map to VRGB for human-interpretable interface layer.

---

## Future Directions

### Will there be reference implementations?

Planned for future releases:
- Python reference implementation
- Schema validation toolkit
- Colorspace transform library with benchmarks
- Example web UI with color pickers + semantic sliders

See [CHANGELOG.md](../CHANGELOG.md) for roadmap.

### Can VRGB support more than 3 dimensions?

Not without losing core benefits (visual interpretability, colorspace theory). If you need >3D:
- Compose multiple VRGB coordinates
- Use traditional embeddings
- Explore other geometric structures (e.g., quaternions for 4D)

### Will VRGB support approximate/probabilistic interpretation?

Potentially. The white paper sketches how approximate computation could trade accuracy for scalability. Future work may explore:
- Locality-sensitive hashing (LSH) for fast approximate nearest-anchor
- Approximate colorspace transforms
- Probabilistic confidence calibration

---

## Getting Started

### I'm new to VRGB. Where should I start?

1. **Read the [Architecture Overview](architecture-overview.md)** for visual/conceptual intro
2. **Skim the [white paper](../VRGB-whitepaper.md)** for formal definitions
3. **Explore [examples](../examples/)** for concrete schemas
4. **Prototype** a schema for your domain
5. **Test** interpretations with different coordinates
6. **Iterate** on anchor placement and confidence thresholds

### Where can I get help?

- **Issues**: Open a GitHub issue with the `question` label
- **Discussions**: Use GitHub Discussions for design patterns and use cases
- **Implementation**: Reference implementations coming soon (see CHANGELOG)

### How can I contribute?

Contributions welcome for:
- Example schemas (new domains)
- Schema design patterns
- Implementation feedback
- Theoretical extensions

See repository README for contribution guidelines.

---

**Last Updated**: November 21, 2025
