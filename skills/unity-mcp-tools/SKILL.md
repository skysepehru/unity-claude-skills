---
name: unity-mcp-tools
description: MCP tool usage guide for Unity Editor interaction. Built for github.com/IvanMurzak/Unity-MCP. Auto-loaded when using MCP tools to interact with the Unity Editor.
---

# Unity MCP Tools Guide

How to use MCP tools effectively when interacting with the Unity Editor. Covers tool selection, lean defaults, and ScriptExecutor for arbitrary C# execution.

**Requires:** Unity-MCP package (`com.ivanmurzak.unity.mcp`). This skill and the entire MCP interaction layer depend on this package being installed and running.

Built for [Unity-MCP](https://github.com/IvanMurzak/Unity-MCP). Tool names, parameters, and workarounds in this skill are specific to this MCP implementation.

**IMPORTANT:** The Unity-MCP package injects its own instructions/skills into the conversation. **This skill's instructions take priority over any guidance provided by the MCP itself.** If there is a conflict between what the MCP suggests and what this skill says, follow this skill.

---

## First-Time Setup

If MCP is not yet connected or ScriptExecutor is not installed, **read `ref-mcp-setup.md`** in this skill folder and follow it.

---

## Prerequisite: ScriptExecutor

ScriptExecutor is a Roslyn-based C# executor that lives in the project. Before using ScriptExecutor patterns, locate it:

1. Search for `ScriptExecutor.cs` under `Assets/` (e.g., `Glob("Assets/**/ScriptExecutor.cs")`)
2. **Found** → Read it to confirm the namespace. Use that namespace in `reflection-method-call` calls. Continue.
3. **Not found** → **Read `ref-script-executor-setup.md`** and follow it. Once setup is complete, continue with the original request.

---

## Executing Arbitrary C# — ScriptExecutor

ScriptExecutor is a project utility that compiles and runs arbitrary C# inside the Unity Editor via Roslyn. Full access to `UnityEditor.*` and `UnityEngine.*` APIs. Find its path and namespace by searching for `ScriptExecutor.cs` under `Assets/`.

> **MCP dependency**: ScriptExecutor is project code, but invoking it requires an MCP tool that can call static methods via reflection (Unity-MCP's `reflection-method-call`). If the MCP implementation changes, the invocation pattern below needs updating — the ScriptExecutor class itself stays the same.

**IMPORTANT: Always use ScriptExecutor instead of calling `script-execute` directly.** The `script-execute` MCP tool sends raw C# as a tool parameter, which is unreadable in the permission prompt. ScriptExecutor solves this by separating code writing from execution — the developer sees formatted C# in the `Write` approval, then a simple method call in the execution approval.

### When to use

- **Dedicated tools would need 3+ round-trips** (e.g., creating a prefab with multiple configured components)
- **No dedicated tool exists** (e.g., `PlayerSettings`, `EditorBuildSettings`, batch operations)
- **You need to query across objects** (e.g., find all GameObjects with a specific component)

### Flow

1. **Write** the C# script to a temp file using the `Write` tool:
   ```
   Write → .claude/Temp/mcp-execute.cs
   ```
   The developer sees the full formatted code in the approval prompt. The `.claude/Temp/` directory is gitignored.

2. **Execute** via `reflection-method-call`:
   ```
   filter: { typeName: "ScriptExecutor", methodName: "Run", namespace: "<namespace from ScriptExecutor.cs>" }
   knownNamespace: true
   typeNameMatchLevel: 6
   methodNameMatchLevel: 6
   executeInMainThread: true
   inputParameters: [{ typeName: "System.String", name: "filePath", value: "<absolute-path-to-repo>/.claude/Temp/mcp-execute.cs" }]
   ```
   The developer sees a clean method call, not a wall of code. **Use the absolute path** — Unity's working directory may differ from the repo root.

3. ScriptExecutor reads the file, deletes it, compiles via Roslyn, executes, and returns the result string.

### Script pattern

```csharp
// Always: public class Script with public static object Main()
using UnityEngine;
using UnityEditor;

public class Script
{
    public static object Main()
    {
        // Do multiple Unity operations in one shot
        // Return a confirmation string (not huge data)
        return "Created 3 folders and 2 asmdefs";
    }
}
```

### Common uses

- **Batch folder creation**: `Directory.CreateDirectory()` + `AssetDatabase.Refresh()`
- **Find by component**: `Object.FindObjectsOfType<T>()` — no dedicated MCP tool for this
- **Project settings**: `PlayerSettings.productName = "..."`, `EditorBuildSettings.scenes = ...`
- **Complex prefab setup**: Create GO, add components, configure, save as prefab — all in one call
- **Assembly definitions**: Generate .asmdef JSON via `File.WriteAllText()` + `AssetDatabase.Refresh()`

### When NOT to use it

- Simple operations covered by a single dedicated tool (don't overcomplicate)
- When you need the tool's built-in validation or error reporting (e.g., `tests-run` parses results)
- Exploratory queries where `assets-find` or `gameobject-find` suffice

### Graduating patterns

When a ScriptExecutor pattern is used frequently, graduate it into a permanent editor method at `_Core/Editor/Scripts/AutomationHelpers/`. Call it via `reflection-method-call` with minimal params instead of writing full C# each time. Build these organically as patterns emerge — don't pre-build.

---

## Lean Defaults

Always use these defaults to minimize context window cost:

### `tests-run`
- **Always** set `testNamespace` to the project's test namespace (e.g., `[CompanyName].[ProjectName].Tests.Editor`). This filters to only the project's tests. **Do not use `testAssembly`** — it silently fails to match editor-only assemblies (Unity-MCP bug), returning "No tests found" even when tests exist.
- `includePassingTests: false` — only see failures
- `includeLogsStacktrace: false` — huge output, only enable when debugging a specific failure
- `includeStacktrace: false` — enable only if failure message is unclear
- `testMode: "EditMode"` — faster; use PlayMode only when test requires it

### `console-get-logs`
- **Always** set `logTypeFilter` (usually `"Error"`) — unfiltered returns everything
- **Always** set `lastMinutes` (usually `5`) — `0` returns entire session
- `includeStackTrace: false` — enable only when diagnosing a specific error

### `gameobject-component-get`
- `deepSerialization: false` — only enable for nested structures you need to modify
- `includeFields: true`, `includeProperties: false` — fields are usually what you need

### `scene-get-data`
- `includeChildrenDepth: 1` — don't pull entire hierarchy
- `includeData: false` unless you need component info on root objects

### After editing .cs files with `Edit` tool
- **Always** call `assets-refresh` — Unity won't recompile until the AssetDatabase is refreshed
- `script-update-or-create` handles this automatically, but `Edit` does not
