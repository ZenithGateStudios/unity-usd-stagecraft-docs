# USD Stagecraft

Load OpenUSD (`.usd` / `.usda` / `.usdc` / `.usdz`) files at runtime in Unity, preview them in the Editor without Play mode, and iterate with file watching and diff reload.

**Version:** 1.0.0 (v1.0)  
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
11. [Roadmap (incl. changelog / v0.3.x baseline)](#roadmap)
12. [macOS security notice](#macos-security-notice)
13. [Known limitations](#known-limitations)
14. [Troubleshooting](#troubleshooting)
15. [Support and documentation](#support-and-documentation)
16. [License](#license)

---

## Supported environments

| Item | Details |
|------|---------|
| Unity | 2022.3 LTS or later |
| macOS | Apple Silicon (arm64) only |
| Windows | x64 |
| Render pipelines | **Built-in RP**, **URP**, and **HDRP** |

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

where `<package-version>` matches this package’s version (for example **1.0.0**).

### From a local package folder (developers)

1. Open **Window → Package Manager**.
2. Click **+** → **Add package from disk…**
3. Select `package.json` from the extracted package folder.

### Repository layout (source development vs DLL distribution)

When working from the **git repository**, C# is split between compiled DLLs and source shipped to end users:

| Path | Source dev | DLL distribution (Asset Store) |
|------|------------|--------------------------------|
| `Runtime/Scripts/Core/` | C# source (compiled to DLL locally) | **Excluded** — logic in `UsdStagecraft.Core.dll` |
| `Runtime/Scripts/UsdStagePreview.cs` | Source | Source (public API) |
| `Runtime/Plugins/Managed/` | Built DLL output from `make` | `UsdStagecraft.Core.dll` |
| `Editor/Core/` | C# source | **Excluded** — `UsdStagecraft.Editor.Core.dll` |
| `Editor/*.cs` (except Core) | C# source | **Excluded** — `UsdStagecraft.Editor.dll` |
| `Editor/Panel/*.uxml` / `*.uss` | Assets | Shipped as assets |
| `ThirdParty/UnityMeshSimplifier/` | Source | **Excluded** — embedded in Editor.Core DLL |

Full build steps: repository `Documents/distribution/dll-distribution.md`.

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
| **GeomSubset (materialBind)** | Child `UsdGeomSubset` prims with material bindings split the mesh into sub-meshes; each subset gets its own material slot on the **MeshRenderer**. |
| **Root subLayers** | `[+]` picks a save path and creates an **in-memory** anonymous edit layer only (no `.usdc` or root `subLayers` on disk yet). **Save** exports the layer to the chosen path and writes root `subLayers` (UE *File > Save* style). In the Editor, paths under `Assets/` are passed to **AssetDatabase.ImportAsset** so `.meta` is created and the file appears in the Project window. **EditTarget** is stored per **UsdStagePreview** instance. Legacy `{name}_unity_session.json` sidecars are migrated once on load. |

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

- **UsdStagePreview** exposes **USD Variants** in its custom Inspector after load. Choosing a variant writes to the **EditTarget** layer and **refreshes** the affected subtree (`UsdLoader.SetVariantAndRefresh`). Persist with **Save Edit Layer**.
- Prims with variant sets also show variant controls on **UsdPrimBinding** objects in the hierarchy.

### Animation

If the stage has time-sampled data, the **UsdStagePreview** Inspector shows **Animation** transport (play/pause, frame slider, wrap mode). Evaluation uses **UsdAnimationPlayer** internally (Xform, PointInstancer, and **`UsdSkel`** skeleton joints).

### Skeletal mesh (`UsdSkel`)

Stages with **`UsdSkel`** prims load as a bone hierarchy with **`SkinnedMeshRenderer`** (linear blend skinning, up to 4 joint influences per vertex). Use the same Animation transport to scrub skeleton animation in the Editor or Play mode.

**Bake to Prefab:** **USD Stage → Bake to Prefab** emits skeleton and Xform **`AnimationClip`** assets, **`AnimatorController`**, **`Skinned_*`** visual prefabs (relinked into the stage hierarchy), and optionally **`StageTimeline.playable`** when Timeline is installed. See [Known limitations](#known-limitations) for skeletal and animation scope limits.

### Edit layers and save

After load, the **Edit Layers** block (custom Inspector) lets you:

- Choose the **EditTarget** from the stage’s layer stack.
- Click **+** to create a new layer file (`.usdc` / path chosen in the save dialog) and add it as a **SubLayer** with that target.
- Click **Save** to flush **Transform** and **Visibility** differences from the Unity hierarchy into the **EditTarget** layer via `SaveEditLayer`.

**Save** writes through the USD stage API; it does **not** overwrite your original root USD file as the primary workflow — session and edit layers are stored as separate files. A typical sidecar edit file name pattern is `{rootName}_unity_edits.usda` next to the root USD (see log output after save).

### Unity ↔ DCC edit cycle (current)

This package supports a **manual** round trip between a DCC tool and **UsdStagePreview**. Live bidirectional sync is planned for **v1.1** (see roadmap).

| Direction | What works today | What does not |
|-----------|------------------|---------------|
| **DCC → Unity** | Save USD on disk → **UsdStagePreview** file watcher runs **DiffReload** (meshes, materials, transforms, visibility, variants from disk). | Prim-level live push without a file save. |
| **Unity → DCC** | Edit **Transform** / **Visibility** in the Hierarchy or **USD Stage** panel → **Save Edit Layer** writes to the **EditTarget** sidecar and updates root `subLayers`. | Materials, lights, mesh geometry. |
| **Variants** | Switch in Inspector / **USD Stage** → stored on **EditTarget** (persist after **Save**). | — |
| **purpose / kind** | Edited in **USD Stage** Properties → EditTarget in memory → **Save** to disk. | — |

**Recommended DCC workflow**

1. Open the same root USD in your DCC and in Unity (**UsdStagePreview** or **USD Stage** window).
2. In Unity, create an edit layer (**+** in **USD Stage** → Layers) if you plan to write edits back.
3. DCC saves → Unity preview updates automatically (hot reload).
4. Unity edits → **File → Save Edit Layer** (or Inspector **Save**).
5. In the DCC, ensure the root USD references the Unity edit layer as a **SubLayer** (Unity **Save** also writes `subLayers` on the root when possible). Reload the stage in the DCC to see overrides.

Unity-managed edit layer files are **ignored** by diff reload so your own saves do not wipe the preview.

### Baked Prefab ↔ USD Stage Preview (current)

**Bake to Prefab** is a **one-way** export from preview to Unity assets, but baked prefabs now keep USD linkage for round trip:

| Step | Behavior |
|------|----------|
| **Bake** | Parent **UsdScene** Prefab + child visual Prefabs; **`UsdStageReference`** on the root stores root USD path, edit layers, and shader settings; **`UsdPrimBinding`** remains on prims. |
| **Rehydrate** | **USD Stage → Open in Preview** (toolbar or **Actions** menu) removes the baked **scene instance**, opens a live **UsdStagePreview**, and applies baked transform overrides. |
| **Write back from Prefab** | **USD Stage → Write to USD** (toolbar or **Actions** menu) opens the stage from **`UsdStageReference`**, writes bound transforms/visibility to the edit layer, and saves. |

Before bake, pending Unity edits can be flushed with **Save Edit Layer** (prompted automatically when needed).

### Preview visual cache

Editor preview stores generated meshes/materials under **`Assets/UsdStagecraftCache/{fileName}__{pathHash8}/`**. The folder name is derived from the USD file path only (not the native stage handle), so reopening the same file reuses the same cache directory. Legacy folders named `__h{handle}` are still picked up when their manifest matches the file path.

### LOD (Bake primary path + thin Importer)

LOD is primarily driven by **Bake** and **`.usd` Reimport**. LOD generation in the preview cache is **OFF** by default (Project Settings → **Generate LOD In Preview Cache**).

| Layer | Where to configure |
|----|----------|
| Project | **Edit → Project Settings → USD Stagecraft → Native Asset** — Profile presets (Default / Hero / Prop / None), defaults for Bake/Import |
| Prim | **USD Stage** properties — `lodProfile` / `generateLod`(written back to USD `customData`) |
| Bake | folder picker — **Override LOD settings** — override Generate LOD / Profile |
| Importer | `.usd` Inspector — same overrides (thin `UsdScriptedImporter`) |

Prim `customData` keys: `unity:lodProfile` (preset name), `unity:generateLod` (`false` explicitly disables). Reduction ratios and thresholds are not stored on Prims. See `Documents/design/lod-design.md` in the repository.

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
- **UsdGeomSubset** — `materialBind` subsets split meshes into sub-meshes with per-subset **UsdPreviewSurface** materials
- **UsdPreviewSurface** — PBR-style materials (see shader table below)
- **Prim hierarchy** — USD scene graph as **GameObject** parent/child chain
- **UsdLux** — **Distant**, **Sphere**, **Disk**, **Cylinder**, **Rect** lights mapped to Unity **Light**; **DomeLight** applied as **environment lighting** (approximate, not full HDR dome IBL); **ShapingAPI** (cone half-angle under 90°) selects **Spot**; **intensity**, **exposure**, and **color** from `UsdLuxLightAPI`
- **HDRP (optional)** — `UsdStagecraft.Hdrp` assembly with `HDRenderPipelineHost` (`HDRP/Lit`, area / spot / disc / tube lights, Dome via GradientSky); **added in v0.4.0 on `main`**. The **v0.3.x** line validated **Built-in** and **URP** first — see `CHANGELOG.md` sections **`[0.3.0]`** and **`[0.4.0]`**.
- **UsdGeomCamera** — **Camera** component where authored
- **USD Physics** — colliders from **PhysicsCollisionAPI** / **PhysicsMeshCollisionAPI** (with guide mesh / approximation behaviors)
- **UsdAnimationPlayer** — time-sampled playback in Editor and Play (Xform, PointInstancer, `UsdSkel` skeleton joints)
- **`UsdSkel` skeletal mesh** — bone hierarchy + `SkinnedMeshRenderer` at runtime preview (LBS, up to 4 influences)
- **Skeleton / Xform `AnimationClip` bake** — joint and transform curves + `AnimatorController` during **Bake to Prefab** and animated **ScriptedImporter** import
- **SkinnedMesh Prefab relink** — `Visuals/Skinned_*/` prefabs wired into the baked stage hierarchy
- **Timeline / Playable** — preview time driver (`UsdStageTimelineBridge`) and baked `StageTimeline.playable` (`UsdTimelineBaker`; requires `com.unity.timeline`)
- **USD Scripted Importer (animated)** — time-sampled USD imports clips, skinned prefabs, and timeline alongside static mesh import
- **Multi-stage preview** — multiple **UsdStagePreview** instances per scene (**Window → USD Stagecraft → USD Stage**)
- **UsdStagePreview** — Editor preview without Play; Play mode async load
- **Variant sets** — Inspector selection on loader and bound prims
- **Hot reload** — file watcher + diff reload for rapid DCC iteration
- **Edit layers** — EditTarget selection, new SubLayer, save Transform / Visibility to EditTarget
- **Root subLayers + Save** — `[+]` registers layers in memory; **Save** writes edit layer + root `subLayers` to disk; EditTarget on `UsdStagePreview`

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
| `SetVariant(primPath, variantSetName, variantName)` | Selects a variant (EditTarget) and refreshes the subtree. Save to persist. |
| `HasPendingUnityEdits()` | True when unsaved Transform/Visibility deltas exist. |

### `UsdStageReference` (baked Prefab root)

Editor-only metadata for Rehydrate and Write-back. Fields (`RootUsdPath`, `EditTargetPath`, etc.) are declared under `#if UNITY_EDITOR` and are **not serialized into Player builds** (Unity 2021.2+). The `UsdStageReference` component itself may remain on baked prefabs at runtime as an empty marker; it has no effect in Player builds.

| Member | Purpose |
|--------|---------|
| `RootUsdPath` | Root USD file used when the prefab was baked (Editor only). |
| `EditTargetPath` / `PersistentSubLayers` | Edit layer stack for rehydrate and write-back (Editor only). |
| Rehydrate | **USD Stage → Open in Preview** (replaces scene instance with live preview) |
| Write-back | **USD Stage → Write to USD** |

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
| `HasAnimation`, time fields, `AnimatedPrims`, `AnimatedSkeletons` | Animation metadata. |
| `PersistentSubLayers`, `PersistentEditTarget` | Used internally across reloads for edit layers. |
| `RefreshAnimatedPrims()` | Rescans time-sampled prims after reload. |

---

## Render pipelines and shaders

| Pipeline | Typical **Shader Name** value |
|----------|-------------------------------|
| Built-in | `Standard` |
| URP | `Universal Render Pipeline/Lit` |
| HDRP | `HDRP/Lit` — **HDRP is an active target** in this branch (see [Known limitations](#known-limitations) for caveats). |

**RectLight note:** on the **Built-in** render pipeline, Unity treats **Rectangle** lights as **baked-only** for real-time views; prefer **URP** (or test **HDRP**) when you need real-time rectangular area lights. **DomeLight** is approximated through the active `IUsdRenderPipelineHost` (for example ambient-style settings on Built-in / URP), not as a full panoramic environment map.

The **Basic Load** sample exposes `shaderName` in the Inspector for quick testing across pipelines.

---

## Roadmap

### Changelog and version baseline

`CHANGELOG.md` lists **`[1.0.0]`** then **`[0.4.0]`**, **`[0.3.0]`**, etc. (newest first).

Authoritative release planning lives in `Documents/planning/ROADMAP.md` at the **Git repository root** (this file ships under `Documentation/` inside the Unity package; the roadmap file is not duplicated inside the package folder).

This **main** line is **package 1.0.0** (**v1.0**) on top of the **v0.4.0** feature set (PointInstancer, HDRP, Bake to Prefab, LOD, ScriptedImporter). **v1.0** adds multi-stage preview, **`UsdSkel`** runtime preview, skeleton / Xform **`AnimationClip`** bake, skinned prefab relink, Timeline integration, animated **ScriptedImporter** import, and DiffReload skinning. **`UsdGeomBasisCurves`** is deferred to **v1.1**.

Release branches may pin earlier versions; this README matches **`package.json`** on `main`.

---

## macOS security notice

If macOS Gatekeeper blocks the native plugin (`NativeUsdBridge.bundle`), follow these steps:

1. Open **System Settings → Privacy & Security**
2. Scroll down to the security section
3. Click **"Open Anyway"** next to the blocked item

Or via Terminal (use the path that exists in your project):

```bash
xattr -dr com.apple.quarantine /path/to/your/UnityProject/Packages/com.zenithgatestudios.usd-stagecraft/Runtime/Plugins/macOS/NativeUsdBridge.bundle
```

If the package was installed via a different route, it may instead live under `Library/PackageCache/com.zenithgatestudios.usd-stagecraft*/Runtime/Plugins/macOS/NativeUsdBridge.bundle`.

---

## Known limitations

- Very large USD files (for example **> 500 MB**) may load slowly or stress memory.
- **USD variants and payloads** are only **partially** supported.
- **HDRP** is integrated but not every DCC / USD combination is validated — report gaps via [Issues](https://github.com/ZenithGateStudios/unity-usd-stagecraft-docs/issues) rather than assuming parity with every native HDRP workflow.
- **DomeLight** is **not** imported as a full panoramic sky texture; expect an **approximate** environment response through the render-pipeline host (GradientSky-style path on HDRP; see [Render pipelines and shaders](#render-pipelines-and-shaders)).
- **Built-in RP** **RectLight** mapping uses Unity’s **baked** rectangle light path for real-time views.
- **`UsdSkel`:** Linear blend skinning only; up to **4** joint influences per vertex. BlendShape and dual-quaternion skinning are not supported (planned for a future release).
- **USD animation editing:** Read, playback, and Unity bake/export only. Writing keyframes back to USD `timeSamples` is planned for **v1.1**.
- **Timeline clip mixing:** Baked `StageTimeline.playable` and preview time driving are supported; blending USD clips with other Unity animation tracks is planned for **v2.0**.
- **Multiple `UsdStagePreview` instances:** Works best with **different** USD files per instance. Opening the **same** file twice logs a warning (OpenUSD shares layer data in memory). **DomeLight** environment is **scene-global** — only one approximate sky is active at a time; unloading one stage restores the remaining stage’s dome when possible.
- **Baked prefab metadata (`UsdStageReference`):** Absolute paths and edit-layer settings are **Editor-only serialized fields** — they do not ship in Player/ROM builds. The component may remain on baked prefabs at runtime as an empty marker with no serialized data.
- **`UsdGeomBasisCurves`** (hair, grass, cables) is **not** supported — planned for **v1.1**.
- Advanced **material writeback** and **layer reorder/mute** are not implemented yet (planned for **v1.2** per roadmap).

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
