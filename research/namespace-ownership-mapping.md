# Namespace Ownership: Full Requirement Code → Owner Mapping

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23

## Principle

- **OAV** (`com.nvidia.oav.*` → `usd.*` future): USD schema/grammar compliance — non-opinionated, applies to ANY USD file
- **SRF** (`com.nvidia.simready.*`): simulation conformance — opinionated conventions for SimReady assets

**Rule:** If a requirement checks that USD is *valid*, OAV owns it. If it checks that an asset conforms to a *convention*, SRF owns it.

## Full Mapping (118 requirements)

### com.nvidia.oav.core (7)

| Code | Description | Notes |
|------|------------|-------|
| AA.001 | Anchored asset paths | General USD hygiene |
| AA.002 | Supported file types | General USD hygiene |
| HI.001 | Single root prim | Valid USD structure |
| HI.003 | Root prim is Xformable | Valid USD structure |
| HI.004 | Stage has defaultPrim | Valid USD structure |
| UN.001 | upAxis is declared | Metadata presence check |
| UN.002 | metersPerUnit is declared | Metadata presence check |

### com.nvidia.oav.geom (31)

| Code | Description |
|------|------------|
| VG.001–VG.029, VG.MESH.001, VG.RTX.001 | All geometry validity checks |

⚠️ **Review needed:**
- `VG.MESH.001` (must contain mesh) — is this general or SimReady-opinionated?
- `VG.RTX.001` (RTX extreme extent limits) — is this general or NVIDIA-specific?
- `VG.025` (asset origin positioning) — could be SimReady convention?

### com.nvidia.oav.physics (21)

| Code | Description |
|------|------------|
| JT.001–JT.003, JT.ART.001–JT.ART.004 | Joint/articulation schema validity |
| RB.001, RB.003, RB.005–RB.012 | Rigid body schema validity |
| RB.COL.001–RB.COL.004 | Collision API schema validity |

⚠️ **Review needed:**
- `JT.ART.001` (articulation capability) and `JT.ART.003` — are these general physics schema checks or SimReady-specific?
- `RB.001` (rigid body capability), `RB.008` (rigid body static), `RB.010–RB.012` — general or SimReady?

### com.nvidia.oav.shade (8)

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
| HI.002 | Exclusive XForm parent | SimReady layout |
| HI.005 | SimReady hierarchy structure | |
| HI.006 | Placeable/posable xformable | SimReady convention |
| HI.007 | SimReady hierarchy rules | |
| HI.008 | Logical geometry grouping | SimReady convention |
| HI.009 | Kinematic chain hierarchy | SimReady convention |
| HI.010 | Undefined prims check | SimReady convention |

### com.nvidia.simready.naming_paths (8)

| Code | Description |
|------|------------|
| NP.001–NP.008 | All naming/path conventions | SimReady-specific |

### com.nvidia.simready.units (5)

| Code | Description |
|------|------------|
| UN.003 | kilogramsPerUnit | SimReady value |
| UN.004 | Corrective transforms | SimReady convention |
| UN.005 | timeCodesPerSecond | SimReady value |
| UN.006 | upAxis = Z | Opinionated (not just "declared") |
| UN.007 | metersPerUnit = 1.0 | Opinionated (not just "declared") |

### com.nvidia.simready.semantic_labels (4)

| Code | Description |
|------|------------|
| SL.001, SL.003, SL.NV.002, SL.QCODE.001 | All semantic labels | SimReady-specific |

### com.nvidia.simready.nonvisual_materials (6)

| Code | Description |
|------|------------|
| NVM.001–NVM.006 | All non-visual materials | SimReady-specific |

### com.nvidia.simready.driven_joints (11)

| Code | Description |
|------|------------|
| DJ.001–DJ.011 | All driven joint conventions | SimReady-specific |

### com.nvidia.simready.base_articulation (2)

| Code | Description |
|------|------------|
| BA.001, BA.002 | Base articulation rules | SimReady-specific |

### com.nvidia.simready.* (misc) (6)

| Code | Namespace | Description |
|------|-----------|------------|
| AA.OV.001 | simready.core | USDZ UDIM limitation (Kit-specific) |
| COL.001 | simready.colliders | Collider convention |
| GSP.001 | simready.graspable | Graspable physics |
| PHYSX.COL.001-002 | simready.physx | PhysX-specific colliders |
| PMT.001 | simready.physics_materials | Physics materials convention |
| RB.MB.001 | simready.physics | Multi-body convention |
| SR.001 | simready.sim_ready | SimReady capability check |

## Summary

| Namespace | Count | Description |
|-----------|-------|-------------|
| `com.nvidia.oav.core` | 7 | Basic USD validity |
| `com.nvidia.oav.geom` | 31 | Geometry validity |
| `com.nvidia.oav.physics` | 21 | Physics schema validity |
| `com.nvidia.oav.shade` | 8 | Material/shader validity |
| **OAV subtotal** | **67** | |
| `com.nvidia.simready.*` | 51 | Simulation conformance |
| **Total** | **118** | |

## Open Questions for Review

1. **VG.MESH.001** (must contain mesh) — Is "assets must have mesh geometry" a general USD check or a SimReady convention? Non-mesh USD files (volumes, curves, points) are perfectly valid.

2. **VG.RTX.001** (RTX extreme extents) — This is NVIDIA RTX-specific. Should it be `com.nvidia.oav.geom` or `com.nvidia.simready.geom`?

3. **VG.025** (asset origin positioning) — "Origin should be at a specific location" feels opinionated/SimReady rather than general USD validity.

4. **JT.ART.001, JT.ART.003** — Are articulation capability and articulation pose checks general USD physics or SimReady-specific?

5. **RB.001, RB.008, RB.010-RB.012** — Some rigid body checks may be SimReady-specific (e.g. "rigid body capability" vs "rigid body schema validity"). Need per-code review.

6. **Should `com.nvidia.oav` be just `oav`?** The `com.nvidia` prefix implies vendor ownership, but OAV checks are meant to be general USD hygiene. Maybe just `oav.geom`, `oav.physics`, etc.?
