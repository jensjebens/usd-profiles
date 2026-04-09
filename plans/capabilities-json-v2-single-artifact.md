# Plan: capabilities.json v2 — Single Artifact, No Generated Enums

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#28
**Author:** Profiles agent

## Motivation

The codegen pipeline produces two parallel representations of the same data:
1. `capabilities.json` — DAG structure only (kind, predecessors, validators, docstring)
2. Generated Python enums — rich metadata (display_name, message, version, tags, parameters, examples)

This creates redundancy, splits the data model, and misaligns with Pixar's `plugInfo.json` pattern where a single declarative JSON file is the source of truth.

## Goal

Make `capabilities.json` the **single generated artifact** that carries both DAG structure AND rich requirement metadata. Eliminate the generated Python enum layer entirely.

## Acceptance Criteria

### omniverse-usd-profiles (codegen + runtime)
- [ ] `capabilities.json` v2 schema defined with `requirements` dict per node
- [ ] Each requirement carries: display_name, message, version, path, compatibility, tags, parameters, examples
- [ ] `CapabilityGraph` accepts both v1 and v2 schema (backward compat)
- [ ] New query methods: `get_requirement(code)`, `get_requirements(node_id)`
- [ ] Requirements returned as objects satisfying the OAV `Requirement` protocol
- [ ] `PythonGenerator._generate_capability_graph_json()` enriched to emit v2
- [ ] Python enum generation methods remain but are no longer called by default
- [ ] All existing tests pass
- [ ] New tests for v2 loading, requirement queries, protocol compatibility

### simready-validators (consumer)
- [ ] Regenerated `capabilities.json` with v2 schema (enriched)
- [ ] `capabilities/` Python package can be removed (or reduced to protocols)
- [ ] `SimReadyPlugin` updated if it references enums
- [ ] Existing tests adapted

## Approach

### 1. Schema v2 for capabilities.json

```json
{
  "schema": "usd-profiles/capability-dag/v2",
  "capabilities": {
    "com.nvidia.simready.capabilities.hierarchy": {
      "kind": "capability",
      "domain": "hierarchy",
      "docstring": "Hierarchy requirements for SimReady assets",
      "predecessors": ["com.nvidia.simready"],
      "validators": ["HI.004", "HI.001"],
      "requirements": {
        "HI.004": {
          "display_name": "stage-has-default-prim",
          "message": "Stage must specify a default prim to define the root entry point.",
          "version": "1.0.0",
          "path": "hierarchy/requirements/stage-has-default-prim.html",
          "compatibility": "other",
          "tags": ["essential"],
          "parameters": [],
          "examples": []
        }
      }
    }
  }
}
```

### 2. CapabilityGraph changes

Add to `CapabilityNode`:
```python
requirements: dict[str, dict[str, Any]] = field(default_factory=dict)
```

Add to `CapabilityGraph`:
```python
def get_requirement(self, code: str) -> RequirementInfo | None
def get_requirements(self, node_id: str) -> list[RequirementInfo]
def get_all_requirements(self) -> list[RequirementInfo]
```

Where `RequirementInfo` is a frozen dataclass that satisfies OAV's `Requirement` protocol.

### 3. Codegen changes

In `_py_generate.py._generate_capability_graph_json()`:
- For each capability/feature node, iterate `store.requirements` and attach the full metadata to the `requirements` dict on each node.
- Bump schema to v2.

### 4. simready-validators changes

- Regenerate `capabilities.json` using updated codegen
- Remove or deprecate the `capabilities/` Python package
- Update `SimReadyPlugin.on_startup()` to use `CapabilityGraph` for requirement metadata

## What's NOT in scope

- Changing OAV's checker decorators (that's a separate issue, affects the asset-validator repo)
- Namespace conformance (#23, blocked)
- Profile detection/stamping (#24, owned by someone else)

## Savings / Simplification

### Lines eliminated
- `_requirements.py` — ~800 lines of generated enum
- `_capabilities.py` — ~1500 lines of generated enum with nested requirement refs
- `_features.py` — ~300 lines of generated enum
- `_profiles.py` — ~200 lines of generated enum
- `_examples.py` — ~400 lines of generated enum
- `_parameters.py` — ~200 lines of generated enum
- `__init__.py` — ~80 lines of generated imports
- **Total: ~3,500 lines of generated Python eliminated**

### Architectural wins
- Single artifact instead of two parallel representations
- Any consumer (browser, CLI, CI) gets full metadata from JSON alone
- Mirrors Pixar's `plugInfo.json` → registry → validators pattern
- Cleaner convergence path: in Phase D, `capabilities.json` feeds `plugInfo.json`
- No Python import needed to browse profiles/requirements

### What stays
- `capabilities.json` (enriched to v2)
- `CapabilityGraph` (enhanced with requirement queries)
- `Requirement` protocol definition (for OAV decorator compatibility)
- Codegen (simplified — outputs only JSON)
