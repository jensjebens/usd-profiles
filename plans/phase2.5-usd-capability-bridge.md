# Phase 2.5 — Define `usd.*` Capabilities Bridging Pixar Validators into the DAG

## Motivation

The capability DAG has `usd`, `usd.core`, and `usd.core.v25_05` as namespace placeholders with no validators attached. Pixar ships 27 validators across 6 plugins, but a profile-driven validation only runs capability-declared validators. This means `com.nvidia.simready.prop_robotics_neutral` runs 10 SimReady validators but misses all 27 Pixar validators — even though they check overlapping and complementary concerns.

Fix: attach Pixar's validators to `usd.*` capabilities and wire SimReady capabilities as descendants so the DAG does the work.

## Acceptance Criteria

1. `usd.*` capability tree defined in `plugInfo.json` with all 27 Pixar validators mapped
2. `GetAllValidatorsForCapability("usd.core.v25_05")` → 27 validators
3. `GetAllValidatorsForCapability("com.nvidia.simready.prop_robotics_neutral")` → 37 validators (27 Pixar + 10 SimReady)
4. Existing `testUsdProfilesBasic.py` still passes
5. New test file `testUsdProfilesCapabilityBridge.py` covering the bridge layer
6. Updated demo script showing combined validator sets
7. Updated `overview.dox` documenting the `usd.*` tree

## Approach

### Step 1: Update `plugInfo.json` — Define `usd.*` validators

Add `validators` arrays to existing `usd.*` capabilities and add new ones:

```
usd                          (no validators — root)
├── usd.core                 → usdValidation:CompositionErrorTest, StageMetadataChecker, AttributeTypeMismatch
├── usd.geom                 → usdGeomValidators:EncapsulationChecker, StageMetadataChecker, SubsetFamilies, SubsetParentIsImageable
├── usd.shade                → usdShadeValidators:* (9 validators)
├── usd.physics              → usdPhysicsValidators:* (4 validators)
├── usd.skel                 → usdSkelValidators:* (2 validators)
├── usd.utils                → usdUtilsValidators:* (5 validators)
└── usd.core.v25_05 [PROFILE] → predecessors: [usd.core, usd.geom, usd.shade, usd.physics, usd.skel, usd.utils]
```

### Step 2: Wire SimReady → usd.* predecessors

Update SimReady capability predecessors to inherit from corresponding `usd.*`:

- `com.nvidia.simready` ← also `usd` (already has this)
- `com.nvidia.simready.geom` ← also `usd.geom`
- `com.nvidia.simready.physics` ← also `usd.physics`
- `com.nvidia.simready.physics.rigidBodies` ← (inherits `usd.physics` via parent)
- `com.nvidia.simready.units` ← also `usd.core` (upAxis/metersPerUnit are core concerns)

### Step 3: Tests

Write `testUsdProfilesCapabilityBridge.py`:
- Test `usd.*` capability validator counts
- Test DAG traversal from SimReady profile reaches Pixar validators
- Test no duplicates when multiple paths lead to same capability
- Test the `usd.core.v25_05` profile as a standalone "core USD hygiene" profile

### Step 4: Update docs

- `overview.dox`: add section on the `usd.*` bridge layer
- `README.md`: update capability tree diagram
- Demo script: show combined validator resolution

## Open Questions

None — this is a clean implementation of Option A from the problem statement, which Jens approved.

## References

- Issue: jensjebens/OpenUSD#32
- Related: jensjebens/usd-profiles#8
- Problem statement: `plans/phase3b-problem-statement.md`
