# Phase 3b — Problem Statement: Reconciling Pixar Validators with the Capability Profile System

## The Problem

Two validator ecosystems exist side-by-side:

1. **Pixar's built-in validators** (27 validators, loaded at startup from 6 plugins) — selected by `schemaTypes` and `keywords`
2. **SimReady validators** (10 ported so far, 90 planned) — selected by capability DAG via `plugInfo.json` `validators[]` field

These two systems don't know about each other. A SimReady profile like `com.nvidia.simready.prop_robotics_neutral` runs only the 10 SimReady validators — it doesn't also run Pixar's `usdGeomValidators:StageMetadataChecker` or `usdPhysicsValidators:RigidBodyChecker`, even though those check overlapping concerns.

This means:
- **Duplicated effort**: SimReady's `UN.001` (upAxis) and Pixar's `usdGeomValidators:StageMetadataChecker` both check if upAxis is declared — but a profile-driven validation only runs the SimReady one.
- **Missed checks**: Pixar's validators cover things SimReady doesn't (e.g. `usdShadeValidators:EncapsulationRulesValidator`, `usdValidation:CompositionErrorTest`). A "good USD citizen" profile should include those.
- **No canonical `usd.*` capabilities**: The Pixar proposal envisions capabilities like `usd.geom`, `usd.shade`, `usd.physics` that map to core USD functionality — but nobody has defined those yet or linked them to the existing validators.

## What We Have

### Pixar's Selection Model (existing)

```
UsdValidationContext(keywords=["UsdPhysicsValidators"])
  → selects 4 validators (RigidBodyChecker, ColliderChecker, ArticulationChecker, PhysicsJointChecker)

UsdValidationContext(schemaTypes=["UsdPhysicsRigidBodyAPI"])
  → selects RigidBodyChecker
```

Selection is schema-driven: "if this schema is applied to a prim, run these validators." This is bottom-up — the scene content determines what validates.

### Our Selection Model (new)

```
CapabilityRegistry.GetAllValidatorsForCapability("com.nvidia.simready.prop_robotics_neutral")
  → walks DAG → collects validators[] from each capability → 10 validators

UsdValidationContext(validators)
  → runs those 10
```

Selection is profile-driven: "if this profile is declared, run these validators." This is top-down — the declared intent determines what validates.

### The Gap

There's no bridge between "schema-driven" (Pixar) and "profile-driven" (ours). Ideally:

```
Profile declares:
  com.nvidia.simready.prop_robotics_neutral
    ← com.nvidia.simready.physics.rigidBodies
      ← usd.physics                              ← THIS DOESN'T EXIST YET
        validators: [usdPhysicsValidators:*]      ← NOR THIS MAPPING
```

## The Design Questions

### 1. Should canonical `usd.*` capabilities exist?

The Pixar proposal envisions them but doesn't define them. We could:

**Option A: Define `usd.*` capabilities ourselves, mapping to Pixar's validators**
```json
"usd.geom": {
    "predecessors": ["usd"],
    "validators": [
        "usdGeomValidators:EncapsulationChecker",
        "usdGeomValidators:StageMetadataChecker",
        "usdGeomValidators:SubsetFamilies",
        "usdGeomValidators:SubsetParentIsImageable"
    ]
}
```
Pro: Clean DAG. SimReady capabilities inherit from `usd.*` and get Pixar validators for free.
Con: We're defining canonical capabilities that Pixar should arguably own.

**Option B: Auto-generate `usd.*` capabilities from Pixar's existing plugin metadata**
```python
# At registry load time, for each validator plugin:
#   "usdGeomValidators" with keyword "UsdGeomValidators"
#   → auto-create capability "usd.geom.validation"
#   → with all validators from that plugin
```
Pro: No manual mapping. Stays in sync as Pixar adds validators.
Con: The auto-generated capability names might not match the Pixar proposal's taxonomy.

**Option C: Don't create `usd.*` capabilities — use a hybrid selection model**
```python
# Profile validation collects:
#   1. Capability-declared validators (our system)
#   2. Schema-type validators (Pixar's system, based on schemas present in scene)
# And merges them into one UsdValidationContext
```
Pro: No new capabilities needed. Both systems work as designed.
Con: The "what validators will run?" question becomes harder to answer — it depends on both the profile AND the scene content.

### 2. How do the `schemaTypes` and `validators[]` selection mechanisms compose?

Pixar's validators use `schemaTypes` to say "run me when a prim has this schema." Our `validators[]` field says "run me when this capability is required." These are complementary:

- **schemaTypes**: "I validate prims with UsdPhysicsRigidBodyAPI" (content-driven)
- **validators[]**: "The physics.rigidBodies capability requires these checks" (intent-driven)

A profile-driven validation should arguably do both:
1. Run the explicitly declared validators from the capability DAG
2. Also run any schema-type validators for schemas that appear in the scene

This gives the best coverage but makes the validator set non-deterministic (it depends on scene content).

### 3. Where do the 15 overlaps land?

From Phase 1, we identified 15 SimReady requirements that overlap with Pixar validators:
- 4 **DELEGATE** (UN.001, UN.002, UN.006, UN.007) — Pixar checks presence, SimReady checks specific values
- 11 **OVERLAP** (RB.003, RB.005, RB.009, RB.COL.004, JT.002, JT.003, JT.ART.002, JT.ART.004, VM.PS.001, VM.BIND.001, AA.002)

For the delegates, the answer is clear: reference Pixar's validator as the base check, add a SimReady validator for the stricter constraint.

For the overlaps, we need to decide per-validator:
- **If Pixar's is a superset**: use `usd.*` capability → Pixar validator. Don't duplicate in SimReady.
- **If SimReady's is stricter**: keep SimReady validator, but declare `usd.*` as a predecessor so Pixar's base check also runs.
- **If they check different things**: keep both, under their respective capabilities.

### 4. What about profiles that should include core USD hygiene?

A SimReady profile should probably include basic USD correctness checks (composition errors, type mismatches, etc.) even though they aren't "SimReady-specific." The Pixar proposal handles this via profile inheritance:

```
usd.core.v25_05 (profile)
  ← usd.core
    validators: [usdValidation:CompositionErrorTest, usdValidation:AttributeTypeMismatch, ...]

com.nvidia.simready.prop_robotics_neutral (profile)
  ← usd.core.v25_05                    ← ADDING THIS PREDECESSOR
  ← com.nvidia.simready.geom
  ← com.nvidia.simready.physics.rigidBodies
  ← ...
```

This way every SimReady profile inherits the core USD hygiene validators via the DAG, without listing them explicitly.

## Proposed Approach

We recommend a phased approach:

### Phase 3b.1: Define canonical `usd.*` capabilities (Option A)

Hand-define the initial `usd.*` capability tree in our `plugInfo.json`:

```
usd
├── usd.geom (validators: usdGeomValidators:*)
├── usd.shade (validators: usdShadeValidators:*)
├── usd.physics (validators: usdPhysicsValidators:*)
├── usd.skel (validators: usdSkelValidators:*)
├── usd.utils (validators: usdUtilsValidators:*)
├── usd.core (validators: usdValidation:CompositionErrorTest, ...)
└── usd.core.v25_05 [PROFILE] (predecessors: usd.core, usd.geom, usd.shade, usd.physics, usd.skel, usd.utils)
```

### Phase 3b.2: Make SimReady profiles inherit from `usd.*`

Update `com.nvidia.simready.prop_robotics_neutral` to include `usd.core.v25_05` as a predecessor. This gives it all 27 Pixar validators for free, plus the 10 SimReady-specific ones.

### Phase 3b.3: Resolve the 15 overlaps

For each overlap, make a determination:
- DELEGATE → remove SimReady validator, rely on the `usd.*` one
- EXTEND → keep SimReady validator for the stricter check, `usd.*` validator runs via inheritance
- KEEP BOTH → different scope, both needed

### Phase 3b.4 (future): Auto-generation

Once the hand-defined capabilities are proven, explore auto-generating `usd.*` capabilities from Pixar's plugin metadata for forward compatibility.

## Open Questions

1. Should we wait for Pixar to define canonical `usd.*` capabilities, or define them ourselves and propose upstream?
2. Is it acceptable for a profile's validator count to vary by OpenUSD version (as Pixar adds/removes validators)?
3. Should the `usd.core.v25_05` profile be version-locked, or should there be a `usd.core.latest` alias?
