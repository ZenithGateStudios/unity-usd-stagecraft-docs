# USD Stagecraft

Load OpenUSD (`.usd` / `.usda` / `.usdc`) files at runtime in Unity.

**Version:** 0.1.2 (Early Access)  
**Publisher:** ZenithGateStudios

---

## Supported Environments

| Item | Details |
|------|---------|
| Unity | 2022.3 LTS or later |
| macOS | Apple Silicon (M1 / M2 / M3) — arm64 |
| Windows | Planned for v1.0 |

> **Note:** Intel Mac (x86_64) is not supported in this Early Access release.

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

### From a local package folder (developers)

1. Open **Window → Package Manager**.
2. Click **+** → **Add package from disk…**
3. Select `package.json` from the extracted package folder.

---

## Setup

1. Add a **UsdStagePreview** component to any GameObject.
2. Set **File Path** to a `.usd` / `.usda` / `.usdc` file path.
3. **In the Editor:** the preview appears when the path is set (no Play mode required). Changing the path reloads the preview.
4. **In Play mode:** enable **Auto Load On Start** (default) or call `Load()` on the component, or use `UsdLoader.LoadAsync` from a script.
5. For runtime loading from code, use `UsdLoader.LoadAsync`. For synchronous loading in the Editor, use `UsdLoader.LoadSync`.

See the **Basic Load** sample for runtime loading and the **Stage Preview** sample for editor preview (menu: **USD Stagecraft → Create Stage Preview Sample Scene**).

---

## Features

- Runtime loading of OpenUSD files
- **UsdGeomMesh** — polygon mesh import
- **UsdPreviewSurface** — PBR material support
- **Prim hierarchy** — preserves USD scene graph as GameObject hierarchy
- **UsdAnimationPlayer** — time-sampled animation playback
- **UsdStagePreview** — Editor preview mode

---

## Basic Usage

```csharp
using UsdStagecraft;

public class Example : MonoBehaviour
{
    async void Start()
    {
        var result = await UsdLoader.LoadAsync("/path/to/scene.usda");
        result.Root.transform.SetParent(transform);
    }
}
```

---

## macOS Security Notice

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

> Notarization will be added in a future release.

---

## Known Limitations (Early Access)

- macOS Apple Silicon only (Windows support coming in v1.0)
- Plugin is not yet notarized (see macOS Security Notice above)
- Large USD files (> 500 MB) may cause slow load times
- USD variants and payloads are partially supported

---

## Support & Documentation

- GitHub: https://github.com/ZenithGateStudios/unity-usd-stagecraft
- Issues: https://github.com/ZenithGateStudios/unity-usd-stagecraft/issues

---

## License

This asset is licensed under the [Unity Asset Store EULA](https://unity.com/legal/as-terms).  
Copyright © 2026 ZenithGateStudios. All rights reserved.

### Third-party open-source software

This package redistributes components such as **OpenUSD**, **Intel oneTBB**, and **MaterialX-related resources** (via the USD `usdMtlx` plugin data shipped inside the native plugin).

- **Where to look:** at the package root (same level as this `Documentation/` folder), open the **`ThirdParty/`** directory.
- **Read:** `ThirdParty/README.txt` — attribution, NOTICE-style text, and the **full Apache 2.0 license text** (covers OpenUSD and oneTBB bundled here; MaterialX is under the same license family per ASF).
