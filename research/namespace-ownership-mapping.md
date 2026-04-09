# Namespace Ownership: Full Requirement Code → Owner Mapping

**Date:** 2026-04-09 (updated with Jens's feedback)
**Ref:** jensjebens/usd-profiles#23

## Principle

- **OAV** (`com.nvidia.usd.*` → `usd.*` future): USD schema/grammar compliance — non-opinionated, applies to ANY USD file
- **SRF** (`com.nvidia.simready.*`): simulation conformance — opinionated conventions for SimReady assets

**Rule:** If a requirement checks that USD is *valid*, OAV owns it. If it checks that an asset conforms to a *convention*, SRF owns it.

**Namespace decision:** OAV requirements use `com.nvidia.usd.*` (not `com.nvidia.oav.*`) to signal intent for upstream into `usd.*`.

## Full Mapping (118 requirements)

### com.nvidia.usd.core (7)

| Code | Description | Notes |
|------|------------|-------|
| AA.001 | Anchored asset paths | General USD hygiene |
| AA.002 | Supported file types | General USD hygiene |
| HI.001 | Single root prim | Valid USD structure |
| HI.003 | Root prim is Xformable | Valid USD structure |
| HI.004 | Stage has defaultPrim | Valid USD structure |
| UN.001 | upAxis is declared | Metadata presence check |
| UN.002 | metersPerUnit is declared | Metadata presence check |

### com.nvidia.usd.geom (28)

| Code | Description |
|------|------------|
| VG.001–VG.024, VG.026–VG.029 | Geometry validity checks |

**Moved to SRF (per review):**
- `VG.MESH.001` → `com.nvidia.simready.geom` (opinionated: must contain mesh)
- `VG.RTX.001` → `com.nvidia.simready.geom` (NVIDIA RTX-specific)
- `VG.025` → `com.nvidia.simready.geom` (opinionated: origin positioning)

### com.nvidia.usd.physics (14)

Only codes that match Pixar's UsdPhysicsValidators scope (schema compliance):

| Code | Description | Pixar equivalent |
|------|------------|-----------------|
| JT.002 | Joint body target exists | JointInvalidPrimRel |
| JT.003 | Joint no multiple targets | JointMultiplePrimsRel |
| JT.ART.002 | No nested articulation | NestedArticulation |
| JT.ART.004 | Articulation not on static body | ArticulationOnStaticBody |
| RB.003 | Rigid body on Xformable | RigidBodyNonXformable |
| RB.005 | Rigid body no instancing | RigidBodyNonInstanceable |
| RB.006 | Rigid body collision API | — |
| RB.007 | Rigid body mass | MassInvalidValues |
| RB.009 | Rigid body no skew | RigidBodyOrientationScale |
| RB.COL.001 | Collision API applied | — |
| RB.COL.002 | Collision mesh check | — |
| RB.COL.003 | Collision mesh quality | — |
| RB.COL.004 | Collision uniform scale | ColliderNonUniformScale |

**Moved to SRF (not in Pixar's scope):**
- `JT.001` → `com.nvidia.simready.physics` (joint capability — opinionated)
- `JT.ART.001` → `com.nvidia.simready.physics` (articulation capability — opinionated)
- `JT.ART.003` → `com.nvidia.simready.physics` (articulation pose — not in Pixar)
- `RB.001` → `com.nvidia.simready.physics` (rigid body capability — opinionated)
- `RB.008` → `com.nvidia.simready.physics` (rigid body static — not in Pixar)
- `RB.010` → `com.nvidia.simready.physics` (not in Pixar)
- `RB.011` → `com.nvidia.simready.physics` (not in Pixar)
- `RB.012` → `com.nvidia.simready.physics` (not in Pixar)

### com.nvidia.usd.shade (8)

| Code | Description |
|------|------------|
| VM.BIND.001, VM.BIND.002 | Material binding validity |
| VM.MAT.001 | Material existence |
| VM.MDL.001, VM.MDL.002 | MDL schema compliance |
| VM.PS.001 | UsdPreviewSurface compliance |
| VM.TEX.001, VM.TEX.002 | Texture validity |

### com.nvidia.simready.hierarchy (7)

| Code | Description |
|------|------------|
| HI.002 | Exclusive XForm parent |
| HI.005 | SimReady hierarchy structure |
| HI.006 | Placeable/posable xformable |
| HI.007 | SimReady hierarchy rules |
| HI.008 | Logical geometry grouping |
| HI.009 | Kinematic chain hierarchy |
| HI.010 | Undefined prims check |

### com.nvidia.simready.geom (3) — moved from OAV

| Code | Description | Reason |
|------|------------|--------|
| VG.MESH.001 | Must contain mesh | Opinionated — volumes, curves, points are valid USD |
| VG.RTX.001 | RTX extreme extents | NVIDIA RTX-specific |
| VG.025 | Asset origin positioning | Opinionated convention |

### com.nvidia.simready.physics (9) — moved from OAV

| Code | Description | Reason |
|------|------------|--------|
| JT.001 | Joint capability | Opinionated — not all assets need joints |
| JT.ART.001 | Articulation capability | Opinionated |
| JT.ART.003 | Articulation pose | Not in Pixar's scope |
| RB.001 | Rigid body capability | Opinionated — not all assets need physics |
| RB.008 | Rigid body static | Not in Pixar's scope |
| RB.010 | Rigid body check | Not in Pixar's scope |
| RB.011 | Rigid body check | Not in Pixar's scope |
| RB.012 | Rigid body check | Not in Pixar's scope |
| RB.MB.001 | Multi-body convention | SimReady-specific |

### com.nvidia.simready.naming_paths (8)

| Code | Description |
|------|------------|
| NP.001–NP.008 | All naming/path conventions |

### com.nvidia.simready.units (5)

| Code | Description |
|------|------------|
| UN.003 | kilogramsPerUnit |
| UN.004 | Corrective transforms |
| UN.005 | timeCodesPerSecond |
| UN.006 | upAxis = Z (opinionated value) |
| UN.007 | metersPerUnit = 1.0 (opinionated value) |

### com.nvidia.simready.semantic_labels (4)

| Code | Description |
|------|------------|
| SL.001, SL.003, SL.NV.002, SL.QCODE.001 | All semantic labels |

### com.nvidia.simready.nonvisual_materials (6)

| Code | Description |
|------|------------|
| NVM.001–NVM.006 | All non-visual materials |

### com.nvidia.simready.driven_joints (11)

| Code | Description |
|------|------------|
| DJ.001–DJ.011 | All driven joint conventions |

### com.nvidia.simready.base_articulation (2)

| Code | Description |
|------|------------|
| BA.001, BA.002 | Base articulation rules |

### com.nvidia.simready.* (misc) (6)

| Code | Namespace | Description |
|------|-----------|------------|
| AA.OV.001 | simready.core | USDZ UDIM limitation (Kit-specific) |
| COL.001 | simready.colliders | Collider convention |
| GSP.001 | simready.graspable | Graspable physics |
| PHYSX.COL.001-002 | simready.physx | PhysX-specific colliders |
| PMT.001 | simready.physics_materials | Physics materials convention |
| SR.001 | simready.sim_ready | SimReady capability check |

## Revised Summary

| Namespace | Count | Description |
|-----------|-------|-------------|
| `com.nvidia.usd.core` | 7 | Basic USD validity |
| `com.nvidia.usd.geom` | 28 | Geometry validity |
| `com.nvidia.usd.physics` | 14 | Physics schema validity (Pixar-aligned) |
| `com.nvidia.usd.shade` | 8 | Material/shader validity |
| **USD subtotal** | **57** | → `usd.*` future |
| `com.nvidia.simready.*` | **61** | Simulation conformance |
| **Total** | **118** | |
