# ScriptExecutor First-Time Setup

One-time setup of ScriptExecutor — a Roslyn-based C# executor that lets you run arbitrary Unity Editor code via MCP.

---

## 1. Explain to the Developer

Tell the developer:

> **ScriptExecutor** is a small utility script that compiles and runs arbitrary C# inside the Unity Editor via Roslyn. It's used instead of the raw `script-execute` MCP tool because it separates code writing from execution, making permission prompts readable.
>
> The mechanism: I write C# to a temp file (`.claude/Temp/mcp-execute.cs`), then call `ScriptExecutor.Run(filePath)` via MCP reflection. The script reads the file, deletes it, compiles it, and runs it. This is a one-time setup.

---

## 2. Determine Path and Namespace

Examine the project's `Assets/` folder structure to find a logical location:

- Look for an existing Editor scripts folder (e.g., `Assets/*/Editor/Scripts/` or `Assets/Editor/`)
- Look for existing namespace patterns in `.cs` files to match the project's conventions
- Look for assembly definitions that cover Editor code

**Default recommendation** (if no clear pattern exists):
- **Path**: `Assets/_Core/Editor/Scripts/AutomationHelpers/ScriptExecutor.cs`
- **Namespace**: `Core.Editor`

**If patterns exist**, adapt:
- If there's a `Assets/MyGame/Editor/` folder, recommend putting it there
- If existing scripts use `CompanyName.ProjectName.Editor` namespace, use that
- The script needs to be under an Editor folder (or covered by an Editor-only asmdef) since it uses `UnityEditor` APIs

Present the recommended path and namespace to the developer and ask for confirmation.

---

## 3. Create ScriptExecutor.cs

Once confirmed, create the script at the agreed path using `script-update-or-create` (which auto-refreshes the AssetDatabase).

Read `ScriptExecutor.cs.template` in this skill folder. Copy its contents, replacing `[CompanyName]` in the namespace with the confirmed namespace.

---

## 4. Ensure Temp Folder

Create `.claude/Temp/` if it doesn't exist. This is where temp scripts are written before execution.

If the project uses git, ensure `.claude/Temp/` is in `.gitignore`. Check for an existing entry — if not present, append:
```
# Claude Code temp files
.claude/Temp/
```

---

## 5. Done

Setup is complete. Return to the main skill and continue with the original request.
