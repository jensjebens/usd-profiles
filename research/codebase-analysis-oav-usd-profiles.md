# Codebase Analysis: OAV + omniverse-usd-profiles

**Date:** 2026-04-08
**Ref:** jensjebens/usd-profiles#15

## omniverse-usd-profiles (v1.11.2)

### Architecture
- **Markdown parser** (`omni.usd_profiles.markdown`): Reads SimReady spec markdown files
- **Model** (`omni.usd_profiles.model`): Dataclasses for Requirement, Capability, Feature, Profile
- **Store** (`omni.usd_profiles.store`): Versioned registries (CapabilityStore, ProfileStore, etc.)
- **Codegen** (`omni.usd_profiles.codegen`): Jinja2 templates → Python code (generates `omni.capabilities`)
- **Serialization** (`omni.usd_profiles.serialization`): JSON encode/decode for specs
- **Sphinx** (`omni.usd_profiles.sphinx`): Documentation generation

### Key observations
1. **No DAG concept** — the model is strictly hierarchical: Profile → Features → Capabilities → Requirements
2. **No runtime query API** — it's purely a codegen tool. No `GetAllValidatorsForCapability()` equivalent
3. **Serialization exists** — JSON encoder/decoder for all model types, could be extended
4. **Store layer** has versioned registries with `find()`, `keys()`, `latest_values()` — good foundation
5. **No entrypoint loading** — reads from filesystem directories, not installed packages

### What needs to change for the roadmap
- Add `capabilities.json` as an output format (alongside generated Python)
- Add DAG semantics: `predecessors` field on capabilities, `kind` tags
- Add runtime mode: load from entrypoint packages, merge DAGs, expose query API
- The store layer is close to what we need but needs DAG traversal methods

## omniverse-asset-validator (OAV)

### Architecture
- **Plugin system** (`_plugins.py`): **Already implements Matt's SDD** — `omni.asset_validator` entrypoint group, `PluginProtocol` with `on_startup()`/`on_shutdown()`, topological sort by package dependencies
- **Registries**: Separate singletons for `CapabilityRegistry`, `FeatureRegistry`, `RequirementsRegistry`, `ProfileRegistry`
- **Engine** (`_engine.py`): `ValidationEngine` runs rules, supports enable/disable by requirement/capability/feature
- **ComplianceChecker**: The actual runner that traverses the stage and runs checkers
- **Categories** (`_categories.py`, `_default_categories.py`): Rule organization

### Key observations
1. **Plugin system is LIVE** — `PluginManager` already discovers and loads `omni.asset_validator` entrypoints with topological sort. `DefaultPlugin` is the built-in registrant.
2. **4-level hierarchy already works** — `CapabilityRegistry`, `FeatureRegistry`, etc. are separate singletons
3. **No DAG support** — Capabilities have `requirements: list[Requirement]` but no `predecessors`
4. **ValidationEngine already supports filtering** by capability, feature, requirement — `enable_capabilities()`, `disable_features()`, etc.
5. **Requirement has a `validator` field** that maps to a `BaseRuleChecker` class

### What needs to change
- OAV doesn't need DAG support — it consumes the flattened 4-level hierarchy
- The adapter layer in `omniverse-usd-profiles` should register into OAV's existing registries
- No changes needed to OAV's plugin system — we just publish a new entrypoint package

## Integration Path

The cleanest approach:

### 1. `omniverse-usd-profiles` gains runtime mode (Phase A)
- New `CapabilityGraph` class with DAG semantics
- Reads `capabilities.json` from installed entrypoint packages
- Query API: predecessors, transitive predecessors, validators, profiles

### 2. OAV adapter is a thin entrypoint package (Phase B)
- Published as e.g. `simready-validators` with `omni.asset_validator` entrypoint
- `on_startup()`:
  1. Uses `omniverse-usd-profiles` runtime to load the DAG
  2. For each profile → flattens features → registers in OAV's `ProfileRegistry`
  3. For each feature → registers in OAV's `FeatureRegistry`
  4. For each capability → registers in OAV's `CapabilityRegistry`
  5. For each requirement with a checker → registers in OAV's `RequirementsRegistry`
- `on_shutdown()`: unregisters everything

### 3. No changes to OAV core needed
- OAV's plugin system already handles discovery, loading, topological sort
- OAV's registries already handle the 4-level hierarchy
- The adapter just bridges between the DAG model and the flat registries

This means:
- **Phase A** is purely in `omniverse-usd-profiles`
- **Phase B** is a new pip package (the adapter + checkers)
- **OAV itself doesn't change** — it just gets a new plugin
