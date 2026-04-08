# Roadmap: Compatibility Strategy for Omniverse Kit 109+ Users

**Ref:** jensjebens/usd-profiles#15
**Date:** 2026-04-08

## Context

Our ProfileAPI and capability-driven validation pipeline is built on Pixar's
`UsdValidation` C++ framework. Omniverse users on Kit 109+ don't have this and
won't for some time. They need profile-driven validation **now**.

## Architecture

`omniverse-usd-profiles` becomes the single package that serves both as a
**codegen tool** (build time) and a **runtime DAG library** (query time).

```
omniverse-usd-profiles
├── CLI / codegen mode
│   ├── Input:  SimReady markdown specs (SRF repo)
│   ├── Output: capabilities.json      (canonical DAG definition)
│   ├── Output: plugInfo.json           (Pixar/UsdValidation format)
│   └── Output: entrypoint package      (pyproject.toml + scaffolding)
│
└── Library / runtime mode
    ├── Discover installed entrypoint packages (importlib.metadata)
    ├── Load capabilities.json from each package
    ├── Merge into unified capability DAG
    ├── Query API (predecessors, validators, profiles, filtering)
    └── OAV adapter (flatten DAG → 4-level hierarchy)
```

### Entrypoint Package Structure

Any vendor can publish capability definitions as a pip-installable package:

```
simready_validators/
├── pyproject.toml              # declares omni.usd_profiles entrypoint
├── capabilities.json           # DAG definition (canonical format)
├── validators/
│   ├── geom_checkers.py        # OAV-compatible checker classes
│   └── physics_checkers.py
└── __init__.py
```

### Capability DAG Format (capabilities.json)

The canonical format for capability graphs. Encodes the DAG with semantic
tags so any consumer can reconstruct the view it needs.

```json
{
  "schema": "usd-profiles/capability-dag/v1",
  "capabilities": {
    "usd": {
      "kind": "namespace",
      "docstring": "Base USD capability",
      "predecessors": []
    },
    "usd.geom": {
      "kind": "capability",
      "domain": "geometry",
      "docstring": "USD geometry validation",
      "predecessors": ["usd"],
      "validators": [
        "usdGeomValidators:EncapsulationChecker",
        "usdGeomValidators:StageMetadataChecker",
        "usdGeomValidators:SubsetFamilies",
        "usdGeomValidators:SubsetParentIsImageable"
      ]
    },
    "com.nvidia.simready.geom": {
      "kind": "feature",
      "domain": "geometry",
      "docstring": "SimReady geometry requirements",
      "predecessors": ["com.nvidia.simready", "usd.geom"],
      "validators": [
        "simready_validators.geom:VG_MESH_001",
        "simready_validators.geom:VG_014",
        "simready_validators.geom:VG_027",
        "simready_validators.geom:VG_025"
      ]
    },
    "com.nvidia.simready.prop_robotics_neutral": {
      "kind": "profile",
      "docstring": "SimReady neutral robotics prop profile",
      "predecessors": [
        "com.nvidia.simready.geom",
        "com.nvidia.simready.physics.rigidBodies",
        "com.nvidia.simready.hierarchy",
        "com.nvidia.simready.units"
      ]
    }
  }
}
```

### Semantic Tags (`kind`)

| `kind` | Description | SimReady equivalent | Example |
|---|---|---|---|
| `namespace` | Grouping node, no validators | — | `usd`, `com.nvidia.simready` |
| `capability` | USD-native capability with validators | Capability | `usd.geom`, `usd.physics` |
| `feature` | Vendor-defined feature with validators | Feature | `com.nvidia.simready.geom` |
| `profile` | Concrete validation target | Profile | `prop_robotics_neutral` |

Individual validator references within a node's `validators[]` array correspond
to SimReady **Requirements**.

This allows:
- **DAG consumers** (Pixar path, our usdProfiles C++) to walk the graph as-is
- **OAV consumers** to reconstruct the 4-level tree by filtering on `kind`
- **UI tools** to group by `domain` for display
- **Other tools** (linters, CI pipelines, documentation generators) to read the
  graph without depending on either validation engine

### Data Flow

```
SimReady markdown specs (SRF repo)
         │
         ▼
omniverse-usd-profiles codegen
         │
    ┌────┴──────────────┐
    ▼                   ▼
capabilities.json    plugInfo.json
(canonical DAG)      (Pixar format)
    │
    ▼
pip-installable entrypoint package
(simready-validators)
    │
    ├─── Kit 109+ user: pip install simready-validators
    │         │
    │         ▼
    │    omniverse-usd-profiles (runtime)
    │         ├── loads DAG from entrypoint
    │         ├── query API
    │         └── OAV adapter → omniverse-asset-validator
    │
    └─── Standalone USD user: plugInfo.json
              │
              ▼
         UsdValidation + usdProfiles C++ library
```

## User Groups

### Group 1: Omniverse Kit 109+ (near term)

```
pip install omniverse-usd-profiles simready-validators
omni_asset_validate --profile com.nvidia.simready.prop_robotics_neutral asset.usd
```

- `omniverse-usd-profiles` discovers `simready-validators` via entrypoints
- Loads the capability DAG, resolves the profile to validators
- OAV adapter maps to the 4-level hierarchy
- OAV runs the checkers

### Group 2: Standalone USD 25.08+ (medium term)

```
# Build OpenUSD with usdProfiles library
usdprofilecheck --profile com.nvidia.simready.prop_robotics_neutral asset.usd
```

- `usdProfiles` C++ CapabilityRegistry loads from plugInfo.json
- UsdValidation framework runs the validators
- Same DAG semantics, different engine

### Group 3: Converged (future)

- Kit ships USD with UsdValidation Python support
- OAV adapter bridge lofts C++ validators into OAV
- One `pip install` gets both engines working
- `omniverse-usd-profiles` runtime detects available engine and uses it

## Phases

### Phase A: Runtime DAG library in omniverse-usd-profiles

- Add `CapabilityGraph` class (Python equivalent of our C++ CapabilityRegistry)
- Load DAG from `capabilities.json` in installed entrypoint packages
- Query API: predecessors, transitive predecessors, validators, profiles
- Namespace filtering, DOT graph output (port from usdprofilecheck)
- **No OAV dependency** — pure library, usable standalone

### Phase B: OAV adapter layer

- Bridge `CapabilityGraph` query results → OAV's Requirement/Capability/Feature/Profile
- Use `kind` tags to map DAG nodes to the 4-level hierarchy
- Register checkers via the OAV `register_rule()` API
- Package as an `omni.asset_validator` entrypoint (Matt's SDD pattern)

### Phase C: Codegen updates

- Update `omniverse-usd-profiles` codegen to output:
  - `capabilities.json` (new canonical format)
  - `plugInfo.json` (Pixar format, derived from capabilities.json)
  - Entrypoint package scaffolding (pyproject.toml, __init__.py, registrant class)
- SRF CI runs codegen → publishes pip-installable packages

### Phase D: Convergence — Exit Strategy

When Kit ships a USD with both `UsdValidation` (incl. Python checker support)
and a native profiles/capability library:

**`omniverse-usd-profiles` narrows back to codegen-only:**
- The runtime DAG library is superseded by USD's native `CapabilityRegistry`
- The query API may survive as a lightweight CI/scripting tool for environments
  that don't link against USD C++, but it's no longer the primary path
- The codegen pipeline remains: SimReady markdown → `plugInfo.json` (Pixar format)
  that feeds directly into USD's native plugin system
- Entrypoint packages shift from publishing `capabilities.json` for Python
  consumption to publishing `plugInfo.json` for the Plug registry (or both,
  for backward compat during transition)

**OAV becomes a thin Kit UI shell:**
- The "temporary" adapter bridge (already in OAV's codebase) becomes the
  permanent architecture — OAV delegates to `UsdValidation` for execution
  and provides Kit-specific UI: viewport panel, progress bars, interactive
  validation, scoring/badges
- OAV-specific rules that aren't in Pixar's validator set remain as OAV
  entrypoint packages, but they register into `UsdValidation` via the adapter
  rather than running in their own framework

**What dissolves:**
- The Python `CapabilityGraph` runtime → replaced by C++ `CapabilityRegistry`
- The OAV adapter layer → replaced by OAV's existing C++→Python bridge, inverted
- The dual-engine detection logic → single engine (UsdValidation)

**What survives:**
- `omniverse-usd-profiles` codegen (SimReady markdown → Pixar formats)
- Entrypoint packages as a distribution mechanism for vendor validation rules
- `capabilities.json` schema as a portable interchange format (CI, docs, tooling)
- The `kind` semantic tags and DAG model — absorbed into the Pixar proposal

This is the best outcome: we build the interim infrastructure that proves the
concept for Kit 109+ users today, and when USD ships native support, our runtime
layers gracefully dissolve while the codegen and spec formats feed into the
native system.

## Open Questions

1. Should the entrypoint group be `omni.usd_profiles` (new, DAG-specific) or
   reuse `omni.asset_validator` (existing, Matt's SDD)?
   - Leaning toward new group: the DAG library is useful beyond just OAV
2. Does `capabilities.json` fully replace the generated Python in `omni.capabilities`,
   or do we maintain both during transition?
3. Version pinning: should `capabilities.json` declare which USD version's validators
   it references (for forward compatibility)?
4. Should `omniverse-usd-profiles` be renamed to reflect its expanded role
   during Phases A-C, or keep the name since it narrows back in Phase D?
