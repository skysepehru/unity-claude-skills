# Unity-MCP First-Time Setup

One-time setup for Unity-MCP connection and tool configuration. Follow this when MCP is not yet connected to the project.

---

## 1. Install Unity-MCP Package

The Unity-MCP package (`com.ivanmurzak.unity.mcp`) must be installed in the Unity project. If not already installed, tell the developer to follow the instructions at [github.com/IvanMurzak/Unity-MCP](https://github.com/IvanMurzak/Unity-MCP).

---

## 2. Connect MCP to Claude Code

Tell the developer to configure the MCP server connection following the Unity-MCP README. They need to:
1. Open Unity with the project loaded
2. Go to **Tools > MCP Unity > Server Window** and start the server
3. Configure Claude Code to connect to the MCP server (the Unity-MCP README has the exact steps)

---

## 3. Verify Connection

Call `editor-application-get-state` to confirm Unity is running and MCP is connected.

If this fails, tell the developer to check that:
- Unity is open with the project loaded
- MCP server is running (Tools > MCP Unity > Server Window)

---

## 4. Configure MCP Tools

Ask the developer to **enable the `tool-set-enabled-state` tool** in the MCP settings — it may be disabled by default. Once enabled, use it to disable redundant tools:

| Tool to Disable | Reason |
|-----------------|--------|
| `script-execute` | Replaced by ScriptExecutor flow (Write temp file + `reflection-method-call`) |
| `script-update-or-create` | Replaced by `Write` tool + `assets-refresh` |
| `script-read` | Replaced by `Read` tool |
| `assets-create-folder` | Replaced by `mkdir -p` + `assets-refresh` |
| `screenshot-scene-view` | Not needed unless visual inspection is required |
| `screenshot-game-view` | Not needed unless visual inspection is required |

Ensure `reflection-method-call` is **enabled** — it's critical for the ScriptExecutor flow.

**IMPORTANT:** After changing tool states, Claude must be restarted for the changes to take effect. Tell the developer to restart Claude after this step.

---

## 5. Set Up ScriptExecutor

**Read `ref-script-executor-setup.md`** in this skill folder and follow it to install ScriptExecutor in the project.

---

## 6. Done

MCP is connected, tools are configured, and ScriptExecutor is ready. Return to the main skill.
