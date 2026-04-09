# Namespace Ownership: OAV and SimReady Requirement Codes

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23

## Background

The Omniverse Asset Validator (OAV) and SimReady Foundation (SRF) both define validation requirements for OpenUSD assets. Currently, all requirement codes (e.g. `VG.014`, `HI.001`, `RB.003`) share a flat namespace with short prefixes that don't indicate which package owns them or what domain they belong to.

This document proposes a clear namespace split between OAV and SRF requirements.

## Problem

Today, a developer seeing requirement code `VG.024` cannot tell:
- Is this an OAV check or a SimReady check?
- Which package provides the checker implementation?
- Where to find the spec markdown?
- Will this check eventually become part of upstream USD validation?

Some requirement codes are implemented by **both** OAV and SRF with different checker classes, creating silent registration conflicts.

## Principle

**OAV** checks that a USD file is **structurally valid** — correct schemas, resolvable references, valid topology. These are non-opinionated checks that apply to any USD file regardless of its purpose. They are candidates for upstreaming into Pixar's `UsdValidation` framework.

**SRF** checks that an asset conforms to **SimReady conventions** — specific metadata values, hierarchy layouts, naming rules, simulation-specific features. These are opinionated requirements that apply only to assets targeting SimReady profiles.

**Rule:** If removing the check would produce an invalid USD file, OAV owns it. If removing the check would produce a valid USD file that just doesn't meet a convention, SRF owns it.

## Proposed Namespace Scheme

### OAV requirements: `com.nvidia.usd.<domain>.<number>`

OAV requirements use the `com.nvidia.usd` prefix to signal that these checks validate core USD correctness and are candidates for upstream into `usd.*` when Pixar's UsdValidation framework matures.

| Namespace | Count | Examples |
|-----------|-------|---------|
| `com.nvidia.usd.core` | 7 | defaultPrim exists, upAxis declared, anchored paths |
| `com.nvidia.usd.geom` | 28 | mesh topology, manifold, normals, primvar indexing |
| `com.nvidia.usd.physics` | 14 | rigid body on Xformable, joint targets exist, no nested articulations |
| `com.nvidia.usd.shade` | 8 | material bindings, UsdPreviewSurface, MDL schema |
| **Total** | **57** | |

### SRF requirements: `com.nvidia.simready.<domain>.<number>`

SRF requirements use the `com.nvidia.simready` prefix for simulation-specific conventions.

| Namespace | Count | Examples |
|-----------|-------|---------|
| `com.nvidia.simready.hierarchy` | 7 | kinematic chain, logical grouping, placeable/posable |
| `com.nvidia.simready.geom` | 3 | must contain mesh, origin positioning, RTX extents |
| `com.nvidia.simready.physics` | 9 | rigid body capability, articulation capability, multi-body |
| `com.nvidia.simready.units` | 5 | upAxis=Z, metersPerUnit=1.0, kilogramsPerUnit |
| `com.nvidia.simready.naming_paths` | 8 | prim naming, file naming, directory structure |
| `com.nvidia.simready.semantic_labels` | 4 | semantic labels required, Q-code validation |
| `com.nvidia.simready.nonvisual_materials` | 6 | non-visual material conventions |
| `com.nvidia.simready.driven_joints` | 11 | driven joint conventions |
| `com.nvidia.simready.base_articulation` | 2 | base articulation rules |
| `com.nvidia.simready.*` (misc) | 6 | graspable, PhysX colliders, physics materials |
| **Total** | **61** | |

### Requirement code format

Current codes like `VG.014` become fully namespaced:

```
# Current (ambiguous)
VG.014
HI.001
RB.003

# Proposed (self-describing)
com.nvidia.usd.geom.014         ← OAV: mesh topology valid
com.nvidia.usd.core.hi.001      ← OAV: single root prim
com.nvidia.usd.physics.rb.003   ← OAV: rigid body on Xformable

com.nvidia.simready.geom.001    ← SRF: must contain mesh (was VG.MESH.001)
com.nvidia.simready.units.006   ← SRF: upAxis = Z (was UN.006)
com.nvidia.simready.hierarchy.002  ← SRF: exclusive XForm parent (was HI.002)
```

## Capability DAG Integration

SRF features inherit OAV's base checks via DAG predecessors. This means a SimReady profile automatically includes USD hygiene validation without listing every OAV requirement explicitly.

```
com.nvidia.simready.features.minimal_neutral
  ├── com.nvidia.simready.capabilities.hierarchy
  ├── com.nvidia.simready.capabilities.units
  ├── com.nvidia.usd.core        ← OAV: defaultPrim, upAxis, metersPerUnit
  └── com.nvidia.usd.geom        ← OAV: topology, normals, manifold
```

When Pixar ships native `usd.geom` validators, `com.nvidia.usd.geom` becomes a thin wrapper that delegates to `usd.geom`, and eventually dissolves entirely.

## Current Duplicates (9 codes)

These requirement codes currently have checker implementations in **both** OAV and SRF:

| Code | OAV Checker | SRF Checker | Proposed Owner |
|------|------------|-------------|----------------|
| AA.001 | AnchoredAssetPathsChecker | AnchoredAssetPathsChecker | OAV (general USD) |
| AA.002 | SupportedFileTypesChecker | SupportedFileTypesChecker | OAV (general USD) |
| HI.001 | HierarchyHasRootChecker | HierarchyHasRootChecker | OAV (valid structure) |
| HI.003 | RootPrimXformableChecker | RootPrimXformableChecker | OAV (valid structure) |
| HI.004 | DefaultPrimChecker | StageHasDefaultPrimChecker | OAV (valid structure) |
| UN.001 | StageMetadataChecker | StageMetadataChecker | OAV (metadata presence) |
| UN.002 | StageMetadataChecker | StageMetadataChecker | OAV (metadata presence) |
| UN.006 | UpAxisZChecker | UpAxisZChecker | SRF (opinionated value) |
| UN.007 | UnitsInMetersChecker | MetersPerUnit1Checker | SRF (opinionated value) |

**Resolution:** Remove the duplicate checker from the non-owning package. The remaining checker keeps the namespaced code.

## Classification Method

For physics requirements, we cross-referenced against Pixar's `UsdPhysicsValidators` (in the OpenUSD `dev` branch) to determine which checks Pixar considers general schema compliance:

- Pixar validates: nested articulations, rigid body Xformable/instancing/scale, joint targets, collider scale, mass values
- Pixar does NOT validate: rigid body capability, articulation capability, articulation pose, specific rigid body configurations

Codes matching Pixar's scope → `com.nvidia.usd.physics`
Codes outside Pixar's scope → `com.nvidia.simready.physics`

## Full Mapping Table

See `research/namespace-ownership-mapping.md` for the complete 118-code mapping.
