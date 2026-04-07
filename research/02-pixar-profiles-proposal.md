# 2. Pixar USD-Profiles Proposal

## Overview

A formal OpenUSD proposal (Version 1.0) for a **declarative capability taxonomy** that enables applications and assets to declare, discover, and validate functional capabilities. Located in the [OpenUSD-proposals](https://github.com/PixarAnimationStudios/OpenUSD-proposals/tree/main/proposals/profiles) repository.

## Purpose

USD's flexibility (custom schemas, proprietary extensions, format variants) creates interoperability challenges. Different apps implement different subsets. The Profiles proposal introduces a standardized, declarative way to:

- Discover and validate a document's compatibility and runtime requirements
- Assess interoperability between scenes and applications
- Determine capability satisfaction for declared functional needs

Analogous to a package manager's dependency graph, but **declarative only** — no runtime resolution, installation, or binding.

## Architecture

### Core Concepts

#### Capabilities
Atomic units of functionality organized in a **directed acyclic graph (DAG)**. Each capability is a named identifier using reverse-domain notation:

```
usd (root)
└── usd.image
    ├── usd.image.jpeg
    ├── usd.image.png
    └── usd.image.avif
```

Graph principles:
- **Ancestral Derivation**: Capabilities inherit hierarchically
- **Interoperability Constraints**: Siblings must share a common descendant
- Naming: `usd.geom.skel`, `usd.shade.material`, `yoyodyne.dimensional.contabulator`

#### Profiles
A Profile is a **tagged capability node** representing a coherent set of functionality defined by its transitive predecessors in the DAG. Created for:

- OpenUSD releases: `usd.core.v23_05`, `usd.core.v23_11`
- Domain standards: `aousd.interchange.v1_0`
- Application targets: `realitykit.v1_0`

Profile evolution uses DAG re-derivation:
```
usd.core.v23_11 → [usd.core.v23_05, usd.new.feature.v23_11]
```

### Three Domains

| Domain | Scope | Examples |
|--------|-------|---------|
| **Layer** | File-level capabilities | File formats, compression, encoding, layer metadata |
| **Prim** | Prim and descendants | Schemas (skel, physics), geometry types, prim metadata |
| **Application** | App-level support | Physics simulation, import/export, UI features |

### Query System

Two query mechanisms:

1. **Explicit Capability Query** — Fast, reads authored `ProfileAPI` metadata on prims. Supports caching and traversal pruning.
2. **Introspective Query** — Exhaustive scene introspection, infers capabilities from schemas. Higher cost, used for validation and capability inference.

Six standard query types:
- Narrowest required capability set
- Specific capability check
- Capability set coverage
- Full read/write interop support
- Proper display for review
- Approximate display for browsing

### Authoring

Two mechanisms:
1. **Explicit authoring** via `ProfileAPI` — applied to prims manually or by tools
2. **Introspective authoring** — tools infer capabilities from schema metadata and author back

Resolution semantics:
- Profiles are locally authored, not aggregated across hierarchies
- Resolved using standard USD value resolution
- Nearest ancestor's ProfileAPI applies if none authored on a prim

### Capability Registration

Capabilities declared in three places:
1. **Plugin domain** (`plugInfo.json`): Broad capabilities at plugin load time
2. **Schema domain** (`schema.usda`): Fine-grained per-schema-type capabilities
3. **Static registration**: Capabilities registered in code

### Versioning

Uses USD schema versioning conventions (underscore + integer):
- `yoyodyne.dimensional.contabulator` → `yoyodyne.dimensional.contabulator_v2`
- NOT semver (considered and rejected as redundant with the DAG structure)

Versioning required for: capability removal, breaking changes, major extensions
NOT required for: schema version bumps, additive changes, doc updates

### Validation Integration

Integrates with Pixar's UsdValidation framework:
- `usdchecker --profile usd.core.v23_05 asset.usda`
- `usdprofile analyze --output-profile asset.usda`
- `usdprofile validate --strict asset.usda`

### Late Binding (Future)

Envisions dynamic schema loading, plugin extension, and capability-driven validation at runtime. Out of scope for v1.0 but architecturally enabled.

### Vendor Extensions

- Canonical capabilities maintained by OpenUSD/AOUSD
- Vendors use unique prefixes: `yoyodyne.`, `realitykit.`
- Extensions must transitively inherit from `usd` root
- Standardization path: extensions may later declare relationships to official capabilities

## Proposed Capability Taxonomy

The proposal includes a draft taxonomy covering:
- Base: `usd`, `usd.format`, `usd.asset`, `usd.typed`
- Geometry: `usd.imageable`, `usd.geom.skel`, `usd.pointInstancer`
- Rendering: `usd.shade`, `usd.volume`, `usd.lighting`, `usd.renderman`, `usd.hydra`
- Animation: `usd.animation`
- Physics: `usd.physics`
- Collections: `usd.collection`

## Status

- **Proposal stage** — in the OpenUSD-proposals repository
- No implementation merged into OpenUSD core yet
- Defines `ProfileAPI` schema but it doesn't exist in the codebase yet
- Command-line tools (`usdchecker --profile`, `usdprofile`) are proposed, not implemented
- Detailed and thorough spec with worked examples
- Explicitly references integration with UsdValidation framework

## Key Differences from NVIDIA's Implementation

| Aspect | Pixar Proposal | NVIDIA `omni.capabilities` |
|--------|---------------|---------------------------|
| Hierarchy | Capability DAG (graph) | Flat: Requirement → Capability → Feature → Profile |
| Versioning | USD-style (`_v2`) | Semver (`1.0.0`) |
| Scope | Layer + Prim + Application domains | Asset validation (prim-centric) |
| Registration | plugInfo.json + schema.usda | Python Enums (code-first) |
| Query system | Explicit + Introspective, with traversal pruning | Registry lookup |
| Granularity | Capability = functional unit | Requirement = atomic rule, Capability = grouping |

## Source

- Repository: [PixarAnimationStudios/OpenUSD-proposals](https://github.com/PixarAnimationStudios/OpenUSD-proposals/tree/main/proposals/profiles)
- Document: `proposals/profiles/README.md` (Version 1.0)
