# unity-claude-skills

A Claude Code plugin that bundles convention skills for **Unity development**. Install it once and Claude Code will follow consistent Unity coding, architecture, performance, folder, Zenject, UGUI, and testing conventions across any project.

Distributed as a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces).

## Install

In any Claude Code session:

```
/plugin marketplace add skysepehru/unity-claude-skills
/plugin install unity-claude-skills@skysepehru
```

That's it. Most skills are **model-invoked** — Claude loads them automatically based on what you're working on (e.g. writing C# for a Unity project). You don't need to call them by name.

To update later:

```
/plugin marketplace update skysepehru
```

## Skills

| Skill | What it's for |
|-------|---------------|
| `unity-coding-conventions` | C# naming, style, and organization for Unity. |
| `unity-architecture-conventions` | Dependency direction, orchestration, and event patterns. |
| `unity-performance` | Avoiding allocations and GC pressure on hot paths. |
| `unity-folder-conventions` | Folder structure when creating files/folders. |
| `zenject-conventions` | Zenject DI conventions and coding patterns. |
| `unity-ugui-development` | Conventions for building UGUI-based UI. |
| `unity-ugui-list` | Generating uGUI list components from a list of data. |
| `unity-mcp-tools` | Driving the Unity Editor via [Unity-MCP](https://github.com/IvanMurzak/Unity-MCP). |
| `unity-testing` | Testing Unity apps — Unity Test Framework and Zenject test patterns. |
| `writing-tests` | Writing/organizing C#/.NET tests (NUnit + NSubstitute, `dotnet test`). |

## Usage notes

- Plugin skills are namespaced, so if you ever want to invoke one explicitly it's `/unity-claude-skills:<skill-name>`.
- After installing or updating, run `/reload-plugins` to pick up changes without restarting.

## License

Released under the [MIT License](LICENSE).
