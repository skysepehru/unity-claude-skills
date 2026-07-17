---
description: Architecture best practices for Unity projects - dependency direction, orchestration, event patterns. ALWAYS Auto-loaded when writing code for Unity Projects.
---

# Unity Architecture Conventions

Follow these architectural patterns when designing and structuring Unity project code.

## Application Startup & Orchestration

**CRITICAL: Every scene must have a visible starting point.**

- A visible starting point means an explicit orchestrator that controls the startup sequence — e.g., `GameManager.StartGame()`, not logic auto-running in `Start()`.
- MonoBehaviours must NOT perform actions or start processes in Awake/Start on their own. Only resolve references or do simple caching.
- Self-contained objects (like pooled enemies) can initialize on instantiation, but top-level systems need external orchestration.
- If there is no clear "who starts the game?" answer when reading the scene, the architecture is wrong.

## Dependency Direction

- Dependencies flow downward: orchestrators → services → leaf components.
- Think of it as a tree: one parent per node, no reaching upward.
- A downstream consumer must not stop, reset, or control systems above it in the hierarchy. If a game-over event needs to stop the beat clock, the game-over owner (or a dedicated orchestrator) should do it — not a downstream consumer that happens to hold a reference.
- When injecting, ask: "does this class need to *know about* this system, or should it just *emit an event* that the system reacts to?"
- Watch for excessive injection count — it often means the class has too many responsibilities.

## Event Lifecycle

- When a state change should stop event handling, **unsubscribe from events** rather than guarding every handler with `if (IsGameOver) return;`.
- The owner of the state transition is responsible for unsubscribing dependents or signaling them to unsubscribe.
- Boolean guards accumulate across the codebase and are easy to forget in new handlers. Unsubscription is a single point of control.

## Events vs Direct Calls

- When multiple independent systems react to the same occurrence (e.g., a grade result affecting score, HP, and combo), publish an event rather than injecting and calling each system directly.
- The publisher should not know about its consumers. If adding a new consumer requires modifying the publisher, use an event instead.
- This prevents the publisher from accumulating injections for every new consumer.
