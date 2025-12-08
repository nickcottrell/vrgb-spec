# VRGB: Stable Latent Coordinates for Semantic Preference Encoding

**A deterministic system for encoding preferences as 3D color coordinates with versioned semantic interpretation**

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/Version-2.6-green.svg)](CHANGELOG.md)

## Overview

VRGB borrows the architectural pattern that made Stable Diffusion transformative—**separate what doesn't change from what must evolve**—and applies it to semantic preferences using colorspace geometry instead of learned latent vectors.

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
│   └── meeting-urgency-schema.json   # Priority/urgency routing
└── assets/                            # Diagrams and visual aids
```

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

Reference implementations and tooling will be developed in separate repositories.

## Related Work

VRGB builds on established research in:

- **Latent diffusion models** (Rombach et al., 2022) — Architectural pattern
- **Perceptual color spaces** (Wyszecki & Stiles, 2000) — Geometric foundations
- **Explainable AI** — Interpretability and auditability requirements

See the [white paper](VRGB-whitepaper.md) for complete references.

---

**Status**: Research specification (v2.6)
**Last Updated**: November 21, 2025
