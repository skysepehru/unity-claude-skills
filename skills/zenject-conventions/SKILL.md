---
name: zenject-conventions
description: Zenject conventions and coding patterns for Unity projects. ALWAYS Auto-loaded when writing code for Unity Projects using Zenject.
---

# Zenject Conventions

Conventions and coding patterns for using Zenject in Unity projects.

**Note:** Original Zenject is no longer maintained. Use [Extenject](https://github.com/Mathijs-Bakker/Extenject), the actively maintained fork. The API is identical.

## MonoBehaviours vs Zenject Abstractions

**Always use MonoBehaviours. Do NOT use IInitializable, ITickable, IDisposable.**

Reasons:
- Easy to see all existing classes from the scene itself
- Easier for new team members to pick up
- Enables inspector injection (best way in Unity - instant access on Awake)
- MonoBehaviours are the cornerstone of Unity's architecture - don't fight it

## Context Hierarchy

**Small projects:**
- Use Project Context + single loaded scene

**Larger projects - Scene Parenting:**
- Load scenes additively, inheriting dependencies from parents
- Base scene initializes fundamental systems, others build on top
- Scenes can be added/removed to alter application state
- Better loading times
- Choose active scene properly

**GameObject Contexts:**
- Use to make game entities as self-contained as possible

## Installer Placement

**MonoInstallers go on the same GameObject as the Context component.** The SceneContext (or GameObjectContext) GameObject hosts both the Context component and all its MonoInstaller components. Do NOT create child GameObjects for installers.

## Factories

**Avoid Zenject's built-in factories** - they are needlessly sophisticated and convoluted.

**Preferred approach:**
1. Create a normal class ending with `Factory`
2. Inject `IInstantiator` into it (not `DiContainer` - limits scope to instantiation only)
3. Perform all creation actions inside
4. Create the factory at composition root
5. Inject this factory into other classes

```csharp
public class EnemyFactory
{
    [Inject] private IInstantiator _instantiator;
    [Inject] private EnemyConfig _config;

    public Enemy Create(Vector3 position)
    {
        var enemy = _instantiator.InstantiatePrefabForComponent<Enemy>(_config.Prefab);
        enemy.transform.position = position;
        return enemy;
    }
}
```

## Singleton Pattern

**General rule:** Never use the traditional singleton pattern. Use Zenject's `AsSingle()` instead.

**Exception - GlobalAccessor pattern:**
When a piece of functionality:
- Cannot use a static method because it has Zenject dependencies
- Is very unique in nature
- Is unlikely to massively change or need abstraction

Use a MonoBehaviour called `GlobalAccessor` with a static `Singleton` field as the single entry point for all singletons.

## Injection Guidelines

**Prefer field injection** over constructor or method injection:
- Easy to use with minimal boilerplate
- Removes constructor boilerplate entirely
- Avoids needing to solve circular dependency issues

**Avoid injection with IDs** - hard to keep track of. Prefer creating a new type.

**Inspector injection vs Zenject injection:**
- If a MonoBehaviour depends on another MonoBehaviour in the scene that is not directly connected or is unique, use DI instead of inspector injection
- Especially if the same MonoBehaviour is referenced in multiple places
- Example: Main camera should be injected via Zenject

**Never inject DiContainer directly** - it becomes a service locator anti-pattern. For factories, inject `IInstantiator` instead (limits scope to instantiation only).

## MonoBehaviour Lifecycle with Zenject

**Order of execution:**
1. `Awake()` called
2. `OnEnable()` called (same call stack as Awake)
3. Zenject injections happen (including `[Inject]` methods)
4. `Start()` called

**Critical rule:** DO NOT USE `[Inject]` METHOD CALLS FOR INITIALIZATION LOGIC.

- Any logic depending on Zenject dependencies should occur in `Start()`
- Never use injection methods to call code between Awake and Start

```csharp
public class PlayerController : MonoBehaviour
{
    [Inject] private IInputService _inputService;
    [SerializeField] private float _moveSpeed;

    private void Awake()
    {
        // Only non-Zenject setup here (e.g., GetComponent calls)
    }

    private void Start()
    {
        // Zenject dependencies are available here
        _inputService.OnMove += HandleMove;
    }
}
```

## Application Startup Pattern

**Rule: Application should always have a visible starting point.**

**On application start:**
- Scripts (especially MonoBehaviours) should NOT perform actions or start processes in Awake/constructors
- Only resolve references or do simple caching
- Significant startup actions (server connection, asset loading, scene loading) should be initiated from the application root

**After initial launch:**
- Self-contained objects (like enemies) can start their own processes on instantiation
- Objects connected to many systems (like player) should have initialization order controlled externally
- The more self-contained, the better

## Design Principles

**Prefer prefabs over scene objects:**
- Make as many prefabs as possible, if it makes sense
- Allows multiple developers to make changes without main scene conflicts

**Composition over inheritance:**
- Avoid deep inheritance trees, use composition instead
- Exception: Well-known structured systems where type safety is crucial
- Even downcasting at runtime is acceptable if it avoids significant complexity

**Avoid premature abstractions:**
- Start with a simple class, create interfaces and abstractions on demand
- If there will only be a few types of a class, an enum with if-else may be better than inheritance
