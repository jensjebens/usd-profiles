# 3. Pixar UsdValidation Framework

## Overview

A C++ (with Python bindings) validation framework built into OpenUSD core, located at `pxr/usdValidation` on the `dev` branch. Provides a systematic way to register, discover, and run validation tests against USD stages, layers, and prims.

## Purpose

Ensure USD assets are robust and interchangeable between different USD workflows by providing:
- A standard way to define validation rules
- A central registry for validator discovery
- Parallel execution of validation tests
- An auto-fix mechanism for common issues

## Architecture

### Core Components

#### UsdValidationValidator
A single validation test instance that can produce zero or more errors when run. Each validator has metadata:

| Field | Description |
|-------|-------------|
| `name` | Qualified name, e.g. `usdGeomValidators:StageMetadataChecker` |
| `pluginPtr` | Pointer to plugin (null for explicit validators) |
| `keywords` | Keywords for filtering/discovery |
| `doc` | Documentation string |
| `schemaTypes` | Associated schema types (e.g. `UsdGeomImageable`) |
| `isTimeDependent` | Whether the validator tests time-varying data |
| `isSuite` | Whether it's a suite of validators |

Validators are **immutable, non-copyable, and immortal** once registered.

#### UsdValidationContext
Runs a set of validators. Created from:
- A vector of validators (manual)
- Metadata queries on the registry (keywords, schema types)

Key feature: `includeAllAncestors` — when filtering by schema type, automatically includes validators for ancestor types (e.g. finding a validator for `UsdGeomSphere` also includes `UsdGeomGprim`, `UsdGeomImageable`).

**Validators run in parallel** via TBB thread pool.

#### UsdValidationRegistry
Central singleton registry. Provides:
- Validator lookup by name, keywords, schema types
- Lazy-loading of plugin validators
- Access to validator suites
- Registration of explicit validators

#### UsdValidationError
Result of a failed validation test:

| Field | Description |
|-------|-------------|
| `Name` | Error name, e.g. `MissingDefaultPrim` |
| `Identifier` | Qualified: `pluginName:validatorName.errorName` |
| `ErrorType` | Severity: None, Error, Warn, Info |
| `ErrorSites` | Where the error occurred: Layer, Stage, Prim, or Property |
| `Message` | Detailed error message |
| `ErrorData` | `VtValue` with additional data for fixers |

#### UsdValidationFixer
Associated with a specific validator and optionally a specific error name. Contains:
- `name` and `description`
- `keywords` for filtering
- `FixerCanApplyFn` — checks if fix is applicable
- `FixerImplFn` — applies the fix
- Fix is applied to a caller-provided `UsdEditTarget`

**Fixers are never called automatically** — the caller must explicitly invoke them.

### Task Functions

Three granularity levels:

| Level | Signature | Use When |
|-------|-----------|----------|
| `UsdValidateLayerTaskFn` | `(SdfLayer) → errors` | Layer metadata, format checks |
| `UsdValidateStageTaskFn` | `(UsdStage, TimeRange) → errors` | Stage-level validation |
| `UsdValidatePrimTaskFn` | `(UsdPrim, TimeRange) → errors` | Prim-level checks (preferred) |

Guideline: use the narrowest granularity possible for performance.

### Registration Paths

| | Explicit | Plugin |
|---|---------|--------|
| Metadata source | Caller provides `ValidatorMetadata` | `plugInfo.json` |
| Discoverability | Only after registration code runs | Metadata visible at startup; lazily loaded |
| Lazy loading | No | Yes |
| Use when | Prototyping, scripts, tests, dynamic rules | Shipping validators in distributed plugins |

### Plugin Registration (C++)

Validators declared in `plugInfo.json`:
```json
{
  "Plugins": [{
    "Info": {
      "Validators": {
        "keywords": ["commonKeyword"],
        "Validator1": {
          "doc": "Validator description",
          "schemaTypes": ["UsdGeomImageable"],
          "keywords": ["keyword1"]
        }
      }
    }
  }]
}
```

Registered via `TF_REGISTRY_FUNCTION(UsdValidationRegistry)` in C++.

### Python Support (Recently Merged)

Full Python binding available with both registration paths:

**Plugin registration:**
- `RegisterPluginLayerValidator(name, callable)`
- `RegisterPluginStageValidator(name, callable)`
- `RegisterPluginPrimValidator(name, callable)`

**Explicit registration:**
- `RegisterLayerValidator(metadata, callable)`
- `RegisterStageValidator(metadata, callable)`
- `RegisterPrimValidator(metadata, callable)`

Python callable signatures:
```python
# Layer: (layer: Sdf.Layer) -> list[ValidationError]
# Stage: (stage: Usd.Stage, timeRange: TimeRange) -> list[ValidationError]
# Prim:  (prim: Usd.Prim, timeRange: TimeRange) -> list[ValidationError]
```

**Performance caveat:** Python validators acquire the GIL on each invocation, so they don't parallelize among themselves and can starve C++ validators. Recommended for prototyping, not production pipelines.

### Built-in Validator Plugins

Located in `pxr/usdValidation/` on the dev branch:

| Plugin | Scope |
|--------|-------|
| `usdGeomValidators` | Geometry validation (e.g. `StageMetadataChecker`) |
| `usdShadeValidators` | Material/shader validation |
| `usdSkelValidators` | Skeletal animation validation |
| `usdPhysicsValidators` | Physics schema validation |
| `usdUtilsValidators` | General utility validators |

Plus `bin/` — likely command-line tooling.

### Time-Dependent Validation

- Default: validates against `GfInterval::GetFullInterval()` (−∞ to +∞)
- Can specify custom time intervals
- Runs against all timeCodes in the interval

## Status

- **On the `dev` branch** of OpenUSD — not yet in a stable release
- Python bindings recently merged
- 5 built-in validator plugin libraries
- Comprehensive C++ and Python APIs
- Framework only — does not define what "correct" means (that's the validators' job)
- No profile/capability concept — purely a validation execution engine

## Key Characteristics

- **Language**: C++ core with Python bindings
- **Parallelism**: TBB-based parallel validator execution
- **Extensibility**: Plugin system + explicit registration
- **Scope**: Execution framework, not a rule set
- **License**: Apache-2.0 (part of OpenUSD)

## Source

- Repository: [PixarAnimationStudios/OpenUSD](https://github.com/PixarAnimationStudios/OpenUSD/tree/dev/pxr/usdValidation)
- Branch: `dev`
- Detailed docs: `pxr/usdValidation/usdValidation/README.md`
