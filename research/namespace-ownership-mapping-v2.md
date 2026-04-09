# Namespace Ownership Mapping v2

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23
**Supersedes:** v1 of this mapping (same file, prior revision)

## Revised Principle

The v1 mapping used "if removing the check would produce an invalid USD file, OAV owns it." This was too narrow — it looked at what Pixar validates **today** (very little) rather than what they **would reasonably accept as an upstream contribution**.

**v2 principle:** A requirement belongs in `com.nvidia.usd.*` (OAV, upstream candidate) if it validates **general-purpose USD correctness or quality** that any USD pipeline would benefit from, regardless of target domain. A requirement belongs in `com.nvidia.simready.*` if it enforces a **convention, budget, or opinion** specific to SimReady's goals (simulation readiness, RTX performance, specific workflows).

**Litmus test:** "Would a Pixar developer reviewing a PR to add this validator to UsdValidation say 'yes, this is generally useful' or 'this is your opinion about how assets should be authored'?"

## Cross-Reference: Pixar UsdValidation Validators (dev branch, 26.05)

### usdGeomValidators (4 validators)
| Validator | Checks |
|-----------|--------|
| StageMetadataChecker | `upAxis` and `metersPerUnit` declared |
| SubsetFamilies | GeomSubset families valid |
| SubsetParentIsImageable | GeomSubset parent is Imageable |
| EncapsulationChecker | No nested Gprims |

### usdShadeValidators (9 validators)
| Validator | Checks |
|-----------|--------|
| EncapsulationMaterialValidator | Only connectable types inside Material |
| EncapsulationRulesValidator | Connectable prims only nest in containers |
| MaterialBindingApiAppliedValidator | MaterialBindingAPI applied if binding exists |
| MaterialBindingCollectionValidator | Collection-based bindings well-formed |
| MaterialBindingRelationships | material:binding properties are relationships |
| NormalMapTextureValidator | Normal map scale/bias/colorSpace correct |
| ShaderSdrCompliance | Shader input types match SDR definitions |
| SubsetMaterialBindFamilyName | materialBind family name on bound subsets |
| SubsetsMaterialBindFamily | materialBind family type restricted |

### usdPhysicsValidators (4 validators)
| Validator | Checks |
|-----------|--------|
| RigidBodyChecker | RB on Xformable, no instancing, no skew, mass values valid |
| ColliderChecker | Uniform scale on non-mesh colliders, Points data valid, mass values |
| ArticulationChecker | No nested articulations, no articulation on static body |
| PhysicsJointChecker | Joint body targets exist, no multiple targets |

### usdValidation core (3 validators)
| Validator | Checks |
|-----------|--------|
| CompositionErrorTest | Composition errors |
| StageMetadataChecker | defaultPrim specified |
| AttributeTypeMismatch | Attribute types match schema |

### usdUtilsValidators (5 validators)
| Validator | Checks |
|-----------|--------|
| FileExtensionValidator | Valid file extensions |
| MissingReferenceValidator | No unresolvable dependencies |
| PackageEncapsulationValidator | USDZ self-contained |
| RootPackageValidator | USDZ root layer format |
| UsdzPackageValidator | USDZ package format |

---

## Revised Mapping

### Geometry (35 codes)

#### `com.nvidia.usd.geom` — upstream candidates (13)

| Code | Name | Rationale |
|------|------|-----------|
| VG.002 | extent | Missing/wrong extents break bbox queries — any pipeline needs this |
| VG.007 | manifold | Non-manifold causes problems for boolean ops, simulation, 3D printing |
| VG.009 | primvar-indexing | Redundant primvar data — general efficiency, upstreamable |
| VG.014 | mesh-topology | Invalid faceVertexCounts/Indices = mathematically broken mesh |
| VG.016 | colocated-points | Duplicate vertices at same position = authoring artifact |
| VG.018 | unused-topology | Unused verts/edges/faces = structural dead data |
| VG.019 | zero-area-faces | Degenerate faces = geometric error any renderer would flag |
| VG.027 | normals-exist | No normals on non-subdivided mesh = bad shading everywhere |
| VG.028 | normals-valid | NaN/zero normals = broken shading in any renderer |
| VG.029 | winding-order | Wrong winding = inverted faces, universally problematic |
| VG.030 | zero-extent | Boundable with zero extent = likely authoring error |
| VG.032 | lamina-faces | Duplicate overlapping faces = geometric error |
| VG.033 | unused-uvs | Unused texture coords = dead data, general cleanup |

#### `com.nvidia.simready.geom` — SimReady conventions (22)

| Code | Name | Rationale |
|------|------|-----------|
| VG.003 | internal-geometry | Valid to have internal geo (cutaway views, X-ray rendering) |
| VG.004 | empty-spaces | Efficiency preference for sim |
| VG.005 | boundable-size | Scale convention |
| VG.006 | mesh-overlap | Sometimes intentional (layered materials, decals) |
| VG.008 | coincident | Sometimes intentional |
| VG.010 | subdivision+normals | Both valid, suboptimal for RTX |
| VG.011 | primvar-usage | Unused primvars may serve downstream tools |
| VG.012 | mesh-small | "Too small" is a budget/convention call |
| VG.013 | tessellation-density | Budget/taste, not correctness |
| VG.015 | identical-timesamples | Redundant but valid |
| VG.017 | primitive-tessellation | Valid to tessellate primitives |
| VG.020 | points-precision | Float precision is a pipeline convention |
| VG.021 | vertex-count | Budget/performance target |
| VG.022 | mesh-duplicate | Use instancing = efficiency preference |
| VG.023 | xform-positioning | Both approaches valid in USD |
| VG.024 | identical-mesh-consistency | Convention |
| VG.025 | asset-at-origin | Convention (SimReady placement rule) |
| VG.031 | non-opaque-thickness | Rendering convention |
| VG.034 | invisible-prims | Valid USD, style preference |
| VG.MESH.001 | geom-shall-be-mesh | UsdGeomSphere etc. are valid USD |
| VG.RTX.001 | rtx-limit | RTX-specific |
| VG.RTX.002 | mesh-count | RTX/budget |

### Hierarchy (10 codes)

#### `com.nvidia.usd.core` — upstream candidates (4)

| Code | Name | Rationale |
|------|------|-----------|
| HI.001 | hierarchy-has-root | Single root prim = good USD hygiene, Pixar validates defaultPrim already |
| HI.003 | root-is-xformable | Root prim should be transformable for any scene assembly |
| HI.004 | stage-has-default-prim | Pixar already validates this (usdValidation:StageMetadataChecker) |
| HI.011 | many-children | Excessive children under one parent = performance issue for any USD consumer |

#### `com.nvidia.simready.hierarchy` — SimReady conventions (6)

| Code | Name | Rationale |
|------|------|-----------|
| HI.005 | xform-common-api-usage | XformCommonAPI is recommended but not required by USD |
| HI.006 | placeable-posable-are-xformable | SimReady placement convention |
| HI.008 | logical-geometry-grouping | Opinionated about asset structure |
| HI.009 | kinematic-chain-hierarchy | Articulation-specific convention |
| HI.010 | intentional-origin-positioning | SimReady origin convention |
| HI.012 | empty-leaves | Valid USD, cleanup preference |

### Units (7 codes)

#### `com.nvidia.usd.core` — upstream candidates (2)

| Code | Name | Rationale |
|------|------|-----------|
| UN.001 | upAxis | Pixar already validates this (usdGeomValidators:StageMetadataChecker) |
| UN.002 | meters-per-unit | Pixar already validates this (usdGeomValidators:StageMetadataChecker) |

#### `com.nvidia.simready.units` — SimReady conventions (5)

| Code | Name | Rationale |
|------|------|-----------|
| UN.003 | kilograms-per-unit | Physics-specific, not all stages have physics |
| UN.004 | corrective-transforms | Convention for how to handle unit differences |
| UN.005 | timecodes-per-second | Only needed if timesamples present; convention |
| UN.006 | upAxis-z | Opinionated value (Z-up); Y-up is equally valid USD |
| UN.007 | meters-per-unit-1 | Opinionated value (1.0); other scales are valid USD |

### Materials (5 codes)

#### `com.nvidia.usd.shade` — upstream candidates (2)

| Code | Name | Rationale |
|------|------|-----------|
| VM.BIND.001 | material-bind-scope | Pixar validates binding API/relationships; scope correctness is general |
| VM.PS.001 | material-preview-surface | Pixar validates SDR compliance; PreviewSurface correctness is general |

#### `com.nvidia.simready.materials` — SimReady conventions (3)

| Code | Name | Rationale |
|------|------|-----------|
| VM.D.001 | material-duplicates | Fewer materials = performance preference |
| VM.MDL.001 | material-mdl-source-asset | MDL is NVIDIA-specific |
| VM.MDL.002 | material-mdl-schema | MDL is NVIDIA-specific |

### Atomic Asset (4 codes)

#### `com.nvidia.usd.core` — upstream candidates (3)

| Code | Name | Rationale |
|------|------|-----------|
| AA.001 | anchored-asset-paths | Anchored paths = portability, general best practice |
| AA.002 | supported-file-types | Pixar already validates this (usdUtilsValidators:FileExtensionValidator) |
| AA.003 | portable-asset-paths | Forward slashes for cross-platform = general best practice |

#### `com.nvidia.simready.asset` — SimReady conventions (1)

| Code | Name | Rationale |
|------|------|-----------|
| AA.OV.001 | ov-usdz-udim-limitation | Omniverse-specific limitation |

### Dense Captions (2 codes)

#### `com.nvidia.simready.captions` — SimReady conventions (2)

| Code | Name | Rationale |
|------|------|-----------|
| DC.001 | dense-caption-capability | Documentation metadata is a SimReady convention |
| DC.002 | additional-dense-captions | Documentation metadata is a SimReady convention |

### Physics — Rigid Bodies (10 codes)

#### `com.nvidia.usd.physics` — upstream candidates (5)

| Code | Name | Rationale |
|------|------|-----------|
| RB.003 | rigid-body-schema-application | Pixar validates: RB on Xformable (RigidBodyChecker) |
| RB.005 | rigid-body-no-instancing | Pixar validates: RB instancing (RigidBodyChecker) |
| RB.009 | rigid-body-no-skew-matrix | Pixar validates: RB scale orientation (RigidBodyChecker) |
| RB.COL.003 | collider-mesh | MeshCollisionAPI only on Mesh = schema correctness |
| RB.COL.004 | collider-non-uniform-scale | Pixar validates: collider scale (ColliderChecker) |

#### `com.nvidia.simready.physics` — SimReady conventions (5)

| Code | Name | Rationale |
|------|------|-----------|
| RB.001 | rigid-body-capability | "Must have RB" is a profile requirement |
| RB.006 | rigid-body-no-nesting | PhysX-specific limitation |
| RB.007 | rigid-body-mass | "Should have mass" is a quality convention |
| RB.COL.001 | collider-capability | "Must have collider" is a profile requirement |
| RB.COL.002 | static-collider | Convention about static vs dynamic |

### Physics — Joints (7 codes)

#### `com.nvidia.usd.physics` — upstream candidates (4)

| Code | Name | Rationale |
|------|------|-----------|
| JT.002 | joint-body-target-exists | Pixar validates: joint targets exist (PhysicsJointChecker) |
| JT.003 | joint-no-multiple-body-targets | Pixar validates: no multiple targets (PhysicsJointChecker) |
| JT.ART.002 | articulation-no-nesting | Pixar validates: nested articulations (ArticulationChecker) |
| JT.ART.004 | articulation-not-on-static-body | Pixar validates: articulation on static body (ArticulationChecker) |

#### `com.nvidia.simready.physics` — SimReady conventions (3)

| Code | Name | Rationale |
|------|------|-----------|
| JT.001 | joint-capability | "Should use joints" is a convention |
| JT.ART.001 | articulation | "Should define articulation" is a quality recommendation |
| JT.ART.003 | articulation-not-on-kinematic-body | PhysX-specific limitation |

### Semantic Labels (4 codes)

#### `com.nvidia.simready.semantics` — all SimReady (4)

| Code | Name | Rationale |
|------|------|-----------|
| SL.001 | semantic-label-capability | SimReady convention |
| SL.003 | semantic-label-schema | SemanticsLabelsAPI is NVIDIA schema |
| SL.NV.002 | semantic-label-time | RTX limitation |
| SL.QCODE.001 | semantic-label-qcode-valid | Wikidata Q-codes are a SimReady convention |

### Non-Visual Materials (6 codes)

#### `com.nvidia.simready.nonvisual_materials` — all SimReady (6)

| Code | Name | Rationale |
|------|------|-----------|
| NVM.001–006 | material-attributes through material-time | All NVIDIA-specific non-visual material conventions |

### Remaining SRF-only codes (not in OAV)

These 40 codes are defined only in SRF specs and stay in `com.nvidia.simready.*`:
- BA.* (base articulation) — 2 codes
- DJ.* (driven joints) — 11 codes  
- NP.* (naming/paths) — 8 codes
- PHYSX.* (PhysX-specific) — 6 codes
- Plus misc codes not present in OAV

---

## Summary

| Namespace | v1 Count | v2 Count | Change |
|-----------|----------|----------|--------|
| `com.nvidia.usd.core` | 7 | 9 | +2 (HI.001, HI.003 moved from hierarchy) |
| `com.nvidia.usd.geom` | 28 | 13 | **-15** |
| `com.nvidia.usd.physics` | 14 | 9 | -5 |
| `com.nvidia.usd.shade` | 8 | 2 | -6 |
| **Total OAV** | **57** | **33** | **-24** |
| **Total SimReady** | **61** | **85** | **+24** |

The big shift is in geometry (-15 to SimReady) and materials (-6 to SimReady). Physics stayed closer because Pixar already validates much of what we had assigned to OAV.
