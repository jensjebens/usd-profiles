# 4. Omniverse Asset Validator

## Overview

An extensible Python framework for validating OpenUSD assets, published on PyPI as [`omniverse-asset-validator`](https://pypi.org/project/omniverse-asset-validator/) (v1.11.2, Feb 2026). Inspired by Pixar's `usdchecker`, it extends validation with additional rules and auto-fixing. Apache-2.0 licensed (NVIDIA).

## Purpose

Provide a modular, extensible validation engine that can:
- Validate USD stages, layers, or entire directories
- Apply automatic fixes for common issues
- Be extended with custom rules
- Integrate with Pixar's UsdValidation framework
- Support the `omni.capabilities` profiles/requirements taxonomy

## Architecture

### Core Components

#### ValidationEngine
Central class that orchestrates validation:
- `validate(path)` — validates a file or directory
- `enableRule()` / `disableRule()` — selective rule activation
- `enableCategory()` — filter by category
- `initRules=True` — loads all registered rules at construction

#### IssueFixer
Applies automatic corrections:
```python
fixer = IssueFixer()
fix_results = fixer.fix(results.issues())
```

#### Rule System
Rules are Python classes inheriting from `BaseRuleChecker`. The codebase contains specialized checkers:

| Checker Module | Domain |
|---------------|--------|
| `_geometry_checker.py` | Mesh topology, normals, extents, welding |
| `_material_checker.py` | Materials, shaders, MDL, UsdPreviewSurface |
| `_layout_checker.py` | Prim hierarchy, kind, encapsulation |
| `_layer_checker.py` | Layer specs, type conformance |
| `_physics_checker.py` | Rigid bodies, colliders, joints, articulations, mass |
| `_misc_checker.py` | Stage metadata, default prims, dangling overs |
| `_performance_checker.py` | ASCII performance, time samples |
| `_utf8_checker.py` | Unicode/UTF-8 name validation |
| `_atomic_asset_checker.py` | USDZ packaging, asset paths |
| `_hierarchy_rules.py` | Hierarchy requirements |
| `_units_rules.py` | Units (upAxis, metersPerUnit) requirements |

#### Built-in Rules (~35+ rule classes)

Key rules include:
- **Stage**: `StageMetadataChecker`, `DefaultPrimChecker`, `DanglingOverPrimChecker`
- **Geometry**: `ExtentsChecker`, `ManifoldChecker`, `SubdivisionSchemeChecker`, `NormalsExistChecker`, `NormalsValidChecker`, `NormalsWindingsChecker`, `ZeroAreaFaceChecker`, `WeldChecker`, `ValidateTopologyChecker`, `IndexedPrimvarChecker`, `UnusedMeshTopologyChecker`, `UnusedPrimvarChecker`
- **Materials**: `TextureChecker`, `NormalMapTextureChecker`, `MaterialPathChecker`, `MaterialOutOfScopeChecker`, `UsdDanglingMaterialBinding`, `UsdMaterialBindingApi`, `MaterialUsdPreviewSurfaceChecker`, `ShaderImplementationSourceChecker`, `MaterialOldMdlSchemaChecker`
- **Physics**: `RigidBodyChecker`, `ColliderChecker`, `PhysicsJointChecker`, `ArticulationChecker`, `MassChecker`
- **Structure**: `PrimEncapsulationChecker`, `KindChecker`, `TypeChecker`, `LayerSpecChecker`, `UsdGeomSubsetChecker`, `UsdLuxSchemaChecker`, `SkelBindingAPIAppliedChecker`
- **Performance**: `UsdAsciiPerformanceChecker`
- **Unicode**: `UnicodeNameChecker`
- **Package**: `UsdzPackageValidator`, `MissingReferenceChecker`

### Integration with `omni.capabilities`

The validator bundles `omni.capabilities` and provides integration layers:
- `_profiles.py` — `ProfileRegistry` wrapping `omni.capabilities.Profiles`
- `_capabilities.py` — `CapabilityRegistry` wrapping `omni.capabilities.Capabilities`
- Maps `omni.capabilities.Requirement` codes to validator rules

### Bridge to Pixar UsdValidation

`_usd_validator_adapter.py` provides:
- `UsdValidatorAdapter` — adapts OV rules to Pixar's validator protocols
- `ValidatorProtocol` — mirrors Pixar's `UsdValidationValidator` interface
- `ValidatorErrorProtocol` — mirrors Pixar's `UsdValidationError`
- `ValidatorErrorSiteProtocol` — mirrors Pixar's error site interface

This is explicitly labeled "Temporary protocol for backward compatibility with UsdValidation" — suggesting planned convergence.

### CLI

```bash
omni_asset_validate asset.usda           # Validate single file
omni_asset_validate ./assets/            # Validate directory
omni_asset_validate --fix asset.usda     # Auto-fix
omni_asset_validate --category Material  # Filter by category
omni_asset_validate --csv-output r.csv   # Export results
omni_asset_validate --no-init-rules --rule StageMetadataChecker asset.usda
```

### Supporting Modules

| Module | Purpose |
|--------|---------|
| `_registry.py` | `VersionedRegistry` with semver-based lookup |
| `_semver.py` | Semantic versioning implementation |
| `_graph_tools.py` | Graph algorithms (for capability DAG?) |
| `_expression.py` | Expression evaluation |
| `_compliance_checker.py` / `_compliance_runners.py` | Compliance framework |
| `_json_reports.py` / `_csv_reports.py` | Report generation |
| `_default_categories.py` / `_categories.py` | Rule categorization |
| `_identifiers.py` | ID management |
| `_mesh_tools.py` | Mesh analysis utilities |
| `_stats.py` | Validation statistics |

## Requirements

- Python 3.10–3.12
- OpenUSD 22.11+ (via `pxr` module)
- Optional: NumPy (for optimizations)
- Pure Python — no compiled extensions

## Status

- **Production** — v1.11.2 on PyPI (Feb 2026)
- Active development (NVIDIA copyright 2024–2026)
- Actively bridging to Pixar's UsdValidation via adapter layer
- Bundles `omni.capabilities` for the profiles/requirements taxonomy
- Well-documented with Omniverse docs site

## Source

- PyPI: [`omniverse-asset-validator`](https://pypi.org/project/omniverse-asset-validator/) v1.11.2
- Docs: [docs.omniverse.nvidia.com/kit/docs/asset-validator](https://docs.omniverse.nvidia.com/kit/docs/asset-validator)
- License: Apache-2.0
