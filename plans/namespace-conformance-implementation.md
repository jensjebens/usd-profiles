# Namespace Conformance: Implementation Plan

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23
**Depends on:** `research/namespace-ownership-mapping.md`

## Current State (inconsistent)

The codegen currently produces IDs from different sources with different conventions:

```
com.nvidia.simready                          ← namespace (codegen hardcoded)
com.nvidia.simready.geometry                 ← capability (from markdown filename)
com.nvidia.simready.FET001_BASE_NEUTRAL      ← feature variant (from markdown Internal ID, SCREAMING_SNAKE)
com.nvidia.simready.Prop-Robotics-Neutral    ← profile (from TOML table name, Title-Case-Hyphens)
com.nvidia.simready.fet_001_minimal          ← feature (from markdown filename, lowercase_snake)
```

Five different naming conventions in one DAG. This happens because:
1. Capability IDs come from markdown directory names (lowercase_snake)
2. Feature variant IDs come from Internal ID fields (SCREAMING_SNAKE)
3. Feature file IDs come from markdown filenames (lowercase_snake)
4. Profile IDs come from TOML table headers (Title-Case-Hyphens)
5. The `com.nvidia.simready.` prefix is hardcoded in codegen

## Target State (consistent)

All IDs should follow a consistent convention:

```
com.nvidia.usd                               ← root namespace
com.nvidia.usd.core                          ← OAV core capability
com.nvidia.usd.geom                          ← OAV geometry capability
com.nvidia.usd.physics                       ← OAV physics capability
com.nvidia.usd.shade                         ← OAV materials capability

com.nvidia.simready                          ← root namespace
com.nvidia.simready.capabilities.hierarchy   ← SRF capability
com.nvidia.simready.capabilities.geom        ← SRF geometry (opinionated)
com.nvidia.simready.features.minimal_neutral ← SRF feature (FET001)
com.nvidia.simready.features.rigid_body_physics_neutral  ← SRF feature (FET003)
com.nvidia.simready.profiles.prop_robotics_neutral       ← SRF profile
```

Convention: lowercase, dot-separated, descriptive names. No SCREAMING_SNAKE, no Title-Case-Hyphens.

## Scope

Apply the `com.nvidia.usd.*` / `com.nvidia.simready.*` namespace split across all repos and codegen outputs.

## What changes

### 1. capabilities.json (DAG node IDs)

**Current:**
```json
"com.nvidia.simready.geometry": { "kind": "capability", ... }
```

**After:**
```json
"com.nvidia.usd.geom": { "kind": "capability", "validators": ["VG.014", ...] }
"com.nvidia.simready.geom": { "kind": "capability", "validators": ["VG.MESH.001", "VG.025", "VG.RTX.001"] }
```

Two capability nodes for geometry: one OAV-owned (`com.nvidia.usd.geom`) with 28 general checks, one SRF-owned (`com.nvidia.simready.geom`) with 3 opinionated checks. SRF inherits from OAV via DAG predecessors:

```json
"com.nvidia.simready.geom": {
    "predecessors": ["com.nvidia.simready", "com.nvidia.usd.geom"],
    ...
}
```

Same pattern we used for the `usd.*` bridge in the C++ path.

### 2. Codegen (`_py_generate.py`)

The `_generate_capability_graph_json` method currently puts everything under `com.nvidia.simready.*`. It needs the mapping table to route codes to the correct namespace.

**Option A:** Hardcoded mapping in codegen (simple, explicit)
**Option B:** Tags in the SRF markdown (source of truth)
**Option C:** Separate mapping file consumed by codegen

### 3. Profile TOML references

Profiles reference features, features reference capabilities. If capability IDs change, the feature→capability links in the DAG need updating. But TOML profiles reference feature IDs (not capability IDs), so profiles themselves don't change.

### 4. SRF validation.py files

Requirement **codes** (VG.014, HI.001, etc.) don't change — they're the stable identifiers. Only the DAG **node IDs** (the capability that contains them) change. The decorators still reference requirement codes, not capability IDs, so **no changes to validation.py files**.

### 5. OAV `_graph_loader.py`

The graph loader resolves by requirement **code** (not by capability namespace), so it works regardless of namespace changes. **No changes needed.**

### 6. `simready_validators.capabilities` (generated enums)

The enum class names (`GeometryRequirements`, etc.) don't change — they're Python identifiers. The requirement codes inside don't change. **No changes needed.**

### 7. `omni.capabilities` (OAV's generated package)

This stays as-is until OAV explicitly migrates. The codes match by value, not by namespace. **No changes for now.**

## What does NOT change

- Requirement codes (VG.014, HI.001, etc.) — stable identifiers
- Python enum class names — stable identifiers
- Checker class names — stable identifiers
- Decorator linkage — by code, not namespace
- Profile/feature structure in TOML — references feature IDs

## What changes

- `capabilities.json` DAG node IDs: split existing nodes into OAV/SRF namespaces
- Codegen: needs the ownership mapping to route codes correctly
- DAG predecessor edges: SRF capabilities inherit from corresponding `com.nvidia.usd.*`
- DOT graph visualization: new color for `com.nvidia.usd.*` nodes

## Risk Assessment

**Low risk:** The namespace only affects DAG node IDs. All functional code (checkers, decorators, graph loader) works on requirement codes which don't change.

**Migration path:** Can be done as a single codegen change since the DAG is regenerated from source. Old `capabilities.json` files stop working but nobody depends on specific node IDs yet (the system is pre-release).

## Repos affected

| Repo | Change | Risk |
|------|--------|------|
| `open-usd-profiles` | Codegen mapping table | Low — single file |
| `simready-validators` | Regenerate capabilities.json | Low — regenerated |
| `simready-foundation` | None | — |
| `asset-validator` | None | — |
| `jensjebens/OpenUSD` | Update plugInfo.json node IDs | Low — aligned with DAG |

## Recommendation

This is a clean change with low blast radius because it only affects DAG node IDs (generated output), not functional code. But it's a good idea to do it **before** anyone starts depending on specific capability node IDs — i.e., now.
