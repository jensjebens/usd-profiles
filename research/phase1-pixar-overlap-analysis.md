# Phase 1 — Pixar UsdValidation Overlap Analysis

## Pixar Built-in Validators (dev branch)

24 validators across 5 plugins:

| Plugin | Validators | Scope |
|--------|-----------|-------|
| **usdGeomValidators** | 4 | Geometry encapsulation, stage metadata, geom subsets |
| **usdShadeValidators** | 9 | Material/shader encapsulation, bindings, normal maps, SDR compliance |
| **usdPhysicsValidators** | 4 | Rigid bodies, colliders, articulations, joints |
| **usdUtilsValidators** | 5 | File extensions, missing references, USDZ packaging |
| **usdSkelValidators** | 2 | Skel binding API |

### Full Validator List

```
usdGeomValidators:EncapsulationChecker          — No nested Gprims
usdGeomValidators:StageMetadataChecker          — upAxis + metersPerUnit declared
usdGeomValidators:SubsetFamilies                — Geom subset family validity
usdGeomValidators:SubsetParentIsImageable        — GeomSubset parent must be Imageable

usdShadeValidators:EncapsulationMaterialValidator — Material prim child types
usdShadeValidators:EncapsulationRulesValidator    — Connectable prim nesting
usdShadeValidators:MaterialBindingApiAppliedValidator — MaterialBindingAPI applied check
usdShadeValidators:MaterialBindingCollectionValidator — Collection-based binding validity
usdShadeValidators:MaterialBindingRelationships  — material:binding must be relationships
usdShadeValidators:NormalMapTextureValidator      — Normal map scale/bias/colorSpace
usdShadeValidators:ShaderSdrCompliance           — Shader input type conformance
usdShadeValidators:SubsetMaterialBindFamilyName  — Subset materialBind family name
usdShadeValidators:SubsetsMaterialBindFamily      — materialBind family type restriction

usdPhysicsValidators:RigidBodyChecker            — RigidBodyAPI validation
usdPhysicsValidators:ColliderChecker             — CollisionAPI validation
usdPhysicsValidators:ArticulationChecker         — ArticulationRootAPI validation
usdPhysicsValidators:PhysicsJointChecker         — PhysicsJoint validation

usdUtilsValidators:FileExtensionValidator        — Package file extension whitelist
usdUtilsValidators:MissingReferenceValidator     — Unresolvable asset dependencies
usdUtilsValidators:PackageEncapsulationValidator — Package self-containment
usdUtilsValidators:RootPackageValidator          — Root USDZ package format
usdUtilsValidators:UsdzPackageValidator          — All USDZ packages format

usdSkelValidators:SkelBindingApiAppliedValidator — SkelBindingAPI applied check
usdSkelValidators:SkelBindingApiValidator        — SkelBindingAPI parent type
```

## Cross-Reference with SimReady Requirements

### Classification

| Type | Count | Meaning |
|------|-------|---------|
| **delegate** | 4 | Pixar validator covers this directly; SimReady profile can reference it |
| **overlap** | 11 | Pixar has equivalent validator; needs detailed comparison for exact semantics |
| **partial** | 3 | Some conceptual overlap but different scope |
| **none** | 72 | SimReady-unique; stays under `com.nvidia.simready.*` |

### DELEGATE (4) — Use Pixar's validator, SimReady adds constraints

These SimReady requirements test the same thing as a Pixar built-in but with stricter parameters. The recommended approach is to **depend on the Pixar validator** and layer SimReady-specific constraints on top.

| NVIDIA Code | SimReady Rule | Pixar Validator | Notes |
|---|---|---|---|
| UN.001 | upaxis | `usdGeomValidators:StageMetadataChecker` | Pixar checks upAxis is declared; SimReady can reference this |
| UN.002 | meters-per-unit | `usdGeomValidators:StageMetadataChecker` | Pixar checks metersPerUnit is declared |
| UN.006 | upaxis-z | `usdGeomValidators:StageMetadataChecker` | SimReady **stricter**: requires upAxis = "Z" specifically |
| UN.007 | meters-per-unit-1 | `usdGeomValidators:StageMetadataChecker` | SimReady **stricter**: requires metersPerUnit = 1.0 specifically |

**Strategy:** Register `usdGeomValidators:StageMetadataChecker` as a dependency. SimReady adds a thin wrapper validator that checks the specific *values* (Z, 1.0) rather than just presence.

### OVERLAP (11) — Pixar has equivalent, needs semantic comparison

| NVIDIA Code | SimReady Rule | Pixar Validator | Notes |
|---|---|---|---|
| RB.003 | rigid-body-schema-application | `usdPhysicsValidators:RigidBodyChecker` | Rigid body must be Xformable |
| RB.005 | rigid-body-no-instancing | `usdPhysicsValidators:RigidBodyChecker` | No scene graph instancing |
| RB.009 | rigid-body-schema-no-skew-matrix | `usdPhysicsValidators:RigidBodyChecker` | No skew in xform |
| RB.COL.004 | collider-non-uniform-scale | `usdPhysicsValidators:ColliderChecker` | Uniform scale for certain shapes |
| JT.002 | joint-body-target-exists | `usdPhysicsValidators:PhysicsJointChecker` | Body0/Body1 targets exist |
| JT.003 | joint-no-multiple-body-targets | `usdPhysicsValidators:PhysicsJointChecker` | Body0/Body1 single target only |
| JT.ART.002 | articulation-no-nesting | `usdPhysicsValidators:ArticulationChecker` | No nested articulation roots |
| JT.ART.004 | articulation-not-on-static-body | `usdPhysicsValidators:ArticulationChecker` | Not on static bodies |
| VM.PS.001 | material-preview-surface | `usdShadeValidators:ShaderSdrCompliance` | UsdPreviewSurface compliance |
| VM.BIND.001 | material-bind-scope | `usdShadeValidators:MaterialBindingApiAppliedValidator` | MaterialBindingAPI applied |
| AA.002 | supported-file-types | `usdUtilsValidators:FileExtensionValidator` | Allowed file extensions |

**Strategy:** These need a detailed code-level comparison between Pixar's C++ implementations and NVIDIA's Python implementations to determine if Pixar's are strict supersets. If so, delegate. If NVIDIA's are stricter or check additional conditions, keep as `com.nvidia.simready.*` extensions that depend on the Pixar base.

### PARTIAL (3) — Different scope, partial conceptual overlap

| NVIDIA Code | SimReady Rule | Pixar Validator | Notes |
|---|---|---|---|
| VG.014 | usdgeom-mesh-topology | `usdGeomValidators:EncapsulationChecker` | Pixar checks nesting; NVIDIA checks topology *validity* (completely different) |
| VG.002 | usdgeom-extent | `usdGeomValidators:StageMetadataChecker` | No dedicated Pixar extent validator exists |
| HI.004 | stage-has-default-prim | `usdUtilsValidators:MissingReferenceValidator` | No dedicated Pixar defaultPrim validator |

**Strategy:** These stay as SimReady-unique validators. The Pixar validators check different things despite being in the same domain.

### NO OVERLAP (72) — SimReady-unique

The remaining 72 requirements have no Pixar equivalent:

| Domain | Count | Examples |
|--------|-------|---------|
| Geometry (VG.*) | 33 | Mesh manifold, normals, subdivision, tessellation, welding, etc. |
| Hierarchy (HI.*) | 9 | Root xformable, logical grouping, kinematic chain, etc. |
| Physics rigid bodies (RB.*) | 5 | Mass, collider mesh, static collider, etc. |
| Materials visual (VM.*) | 3 | MDL source/schema, duplicates |
| Nonvisual materials (NVM.*) | 6 | All 6 — entirely SimReady-specific |
| Units (UN.*) | 3 | kilogramsPerUnit, timeCodesPerSecond, corrective transforms |
| Atomic asset (AA.*) | 3 | Anchored paths, portable paths, USDZ UDIM limitation |
| Semantic labels (SL.*) | 4 | All 4 — entirely SimReady-specific |
| Dense captions (DC.*) | 2 | All 2 — entirely SimReady-specific |
| Physics joints (JT.*) | 3 | Joint capability, articulation, kinematic body limitation |
| Physics rigid bodies (RB.*) | 1 | PhysX nesting limitation |

These are the core value of SimReady — simulation-specific requirements that go well beyond what core USD validation covers. They all belong under `com.nvidia.simready.*`.

## Summary

```
90 SimReady requirements
├── 4  DELEGATE to Pixar built-in (reference + add constraints)
├── 11 OVERLAP with Pixar (need code-level comparison)
├── 3  PARTIAL overlap (different scope — keep as SimReady)
└── 72 NO OVERLAP (SimReady-unique — com.nvidia.simready.*)
```

**80% of SimReady requirements are unique** and have no Pixar equivalent. The overlap is concentrated in physics validators (8/11) which makes sense — both NVIDIA and Pixar implement UsdPhysics schema validation.

## Next Steps

1. **Code-level comparison** of the 11 OVERLAP validators — read Pixar's `validators.cpp` for each plugin to determine exact check semantics
2. **Decide** for each overlap: delegate fully, delegate + extend, or keep separate
3. **Update CSV** with final disposition
