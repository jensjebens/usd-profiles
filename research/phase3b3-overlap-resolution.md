# Phase 3b.3 — Overlap Resolution: Decision Matrix

**Ref:** jensjebens/usd-profiles#13
**Date:** 2026-04-08

## Methodology

For each of the 15 overlapping SimReady/Pixar validators, I read both implementations
and classified the relationship as:

- **DELEGATE** → Pixar's validator is a strict superset; remove SimReady's, rely on `usd.*` inheritance
- **EXTEND** → Pixar checks a subset; keep SimReady's for the stricter constraint, Pixar runs via DAG inheritance as the base check
- **KEEP BOTH** → Different scope or semantics; both needed

---

## DELEGATE (4 original + 5 promoted from OVERLAP = 9 total)

These can be **removed from SimReady** — Pixar's implementation covers them completely.

### UN.001/UN.002 — upAxis/metersPerUnit declared
- **Pixar:** `usdGeomValidators:StageMetadataChecker` checks that upAxis and metersPerUnit are authored on the stage
- **SimReady:** UN.001 checks the same thing
- **Verdict: DELEGATE** — Pixar's check is identical. Remove SimReady's UN.001/UN.002.
- Note: UN.006 (upAxis=Z) and UN.007 (metersPerUnit=1.0) are **EXTEND** — they add value constraints on top.

### RB.003 — Rigid body must be Xformable
- **Pixar `RigidBodyChecker`:** `if (!usdPrim.IsA<UsdGeomXformable>())` → error `rigidBodyNonXformable`
- **SimReady `RB.003`:** `if not prim.IsA(UsdGeom.Xformable)` → error
- **Verdict: DELEGATE** — Identical check. Pixar's C++ version is canonical.

### RB.005 — Rigid body no instancing
- **Pixar `RigidBodyChecker`:** Checks `prim.IsInstanceProxy()`, also considers kinematic/enabled state before reporting
- **SimReady `RB.005`:** Checks `prim.IsInstanceProxy() or prim.IsInPrototype()`
- **Pixar is actually MORE thorough** — it also handles the kinematic/enabled edge case (allowed for kinematic or disabled bodies). SimReady is stricter (always errors on instances) but Pixar's is more correct.
- **Verdict: DELEGATE** — Pixar's is more nuanced and correct.

### RB.COL.004 — Collider non-uniform scale
- **Pixar `ColliderChecker`:** Checks `ScaleIsUniform()` for Sphere, Capsule, Cylinder, Cone, Points — errors on non-uniform scale
- **SimReady `RB.COL.004`:** Same concept
- **Verdict: DELEGATE** — Pixar checks the exact same primitives for uniform scale.

### JT.002 — Joint body target exists
- **Pixar `PhysicsJointChecker`:** `CheckJointRel()` verifies body0/body1 targets resolve to existing prims
- **SimReady `JT.002`:** Same check
- **Verdict: DELEGATE** — Identical semantics.

### JT.003 — Joint no multiple body targets
- **Pixar `PhysicsJointChecker`:** Checks `targets0.size() > 1 || targets1.size() > 1`
- **SimReady `JT.003`:** Same check
- **Verdict: DELEGATE** — Identical.

### JT.ART.002 — No nested articulation roots
- **Pixar `ArticulationChecker`:** `CheckNestedArticulationRoot()` walks parents looking for ArticulationRootAPI
- **SimReady `JT.ART.002`:** Same walk-up check
- **Verdict: DELEGATE** — Identical logic.

### JT.ART.004 — Articulation not on static body
- **Pixar `ArticulationChecker`:** Checks if RigidBodyAPI is present with `rigidBodyEnabled=false` → error
- **SimReady `JT.ART.004`:** Same check
- **Verdict: DELEGATE** — Identical.

### VM.BIND.001 — MaterialBindingAPI applied
- **Pixar `MaterialBindingApiAppliedValidator`:** Checks that prims with material:binding relationships have MaterialBindingAPI applied
- **SimReady `VM.BIND.001`:** Same check
- **Verdict: DELEGATE** — Pixar's is the canonical implementation.

---

## EXTEND (4) — Keep SimReady's, Pixar runs as base check via DAG

### UN.006 — upAxis must be "Z"
- **Pixar:** Checks upAxis is *declared* (any value)
- **SimReady:** Checks upAxis == "Z" specifically
- **Verdict: EXTEND** — Pixar runs as base (via `usd.geom` inheritance), SimReady adds the value constraint.

### UN.007 — metersPerUnit must be 1.0
- **Pixar:** Checks metersPerUnit is *declared*
- **SimReady:** Checks metersPerUnit == 1.0
- **Verdict: EXTEND** — Same pattern as UN.006.

### RB.009 — No skew matrix on rigid bodies
- **Pixar `RigidBodyChecker`:** Checks for non-identity `ScaleOrientation` when scale is non-uniform — this is about scale orientation, not general skew
- **SimReady `RB.009`:** Checks for any skew in the xform matrix (broader definition)
- **Verdict: EXTEND** — Pixar checks a specific case (scale orientation); SimReady may want to check general matrix skew. Keep SimReady's for the broader check; Pixar's runs via inheritance.

### VM.PS.001 — UsdPreviewSurface material compliance
- **Pixar `ShaderSdrCompliance`:** Validates shader inputs match Sdr registry types — broad shader compliance
- **SimReady `VM.PS.001`:** Specifically checks for UsdPreviewSurface presence and correctness
- **Verdict: EXTEND** — Pixar's is general shader compliance; SimReady adds the specific UsdPreviewSurface requirement.

---

## KEEP BOTH (2) — Different scope

### AA.002 — Supported file types
- **Pixar `FileExtensionValidator`:** Checks file extensions in USDZ packages (UsdzValidators keyword)
- **SimReady `AA.002`:** Checks that referenced asset file types are in an allowed set (broader scope, not just USDZ)
- **Verdict: KEEP BOTH** — Different scope. Pixar's is USDZ-specific; SimReady's applies to all assets.

### VG.014, VG.002, HI.004 (PARTIAL from Phase 1)
- Already classified as PARTIAL / NO OVERLAP in Phase 1
- Confirmed: different checks despite same domain
- **Verdict: KEEP** — These stay as SimReady-unique.

---

## Summary

```
15 overlapping requirements:
├── 9  DELEGATE → remove SimReady validator, use Pixar via usd.* bridge
│   ├── UN.001, UN.002 (upAxis/metersPerUnit declared)
│   ├── RB.003, RB.005 (rigid body xformable, no instancing)
│   ├── RB.COL.004 (collider uniform scale)
│   ├── JT.002, JT.003 (joint targets)
│   ├── JT.ART.002, JT.ART.004 (articulation nesting/static body)
│   └── VM.BIND.001 (material binding API applied)
│
├── 4  EXTEND → keep SimReady, Pixar runs as base via DAG inheritance
│   ├── UN.006 (upAxis=Z), UN.007 (metersPerUnit=1.0)
│   ├── RB.009 (no skew — broader than Pixar's scale orientation check)
│   └── VM.PS.001 (UsdPreviewSurface — more specific than ShaderSdrCompliance)
│
└── 2  KEEP BOTH → different scope
    ├── AA.002 (file types — SimReady broader than USDZ)
    └── 3 PARTIAL (already classified in Phase 1)
```

## Impact on Validator Count

Current SimReady profile (`prop_robotics_neutral`):
- 10 SimReady validators + 11 Pixar bridge = **21 total**

After delegation (removing 9 redundant SimReady validators that we currently implement):
- The 9 delegated checks are covered by Pixar validators inherited via `usd.*`
- Some of the delegated validators (RB.003, RB.005) are in our current 10
- Net effect: fewer SimReady-specific validators, same coverage

## Next Steps

1. Remove delegated validators from `simready_validators.py`
2. Update `plugInfo.json` validators[] arrays
3. Add the 4 EXTEND validators that we haven't implemented yet (UN.006, UN.007, RB.009, VM.PS.001)
4. Update tests
