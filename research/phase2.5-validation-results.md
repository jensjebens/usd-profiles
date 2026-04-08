# Phase 2.5 Validation Results — SimReady Foundation Assets

**Date:** 2026-04-08
**Ref:** jensjebens/OpenUSD#35

## Setup

Ran Pixar-equivalent validation checks (simulating the 11 Pixar validators
inherited via the `usd.*` capability bridge) against all 34 root-level assets
in the [NVIDIA SimReady Foundation](https://github.com/NVIDIA/simready-foundation) repo.

### Validator Coverage

| Source | Count | Validators |
|--------|-------|-----------|
| SimReady-specific | 10 | VG.MESH.001, VG.014, VG.025, VG.027, RB.003, RB.005, RB.COL.001, HI.001, HI.004, UN.001 |
| Pixar bridge (usd.core) | 3 | CompositionErrorTest, StageMetadataChecker, AttributeTypeMismatch |
| Pixar bridge (usd.geom) | 4 | EncapsulationChecker, StageMetadataChecker, SubsetFamilies, SubsetParentIsImageable |
| Pixar bridge (usd.physics) | 4 | RigidBodyChecker, ColliderChecker, ArticulationChecker, PhysicsJointChecker |
| **Total** | **21** | |

## Results

- **34 assets checked**
- **33 passed** all Pixar-equivalent checks
- **1 failed** — caught by `usdValidation:CompositionErrorTest` (NEW via bridge)

### 🔴 Bug Found: UR10 PhysX variant has broken reference

**Asset:** `sample_content/common_assets/robots_general/ur10/simready_physx_usd/ur10.usda`

**Error:** Unresolved reference prim path — `physx.usda` references `/colliders/base_link`
but `/colliders` doesn't exist in any layer in the composition.

**Details:**
- `payloads/Physics/physx.usda` defines `/ur10/base_link/collisions` with
  `prepend references = </colliders/base_link>`
- No layer in the composition contains a `/colliders` prim
- The `simready_usd` variant of the same asset has **no errors** (it doesn't have `physx.usda`)
- The `simready_physx_usd` variant has `physx.usda` as an extra layer that adds PhysX-specific
  overrides, but the reference target is missing

**This is a real bug** in the SimReady Foundation repo — a dangling internal reference
in the PhysX physics configuration for the UR10 robot arm.

**Impact:** The collision mesh for `base_link` in the PhysX variant is effectively
missing. Physics simulation of the UR10 with PhysX colliders would have an invisible
base link.

### Why SimReady-only validation missed this

SimReady's 10 custom validators check things like:
- Mesh orientation, scale, hierarchy structure
- Rigid body configuration
- Units and up-axis

None of them check **composition integrity** — whether all references and payloads
resolve correctly. That's exactly what Pixar's `CompositionErrorTest` validator does.

## Conclusion

The `usd.*` capability bridge proves its value: even on NVIDIA's own well-curated
SimReady assets, it caught a real composition bug that was invisible to the existing
SimReady validation pipeline.

This validates the design decision to make vendor profiles inherit from canonical
`usd.*` capabilities. Core USD hygiene checks (composition, types, metadata) are
complementary to domain-specific checks and should run alongside them.
