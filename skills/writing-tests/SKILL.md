---
name: writing-tests
description: Conventions for writing and organizing C#/.NET tests in this repo — NUnit + NSubstitute, run via `dotnet test`. Covers the goal (unit-test everything), the three kinds of test (unit / integration / end-to-end), how to make code testable, and the hard rules. Unity gotchas are noted at the very end. Use whenever creating or editing a .NET test.
---

# Writing tests

## Goal: unit-test everything

Every unit is tested, in isolation — its collaborators mocked — so a bug anywhere else can't fail its tests. That's the default, and we aim for it aggressively.

**Not unit-testing something is a compromise, never a stylistic choice.** There are exactly two reasons to skip:

1. **Genuinely hard to unit-test** — code whose only job is to cross a boundary: a database, an external service or SDK, the filesystem, the network, secrets, the clock, randomness. Skipping it is reluctant; cover it with an **integration test** instead.
2. **It would only test the C# compiler.** A composition root (pure `new` wiring) or a one-line forwarder has nothing to assert but that the language works. **Never test the compiler.**

Aggressive unit isolation is safe *because* integration tests cover the real composition — the two layers are a pair, not alternatives.

## The three kinds of test

**Unit** — SUT is **one class**. Every collaborator is a NSubstitute mock (`Substitute.For<IFoo>()`), the SUT is freshly built per test in `[SetUp]`. Mock *everything* the SUT talks to, **including pure collaborators** (through their interface) — control them with `.Returns(...)`, verify with `Received()`/`DidNotReceive()`. Lives in its own folder `<Component>/<Component>Tests.cs`, namespace `….<Component>Tests`; one folder per SUT; shared builders live in a local `Definitions/`.
  - *Contract / snapshot variant (still a unit test):* pin a **data contract** instead of logic — feed a **frozen JSON literal** of already-stored bytes (a snapshot, *not* generated from the model), deserialize, and assert the fields. Guards against a rename/removal/retype that would orphan persisted data. **Constants whose value is tied to written data** (storage keys, endpoint names, enum→number mappings) get pinned the same way. (Don't assert an arbitrary constant's value — tautology.)

**Integration** — SUT is the **assembled system driven through its public API**, with the REAL units wired together. Fake **only the outermost boundary** — a *stateful* in-memory double that behaves like the real dependency (and round-trips serialization the same way, so serialization bugs surface too). Assert against the real end-state — what actually landed in the double. This is where a hand-written stateful fake is the right tool.

**End-to-end** — the whole system against the **real external systems** (real auth, database, configuration, third-party services), not doubles. Verifies what no mock can — dependency-injection wiring, real serialization at the boundaries, auth, and environment/config resolution. Slow and environment-dependent.

## Make code testable (design-for-test)

The point of most refactors here is to create seams so units can be isolated:

- Extract logic into classes with **constructor-injected interfaces**. Prefer a **public seam over `internal` + `InternalsVisibleTo`**.
- Wrap anything crossing a boundary (I/O, SDK, secrets, clock, randomness) behind an interface so callers mock it.
- **A SUT's collaborators can't be statics** — a static call can't be substituted, so it breaks isolation. Make the collaborator an **instance behind an interface** (drop the static). If a static can't be removed because other callers still depend on it, keep it and add a thin instance that implements the interface and forwards to it — a bridge, not the goal. **Put the interface with the code it abstracts** (code in a shared library → interface in that library; consumers depend on it, don't re-declare it in the consumer). A static is **always a compromise** on testability (you can't inject or substitute it) — sometimes an acceptable one (a helper tested directly as its own SUT, a leaf utility you consciously don't isolate, a constant), but never a first-class choice.
- When a collaborator must be **created dynamically** (it needs runtime context, so it can't be a constructor-injected singleton), inject a **factory** for it rather than `new`-ing it inline — a factory is mockable, an inline `new` isn't. Leave a seam (e.g. an overridable factory or an extra constructor parameter) so integration tests can substitute the outermost boundary while keeping the real wiring.
- Thin adapters (endpoints, etc.) just delegate to injected services — the humble object; the logic lives in the tested service.

## Rules for every test

- Test through the **public API**; name **`Method_Scenario_ExpectedBehavior`**; **Arrange / Act / Assert**.
- **`Assert.Multiple(() => { … })`** when a test has several related asserts; otherwise one assert-concept per test.
- **Never assert an exception's type or message** — bare `Assert.Catch` (sync) / `Assert.CatchAsync` (async). Encode the reason in the test name.
- **Named `const`s** for meaningful values; trivial structural `0`/`1` may stay inline; express expected results as arithmetic on named inputs where it reads clearly.
- **No `if`/loop/switch driving assertions.** (Data-construction loops inside `Definitions/` builders are fine — that's arrange, not assertion logic.)
- **Deterministic & isolated** — fresh SUT per test, no shared mutable state, no ordering dependence, no real clock/randomness.

## Project layout

- The test project is a **sibling dir** of the code it tests (never nested — a nested SDK project gets globbed into the parent's compile). Add it to the `.sln`; run `dotnet test <sln>`.
- Stack: NUnit 4 + NUnit3TestAdapter + Microsoft.NET.Test.Sdk, plus NSubstitute (and a JSON serializer if your tests round-trip DTOs).
- After a testability refactor, `dotnet build` the code under test (Debug **and** Release) before writing tests.

## Unity gotchas (the above is .NET; Unity differs)

Unity tests run under the **Unity Test Framework (NUnit 3.x)** via asmdefs — not `dotnet test` — in `Tests/Editor/` (EditMode) and `Tests/Runtime/` (PlayMode).

- **NSubstitute must be installed** (add its DLLs to the project) so mocking works the same as in .NET. (Same folder-per-SUT + `Definitions/` convention.)
- No `Assert.CatchAsync`. Time-based / coroutine behavior uses `[UnityTest]` + `yield return`; use generous timing margins and never assert on real frame time.
- MonoBehaviour lifecycle, coroutines, and scenes need **PlayMode**; pure C# logic belongs in **EditMode**.
- Everything else holds: naming, AAA, isolation, named consts, bare `Assert.Catch`, one folder per SUT.
