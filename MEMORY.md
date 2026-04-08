# MEMORY.md - Profiles Long-Term Memory

## 2026-04-07 — Project Kickoff
- Unification research: Omniverse USD_Profiles, Pixar USD-Profiles proposal, Pixar UsdValidation, Omniverse Asset Validator, NVIDIA SimReady Foundation
- GitHub: https://github.com/jensjebens/usd-profiles
- Goal: understand each project, summarise, identify overlap and unification path

## Five Projects — Research Complete (2026-04-07)
1. **Omniverse USD_Profiles (`omni.capabilities`)** — NOT a standalone PyPI package. **Generated** by `omniverse-usd-profiles` from markdown specs. Bundled inside `omniverse-asset-validator` 1.11.2. Defines Requirement → Capability → Feature → Profile hierarchy with ~90 rules, 10 capabilities, 2 features. Profile layer still empty in the generated package. Semver versioned.
2. **Pixar USD-Profiles proposal** — Formal proposal for declarative capability DAG. Three domains (Layer/Prim/Application). Uses USD-style versioning (`_v2`). ProfileAPI for authoring on prims. Not implemented yet.
3. **Pixar UsdValidation** — C++ framework with Python bindings on `dev` branch. Execution engine (not rules). TBB parallel validation. Plugin + explicit registration. 5 built-in validator plugins (geom, shade, skel, physics, utils).
4. **Omniverse Asset Validator** — Production Python validator (PyPI v1.11.2). 35+ rule classes. Bundles omni.capabilities. Has UsdValidation adapter bridge (labeled "temporary"). CLI: `omni_asset_validate`.
5. **NVIDIA SimReady Foundation** — [github.com/NVIDIA/simready-foundation](https://github.com/NVIDIA/simready-foundation). The **umbrella project** connecting the full pipeline: markdown specs → `omniverse-usd-profiles` (parser+codegen, PyPI v1.11.0) → `omni.capabilities` (generated) → `omniverse-asset-validator` (validation). Defines 4 production profiles (Prop-Robotics-Neutral, Prop-Robotics-Physx, Robot-Body-Neutral, Robot-Body-Runnable). Uses Git LFS, Python 3.12+, Sphinx doc gen with scoring/badges.

### Key Insight — Full Pipeline
```
SimReady markdown specs → omniverse-usd-profiles (codegen) → omni.capabilities (generated) → omniverse-asset-validator (engine)
```
The projects form complementary layers, not competitors:
- Pixar Profiles = "what capabilities exist" (taxonomy)
- Pixar UsdValidation = "how to run validation" (engine)
- SimReady Foundation = "the spec repository" (source of truth)
- omniverse-usd-profiles = "spec to code" (compiler)
- omni.capabilities = "what rules exist" (compiled output)
- OV Asset Validator = "concrete rules + engine" (runtime)

### Deliverables
- `research/00-comparison-matrix.md` — side-by-side comparison (6 projects)
- `research/01-omniverse-usd-profiles.md` through `05-nvidia-simready-foundation.md` — individual summaries
- `research/phase1-capability-mapping.csv` — 90 requirements mapped to `com.nvidia.simready.*` namespace
- `research/phase1-pixar-overlap-analysis.md` — cross-reference with 24 Pixar UsdValidation validators
- All committed and pushed to jensjebens/usd-profiles

## Phase 2a — ProfileAPI Implementation (2026-04-07)
- Branch: `feature/usd-profiles-impl` on `jensjebens/OpenUSD` (6 commits)
- **PR #26** on jensjebens/OpenUSD — ready for internal review
- Full independent build working via `build_usd.py` at `/home/horde/usd_profiles_build/`
- Build trick: Newton's build at `/home/horde/.openclaw/workspace-newton-usd/usd_build/` has compatible TBB 2020.3 — can use for quick targeted builds with `-DCMAKE_CXX_FLAGS="-isystem $NEWTON/include"` to override system TBB 2021.5
- CapabilityRegistry: loads 10 capabilities, 3 profiles from plugInfo.json, full DAG traversal working
- Doxygen HTML docs: 66 pages on `gh-pages` branch → https://jensjebens.github.io/usd-profiles/
- GitHub issues: #1 closed, #2-#5 open (Phase 2a-d)

### Build Fixes Learned
1. `generatedSchema.classes.txt` must have `# Resource Files` section listing plugInfo.json, generatedSchema.usda, schema.usda
2. All schema classes must be in `PUBLIC_CLASSES` in CMakeLists.txt (not just custom C++ classes)
3. CapabilityRegistry: `plugin->GetMetadata()` returns the `Info` dict directly (Plug framework strips outer wrapper) — don't double-nest with `metadata["Info"]["Capabilities"]`

## Phase 2c — Next: Port SimReady Validators (starting)
- Goal: port ~10 SimReady rules as UsdValidation plugins (Python first)
- Focus: geometry + physics rules for a mini Prop-Robotics-Neutral profile
- End-to-end story: ProfileAPI → profile → capabilities → validators → UsdValidationContext

## Phase 2.5 — usd.* Capability Bridge (2026-04-08)
- Defined 6 `usd.*` capabilities mapping all 27 Pixar validators into the DAG
- Wired SimReady capabilities to inherit: simready.geom → usd.geom, etc.
- SimReady prop_robotics_neutral: 21 validators (10 SimReady + 11 Pixar)
- PR: jensjebens/OpenUSD#33, Issue: usd-profiles#11 (closed)
- **Found real bug in SimReady Foundation:** UR10 PhysX variant has dangling `/colliders/base_link` reference — caught by CompositionErrorTest

## CLI Features (2026-04-08)
- `usdprofilecheck graph` — DOT output, namespace filtering, color-coded
- `usdprofilecheck detect` — profile detection, per-feature pass/partial/fail
- `validate --stamp` — write ProfileAPI to assets after validation
- `validate --metadata` — JSON output for CMS systems
- PRs: #36 (graph), #37 (detect+stamp)

## Kit 109+ Compatibility Roadmap (2026-04-08)
- `omniverse-usd-profiles` becomes codegen tool + runtime DAG library (dual mode)
- Canonical format: `capabilities.json` with semantic `kind` tags (namespace/capability/feature/profile)
- Entrypoint packages publish DAG definitions + checker implementations via `importlib.metadata`
- OAV adapter flattens DAG → 4-level hierarchy (Requirement/Capability/Feature/Profile)
- Phase D exit: when USD ships native profiles, runtime dissolves, codegen survives
- Plan: `plans/roadmap-kit109-compatibility.md`
- Reference: Matt Kuruc's SDD at `research/sdd-asset-validator-entrypoint-plugins.md`
- Forked: jensjebens/simready-foundation

## GitHub Project Board
- "SimReady, USDValidation and USDProfiles" on jensjebens
- All issues on jensjebens/usd-profiles (not OpenUSD)
- PRs on jensjebens/OpenUSD

## Shared Skills
- `usd-asset-validator` skill already exists with omniverse-asset-validator docs
- `dev-methodology` — plan→review→test→implement→demonstrate
