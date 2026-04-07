# 5. NVIDIA SimReady Foundation

## Overview

A central repository for defining **simulation content specifications** based on various runtime use cases. Located at [`nvidia/simready-foundation`](https://github.com/NVIDIA/simready-foundation) (under the `nvidia` GitHub org, not `NVIDIA-Omniverse`). Defines guidelines and requirements for OpenUSD content so that assets work reliably across rendering, simulation, robotics, and AI training workflows within NVIDIA Omniverse.

## Purpose

SimReady assets are "building blocks for industrial virtual worlds" — more than visual 3D objects, they include:
- Semantic labeling
- Dense captions
- Non-visual sensor attributes (e.g. surface coating, material properties)
- Physical properties (USDPhysics: rigid bodies, colliders, joints, mass)

The SimReady Foundation provides the **specification and validation pipeline** to ensure assets meet these standards.

## Architecture

### The Full Stack (revealed)

SimReady Foundation ties together multiple NVIDIA packages into a coherent pipeline:

```
Markdown specs (nv_core/sr_specs/docs/)
        │
        ▼  omniverse-usd-profiles (parser + codegen)
        │
        ▼  omni.capabilities (generated Python enums)
        │
        ▼  omniverse-asset-validator (validation engine)
        │
        ▼  validate_asset.py (profile-based validation)
```

**This is the key discovery: SimReady Foundation is the "umbrella" project that connects the spec → codegen → validation pipeline.**

### Three packages working together

| Package (PyPI) | Version | Role |
|----------------|---------|------|
| `omniverse-usd-profiles` | 1.11.0 | Markdown parser → Python code generator |
| `omniverse-asset-validator` | 1.11.2 | Validation engine + rule implementations |
| `usd-core` | ≥22.11 | USD Python bindings |

### `omniverse-usd-profiles` (the missing piece)

This standalone PyPI package (`omni.usd_profiles`) is the **spec-to-code pipeline**:

| Module | Purpose |
|--------|---------|
| `markdown/` | Parses markdown spec files (capabilities, features, profiles, requirements, examples, parameters) |
| `model/` | Data model: `Model`, `Capability`, `Feature`, `Profile`, `Requirement`, `Version`, `Tag`, `Parameter`, `Example` |
| `codegen/` | **Python code generator** — uses Jinja2 templates to produce `omni.capabilities` module from markdown specs |
| `serialization/` | JSON encode/decode for the model |
| `sphinx/` | Sphinx extensions for doc generation (requirement tables, capability reports, badges) |
| `store/` | `SpecificationsStore` for loading/saving parsed specs |

Key insight: `omni.capabilities` (bundled in the asset validator) is **generated output** from `omniverse-usd-profiles` processing the SimReady markdown specs.

### Layered Hierarchy (same as omni.capabilities)

| Layer | Purpose | Example |
|-------|---------|---------|
| **Requirement** | Single testable rule | "The stage must define a default prim" (SAMP.001) |
| **Capability** | Groups related requirements | Sample (SAMP), Visualization/Geometry (VG), Units (UN) |
| **Feature** | Cross-capability requirement set describing a queryable property | Minimal Placeable Visual, RBD Physics, Driven Joints |
| **Profile** | Bundle of features for a use case | Prop-Robotics-Neutral, Robot-Body-Isaac |

### Profiles (Production)

| Profile | Description |
|---------|-------------|
| **Prop-Robotics-Neutral** | Neutral-format props suitable for robotics pipelines |
| **Prop-Robotics-Physx** | Props with PhysX rigid-body physics |
| **Robot-Body-Neutral** | Neutral robot body with physics |
| **Robot-Body-Runnable** | PhysX robot body, runnable in simulation |

### Feature Dependencies

Features can have dependencies on other features (stored in `custom_data.dependencies`). The validation sample resolves these recursively, collecting all transitive requirements.

### Validation Workflow

`validate_asset.py` demonstrates the end-to-end flow:
1. Load specs from markdown (`sample_requirements/`, `sample_features/`, `sample_profiles/`)
2. Look up profile by ID + version
3. For each capability in the profile, resolve all requirements (including transitive feature dependencies)
4. Map requirements to validator rules via `RequirementsRegistry`
5. Run validation engine
6. Build feature-level pass/fail summary

### Spec Documentation System

The `nv_core/sr_specs/docs/` directory contains:
- `capabilities/` — Markdown spec files per capability (with `requirements/` subdirectories)
- `features/` — Feature definitions
- `profiles/` — Profile definitions
- `guides/` — End-to-end guides
- `indexes/` — Generated status reports

Custom Sphinx extensions auto-generate:
- Requirement tables from markdown
- Capability scoring and status badges (Complete ≥90%, Development 33–89%, Draft <33%)
- Priority-sorted requirement lists (essential → correctness → high-quality → performance)

### Repo Structure

```
nvidia/simready-foundation/
├── nv_core/
│   ├── sr_specs/          # Full SimReady specifications
│   │   └── docs/
│   │       ├── capabilities/
│   │       ├── features/
│   │       ├── profiles/
│   │       ├── guides/
│   │       └── indexes/
│   └── validator_sample/   # Validation sample code
│       ├── validate_asset.py
│       ├── requirements.txt
│       ├── sample_requirements/
│       ├── sample_features/
│       ├── sample_profiles/
│       └── loading/
└── README.md
```

Uses **Git LFS** for binary/USD asset files.

## Status

- **Active development** — public GitHub repo (2026 copyright)
- Requires Python 3.12+
- Production specs in `nv_core/sr_specs/` (noted as requiring additional dependencies not yet public)
- Validator sample works with the three PyPI packages
- Four production profiles defined
- Apache-2.0 licensed

## Relationship to Other Projects

| Related Project | Relationship |
|----------------|-------------|
| `omniverse-usd-profiles` | **Consumed directly** — parses the markdown specs and generates code |
| `omni.capabilities` | **Generated output** — produced by `omniverse-usd-profiles` from these specs |
| `omniverse-asset-validator` | **Validation engine** — executes rules mapped to requirements defined here |
| Pixar USD-Profiles Proposal | **Conceptual alignment** — both define capabilities/profiles, different architecture |
| Pixar UsdValidation | **Indirect** — OV asset validator bridges to it; SimReady specs could eventually target it |

## Source

- GitHub: [nvidia/simready-foundation](https://github.com/NVIDIA/simready-foundation)
- PyPI dependency: `omniverse-usd-profiles` v1.11.0
- Docs: [docs.omniverse.nvidia.com/simready](https://docs.omniverse.nvidia.com/simready/latest/index.html)
- License: Apache-2.0
