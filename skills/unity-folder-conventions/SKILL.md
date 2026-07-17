---
name: unity-folder-conventions
description: Folder structure conventions for Unity projects. ALWAYS Auto-loaded when creating files or folders in Unity projects.
---

# Unity Folder Conventions

Follow these conventions when creating, moving, or placing files and folders in the Unity project.

---

## Quick Reference: Where Does This File Go?

| Asset Type | Path Pattern | Example |
|---|---|---|
| Feature script | `_Module/{Feature}/Scripts/{Role}/` | `_App/Scoring/Scripts/Services/ScoreManager.cs` |
| Feature prefab | `_Module/{Feature}/Prefabs/` | `_App/Gameplay/Prefabs/LineSpawner.prefab` |
| Feature UI prefab | `_Module/{Feature}/Prefabs/UI/` | `_App/Scoring/Prefabs/UI/ScoreUIPanel.prefab` |
| Feature data (SO) | `_Module/{Feature}/Data/` | `_App/Gameplay/Data/DifficultyConfig.asset` |
| Feature scene | `_Module/{Feature}/Scenes/` | `_App/Gameplay/Scenes/GameplayScene.unity` |
| Feature art | `_Module/{Feature}/Art/` | `_App/Gameplay/Art/Models/` |
| Feature audio | `_Module/{Feature}/AudioClips/` | `_App/Audio/AudioClips/SFX/` |
| Feature installer | `_Module/{Feature}/Scripts/Installers/` | `_App/Scoring/Scripts/Installers/ScoringInstaller.cs` |
| Shared script | `_Module/Shared/Scripts/` | `_App/Shared/Scripts/Services/` |
| Shared data model | `_Module/Shared/Scripts/DataModel/{Domain}/` | `_App/Shared/Scripts/DataModel/` |
| Shared prefab | `_Module/Shared/Prefabs/` | `_App/Shared/Prefabs/UI/Button.prefab` |
| Shared scene | `_Module/Shared/Scenes/` | `_App/Shared/Scenes/BaseScene.unity` |
| Root Zenject installer | `_Module/Shared/Scripts/Installers/` | `_App/Shared/Scripts/Installers/BaseSceneInstaller.cs` |
| Zenject config asset | `_Module/Shared/Zenject/Resources/` | `_App/Shared/Zenject/Resources/` |
| Config SO | `_Module/Shared/Data/` | `_App/Shared/Data/AppConfiguration.asset` |
| Core runtime script | `_Core/{Category}/Scripts/` | `_Core/Data/Scripts/IIdentifiable.cs` |
| Core editor script | `_Core/Editor/Scripts/{Category}/` | `_Core/Editor/Scripts/AutomationHelpers/ScriptExecutor.cs` |
| Editor script | `_Module/Editor/{Feature}/Scripts/` | `_App/Editor/Scoring/Scripts/ScoreInspector.cs` |
| Edit mode test | `_Module/Tests/Editor/{Feature}/{Role}/` | `_App/Tests/Editor/Scoring/Services/ScoreManagerTests.cs` |
| Play mode test | `_Module/Tests/Runtime/{Feature}/{Role}/` | `_App/Tests/Runtime/Scoring/Services/ScoreManagerIntegrationTests.cs` |

---

## Top-Level Structure

```
Assets/
├── _Core/                  # Shared/foundational code used by all modules
├── _AppName/               # Main application module
├── Plugins/                # All third-party libraries and assets
├── Settings/               # Render pipeline and project settings
└── ...                     # Root-level folders only when Unity/plugins require it
```

- Prefix first-party modules with underscore (`_`).
- `_Core` = shared code consumed by other modules. App-specific code goes in `_AppName`.
- All third-party code in `Plugins/`.
- Never place first-party assets loose at `Assets/` root.

---

## Module Layout (Feature-First)

Each feature gets its own top-level folder containing all asset types. Cross-feature assets go in `Shared/`. Tests are centralized but mirror the source structure. The `Tests/` folder and test asmdefs only exist if the project uses tests.

```
_Module/
├── Module.asmdef               # Single production assembly (no underscore in asmdef name)
├── {Feature}/
│   ├── Scripts/
│   ├── Prefabs/
│   ├── Scenes/
│   ├── Data/
│   ├── Art/
│   ├── AudioClips/
│   ├── Videos/
│   └── Fonts/
├── {Feature}/
│   └── ...
├── Shared/
│   ├── Scripts/
│   ├── Prefabs/
│   ├── Scenes/
│   ├── Data/
│   ├── Art/
│   ├── Fonts/
│   ├── Zenject/
│   │   └── Resources/
│   └── Misc/
├── Editor/                         # Editor-only code (created on demand)
│   ├── Module.Editor.asmdef        # Separate editor assembly
│   └── {Feature}/Scripts/          # Mirrors feature structure
└── Tests/
    ├── Editor/
    │   ├── Module.Tests.Editor.asmdef
    │   └── {Feature}/{Role}/       # Mirrors source structure
    └── Runtime/
        ├── Module.Tests.Runtime.asmdef
        └── {Feature}/{Role}/       # Mirrors source structure
```

- A feature folder contains **everything** that feature needs.
- Multi-feature or featureless assets go in `Shared/`.
- Create sub-folders only as needed. Not every feature needs Art/ or AudioClips/.
- `Shared/` follows the same internal structure as a feature folder.
- `Editor/` is created on demand when editor tooling is first needed. It gets its own asmdef (`Module.Editor.asmdef`) that references the runtime module.

---

## Scripts

### Sub-roles within a feature

```
{Feature}/Scripts/
├── UI/            # UI panels, canvas controllers
├── Behaviour/     # Game logic, managers
├── Data/          # Data classes, SO definitions
├── Services/      # Service implementations
└── Installers/    # Zenject installers
```

### Service interface pattern

```
{Feature}/Scripts/Services/
├── I{ServiceName}.cs           # Interface
├── {ServiceName}.cs            # Implementation
└── Dummy{ServiceName}.cs       # Optional mock/stub
```

### Core module (`_Core`)

Organized by **technical category**, not by feature. Has its own assembly definitions.

```
_Core/
├── Core.asmdef                 # Core runtime assembly
├── {Category}/
│   ├── Scripts/
│   ├── Prefabs/
│   └── ...
├── Shared/
│   └── Scripts/                # Cross-category runtime scripts (e.g., ContextInfo)
└── Editor/
    ├── Core.Editor.asmdef      # Core editor assembly
    └── Scripts/
        └── {Category}/         # Editor-only tools organized by category
```

- Categories are broad technical domains (e.g., `Data/`, `UI/`, `Networking/`). If a category only has 1-2 scripts, it belongs in the app module's `Shared/` instead.
- `Core.Editor.asmdef` references `Core`.
- The app module's asmdef references `Core`. The app editor asmdef references `Core` and `Core.Editor`.

---

## Prefabs

- **Feature-specific:** `{Feature}/Prefabs/` (UI prefabs in `{Feature}/Prefabs/UI/`)
- **Generic reusable UI:** `Shared/Prefabs/UI/`
- **Runtime service objects:** `Shared/Prefabs/Services/`
- **Environment/level prefabs:** `Shared/Prefabs/Environments/`

---

## ScriptableObjects & Data

| Load Strategy | Path | Use Case |
|---|---|---|
| Design-time (inspector refs) | `{Feature}/Data/` or `Shared/Data/` | Config, definitions |
| Runtime (`Resources.Load`) | `{Feature}/Resources/` or `Shared/Resources/` | Dynamic content |
| Zenject injection | `Shared/Zenject/Resources/` | DI config |

- Feature-specific SOs in `{Feature}/Data/`. Subfolders for collections.
- App-wide config SOs in `Shared/Data/`.

---

## Scenes

- Core/bootstrap scenes in `Shared/Scenes/`.
- Feature-specific scenes in `{Feature}/Scenes/`.
- Scene-generated data (LightingData, NavMesh) in auto-created subfolder named after the scene.

---

## Tests

**If the project doesn't use tests, this entire section does not apply — no test folders or test asmdefs exist.**

Tests are centralized in `_Module/Tests/` (one production asmdef, two test asmdefs) but **mirror the source folder structure exactly**.

### Path mirroring rule

Given a source file, derive the test path by:
1. Replace `_Module/` → `_Module/Tests/Editor/` (or `Tests/Runtime/`)
2. Drop `/Scripts` from the path
3. Append `Tests` to the filename

| Source | Test |
|---|---|
| `_App/Scoring/Scripts/Services/ScoreManager.cs` | `_App/Tests/Editor/Scoring/Services/ScoreManagerTests.cs` |
| `_App/Input/Scripts/Behaviour/SwipeDetector.cs` | `_App/Tests/Editor/Input/Behaviour/SwipeDetectorTests.cs` |
| `_App/Shared/Scripts/Services/AudioService.cs` | `_App/Tests/Editor/Shared/Services/AudioServiceTests.cs` |

### Assembly definitions

- `Core.asmdef` — core runtime assembly at `_Core/` root
- `Core.Editor.asmdef` — core editor assembly at `_Core/Editor/`, references: `Core`
- `Module.asmdef` — production assembly at module root (no underscore prefix in asmdef names), references: `Core`
- `Module.Editor.asmdef` — editor tools at `_Module/Editor/` (created on demand), references: `Module`, `Core`, `Core.Editor`
- `Module.Tests.Editor.asmdef` — edit mode tests, references: `Module`, `Core`, `nunit.framework`, `Zenject`, `Zenject.TestFramework`
- `Module.Tests.Runtime.asmdef` — play mode tests, references: `Module`, `Core`, `nunit.framework`, `UnityEngine.TestRunner`, `Zenject`, `Zenject.TestFramework`

### When to use Edit Mode vs Play Mode

- **Edit Mode:** Pure logic, calculations, state machines, services — anything that doesn't need MonoBehaviour lifecycle
- **Play Mode:** Tests that require `Start()`/`Update()`, coroutines, physics, scene loading, or full Zenject container setup
