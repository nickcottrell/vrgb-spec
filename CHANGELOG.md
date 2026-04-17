# Changelog

All notable changes to the VRGB specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.7] - 2026-04-17

### Added

- §9 **Geometric Security Model**: four assertive claims (accountable privacy as distinct category; geometric-derivable security; VRGB as privacy primitive; spectral binding as economic deterrent), explicit threat model, accountable-privacy definition, four manifestation patterns (cryptographic commitment, ZK proofs, geometric conformance, consent-gated audit), Tor comparison, geometric-impossibility design constraint.
- §10 **Operating Doctrine**: stance, policy, the forkable pattern (stance → threat model → architectural constraints → cryptographic primitives → published contract), audience / not-for clause, observers-welcome clause, propagation doctrine.

### Changed

- Renumbered downstream sections: Implementation Sketch (was §9, now §11), Related Work (was §10, now §12), Conclusion (was §11, now §13). Subsection numbering updated accordingly.
- Table of contents updated to reflect new sections and renumbered anchors.

### Rationale

Previously, the security stance and operating doctrine lived in separate vault entries and planning notes. Folding them into the canonical whitepaper creates a single provenance chain: anyone reading the technical spec now sees the security model and doctrine backing it, and cannot quote the doctrine without the math that supports it.

## [2.6] - 2025-11-21

### Added

- Initial public release of VRGB specification
- Complete formalization of Stable Latent / Mutable Interpretation (SLI) pattern
- Detailed colorspace transform specifications with gamut handling
- Schema versioning and lineage system
- Confidence metric framework with cascading heuristics
- Security and auditability section with Merkle-style schema hashing
- Interpretation audit record format
- Complete schema example (blood panel analysis) in Appendix A
- Implementation sketch with Python-like API design
- Cross-domain interpretation framework
- Schema design guidelines

### Documented

- Four-layer architecture (storage, interpretation, application, governance)
- Four design invariants (I1-I4) for system stability
- Information capacity analysis (24-bit address space implications)
- Geometric properties of LAB, HSL, HSV colorspace manifolds
- Hybrid architecture patterns with high-dimensional embeddings
- Related work positioning vs. latent diffusion, embeddings, color theory

### Clarified

- VRGB is a pattern instantiation, not a learned model
- 3D colorspace capacity constraints and appropriate use cases
- Deterministic interpretation vs. stochastic diffusion models
- Semantic privacy boundaries (coordinates public, schemas private)
- Transform clipping policies for gamut-dependent colorspaces
- Lineage consistency requirements for schema evolution

## [Unreleased]

### Planned

- Reference implementation in Python
- Schema validation toolkit
- Colorspace transform library with benchmark suite
- Example schemas for additional domains (fitness, content moderation, routing)
- Confidence calibration guidelines based on domain testing
- Migration guide for schema version upgrades
- Performance benchmarks for interpretation at scale

---

**Note**: Prior to v2.6, VRGB existed as internal design documents and prototypes. This changelog begins with the first public specification release.
