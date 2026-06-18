# Changelog

All notable changes to USD Stagecraft will be documented in this file.

## [0.1.5] - 2026-06-18

### Changed
- English package metadata (`package.json` descriptions) for Asset Store compliance (Section 2.4.1).
- English XML documentation, comments, tooltips, and user-facing log messages across public C# APIs.

### Notes
- Publisher documentation: https://zenithgatestudios.github.io/unity-usd-stagecraft-docs/

## [0.1.4] - 2026-06-13
 - Minor bug fixes.

## [0.1.2] - 2026-05-08

### Added
- `ThirdParty/README.txt` — attribution, NOTICE, and full Apache 2.0 text for redistributed OpenUSD 0.26.5, Intel oneTBB, plus MaterialX / usdMtlx notice. See `Documentation/README.md` for a short guide.

### Notes
- When upgrading the OpenUSD SDK under `Core/external/USD`, refresh `ThirdParty/README.txt` (version strings, NOTICE alignment, and the Apache 2.0 section if upstream license terms change), and the version in `include/pxr/pxr.h` as applicable.

---

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
- macOS Apple Silicon (arm64) native plugin (`NativeUsdBridge.bundle`)

### Known Limitations
- macOS Apple Silicon only (Intel Mac support planned for v0.2.0, Windows for v1.0)
- Intel Mac (x86_64) not supported
- USD variants and payloads are partially supported
