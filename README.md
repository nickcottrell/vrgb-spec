# VRGB: Stable Latent Coordinates for Semantic Preference Encoding

**A deterministic system for encoding preferences as 3D color coordinates with versioned semantic interpretation**

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/Version-2.6-green.svg)](CHANGELOG.md)

## Overview

Virtual RGB (VRGB) borrows the architectural pattern that made Stable Diffusion transformative—**separate what doesn't change from what must evolve**—and applies it to semantic preferences using colorspace geometry instead of learned latent vectors.

```
Fixed 3D Colorspace Coordinate (#FF3300)
    ↓
Versioned Interpretation Schema (LAB, HSL, RGB)
    ↓
Domain-Specific Parameters (severity, urgency, preferences)
```

Unlike image diffusion models, VRGB is **deterministic**, **interpretable**, and uses **hand-designed geometric structures** rather than learned representations. The same coordinate can be interpreted across multiple domains—medical urgency, dietary preferences, meeting priority—while remaining stable as semantic definitions evolve.

## Core Insight

When Stable Diffusion separated stable latent coordinates from mutable decoder models, it solved a fundamental problem: **how to evolve AI systems without migrating stored data**. VRGB demonstrates this pattern works beyond learned representations:

- **Stable**: Coordinates stored as hex colors (`#RRGGBB`) never change
- **Mutable**: Schemas can evolve, improve, or be A/B tested without touching stored data
- **Interpretable**: 3D geometric space is human-visualizable and auditable
- **Cross-domain**: One coordinate, multiple interpretations across contexts

## Key Features

- **24-bit address space** (16.7M discrete coordinates) using standard colorspace geometry
- **Deterministic interpretation** through versioned JSON schemas
- **Confidence metrics** for every interpretation with configurable thresholds
- **Complete audit trails** with schema hashing and interpretation logging
- **Transform-agnostic** support for RGB, HSL, HSV, LAB, XYZ and custom colorspaces
- **Semantic privacy boundaries** separating coordinates (shareable) from schemas (access-controlled)

## Documentation

- **[Full White Paper](VRGB-whitepaper.md)** — Complete technical specification (v2.6)
- **[Architecture Overview](docs/architecture-overview.md)** — Simplified visual explanation
- **[FAQ](docs/faq.md)** — Common questions and use cases
- **[Examples](examples/)** — Complete schema implementations
- **[Changelog](CHANGELOG.md)** — Version history

## Quick Start

### Understanding VRGB in 3 Steps

**1. Store a Coordinate**
```json
{
  "user_id": "alice",
  "preference_coordinate": "#FF3300"
}
```

**2. Define a Schema**
```json
{
  "domain": "medical_urgency",
  "transform": { "colorspace": "lab" },
  "dimensions": {
    "L": { "param": "severity", "range": [0, 100] },
    "a": { "param": "urgency", "range": [-128, 127] },
    "b": { "param": "temporal", "range": [-128, 127] }
  }
}
```

**3. Interpret Across Contexts**
```python
# Same coordinate, different interpretations
medical_schema.interpret("#FF3300")
# → {severity: 87, urgency: 92, confidence: 0.92}

dietary_schema.interpret("#FF3300")
# → {intensity: "strong", style: "bold", confidence: 0.85}
```

See [examples/](examples/) for complete, production-ready schemas.

## Use Cases

VRGB is designed for scenarios where:

- **Semantics evolve** faster than data can migrate (policy changes, clinical guidelines)
- **Multiple models coexist** (A/B testing, gradual rollouts, multi-tenant systems)
- **Interpretability matters** (regulated domains, audit requirements, explainable AI)
- **Cross-domain reuse** creates value (unified user profiles, correlation analysis)

### Appropriate Domains

- Clinical severity and urgency scoring
- Dietary preferences and restrictions
- Content moderation policies
- Meeting/task prioritization
- Style and aesthetic preferences
- Service routing decisions

### Not Appropriate For

- High-dimensional semantic embeddings (use transformers/CLIP instead)
- Fine-grained continuous parameters (> 10³ distinct values)
- Learned emergent representations
- Real-time latency-critical systems requiring approximate computation

## Repository Structure

```
vrgb-spec/
├── README.md                          # This file
├── VRGB-whitepaper.md                 # Complete technical specification
├── LICENSE                            # Apache 2.0 license
├── CITATION.cff                       # Academic citation format
├── CHANGELOG.md                       # Version history
├── .gitignore                         # Git ignore rules
├── docs/                              # Additional documentation
│   ├── architecture-overview.md       # Visual/simplified explanation
│   └── faq.md                         # Common questions
├── examples/                          # Schema examples
│   ├── README.md                      # Example documentation
│   ├── blood-panel-schema.json       # Medical severity example
│   ├── recipe-recommendation-schema.json  # Dietary preferences
│   ├── meeting-urgency-schema.json   # Priority/urgency routing
│   └── kafka-event-routing-schema.json # Kafka semantic event routing
└── assets/                            # Diagrams and visual aids
```

## Development Setup

VRGB is a specification-first repository: you contribute by improving schemas, examples, and documentation.

### Prerequisites

- **Git** for cloning and submitting pull requests
- **Python 3.10+** if you want to run the JSON loading/snippet checks shown in docs
- A **JSON-aware editor** (VS Code, Cursor, or equivalent) for schema authoring

### Clone and open locally

```bash
git clone https://github.com/nickcottrell/vrgb-spec.git
cd vrgb-spec
```

There are no package dependencies to install for core contribution work in this repository.

### Validate your changes locally

- Ensure edited JSON files parse cleanly
- Compare your schema against the structure in [examples/README.md](examples/README.md)
- Re-check anchor coverage and confidence thresholds before opening a PR

### Editor and tooling recommendations

- **VS Code + JSON language support** for schema editing and quick syntax feedback
- **Markdown preview** for documentation changes
- Optional CLI helpers for quick checks:
  - `python -m json.tool <file>`
  - `jq . <file>` (if installed)

## Your First Schema (Step-by-Step)

Use the blood panel example as a starting point so you follow established VRGB conventions.

### 1. Copy an existing example

```bash
cp examples/blood-panel-schema.json examples/my-first-schema.json
```

### 2. Update identity fields

Edit these fields first:

- `domain` (what the schema is for)
- `lineage_id` (transform family identifier)
- `schema_version` (semantic version)
- `description` (clear, domain-specific intent)

### 3. Define your dimensions

Map each transform axis to a stable semantic parameter:

```json
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
}
```

### 4. Add anchors that cover meaningful regions

Start with 4-6 anchors that represent low/mid/high semantics and key boundaries:

```json
"anchors": [
  {
    "id": "routine_screening",
    "latent": "#707070",
    "label": "Routine Annual Screening"
  },
  {
    "id": "acute_concern",
    "latent": "#FF3300",
    "label": "Acute Clinical Concern"
  }
]
```

### 5. Validate syntax and required keys

```bash
python -m json.tool examples/my-first-schema.json >/dev/null
```

```python
import json

required = ["domain", "lineage_id", "schema_version", "transform", "dimensions", "anchors", "quality_config"]
with open("examples/my-first-schema.json", "r", encoding="utf-8") as f:
    schema = json.load(f)

missing = [k for k in required if k not in schema]
if missing:
    raise ValueError(f"Missing required keys: {missing}")

print("Schema shape check passed.")
```

### 6. Test the schema qualitatively

- Review whether anchors reflect real semantic boundaries in your domain
- Confirm ranges match the transform you selected
- Ensure `quality_config.min_confidence` fits domain risk (higher for safety-critical domains)
- Compare with [examples/blood-panel-schema.json](examples/blood-panel-schema.json) and [examples/recipe-recommendation-schema.json](examples/recipe-recommendation-schema.json)

## Local Testing

This repository does not include a dedicated automated test runner. Local testing is schema and documentation focused.

### Common contributor commands

```bash
# Validate a single schema file
python -m json.tool examples/blood-panel-schema.json >/dev/null

# Validate all example schemas
for f in examples/*.json; do python -m json.tool "$f" >/dev/null; done

# Check repository status before committing
git --no-pager status --short
```

### What to verify before opening a PR

- JSON is valid and consistently formatted
- `dimensions` ranges align with the declared colorspace
- Anchor set is not sparse for the intended confidence thresholds
- Documentation links and examples still resolve correctly

## Related Projects

Reference implementations and tooling that build on this specification:

- **[vrgb-kafka](https://github.com/nickcottrell/vrgb-kafka)** — Semantic Kafka event routing using fixed-size color headers instead of payload inspection
- **[vrgb-benchmarks](https://github.com/nickcottrell/vrgb-benchmarks)** — Performance benchmarks for VRGB-based routing and interpretation

## Citation

If you reference VRGB in academic work, please cite:

```bibtex
@techreport{cottrell2025vrgb,
  author = {Cottrell, Nicholas},
  title = {VRGB: Stable Latent Coordinates for Semantic Preference Encoding},
  year = {2025},
  month = {November},
  version = {2.6},
  url = {https://github.com/nickcottrell/vrgb-spec}
}
```

See [CITATION.cff](CITATION.cff) for machine-readable citation metadata.

## License

Copyright 2025 Nicholas Cottrell

Licensed under the Apache License, Version 2.0 (the "License"). See [LICENSE](LICENSE) for the full license text.

## Contributing

This repository contains the formal specification for VRGB. For discussion of:

- **Theoretical extensions** — Open an issue with the `theory` label
- **Implementation questions** — Open an issue with the `implementation` label
- **Schema design patterns** — Share in discussions or submit example PRs

### How to contribute

We welcome contributions in three main areas:

1. **Schemas and examples** — Add new domain schemas under `examples/` or improve anchor coverage, confidence tuning, and documentation in existing examples.
2. **Specification and docs** — Clarify architecture, add design guidance, and improve onboarding content in `README.md` and `docs/`.
3. **Ecosystem alignment** — Improve references to implementation repositories (for example, `vrgb-kafka`) so patterns remain consistent between spec and practice.

### Good first contributions

- Add a new example schema for a clearly bounded domain
- Improve explanatory notes for one existing example in `examples/README.md`
- Add FAQ entries for schema evolution, lineage, or confidence tuning edge cases
- Fix unclear wording or broken links in docs

### Pull request checklist

Before submitting, confirm:

- [ ] Your change is scoped and clearly documented
- [ ] Any new/edited JSON schema parses successfully
- [ ] You explained domain rationale (dimensions, anchors, thresholds)
- [ ] Related docs were updated when behavior/meaning changed
- [ ] Commit and PR messages describe *why* the change is needed

If you are unsure about schema design tradeoffs, open a discussion or draft PR early.

### Code of conduct

Please follow the standards in [GitHub Community Guidelines](https://docs.github.com/en/site-policy/github-terms/github-community-guidelines). If this repository adopts a dedicated `CODE_OF_CONDUCT.md`, follow that document as the project-specific source of truth.

Reference implementations and tooling are developed in separate repositories.

## Troubleshooting

### JSON parse errors

- Run `python -m json.tool <file>` to find structural issues
- Watch for trailing commas, unmatched braces, or invalid quote usage

### Unsure which colorspace to use

- Use **LAB** for continuous severity/urgency style scoring
- Use **HSL** for category + intensity style mappings
- Use **RGB** for largely independent orthogonal axes

See [examples/README.md](examples/README.md) for pattern guidance.

### Low confidence in large regions

Low confidence usually means anchor coverage is too sparse. Add anchors at:

- semantic extremes
- decision boundaries
- common real-world cases

### Lineage/version confusion

- Keep the same `lineage_id` when refining semantics within the same transform family
- Start a new lineage if you change transform strategy (for example, HSL → LAB)
- Increment `schema_version` whenever interpretation semantics change

## Related Work

VRGB builds on established research in:

- **Latent diffusion models** (Rombach et al., 2022) — Architectural pattern
- **Perceptual color spaces** (Wyszecki & Stiles, 2000) — Geometric foundations
- **Explainable AI** — Interpretability and auditability requirements

See the [white paper](VRGB-whitepaper.md) for complete references.

---

**Status**: Research specification (v2.6)
**Last Updated**: November 21, 2025
