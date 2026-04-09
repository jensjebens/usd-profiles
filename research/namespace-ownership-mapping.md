# Namespace Ownership: Requirement Code → Owner Mapping

**Date:** 2026-04-09
**Ref:** jensjebens/usd-profiles#23

## Principle

- **OAV** (`com.nvidia.oav.*` → `usd.*` future): USD schema/grammar compliance — non-opinionated
- **SRF** (`com.nvidia.simready.*`): simulation conformance — opinionated conventions

## Mapping

### OAV-only (general USD checks, no SRF equivalent)

These stay in OAV, namespace `com.nvidia.oav.*`:

| Code | Checker | Domain | Future |
|------|---------|--------|--------|
| AA.003 | PortableAssetPathChecker | atomic_asset | usd.utils |
| AA.OV.001 | UsdzUdimLimitationChecker | atomic_asset | usd.utils |
| JT.002 | PhysicsJointChecker | physics | usd.physics |
| JT.003 | PhysicsJointChecker | physics | usd.physics |
| JT.ART.002 | ArticulationChecker | physics | usd.physics |
| JT.ART.004 | ArticulationChecker | physics | usd.physics |
| RB.003 | RigidBodyChecker | physics | usd.physics |
| RB.005 | RigidBodyChecker | physics | usd.physics |
| RB.007 | MassChecker | physics | usd.physics |
| RB.009 | RigidBodyChecker | physics | usd.physics |
| RB.COL.004 | ColliderChecker | physics | usd.physics |
| VG.002 | ExtentsChecker | geometry | usd.geom |
| VG.007 | ManifoldChecker | geometry | usd.geom |
| VG.009 | IndexedPrimvarChecker | geometry | usd.geom |
| VG.010 | SubdivisionSchemeChecker | geometry | usd.geom |
| VG.011 | UnusedPrimvarChecker | geometry | usd.geom |
| VG.014 | ValidateTopologyChecker | geometry | usd.geom |
| VG.016 | WeldChecker | geometry | usd.geom |
| VG.018 | UnusedMeshTopologyChecker | geometry | usd.geom |
| VG.019 | ZeroAreaFaceChecker | geometry | usd.geom |
| VG.020 | PointsPrecisionChecker | geometry | usd.geom |
| VG.025 | AssetOriginPositioningChecker | geometry | usd.geom |
| VG.027 | NormalsExistChecker | geometry | usd.geom |
| VG.028 | NormalsValidChecker | geometry | usd.geom |
| VG.029 | NormalsWindingsChecker | geometry | usd.geom |
| VG.MESH.001 | ContainsMeshChecker | geometry | usd.geom |
| VG.RTX.001 | AlmostExtremeExtentChecker | geometry | usd.geom |
| VM.BIND.001 | MaterialOutOfScopeChecker | materials | usd.shade |
| VM.MDL.001 | MaterialPathChecker | materials | usd.shade |
| VM.MDL.002 | MaterialOldMdlSchemaChecker | materials | usd.shade |
| VM.PS.001 | MaterialUsdPreviewSurfaceChecker | materials | usd.shade |

### SRF-only (SimReady-specific, no OAV equivalent)

These stay in SRF, namespace `com.nvidia.simready.*`:

| Code | Checker | Domain |
|------|---------|--------|
| HI.002 | ExclusiveXFormParentChecker | hierarchy |
| HI.006 | PlaceablePosableXformableChecker | hierarchy |
| HI.008 | LogicalGeometryGroupingChecker | hierarchy |
| HI.009 | KinematicChainHierarchyChecker | hierarchy |
| HI.010 | UndefinedPrimsChecker | hierarchy |
| NP.001-008 | (8 naming/path checkers) | naming_paths |
| SR.001 | SimReadyCapabilityChecker | sim_ready |
| UN.003 | KilogramsPerUnitChecker | units |
| UN.004 | UnitsCorrectiveTransformChecker | units |
| UN.005 | TimeCodesPerSecondChecker | units |

### DUPLICATED (both OAV and SRF have checkers — need resolution)

| Code | OAV Checker | SRF Checker | Owner should be |
|------|------------|-------------|-----------------|
| AA.001 | AnchoredAssetPathsChecker | AnchoredAssetPathsChecker | **OAV** (general USD hygiene) |
| AA.002 | SupportedFileTypesChecker | SupportedFileTypesChecker | **OAV** (general USD hygiene) |
| HI.001 | HierarchyHasRootChecker | HierarchyHasRootChecker | **OAV** (valid USD structure) |
| HI.003 | RootPrimXformableChecker | RootPrimXformableChecker | **OAV** (valid USD structure) |
| HI.004 | DefaultPrimChecker | StageHasDefaultPrimChecker | **OAV** (valid USD structure) |
| UN.001 | StageMetadataChecker | StageMetadataChecker | **OAV** (metadata presence) |
| UN.002 | StageMetadataChecker | StageMetadataChecker | **OAV** (metadata presence) |
| UN.006 | UpAxisZChecker | UpAxisZChecker | **SRF** (opinionated value: Z) |
| UN.007 | UnitsInMetersChecker | MetersPerUnit1Checker | **SRF** (opinionated value: 1.0) |

## Action Items

1. Remove SRF checkers for codes that OAV owns (AA.001, AA.002, HI.001, HI.003, HI.004, UN.001, UN.002)
2. Remove OAV checkers for codes that SRF owns (UN.006, UN.007)
3. Update namespaces in capabilities.json codegen
4. SRF profiles inherit OAV base checks via the DAG (same pattern as usd.* bridge)
