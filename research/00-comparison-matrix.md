# Comparison Matrix — USD Validation & Profiles Projects

## Project Scope

| Dimension | Pixar USD-Profiles Proposal | Pixar UsdValidation | NVIDIA SimReady Foundation | NVIDIA `omni.usd_profiles` | NVIDIA `omni.capabilities` | NVIDIA Asset Validator |
|-----------|---------------------------|--------------------|-----------------------------|---------------------------|------------------------------|----------------------|
| **Primary purpose** | Declarative capability taxonomy for interoperability | Validation execution framework | Simulation content spec repository | Spec parser + Python code generator | Requirements/capabilities data model (generated) | Extensible asset validation engine |
| **What it defines** | What capabilities exist and how they relate | How to run and register validators | What SimReady assets must satisfy | How to turn markdown specs into code | What rules/capabilities/features exist | Concrete validation rules + auto-fix |
| **Analogy** | Package dependency graph | Test runner (pytest) | Test specifications (markdown) | Schema compiler | Test metadata catalogue (compiled output) | Test suite (with runner) |
| **Language** | Spec / proposal document | C++ with Python bindings | Markdown + Python | Pure Python (Jinja2 codegen) | Pure Python (generated) | Pure Python |
| **Status** | Proposal (not implemented) | On `dev` branch (not released) | Public repo (2026) | PyPI v1.11.0 | Generated, bundled in OV validator | Production (PyPI v1.11.2) |
| **License** | N/A (proposal) | Apache-2.0 (OpenUSD) | Apache-2.0 (NVIDIA) | Apache-2.0 (NVIDIA) | Apache-2.0 (NVIDIA) | Apache-2.0 (NVIDIA) |

## Conceptual Model

| Concept | Pixar Profiles | Pixar UsdValidation | NVIDIA `omni.capabilities` | NVIDIA Asset Validator |
|---------|---------------|--------------------|-----------------------------|----------------------|
| **Atomic unit** | Capability (DAG node) | Validator (test function) | Requirement (code + message) | Rule (BaseRuleChecker) |
| **Grouping** | Profile (tagged DAG node) | Suite | Capability → Feature → Profile | Category |
| **Versioning** | USD-style (`_v2`) | N/A | Semver (`1.0.0`) | Package version |
| **Graph structure** | DAG with inheritance | Flat (registry) | 4-level hierarchy | Flat (categories) |
| **Scope tags** | Domain (Layer/Prim/App) | Schema types, keywords | Compatibility (`core-usd`, `rtx`, etc.) | Categories |
| **Extensibility** | plugInfo.json + schema.usda | Plugin system + explicit | Python Enums (code-first) | Rule subclasses |

## Validation Architecture

| Aspect | Pixar Profiles | Pixar UsdValidation | NVIDIA `omni.capabilities` | NVIDIA Asset Validator |
|--------|---------------|--------------------|-----------------------------|----------------------|
| **Runs validation?** | No (query/declaration only) | Yes (execution engine) | No (data model only) | Yes (full engine) |
| **Parallel execution** | N/A | TBB threads (C++) | N/A | Sequential (Python) |
| **Auto-fix** | N/A | Yes (`UsdValidationFixer`) | N/A | Yes (`IssueFixer`) |
| **CLI tool** | Proposed: `usdprofile`, `usdchecker --profile` | Part of framework | None | `omni_asset_validate` |
| **Report formats** | N/A | Errors with sites | N/A | JSON, CSV |
| **Time-dependent** | N/A | Yes (TimeRange) | N/A | Limited |

## Query & Discovery

| Aspect | Pixar Profiles | Pixar UsdValidation | NVIDIA `omni.capabilities` | NVIDIA Asset Validator |
|--------|---------------|--------------------|-----------------------------|----------------------|
| **Query model** | Explicit (ProfileAPI) + Introspective (scene scan) | Registry lookup by name/keywords/schema | VersionedRegistry with semver lookup | Engine with rule enable/disable |
| **Traversal pruning** | Yes (key design feature) | Via prim predicates | N/A | N/A |
| **Scene introspection** | Capability inference from schemas | Validator dispatched per-prim | N/A | Per-prim rule application |
| **Application queries** | Yes (what can this app do?) | N/A | Protocol-based duck typing | N/A |

## Overlap & Relationship

```
┌──────────────────────────────────────────────────────────────────────┐
│                  Pixar USD-Profiles Proposal                         │
│   (Declarative capability taxonomy — the "what")                     │
│                                                                      │
│   ┌──────────────────────┐                                           │
│   │ Pixar UsdValidation  │    NVIDIA Stack:                          │
│   │ (Execution engine —  │                                           │
│   │  the "how")          │    SimReady Foundation (markdown specs)    │
│   └──────────┬───────────┘            │                              │
│              │                        ▼                              │
│              │              omniverse-usd-profiles (parser+codegen)  │
│              │                        │                              │
│              │                        ▼                              │
│              │              omni.capabilities (generated enums)      │
│              │                        │                              │
│              │     ┌──────────────────┘                              │
│              └─────┤  NVIDIA Asset Validator                         │
│                    │  (rules + engine + UsdValidation bridge)        │
│                    └─────────────────────────────────────            │
└──────────────────────────────────────────────────────────────────────┘
```

### Key Relationships

1. **SimReady Foundation → `omniverse-usd-profiles` → `omni.capabilities`**: This is a spec-to-code pipeline. SimReady defines requirements, capabilities, features, and profiles as markdown files. `omniverse-usd-profiles` parses them and generates Python code (`omni.capabilities`). This generated code is bundled into `omniverse-asset-validator`.

2. **Pixar Profiles ↔ UsdValidation**: The Profiles proposal explicitly references UsdValidation for its validation story (`usdchecker --profile`). They are complementary: Profiles defines *what to check*, UsdValidation defines *how to check*.

3. **NVIDIA stack ↔ Pixar Profiles**: Both define a taxonomy of capabilities/requirements, but with different structures (flat hierarchy vs. DAG) and different versioning (semver vs. USD-style). NVIDIA's is more concrete (actual requirement enums with codes), Pixar's is more architectural (how capabilities relate and propagate).

4. **NVIDIA Asset Validator ↔ UsdValidation**: The asset validator has an explicit adapter layer (`_usd_validator_adapter.py`) to bridge to Pixar's UsdValidation protocols. This is labeled "temporary" — suggesting planned convergence.

5. **SimReady Foundation profiles**: The first concrete profiles are simulation-focused (Prop-Robotics-Neutral, Robot-Body-Isaac, etc.), demonstrating the system's primary use case: ensuring USD assets are simulation-ready.

## Gap Analysis

| Gap | Description |
|-----|-------------|
| **No unified taxonomy** | Pixar uses DAG-based capabilities; NVIDIA uses flat requirement codes. No mapping between `usd.geom.skel` (Pixar) and `VG.014` (NVIDIA). |
| **Versioning mismatch** | Pixar uses `_v2` suffix; NVIDIA uses semver `1.0.0`. Different granularity and semantics. |
| **No ProfileAPI implementation** | Pixar proposes `ProfileAPI` for authoring on prims, but it doesn't exist yet. NVIDIA has no equivalent — profiles are engine-side only. |
| **No scene-level capability inference** | Pixar proposes introspective queries that scan a scene and infer capabilities. NVIDIA's system is validation-focused, not discovery-focused. |
| **Application domain missing** | Pixar proposes "what can this app do?" queries. Neither NVIDIA project addresses this. |
| **Layer domain missing** | Pixar proposes layer-level capabilities. NVIDIA's requirements are mostly prim/stage-focused. |
| **Python-only vs C++** | NVIDIA's entire stack is Python. Pixar's UsdValidation is C++ with Python bindings. Performance implications for large-scale validation. |
| **No standard rule naming** | NVIDIA uses `VG.014`, `RB.005`, etc. Pixar proposes `usd.geom.subdiv`. No rosetta stone between them. |

## Maturity Comparison

| Project | Maturity | Production Ready? |
|---------|----------|-------------------|
| Pixar USD-Profiles Proposal | Early (proposal document) | No — no implementation |
| Pixar UsdValidation | Pre-release (dev branch) | No — not in stable release |
| NVIDIA `omni.capabilities` | Active development (v1.11.2) | Partial — bundled, no standalone |
| NVIDIA Asset Validator | Production (v1.11.2, PyPI) | Yes — actively used |
