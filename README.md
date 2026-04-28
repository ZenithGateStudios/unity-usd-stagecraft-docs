# USD Stagecraft

Load OpenUSD (`.usd` / `.usda` / `.usdc`) files at runtime in Unity.

**Version:** 0.1.0 (Early Access)  
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

1. Open **Window > Package Manager** in Unity.
2. Click **+** → **Add package from disk…**
3. Select `package.json` from the extracted folder.

Or use the Unity Asset Store to install directly.

---

## Setup

1. Add a **UsdSceneLoader** component to any GameObject.
2. Set the **USD File Path** field to a `.usd` / `.usda` / `.usdc` file path.
3. Press **Load** in the Inspector or call `UsdLoader.Load(path)` from a script.

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
using UsdRuntimeLoader;

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

## Known Limitations (Early Access)

- macOS Apple Silicon only (Windows support coming in v1.0)
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
