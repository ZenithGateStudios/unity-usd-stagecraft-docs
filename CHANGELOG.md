# Changelog

All notable changes to USD Stagecraft will be documented in this file.

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
- macOS Apple Silicon only (Windows support planned for v1.0)
- Intel Mac (x86_64) not supported
- USD variants and payloads are partially supported
