# Changelog

All notable changes to USD Stagecraft will be documented in this file.

## [1.0.0] - 2026-07-09

### Added
- **`UsdSkel` runtime preview** — Bone hierarchy + `SkinnedMeshRenderer`, time-sampled joint evaluation via `UsdAnimationPlayer` (LBS, up to 4 influences; native: `UsdBridgeSkel.cpp`; Unity: `UsdSkelHandler`, `UsdSkelRig`)
- **Skeleton + Xform `AnimationClip` bake** — `UsdAnimationClipBaker` samples joint and transform curves into Generic rig clips + `AnimatorController` during Bake to Prefab / Import
- **SkinnedMesh Prefab promotion + relink** — `Visuals/Skinned_*/` prefab output and stage hierarchy relink (`UsdBakeToPrefab.RelinkSkinnedVisualInstances`)
- **Timeline / Playable integration** — Layer A: `UsdStageTimelineBridge` + preview `PlayableDirector`; Layer B: `UsdTimelineBaker` generates `StageTimeline.playable` on bake
- **`UsdScriptedImporter` skeletal mesh + clip support** — Animated USD files import via `LoadSync` with clips, skinned prefabs, and timeline
- **DiffReload skinning** — Deferred `AttachSkinnableMeshesInSubtree` pass after hot reload
- **Read-only animation track list** — `anim-tracks` ListView in USD Stage panel
- **Multi-stage preview** — Multiple `UsdStagePreview` instances per scene (different USD files recommended; same-file duplicate open warns; DomeLight environment is scene-global with reference counting)
- **`com.unity.timeline` dependency** — Optional assembly `UsdStagecraft.Timeline` with `USD_STAGECRAFT_TIMELINE` define

### Deferred to v1.1
- **`UsdGeomBasisCurves`** — hair, grass, cables, etc.

### Notes
- **v1.0** formal release on `main` / package **1.0.0**. See `Documents/planning/ROADMAP.md` for the authoritative milestone list.
- If you publish a separate changelog (for example `unity-usd-stagecraft-docs` on GitHub), keep it in sync with this file.

---

## [0.4.0] - TBD

### Added
- **instanceable / PointInstancer (provisional Mesh expansion)** — Lists `instanceable` reference instance proxies as child Prims and displays referenced Meshes. `PointInstancer` reflects positions / orientations and duplicates each instance as separate Mesh groups (GPU instancing not yet supported; planned follow-up)（native: `UsdBridgeStage.cpp`, `UsdBridgeGeom.cpp`, `UsdBridgePointInstancer.cpp`; Unity: `UsdLoader`, `UsdPrimAdapter`）
- **HDRP** (optional `UsdStagecraft.Hdrp` assembly) — `HDRenderPipelineHost` using `HDRP/Lit` for `UsdPreviewSurface`, `HDAdditionalLightData` for rectangular / disc / tube / spot-shaped lights, and DomeLight approximated via a GradientSky volume (`Runtime/Scripts/Host/Hdrp/`)
- **USD Stage panel — property editing** — Inline editing of `xformOp:translate` / `xformOp:rotateXYZ` / `xformOp:scale` (writes back via `Transform.hasChanged`), `purpose` (default/render/proxy/guide) and `kind` (assembly/group/component/subcomponent) dropdowns, and a read-only `references` viewer (native: `USD_SetPrimPurpose`, `USD_GetPrimKind`, `USD_SetPrimKind`, `USD_GetReferenceCount`, `USD_GetReferencePath`)

### Changed
- **Editor assembly precompiled** — `UnityPackage/Editor/**/*.cs` is now built ahead of time as `UsdStagecraft.Editor.dll` (Unity 2022.3 LTS managed) and shipped from `Editor/UsdStagecraft.Editor.dll` in distribution packages. UXML / USS panel assets are still shipped as source under `Editor/Panel/` and resolved through `AssetDatabase` so the panel works for both UPM (`Packages/...`) and `.unitypackage` installs. Source-mode development continues to use `UsdStagecraft.Editor.asmdef` unchanged.
- **PointInstancer prototype child enumeration** — Relaxed `USD_GetPrimChildCount` / `USD_GetPrimChildPath` predicates like Mesh traversal so C# expansion can walk prototype subtrees under `over`（`UsdBridgeStage.cpp`）

### Removed
- **macOS Intel (x86_64) support** — macOS native plugins and OpenUSD SDK are **Apple Silicon (arm64) only**. Windows x64 is unchanged.

### Notes
- **v0.3.x baseline:** `UsdGeomSubset`, extended **UsdLux**, and `.usdz` support are recorded under **`[0.3.0]`** below (they first shipped on the `release-v0.3.x` line and remain part of this package).
- Updated `Documentation/README.md` and `Documents/planning/ROADMAP.md` for **v0.4.0** / `main` (PointInstancer, instanceable, and HDRP on top of the v0.3.0 feature set).
- If you publish a separate changelog (for example `unity-usd-stagecraft-docs` on GitHub), keep it in sync with this file.

---

## [0.3.0] - TBD

### Added
- **`UsdGeomSubset` (materialBind)** — Sub-mesh splitting from face subsets and multiple `UsdPreviewSurface` assignments via `MeshRenderer.sharedMaterials`
- **UsdLux extensions** — Approximate environment lighting for `DomeLight` through the render-pipeline host; `RectLight` width and height; spot shaping from `UsdLuxShapingAPI` cone angle and softness; intensity and color including `inputs:exposure` (native: `Core/src/UsdBridgeLight.cpp`; Unity: `UsdPrimAdapter` / `DefaultUnityRenderPipelineHost`)
- **`.usdz`** — Local file paths accepted by `UsdLoader` and `UsdFileWatcher`

### Notes
- Primary **release line** for these items: branch **`release-v0.3.x`** / package **0.3.0**. The **`main` / 0.4.0** tree includes the same capabilities and adds **[0.4.0]** HDRP integration.

---

## [0.2.0] - TBD

### Added
- Windows (x86_64) native plugin support
- macOS Intel (x86_64) native plugin support
- Apple Silicon / Intel universal2 binary (`NativeUsdBridge.bundle`)
- Third-party OSS attribution and full Apache 2.0 license text in `ThirdParty/README.txt` (OpenUSD, oneTBB, MaterialX / usdMtlx notice); see `Documentation/README.md`.

### Changed
- CMake build configuration updated for universal2
- OpenUSD SDK layout unified under `Core/external/USD/{arm64,x86_64,win-x86_64}/lib` with a shared `include/`; link library lists are shared with NativeUsdBridge (CMake / Makefile) via `Core/external/USD/LinkedLibraries.txt`

### Notes
- When upgrading the OpenUSD SDK under `Core/external/USD`, refresh `ThirdParty/README.txt` (version strings, NOTICE alignment, and the Apache 2.0 section if upstream license terms change), and the version in `include/pxr/pxr.h` as applicable.

---

## [0.1.2] - 2026-05-08

### Added
- `ThirdParty/README.txt` — attribution, NOTICE, and full Apache 2.0 text for redistributed OpenUSD 0.26.5, Intel oneTBB, plus MaterialX / usdMtlx notice. See `Documentation/README.md` for a short guide.

### Notes
- When upgrading the OpenUSD SDK under `Core/external/USD`, refresh `ThirdParty/README.txt` (version strings, NOTICE alignment, and the Apache 2.0 section if upstream license terms change), and the version in `include/pxr/pxr.h` as applicable.

## [0.1.1] - 2026-04-29

- Minor bug fixes.
- Minor update.

## [0.1.0] - 2026-04-29 (Early Access)

### Added
- Runtime loading of OpenUSD (`.usd` / `.usda` / `.usdc`) files
- **UsdLoader** — async/sync USD file loading API
- **UsdMeshBuilder** — UsdGeomMesh to Unity Mesh conversion
- **UsdMaterialLoader** — UsdPreviewSurface PBR material support
- **UsdTextureCache** — texture caching to reduce redundant loads
- **UsdPrimAdapter** — USD Prim to GameObject hierarchy mapping
- **UsdPrimBinding** — runtime Prim-to-component binding
- **UsdAnimationPlayer** — time-sampled animation playback
- **UsdStagePreview** — Editor preview mode for USD stages
- **UsdFileWatcher** — hot-reload on USD file change (Editor only)
- **UsdSessionData** — per-session stage state management
- **VariantSetInfo** — USD variant set query support
- Inspector edit-layer UI (EditTarget switch, create, save)
- USD Physics collision import (`PhysicsCollisionAPI` / `PhysicsMeshCollisionAPI`)
- Unity → USD writeback (Transform / Visibility on save)
- macOS Apple Silicon (arm64) native plugin (`NativeUsdBridge.bundle`)

### Known Limitations
- At release time, macOS Apple Silicon only; Intel Mac and Windows were planned for v0.2.0
- USD variants and payloads are partially supported
