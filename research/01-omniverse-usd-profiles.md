# 1. Omniverse USD Profiles (`omni.capabilities`)

## Overview

NVIDIA's implementation of a USD profiles/capabilities system, shipped as the `omni.capabilities` Python package bundled inside `omniverse-asset-validator` ≥ 1.11.2. **Generated** by `omniverse-usd-profiles` (PyPI v1.11.0) from markdown spec files in the `nvidia/simready-foundation` repository. Not published independently on PyPI. Apache-2.0 licensed (NVIDIA copyright 2025–2026).

## Purpose

Defines a structured, versioned taxonomy of **requirements**, **capabilities**, **features**, and **profiles** for validating USD assets. The system lets tool authors declare what properties a "good" USD asset should have, grouped into progressively broader units.

## Architecture

### Four-level hierarchy

```
Requirement → Capability → Feature → Profile
```

1. **Requirement** — The atomic unit. A single testable rule with a unique code (e.g. `VG.014`), version, human-readable message, compatibility tag (e.g. `core-usd`, `rtx`, `physx`, `open-usd`), documentation path, and tags (e.g. `correctness`, `performance`, `essential`, `limitation`). May have parameters and examples.

2. **Capability** — Groups related requirements. Examples: `geometry`, `hierarchy`, `units`, `materials`, `physics_rigid_bodies`, `physics_joints`, `semantic_labels`, `dense_captions`, `nonvisual_materials`, `atomic_asset`. Each has an id, semver version, docs path, and a list of `Requirement` enums.

3. **Feature** — A cross-capability selection of requirements forming a user-facing use case. Examples: `minimal_placeable_visual` (v1.0.0), `performant_placeable_visual` (v0.0.1). Features cherry-pick specific requirements from across capabilities.

4. **Profile** — The top-level grouping. Has an id, version, features list, and capabilities list. The `Profiles` enum is currently empty (no profiles defined yet), suggesting this layer is still under development.

### Key implementation details

- All data structures are **frozen dataclasses + Enums** — immutable, self-documenting, discoverable at import time.
- `VersionedRegistry[T]` provides singleton registries with semver-based lookup (`find(id, version=None)` returns latest by default).
- Protocol classes (`RequirementProtocol`, `CapabilityProtocol`, `FeatureProtocol`, `ProfileProtocol`) enable duck-typing against external implementations.
- Compatibility tags scope rules to runtimes: `core-usd` (universal), `open-usd` (OpenUSD spec), `rtx` (NVIDIA RTX), `physx` (PhysX), `kit` (Omniverse Kit), `kit-107+`, `omniverse`, `other`.
- Requirements can carry **parameters** (e.g. `USE_GPU`, `CHECK_TRANSPARENCY`, `SIZE_THRESHOLD`, `TRANSFORM_TOLERANCE`) that configure rule behavior.
- Requirements carry **examples** with snippets (language-tagged: `python`, `usda`) and expected results (`OK`, `NOK`).

### Integration with asset validator

The `omniverse-asset-validator` package imports from `omni.capabilities` and maps these requirements to its own validation rules. The `_profiles.py` and `_capabilities.py` within the asset validator provide `ProfileRegistry` and `CapabilityRegistry` wrappers.

### Bridge to Pixar UsdValidation

`_usd_validator_adapter.py` provides protocol compatibility with Pixar's `UsdValidation` framework (`ValidatorProtocol`, `ValidatorErrorProtocol`, `ValidatorErrorSiteProtocol`), suggesting NVIDIA is actively bridging between the two systems.

## Current Capabilities (v1.11.2)

| Capability | Requirements | Tags |
|---|---|---|
| `geometry` | 35 rules (VG.*) | correctness, performance, essential |
| `hierarchy` | 10 rules (HI.*) | correctness, performance, essential |
| `units` | 7 rules (UN.*) | correctness, essential |
| `materials` | 5 rules (VM.*) | correctness, performance |
| `physics_rigid_bodies` | 10 rules (RB.*) | correctness, essential, high-quality, limitation |
| `physics_joints` | 7 rules (JT.*) | correctness, essential, high-quality, limitation |
| `atomic_asset` | 4 rules (AA.*) | essential, limitation |
| `semantic_labels` | 4 rules (SL.*) | correctness, essential, limitation |
| `dense_captions` | 2 rules (DC.*) | essential, high-quality |
| `nonvisual_materials` | 6 rules (NVM.*) | correctness, essential |

**Total: ~90 requirements across 10 capabilities and 2 features.**

## Status

- **Active development** (copyright 2025–2026)
- Bundled in `omniverse-asset-validator` but architecturally separate
- Profile layer (the top-level `Profiles` enum) is empty — no concrete profiles defined yet
- Features layer has two entries (`minimal_placeable_visual`, `performant_placeable_visual`)
- Uses semver for all entities
- No standalone PyPI package found; may be planned

## Source

- Package: `omniverse-asset-validator` 1.11.2 (PyPI)
- Module: `omni.capabilities` (bundled, no separate distribution)
- License: Apache-2.0
