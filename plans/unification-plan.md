# Unification Plan — Converging on Pixar's USD Foundations

## Motivation

The NVIDIA stack (`simready-foundation` → `omniverse-usd-profiles` → `omni.capabilities` → `omniverse-asset-validator`) has built a parallel implementation of concepts that Pixar is standardising in OpenUSD core:

- **Pixar USD-Profiles proposal** → declarative capability taxonomy (DAG-based)
- **Pixar UsdValidation** → validation execution framework (C++ with Python bindings)

Both NVIDIA and Pixar define capabilities, profiles, validators, fixers, and registries — but with different architectures, naming, versioning, and extensibility models. This creates fragmentation: two competing ways to describe and validate the same assets.

**Goal:** Converge the SimReady Foundation specs and validation onto Pixar's proposals and frameworks as the underlying foundation, eliminating duplication while preserving the valuable content (rules, requirements, profiles) that NVIDIA has built.

## Acceptance Criteria

1. SimReady requirements map to Pixar capability identifiers (e.g. `VG.014` → `usd.geom.mesh.topology`)
2. Validators register as Pixar `UsdValidation` plugins (not parallel `BaseRuleChecker` classes)
3. Profiles are expressible as Pixar `ProfileAPI` declarations on prims
4. The codegen pipeline (`omniverse-usd-profiles`) can target Pixar's `plugInfo.json` + `schema.usda` format
5. The existing SimReady content (90+ requirements, 10 capabilities, 4 profiles) is preserved, not rewritten from scratch

## Current State → Target State

```
CURRENT (parallel stacks):

  Pixar Proposal (paper)          NVIDIA (production)
  ────────────────────           ─────────────────────
  Capability DAG (unimpl.)       omni.capabilities (Python enums)
  ProfileAPI (unimpl.)           omni.usd_profiles (codegen)  
  UsdValidation (dev branch)     omni.asset_validator (BaseRuleChecker)
  usdchecker --profile (prop.)   omni_asset_validate (CLI)

TARGET (unified):

  SimReady Specs (markdown source of truth)
        │
        ▼  codegen (updated omniverse-usd-profiles)
        │
  ┌─────┴──────────────────────────────┐
  │  Pixar plugInfo.json + schema.usda │  ← capability declarations
  │  Pixar ProfileAPI schema           │  ← profile authoring on prims
  │  Pixar UsdValidation plugins       │  ← validator implementations
  └────────────────────────────────────┘
        │
        ▼  validation runtime
  Pixar UsdValidation (C++ engine, TBB parallel)
        │
        ▼  tools
  usdchecker --profile <profile-name> asset.usda
```

## Approach — Three Phases

### Phase 1: Mapping (research + design, no code changes)

**1a. Capability name mapping**
Create a rosetta stone between NVIDIA requirement codes and Pixar capability identifiers:

| NVIDIA Code | NVIDIA Name | Proposed Capability ID |
|-------------|-------------|------------------------|
| `VG.014` | `usdgeom-mesh-topology` | `com.nvidia.simready.geom.mesh.topology` |
| `VG.027` | `usdgeom-mesh-normals-exist` | `com.nvidia.simready.geom.mesh.normals` |
| `RB.003` | `rigid-body-schema-application` | `com.nvidia.simready.physics.rigidBody` |
| `UN.001` | `upaxis` | `com.nvidia.simready.stage.upAxis` |
| `SL.001` | `semantic-label-capability` | `com.nvidia.simready.semantics.labels` |
| ... | ... | ... |

SimReady requirements use NVIDIA's reverse domain notation (`com.nvidia.simready.*`) as vendor extensions per the Pixar proposal's extension rules. This clearly distinguishes them from core `usd.*` capabilities while preserving full traceability to existing NVIDIA requirement codes.

Core USD validation rules (e.g. those that duplicate what Pixar's built-in `usdGeomValidators` already check) should map to the canonical `usd.*` namespace instead, avoiding duplication. SimReady-specific rules (semantic labels, dense captions, non-visual materials, RTX-specific) belong under `com.nvidia.simready.*`.

Per the Pixar proposal, vendor extensions **must transitively inherit from `usd`**:
```
usd (root)
└── com.nvidia.simready
    ├── com.nvidia.simready.geom
    │   ├── com.nvidia.simready.geom.mesh.topology
    │   └── com.nvidia.simready.geom.mesh.normals
    ├── com.nvidia.simready.physics
    │   ├── com.nvidia.simready.physics.rigidBody
    │   └── com.nvidia.simready.physics.joints
    ├── com.nvidia.simready.semantics
    │   └── com.nvidia.simready.semantics.labels
    └── com.nvidia.simready.stage
        └── com.nvidia.simready.stage.upAxis
```

**1b. Hierarchy alignment**
Map NVIDIA's 4-level hierarchy to Pixar's DAG:

| NVIDIA Layer | Pixar Equivalent | Notes |
|-------------|-----------------|-------|
| Requirement | Validator (test function) | 1:1 — each requirement becomes a UsdValidation validator |
| Capability | Capability (DAG node) | Group of validators, registered via plugInfo.json |
| Feature | Capability (tagged as feature) | Cross-cutting set — becomes a composite DAG node |
| Profile | Profile (tagged capability node) | Top-level — expressed via ProfileAPI on prims |

**1c. Versioning alignment**
Decide on versioning strategy:
- Option A: Adopt Pixar's `_v2` suffix for capabilities, keep semver for package versions only
- Option B: Contribute semver support to the Pixar proposal (stronger expressiveness)
- Option C: Map semver to USD-style (e.g. `1.0.0` → `_v1`, `2.0.0` → `_v2`, minor/patch handled at validator level)

**1d. Domain mapping**
Map NVIDIA's compatibility tags to Pixar's three domains and namespaces:

| NVIDIA Tag | Pixar Domain | Capability Namespace | Notes |
|-----------|-------------|---------------------|-------|
| `core-usd`, `open-usd` | Prim / Layer | `usd.*` (canonical) | Delegate to Pixar's built-in validators where they exist |
| `rtx` | Application | `com.nvidia.simready.rtx.*` | RTX-specific limitations/requirements |
| `physx` | Application | `com.nvidia.simready.physx.*` | PhysX-specific limitations |
| `kit`, `kit-107+` | Application | `com.nvidia.kit.*` | Omniverse Kit-specific |
| `omniverse` | Application | `com.nvidia.omniverse.*` | General Omniverse vendor extension |

Rules tagged `core-usd` / `open-usd` that overlap with Pixar's built-in validators (e.g. `usdGeomValidators:StageMetadataChecker`) should reference the canonical `usd.*` capability rather than duplicating under `com.nvidia.simready.*`.

### Phase 2: Infrastructure (make UsdValidation the engine)

**2a. Implement USD-Profiles proposal (or contribute to it)**
- Implement `ProfileAPI` schema in OpenUSD (or as a codeless schema plugin)
- Implement capability registration via `plugInfo.json` metadata
- Implement capability queries (explicit + introspective)
- Contribute back to Pixar or maintain as a plugin until upstream merges

**2b. Codegen targeting Pixar formats**
Update `omniverse-usd-profiles` codegen to output:
- `plugInfo.json` capability declarations (instead of Python enums)
- `schema.usda` schema-level capability metadata
- `UsdValidation` plugin registration code (C++ or Python)
- Profile definitions as `ProfileAPI`-compatible data

**2c. Port validators to UsdValidation**
For each `BaseRuleChecker` in `omniverse-asset-validator`:
1. Create a `UsdValidatePrimTaskFn` / `UsdValidateStageTaskFn` / `UsdValidateLayerTaskFn`
2. Register via `TF_REGISTRY_FUNCTION(UsdValidationRegistry)` or Python equivalent
3. Map NVIDIA requirement codes to validator names: `com.nvidia.simready:VG.014`
4. Port fixers to `UsdValidationFixer`

Python validators first (faster iteration), C++ for performance-critical rules later.

**2d. Profile-based validation CLI**
Leverage Pixar's proposed `usdchecker --profile`:
```bash
usdchecker --profile com.nvidia.simready.prop_robotics_neutral asset.usda
```

### Phase 3: Migration (swap out the NVIDIA engine)

**3a. Adapter deprecation**
The existing `_usd_validator_adapter.py` bridge becomes unnecessary — validators are native UsdValidation plugins.

**3b. `omniverse-asset-validator` evolution**
Two options:
- **Option A (thin wrapper):** Keep as a convenience layer that configures `UsdValidationContext` with SimReady profiles, adds JSON/CSV reporting, and provides the `omni_asset_validate` CLI
- **Option B (deprecate):** Move all value-adds (reporting, CLI) into upstream `usdchecker` contributions

**3c. `omni.capabilities` phase-out**
Generated Python enums replaced by Pixar's capability registry (loaded from `plugInfo.json` at plugin load time). The `VersionedRegistry` and `ProfileRegistry` classes become thin wrappers around `UsdValidationRegistry` queries.

## Open Questions

1. **Pixar proposal status:** Is the USD-Profiles proposal actively being implemented, or stalled? This determines whether we implement it ourselves vs. wait for upstream.
2. **UsdValidation release timeline:** When does `pxr/usdValidation` land in a stable OpenUSD release? If it's imminent, we build on it. If not, we may need to vendor or build against `dev`.
3. **Python validator performance:** Pixar notes Python validators can starve C++ ones. For SimReady's 90+ rules, is Python-first acceptable or do we need C++ from the start?
4. **Backward compatibility:** Existing SimReady users depend on `omni_asset_validate` CLI and `omni.capabilities` Python API. What's the migration path?
5. **Contribution model:** Fork + PR to OpenUSD? Separate plugin repo? NVIDIA-internal process?

## Risks

| Risk | Mitigation |
|------|-----------|
| Pixar Profiles proposal changes significantly | Phase 1 mapping is low-cost; re-map if proposal evolves |
| UsdValidation API breaks before stable release | Pin to specific dev commit; contribute stability patches |
| Python validator perf insufficient for large scenes | Phase 2c: C++ path for hot validators; Python for prototyping |
| NVIDIA won't adopt upstream approach | Plan works as a plugin either way — doesn't require NVIDIA buy-in |

## Non-Goals

- We are NOT proposing changes to the Pixar USD-Profiles proposal's design
- We are NOT rewriting SimReady requirements from scratch
- We are NOT building a new validation framework — we're adopting Pixar's
