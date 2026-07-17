---
name: unity-testing
description: Guide for testing Unity applications. Covers Unity Test Framework, what's testable vs requires human validation, and Zenject testing patterns.
---

# Unity Testing Guide

Testing reference for Unity projects using Zenject.

---

## Edit Mode vs Play Mode

| Use Edit Mode when... | Use Play Mode when... |
|----------------------|----------------------|
| Pure logic, calculations, data transforms | Needs `Start()`, `Update()`, or other lifecycle |
| State machines, service logic | Coroutines or async operations |
| Event wiring (subscribe/fire/assert) | Physics, scene loading |
| Anything without MonoBehaviour dependency | Time-based behavior (timers, cooldowns) |
| | Multi-system integration through full container |

**Rule of thumb:** No MonoBehaviour lifecycle needed and no timing → Edit Mode. Otherwise → Play Mode.

---

## Testable vs Human Validation

| Testable (automated) | Human validation (manual) |
|---------------------|--------------------------|
| Logic and calculations | Visual appearance |
| State transitions | Animation feel |
| Event wiring and callbacks | Game feel / fun |
| Data transformations | Audio quality |
| Service interactions | Input responsiveness on device |
| Boundary conditions (min/max/zero) | Performance / frame rate |

**Design principle:** Keep logic separate from visuals. A service that computes results is trivially testable. A MonoBehaviour that displays those results requires manual validation. The more logic lives in injectable services, the more you can test automatically. Test the public interface — how the class will actually be used — not its internals.

---

## Zenject Test Fixtures

### ZenjectUnitTestFixture (Edit Mode)

Isolated DI container per test. Bind only what you need.

```csharp
using NUnit.Framework;
using Zenject;

public class ScoreServiceTests : ZenjectUnitTestFixture
{
    [Inject] private ScoreService _scoreService;

    [SetUp]
    public void Setup()
    {
        Container.Bind<ScoreService>().AsSingle();
        Container.Inject(this);
    }

    [Test]
    public void AddScore_IncreasesTotal()
    {
        _scoreService.Add(100);
        Assert.AreEqual(100, _scoreService.Total);
    }
}
```

### ZenjectIntegrationTestFixture (Play Mode)

Full container with MonoBehaviour lifecycle. Use when testing systems that depend on `Start()`.

```csharp
using System.Collections;
using NUnit.Framework;
using UnityEngine.TestTools;
using Zenject;

public class GameFlowTests : ZenjectIntegrationTestFixture
{
    [UnityTest]
    public IEnumerator GameOver_WhenHPReachesZero()
    {
        PreInstall();
        Container.Bind<HPService>().AsSingle();
        Container.Bind<GameStateService>().AsSingle();
        PostInstall();

        yield return null; // Let Start() execute

        var hp = Container.Resolve<HPService>();
        var state = Container.Resolve<GameStateService>();

        hp.TakeDamage(hp.MaxHP);

        Assert.AreEqual(GameState.Over, state.Current);
    }
}
```

---

## Pitfalls

- **Injection timing:** Dependencies are available in `Start()`, not `Awake()`. Tests that create MonoBehaviours must `yield return null` after setup to let `Start()` run before asserting.
- **Edit Mode cannot yield:** No `WaitForSeconds`, no coroutines, no frame advancement. If your test needs any of these → Play Mode.
- **`Time.deltaTime` is zero in Edit Mode:** Time-dependent logic must be tested in Play Mode, or accept a time parameter for testability.
- **Project convention — no IInitializable/ITickable/IDisposable:** Use MonoBehaviour lifecycle instead. This means integration tests often need Play Mode + `yield return null` for `Start()` to fire.
- **PostInstall before resolving:** In `ZenjectIntegrationTestFixture`, always call `PostInstall()` before resolving or injecting. `yield return null` after PostInstall for `Start()`.

---

## Running Tests

Use MCP tools to run tests and read failure details — not the Unity Editor test runner UI.

---

## References (don't duplicate — look here instead)

- **Test file locations and path mirroring** → `unity-folder-conventions` skill
- **Naming conventions** → `unity-coding-conventions` skill
