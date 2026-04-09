# Namespace Conformance: Implementation Plan

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23
**Prerequisite:** Stakeholder alignment on `research/namespace-ownership-mapping.md`

## Background

The Omniverse Asset Validator (OAV) and SimReady Foundation (SRF) both validate OpenUSD assets. OAV provides ~40 general-purpose checker classes. SRF provides ~84 simulation-specific checker classes plus spec definitions for 118 requirements across 6 profiles, 22 feature variants, and 16 capabilities.

A capability DAG system has been built (`omniverse-usd-profiles` library) that connects these pieces: SRF profiles → features → capabilities → requirements → checker classes. This DAG drives profile-based validation — "validate this asset against Prop-Robotics-Neutral" runs exactly the right set of checkers.

This plan addresses two problems in the current system:

1. **Namespace inconsistency** — DAG node IDs use 5 different naming conventions
2. **Ownership ambiguity** — requirement codes don't indicate which package owns them

## Current State

The capability DAG currently produces IDs from different sources with inconsistent conventions:

```
com.nvidia.simready                          ← namespace (hardcoded in codegen)
com.nvidia.simready.geometry                 ← capability (from markdown directory name)
com.nvidia.simready.FET001_BASE_NEUTRAL      ← feature variant (from Internal ID in markdown)
com.nvidia.simready.Prop-Robotics-Neutral    ← profile (from TOML table header)
com.nvidia.simready.fet_001_minimal          ← feature (from markdown filename)
```

Requirement codes (e.g. `VG.014`) are flat strings with short prefixes that don't indicate which package owns them or what domain they belong to.

## Target State

All IDs follow a consistent convention: **lowercase, dot-separated, fully namespaced**.

### DAG node IDs

```
com.nvidia.usd.core                                    ← OAV capability
com.nvidia.usd.geom                                    ← OAV capability
com.nvidia.usd.physics                                 ← OAV capability
com.nvidia.usd.shade                                   ← OAV capability

com.nvidia.simready                                    ← SRF root namespace
com.nvidia.simready.capabilities.hierarchy             ← SRF capability
com.nvidia.simready.capabilities.geom                  ← SRF capability (opinionated)
com.nvidia.simready.features.minimal_neutral           ← SRF feature variant
com.nvidia.simready.features.rigid_body_physics_physx  ← SRF feature variant
com.nvidia.simready.profiles.prop_robotics_neutral     ← SRF profile
```

### Requirement codes

```
com.nvidia.usd.geom.014                 ← OAV: mesh topology valid
com.nvidia.usd.physics.rb.003           ← OAV: rigid body on Xformable
com.nvidia.simready.geom.001            ← SRF: must contain mesh
com.nvidia.simready.units.006           ← SRF: upAxis = Z
```

### DAG predecessor edges

SRF features inherit OAV checks via DAG predecessors:

```
com.nvidia.simready.features.minimal_neutral
  ├── com.nvidia.simready.capabilities.hierarchy
  ├── com.nvidia.simready.capabilities.units
  ├── com.nvidia.usd.core
  └── com.nvidia.usd.geom
```

## What Changes

### 1. Codegen mapping table

Add a mapping in `omniverse-usd-profiles` that:
- Routes each requirement code to `com.nvidia.usd.*` or `com.nvidia.simready.*`
- Normalizes all DAG node IDs to lowercase dot-separated
- Normalizes feature variant IDs from `FET001_BASE_NEUTRAL` to `minimal_neutral`
- Normalizes profile IDs from `Prop-Robotics-Neutral` to `prop_robotics_neutral`

### 2. Renumber requirement codes

Current codes (`VG.014`, `HI.001`) become fully namespaced (`com.nvidia.usd.geom.014`). This is a breaking change for any code that references requirement codes by string.

**Impact:** OAV checker decorators, SRF validation.py decorators, generated `omni.capabilities` enums, generated `simready_validators.capabilities` enums. All need updating.

### 3. Split capabilities by ownership

Current: one `geometry` capability with 31 requirements.
After: `com.nvidia.usd.geom` (28 requirements) + `com.nvidia.simready.geom` (3 requirements).

SRF's geometry capability inherits from OAV's via the DAG:
```json
"com.nvidia.simready.capabilities.geom": {
    "predecessors": ["com.nvidia.simready", "com.nvidia.usd.geom"]
}
```

### 4. Regenerate all outputs

Run codegen with the new mapping → new `capabilities.json`, new generated Python enums, new DAG.

## Migration Considerations

This is a pre-release system — no external users depend on specific requirement codes or DAG node IDs yet. The migration window is now, before the first public release.

**Breaking changes:**
- Requirement code strings change everywhere (decorators, enums, tests)
- DAG node IDs change (capabilities.json, graph queries)
- OAV `omni.capabilities` enums would need regeneration

**Non-breaking:**
- Checker class names stay the same
- Validation logic stays the same
- Profile/feature structure stays the same
- The graph loader resolves by code, so it adapts automatically once codes are updated

## Repos Affected

| Repo | Change | Scope |
|------|--------|-------|
| `open-usd-profiles` | Codegen mapping table + ID normalization | Moderate |
| `simready-foundation` | Update requirement codes in spec markdown + validation.py decorators | Large (118 codes) |
| `asset-validator` | Update requirement codes in `omni.capabilities` + checker decorators | Large |
| `simready-validators` | Regenerate capabilities.json + capabilities enums | Small (regenerated) |
| `jensjebens/OpenUSD` | Update plugInfo.json capability node IDs | Small |

## Recommendation

Get stakeholder alignment on the namespace mapping first (the "what"), then implement the code migration (the "how"). The mapping is documented in `research/namespace-ownership-mapping.md` — 57 codes to `com.nvidia.usd.*`, 61 codes to `com.nvidia.simready.*`.
