---
description: Performance best practices for Unity C# code - allocations, hot paths, GC pressure. ALWAYS Auto-loaded when writing code for Unity Projects.
---

# Unity Performance Conventions

Follow these conventions to avoid common performance pitfalls in Unity C# code.

## LINQ & Allocations in Hot Paths

- **Never use LINQ in hot paths** — `Update()`, `FixedUpdate()`, input handlers, physics callbacks, and any method called frequently.
- LINQ methods (`Where`, `FirstOrDefault`, `Select`, etc.) allocate enumerator objects on each call, creating GC pressure.
- Use manual `for`/`foreach` loops instead.
- Properties that expose LINQ queries (e.g., `public IEnumerable<T> ActiveItems => _items.Where(...)`) allocate on every access — cache the result or use manual iteration.

## Collection Allocation

- Do not create new collections (`new List<T>()`) in methods called frequently — pre-allocate and reuse.
- For methods that return filtered subsets, prefer passing in a pre-allocated list to fill, or cache the result and invalidate on change.

## General GC Awareness

- Avoid boxing value types (passing structs as `object`).
- String concatenation in hot paths — use `StringBuilder` or `string.Create`.
- Closures/lambdas in hot paths capture variables and allocate — extract to methods when possible.
