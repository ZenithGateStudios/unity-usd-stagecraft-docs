# USD Stagecraft

Load OpenUSD (`.usd` / `.usda` / `.usdc` / `.usdz`) files at runtime in Unity, preview them in the Editor without Play mode, and iterate with file watching and diff reload.

**Version:** 0.1.5 (Early Access)<br>
**Publisher:** ZenithGateStudios

---

## Table of contents

1. [Supported environments](#supported-environments)
2. [File formats and paths](#file-formats-and-paths)
3. [Installation](#installation)
4. [Quick start — Editor preview](#quick-start--editor-preview)
5. [Quick start — Runtime (samples)](#quick-start--runtime-samples)
6. [Core concepts](#core-concepts)
7. [Workflows](#workflows)
8. [Features (summary)](#features-summary)
9. [Component and API reference](#component-and-api-reference)
10. [Render pipelines and shaders](#render-pipelines-and-shaders)
11. [Roadmap](#roadmap)
12. [macOS security notice](#macos-security-notice)
13. [Known limitations (Early Access)](#known-limitations-early-access)
14. [Troubleshooting](#troubleshooting)
15. [Support and documentation](#support-and-documentation)
16. [License](#license)

---

## Supported environments

| Item | Details |
|------|---------|
| Unity | 2022.3 LTS or later |
| macOS | Apple Silicon (M1 / M2 / M3) — **arm64 only** |
| Windows | Planned for v1.0 (see [Roadmap](#roadmap)) |
| Render pipelines | **Built-in RP** and **URP** (primary targets; see [Render pipelines and shaders](#render-pipelines-and-shaders)) |

> **Note:** Intel Mac (x86_64) is not supported in this Early Access release. Broader macOS support is planned for v0.2.0.

---

## File formats and paths

- **Supported extensions:** `.usd`, `.usda`, `.usdc`, `.usdz` (local file paths).
- **Absolute paths:** Any readable path on disk (typical for DCC exports next to your project).
- **StreamingAssets:** Recommended for **runtime** builds. Place USD files under `Assets/StreamingAssets/` and resolve with `Path.Combine(Application.streamingAssetsPath, "file.usda")` (see the **Basic Load** sample).
- **Textures** referenced from USD are resolved relative to the USD file; changing a texture on disk triggers a material refresh when using **UsdStagePreview** file watching.

---

## Installation

### From the Unity Asset Store (recommended)

1. Open **Window → Package Manager** in Unity.
2. Set the package source to **My Assets**.
3. Select **USD Stagecraft**, then **Download** and **Import**.
4. Confirm the package appears under `Packages/com.zenithgatestudios.usd-stagecraft/` in your project.

### Import samples (optional)

1. In **Package Manager**, select **USD Stagecraft**.
2. Expand **Samples** and import **Basic Load** and/or **Stage Preview**.

After import, sample assets live under:

`Assets/Samples/USD Stagecraft/<package-version>/<SampleName>/`

where `<package-version>` matches this package’s version (for example **0.1.5**).

### From a local package folder (developers)

1. Open **Window → Package Manager**.
2. Click **+** → **Add package from disk…**
3. Select `package.json` from the extracted package folder.

---

## Quick start — Editor preview

Goal: see a USD stage in the **Scene** view **without** pressing Play.

1. Add an empty GameObject (or use the Stage Preview sample scene — menu **USD Stagecraft → Create Stage Preview Sample Scene** after importing that sample).
2. Add the **UsdStagePreview** component.
3. Set **File Path** to a valid USD file (absolute path, or use **StagePreviewSample** with a file name under `StreamingAssets/`).
4. The hierarchy appears under the GameObject; in Edit mode, generated objects use **HideFlags** so they are **not saved into the scene file** (preview only).

Changing **File Path**, **Shader Name**, or **Scale** in the Inspector reloads the preview in the Editor.

---

## Quick start — Runtime samples

### Basic Load

1. Import the **Basic Load** sample.
2. Copy `sample.usda` (and optionally `stagecraft_demo.usda`) into `Assets/StreamingAssets/`.
3. Create a scene with a GameObject that has **BasicLoadSample** (see sample README).
4. Enter **Play**, then use the on-screen **Load** / **Unload** buttons.

### Stage Preview sample

1. Import **Stage Preview**.
2. Use **USD Stagecraft → Create Stage Preview Sample Scene**, then assign **File Path** to the included `sample.usda` (see sample README for the exact path under `Assets/Samples/...`).

---

## Core concepts

| Concept | Description |
|---------|-------------|
| **Stage** | One OpenUSD stage opened from a root file path (`UsdLoader` / **UsdStagePreview**). |
| **Prim** | A node in the USD scene graph; imported as a **GameObject** with optional mesh, light, camera, or collider. |
| **LoadResult** | Result of `UsdLoader.LoadAsync` / `LoadSync`: success flag, **Root** object, **NodeMap** (prim path → GameObject), stage handle, animation metadata. |
| **UsdPrimBinding** | Component on imported prims carrying the USD **prim path**; used for per-prim variant UI in the Inspector. |
| **Session sidecar** | JSON file `{rootUsdBaseName}_unity_session.json` next to the root USD file: remembers **SubLayers** and **EditTarget** across Editor sessions (non-destructive to the original USD). |

---

## Workflows

### Editor vs Play mode (`UsdStagePreview`)

| Mode | Loading | Notes |
|------|---------|------|
| **Edit mode** | Synchronous `UsdLoader.LoadSync` | Preview children are **not saved** in the scene (`DontSaveInEditor` / `DontSaveInBuild`). |
| **Play mode** | Asynchronous `UsdLoader.LoadAsync` (via `Load()` or **Auto Load On Start**) | Standard Unity lifecycle; unload when done. |

- **File Path:** Root USD file to open.
- **Shader Name:** Default `Standard`; if left at default on Awake, the component tries to pick **Built-in** vs **URP** vs **HDRP/Lit** from the active **GraphicsSettings** render pipeline asset.
- **Scale:** `0` means **auto** from USD `metersPerUnit`; set explicitly to override world scale.
- **Auto Load On Start:** When enabled (default), Play mode loads **File Path** on `Start`.

### Hot reload and diff reload

When **UsdStagePreview** has an active load, it watches the root USD path and **layer dependencies**. On change:

- USD layer changes trigger **stage reload** and **diff** updates to the hierarchy where applicable.
- Texture changes invalidate the texture cache and **refresh materials**.
- Changes to Unity-managed **persistent SubLayer** files (created via the **+** edit-layer flow) are **filtered out** from diff reload so your in-editor edits are not wiped by accidental file events.

Advanced: `UsdLoader.DiffReload(LoadResult, changedPaths)` exists for custom tooling; **call only from the main thread**.

### Variants

- **UsdStagePreview** exposes **USD Variants** in its custom Inspector after load. Choosing a variant calls `SetVariant`, which updates the session layer and **refreshes** the affected subtree (`UsdLoader.SetVariantAndRefresh`). The **source USD file on disk is not modified** by variant selection.
- Prims with variant sets also show variant controls on **UsdPrimBinding** objects in the hierarchy.

### Animation

If the stage has time-sampled data, the **UsdStagePreview** Inspector shows **Animation** transport (play/pause, frame slider, wrap mode). Evaluation uses **UsdAnimationPlayer** internally.

### Edit layers and save

After load, the **Edit Layers** block (custom Inspector) lets you:

- Choose the **EditTarget** from the stage’s layer stack.
- Click **+** to create a new layer file (`.usdc` / path chosen in the save dialog) and add it as a **SubLayer** with that target.
- Click **Save** to flush **Transform** and **Visibility** differences from the Unity hierarchy into the **EditTarget** layer via `SaveEditLayer`.

**Save** writes through the USD stage API; it does **not** overwrite your original root USD file as the primary workflow — session and edit layers are stored as separate files. A typical sidecar edit file name pattern is `{rootName}_unity_edits.usda` next to the root USD (see log output after save).

### Scripting (`UsdLoader`)

Minimal async load:

```csharp
using UsdStagecraft;
using UnityEngine;

public class LoadUsdOnce : MonoBehaviour
{
    private LoadResult _result;

    private async void Start()
    {
        _result = await UsdLoader.LoadAsync("/absolute/path/to/scene.usda", shaderName: "Standard", scale: 0f);
        if (!_result.IsSuccess)
        {
            Debug.LogError(_result.ErrorMessage);
            return;
        }
        _result.Root.transform.SetParent(transform, worldPositionStays: false);
    }

    private void OnDestroy()
    {
        if (_result != null && _result.IsSuccess)
            UsdLoader.Unload(_result);
    }
}
```

- **`LoadSync`** is **Editor-only** and blocks the main thread — suitable for Editor tooling, not for Play mode.
- Always **`Unload`** a successful `LoadResult` when discarding the hierarchy to close the native stage and destroy spawned objects.

---

## Features (summary)

- **Runtime loading** — `UsdLoader.LoadAsync` / `LoadSync` (Editor), `Unload`
- **UsdGeomMesh** — mesh import with normals and UVs where authored
- **UsdPreviewSurface** — PBR-style materials (see shader table below)
- **Prim hierarchy** — USD scene graph as **GameObject** parent/child chain
- **UsdLux** — basic light types mapped to Unity **Light** (e.g. distant, sphere, rect, disk, cylinder — see code for exact mapping)
- **UsdGeomCamera** — **Camera** component where authored
- **USD Physics** — colliders from **PhysicsCollisionAPI** / **PhysicsMeshCollisionAPI** (with guide mesh / approximation behaviors)
- **UsdAnimationPlayer** — time-sampled playback in Editor and Play (via **UsdStagePreview**)
- **UsdStagePreview** — Editor preview without Play; Play mode async load
- **Variant sets** — Inspector selection on loader and bound prims
- **Hot reload** — file watcher + diff reload for rapid DCC iteration
- **Edit layers** — EditTarget selection, new SubLayer, save Transform / Visibility to EditTarget
- **Session sidecar** — `{name}_unity_session.json` for SubLayer / EditTarget persistence

---

## Component and API reference

### `UsdStagePreview` (MonoBehaviour)

| Member | Purpose |
|--------|---------|
| `FilePath` | Root USD path to open. |
| `ShaderName` | Unity shader for `UsdPreviewSurface` materials. |
| `Scale` | World scale; `0` = auto from `metersPerUnit`. |
| `AutoLoadOnStart` | If true, loads in Play mode on `Start`. |
| `Load()` | Starts async load in Play mode. |
| `Unload()` | Destroys loaded hierarchy and closes the stage. |
| `SaveEditLayer()` | Writes Transform/Visibility deltas to the current EditTarget layer file. |
| `SetVariant(primPath, variantSetName, variantName)` | Selects a variant and refreshes the subtree. |

### `UsdLoader` (static)

| API | Purpose |
|-----|---------|
| `LoadAsync(path, shaderName = "Standard", scale = 0f)` | Async load for Play mode / runtime. |
| `LoadSync(path, shaderName = "Standard", scale = 0f)` | Sync load; **Editor only**. |
| `Unload(LoadResult)` | Destroys **Root** and closes the stage. |
| `SetVariantAndRefresh(result, primPath, setName, variantName)` | Low-level variant refresh (used by **UsdStagePreview**). |
| `DiffReload(result, changedFilePaths)` | Apply filesystem-driven updates (**main thread**). |

### `LoadResult`

| Property / method | Purpose |
|-------------------|---------|
| `IsSuccess` | True if load completed. |
| `ErrorMessage` | Populated when `IsSuccess` is false. |
| `Root` | Root **GameObject** for the imported stage. |
| `NodeMap` | `primPath` → **GameObject** for advanced scripts. |
| `StageHandle` | Native stage handle (do not close manually if using `Unload`). |
| `ShaderName` | Shader used to build materials. |
| `HasAnimation`, time fields, `AnimatedPrims` | Animation metadata. |
| `PersistentSubLayers`, `PersistentEditTarget` | Used internally across reloads for edit layers. |
| `RefreshAnimatedPrims()` | Rescans time-sampled prims after reload. |

---

## Render pipelines and shaders

| Pipeline | Typical **Shader Name** value |
|----------|-------------------------------|
| Built-in | `Standard` |
| URP | `Universal Render Pipeline/Lit` |
| HDRP | `HDRP/Lit` (may be auto-selected when an HDRP pipeline asset is active; **full HDRP parity is not guaranteed** in Early Access — see [Roadmap](#roadmap)) |

The **Basic Load** sample exposes `shaderName` in the Inspector for quick testing across pipelines.

---

## Roadmap

High-level planning lives in the product repository: `Documents/ROADMAP.md` (same monorepo as this package).

Current highlights from that file for context:

- **v0.2.0** — Broader macOS native binary coverage (Intel / universal2).
- **v1.0** — Windows support, stronger HDRP story, notarization, and additional pipeline features as listed there.

This README describes **v0.1.5** behavior; newer branches may ship different platform matrices.

---

## macOS security notice

The native plugin (`NativeUsdBridge.bundle`) is not notarized with Apple Developer ID in this Early Access release.

If macOS Gatekeeper blocks the plugin, follow these steps:

1. Open **System Settings → Privacy & Security**
2. Scroll down to the security section
3. Click **"Open Anyway"** next to the blocked item

Or via Terminal (use the path that exists in your project):

```bash
xattr -dr com.apple.quarantine /path/to/your/UnityProject/Packages/com.zenithgatestudios.usd-stagecraft/Runtime/Plugins/macOS/NativeUsdBridge.bundle
```

If the package was installed via a different route, it may instead live under `Library/PackageCache/com.zenithgatestudios.usd-stagecraft*/Runtime/Plugins/macOS/NativeUsdBridge.bundle`.

> Notarization is planned for a future release (see roadmap).

---

## Known limitations (Early Access)

- **macOS Apple Silicon only** (Windows support planned for v1.0; Intel Mac planned for v0.2.0 — see [Supported environments](#supported-environments)).
- Plugin is **not yet notarized** (see [macOS security notice](#macos-security-notice)).
- Very large USD files (for example **> 500 MB**) may load slowly or stress memory.
- **USD variants and payloads** are only **partially** supported; complex compositions may not match a full USD viewer.
- **HDRP** is not a fully validated target in this release; prefer **Built-in** or **URP** for predictable results.

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| **Nothing appears in Edit mode** | **File Path** empty or wrong; macOS quarantine on the bundle (see security notice); Console for `[UsdStagePreview]` errors. |
| **Pink materials** | **Shader Name** does not match the active render pipeline (see [Render pipelines and shaders](#render-pipelines-and-shaders)). |
| **Load fails at runtime** | Path must be readable on device; use **StreamingAssets** or a known absolute path. Inspect `LoadResult.ErrorMessage` after `LoadAsync`. |
| **Hot reload does nothing** | File watcher listens to the root path and **layer** paths; ensure saves flush to disk. Some Unity-managed edit-layer files are intentionally ignored for reload. |
| **Save disabled or warns** | Create an EditTarget with **+** first; **Save** requires a valid EditTarget layer path. |

---

## Support and documentation

- **Package documentation (this file):** shipped under `Documentation/README.md` and mirrored to the web.
- **Online manual / changelog / license:** see `package.json` in this package for URLs:
  - Documentation: https://zenithgatestudios.github.io/unity-usd-stagecraft-docs/
  - Changelog: https://github.com/ZenithGateStudios/unity-usd-stagecraft-docs/blob/main/CHANGELOG.md
  - Licenses: https://github.com/ZenithGateStudios/unity-usd-stagecraft-docs/blob/main/LICENSE.txt
- **Repository:** https://github.com/ZenithGateStudios/unity-usd-stagecraft
- **Issues:** https://github.com/ZenithGateStudios/unity-usd-stagecraft-docs/issues

---

## License

This asset is licensed under the [Unity Asset Store EULA](https://unity.com/legal/as-terms).  
Copyright © 2026 ZenithGateStudios. All rights reserved.

### Third-party open-source software

This package redistributes components such as **OpenUSD**, **Intel oneTBB**, and **MaterialX-related resources** (via the USD `usdMtlx` plugin data shipped inside the native plugin).

- **Where to look:** at the package root (same level as this `Documentation/` folder), open the **`ThirdParty/`** directory.
- **Read:** `ThirdParty/README.txt` — attribution, NOTICE-style text, and the **full Apache 2.0 license text** (covers OpenUSD and oneTBB bundled here; MaterialX is under the same license family per ASF).
