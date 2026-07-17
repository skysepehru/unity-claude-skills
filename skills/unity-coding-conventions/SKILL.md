---
description: C# coding conventions for Unity projects - naming, style, and organization. ALWAYS Auto-loaded when writing code for Unity Projects.
---

# Unity Coding Conventions

Follow these conventions when writing C# code for Unity projects.

## Repository & File Naming

- Repositories: kebab-case ending with "-unity" (e.g., `my-game-unity`)
- Files and directories: PascalCase
    - In case of script files, its name will be the same as the most important definition inside that file (class/struct/enum, etc.)
- Special ordering prefix: _PascalCase (to appear first alphabetically, should be used sparingly, mostly for folders)

## Abbreviation Rules

**In PascalCase:**
- 3+ letters: First capitalized, rest lowercase → `GpsManager`
- 2 letters: Both capitalized → `UIManager`
- Exception: "id" is always `Id` in PascalCase → `UserId`, `GetId()`

**In camelCase (including _camelCase, m_camelCase):**
- All lowercase → `_gpsManager`, `m_uiManager`

## Class & Method Naming

- Classes and structs: PascalCase
- Methods: PascalCase
- Method arguments: camelCase
- Local variables: camelCase
- Method names must accurately describe what the method does — a method that retrieves an available item should be `GetAvailable()` or `Rent()`, not `Activate()` if it doesn't perform activation

## Namespace Conventions

**One namespace per assembly.** The namespace matches the assembly name, prefixed with the company/username:

| Assembly (asmdef name) | Namespace | Purpose |
|------------------------|-----------|---------|
| `Core` | `[CompanyName].Core` | Shared/foundational runtime code (`_Core/`) |
| `Core.Editor` | `[CompanyName].Core.Editor` | Core editor tools & automation (`_Core/Editor/`) |
| `[ProjectName]` | `[CompanyName].[ProjectName]` | Runtime code |
| `[ProjectName].Editor` | `[CompanyName].[ProjectName].Editor` | Editor tools |
| `[ProjectName].Tests.Editor` | `[CompanyName].[ProjectName].Tests.Editor` | Edit mode tests |
| `[ProjectName].Tests.Runtime` | `[CompanyName].[ProjectName].Tests.Runtime` | Play mode tests |

Example for `COMPANY_NAME = acme`, project `MyGame`:
- `acme.Core`, `acme.Core.Editor`
- `acme.MyGame`, `acme.MyGame.Editor`
- `acme.MyGame.Tests.Editor`, `acme.MyGame.Tests.Runtime`

Every `.cs` file must have a namespace. The asmdef `rootNamespace` field is set to the assembly's namespace so Unity auto-fills it.

- Avoid sub-namespaces within an assembly for categorization — use folders instead
    - Rationale: Too many coupled namespaces add boilerplate without improving readability
- File-scoped namespaces: Use `namespace Foo;` if C# 10 is enabled, otherwise traditional block style
- Using directives: Place outside namespace, no specific ordering required

## Field Naming

| Field Type | Convention | Example |
|------------|------------|---------|
| Public/Protected fields | PascalCase | `public int Health` |
| Properties | PascalCase | `public int Health { get; set; }` |
| Private immutable | _camelCase | `[SerializeField] private Transform _target` |
| Private mutable | m_camelCase | `private float m_currentSpeed` |
| Coroutine refs | m_camelCase + Coroutine suffix | `private Coroutine m_movementCoroutine` |
| Static mutable | s_camelCase | `private static int s_instanceCount` |
| Static readonly | _camelCase | `private static readonly float _defaultSpeed` |
| Constants | ALL_CAPS | `private const float MAX_SPEED` |

**Properties vs Public Fields:**
- Never use plain public fields (except ScriptableObjects — see below)
- **Auto-property** `{ get; private set; }` — use when no custom logic is needed. This is the default choice.
- **Backing field** `private int m_health; public int Health => m_health;` — use only when:
  - The setter contains validation or side-effect logic
  - The backing field serves double duty (e.g., `[SerializeField]` for inspector exposure alongside a public getter)
  - The field needs internal mutation with complex logic that a simple setter can't express
- For Unity inspector exposure: `[field: SerializeField] public int Health { get; private set; }`

**Why "_camelCase" for "immutable"?**
In common Unity workflows, many fields cannot be marked `readonly` even though we would want them to be. These fields are set once during initialization and never changed - they are "ideally" immutable. Use `_camelCase` for these fields to signal intent:
- `[SerializeField]` injections (set by inspector, never reassigned)
- `[Inject]` injections (set by Zenject, never reassigned)
- Fields resolved in constructors, Awake, or equivalent initialization methods

## Events

- Events: PascalCase with "On" prefix and past tense → `OnButtonClicked`, `OnDamageReceived`
- Handler methods: Use "Handle" prefix → `HandleButtonClicked`, `HandleDamageReceived`

## Other Naming

- Interfaces: Start with "I" → `ICopyable`
- Enums: Plural name → `States`
- Enum values: Always explicitly numbered starting from 0 → `None = 0, Running = 1, Stopped = 2`
- Enum instances: Singular → `private States m_state`

## ScriptableObjects

- Naming: End with "SO" → `PlayerConfigSO`, `WeaponDataSO`
- Use public fields for data (exception to the properties preference)

## Data Models

- Naming: End with "Model" → `PlayerModel`, `EnemyModel`
- Use auto-properties `{ get; set; }` for data fields
- Data models are plain C# classes/structs for data transfer (DTOs, API responses, serialization)

## Async

- Async methods end with "Async" → `ConnectToServerAsync`
- CancellationToken parameter: `cancellationToken` (full name), always last parameter, with `= default`
- CancellationTokenSource fields: `m_operationCts` (mutable) or `_lifetimeCts` (immutable)

**Exception handling in async:**
- Always handle exceptions properly - never fire-and-forget without error handling
- Use try-catch in async methods, log errors before re-throwing if needed
- For fire-and-forget calls, wrap in a method that catches and logs exceptions

## Coroutines

- Coroutine methods (returning `IEnumerator`): End with "Coroutine" → `MovementCoroutine()`
- Coroutine reference fields: End with "Coroutine", follow mutable field convention → `private Coroutine m_movementCoroutine`

## Update vs Coroutines/UniTask

- Reserve `Update()` for continuous per-frame logic that truly runs every frame (e.g., input polling, smooth animation).
- If an `Update()` body begins with `if (!m_isRunning) return;` or similar guard — that is a signal it should be a coroutine or UniTask instead.
- Prefer coroutines or UniTask for state-driven loops, timed sequences, or logic that activates/deactivates during gameplay.

## Code Style

- Curly braces on separate lines (Allman style)
- `var`: Use for complex types, explicit for primitives (`int`, `string`, `bool`, `float`)
- Expression bodies (`=>`): Always use when body is a single expression
- Modifier order: `public static readonly` (access → static → readonly)
- Never use `#region`

**Local methods:**
- Place at the end of the containing method
- If void method, add explicit `return;` before local methods:
```csharp
private void DoWork()
{
    var data = GetData();
    ProcessData(data);

    return;

    int GetData() => 42;
    void ProcessData(int d) => Debug.Log(d);
}
```

## Enum Scoping

- If enum is only used inside one class: declare inside that class as private
- If rarely used outside: declare inside as public
    - Benefit: Easier naming without project-wide uniqueness concerns

## Member Ordering

Order members within a class from top to bottom:
1. Nested types (classes, structs, enums)
2. Constants (`const`)
3. Readonly fields (`readonly`)
4. Immutable fields (`_camelCase`)
5. Zenject injections (`[Inject]`)
6. Inspector injections (`[SerializeField]`, `[SerializeReference]`, `[field: SerializeField]` properties)
7. Mutable fields (`m_camelCase`)
8. Delegate definitions
9. Events / delegate instances
10. Public fields
11. Properties
12. Initializers (constructor, Awake)
13. Methods - ordered logically (caller above callee, Clean Code style)

**Unity lifecycle methods:** Place initialization methods first (Awake, OnEnable, Start), disposal methods last (OnDisable, OnDestroy). No strict ordering for other lifecycle methods.

**Closely related fields:** When fields have a tight relationship, place them adjacent to each other within their section. Specifically, a backing field immediately precedes its property:
```csharp
// Properties section
private int m_health;
public int Health => m_health;

private string m_playerName;
public string PlayerName { get => m_playerName; set => m_playerName = value; }
```

## Null Handling

- Prefer `?.` and `??` operators
- **Warning**: Do NOT use `?.` or `??` for `UnityEngine.Object` - Unity overloads `==` for destroyed object detection, so use explicit null checks instead
- **No silent null returns:** Early returns on unexpected null must include `Debug.LogWarning` so failures are visible during development. Silent `if (x == null) return;` hides bugs.

## LINQ

- Use method syntax only (`.Where().Select()`)
- Never use query syntax (`from x in y`)
- For performance implications of LINQ in hot paths, see `unity-performance` conventions

## Generics & Tuples

**Generic type parameters:**
- Use descriptive names (`TEntity`, `TResult`) when meaning isn't obvious
- Single `T` is acceptable if clear from class name

**Tuples:**
- Always name elements: `(int x, int y)` not `(int, int)`

## Method Parameters

- Avoid boolean parameters - use enums or separate methods instead

## Constants Location

- Class-specific constants: Keep in the class that uses them
- Shared/project-wide constants: Place in a dedicated constants file (e.g., `GameConstants.cs`)
- `const` is for true invariants only — math constants, fixed protocol values. Gameplay-tuning values (speeds, thresholds, durations) should be `[SerializeField]` or ScriptableObject config so they're adjustable without recompilation.

## Comments

- Only for convoluted sections of code
- Code should be self-explanatory without comments
- If you need many comments, consider refactoring the code
