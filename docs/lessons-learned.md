# Lessons learned

[Back to README](../README.md) · [Reverse engineering](reverse-engineering.md) · [Testing](testing.md)

This reflection distinguishes recorded outcomes from engineering takeaways and possible future work. It makes no claim about measured productivity gains or complete equivalence to the original game.

## MCP tool orchestration

**Observed:** REA exposed browser discovery and passive page inspection through MCP. Discovery confirmed the target before source capture; the analysis did not require configuration changes or page interaction.

**Takeaway:** Use the smallest tool sequence that answers the question. Confirm target identity, inspect the required data, and retain evidence identifiers and capture limits. Tool availability alone does not establish that a particular target or analysis provider is ready.

## Evidence-based investigation

**Observed:** The captured `index.js` established the default speed values and actual update logic. Its SHA-256 and reported byte count identify the inspected source. The original uses `SPEED`, not an `INITIAL_SPEED` constant.

**Takeaway:** Trace assignments and callers rather than relying on names, appearance, or assumptions. Separate source facts from runtime observations. An inspected function is evidence of implemented behavior, not proof of execution; source availability also means this investigation should not be presented as recovery of proprietary internals.

## Frame-dependent versus elapsed-time behavior

**Observed:** The original speed increment has no elapsed-time multiplier, while movement does. The reconstruction retains this distinction and makes one speed decision per eligible update, even when elapsed time is zero.

**Takeaway:** A time-scaled approximation can be a good design elsewhere but would change this project's investigated behavior. Fidelity should be judged against an explicit behavioral contract. Actual eligible update frequency determines wall-clock acceleration; no frame-rate benchmark was performed here.

## Floating-point boundaries

**Observed:** Repeated increments from 6 produced `12.999999999997897` at update 7,000 and `13.000999999997896` at update 7,001. The threshold allows the extra increment. Tests combine numeric tolerance with strict below/above-threshold assertions.

**Takeaway:** Test immediately before and after a branch boundary, not only distant checkpoints. A tolerance appropriate for arithmetic equality should not erase the distinction that controls the branch. Clamping to exactly 13 would conceal the behavior this project aims to preserve.

## Swept collision detection

**Observed:** At speed 20 and a 50 ms update, an obstacle moved 60 pixels across the dinosaur without overlapping at either endpoint. The original local implementation missed it. The revised implementation checks relative-motion overlap intervals on both axes, and the deterministic regression now enters game-over.

**Takeaway:** Endpoint checks are insufficient when displacement can exceed a hitbox's effective width. Collision precision and update frequency are separate concerns: fixing collision coverage should not introduce extra acceleration increments. The implemented sweep remains an approximation using inset rectangles and linear motion between endpoints.

## Unit, integration, and mutation tests

**Observed:** Pure-function tests checked the production helper but missed faults in its caller. Integration tests now exercise `updateGame()`, restart, visibility timing, and keyboard routing. The recorded result is 15 unit plus 52 integration checks, with ten selected faulty variants rejected in memory.

**Takeaway:** Test a rule in isolation and then test how the application uses it. Mutation checks provide concrete evidence that selected defects trigger failures; they do not prove universal fault detection. Browser UAT is another layer: the owner's seven completed checks supplement the mocks but do not establish cross-browser or accessibility coverage.

## Input focus and state ownership

**Observed:** The earlier blanket exclusion of button targets prevented ArrowUp from reaching the game after control interaction. Start/Restart now return focus to the canvas, inputs keep native keyboard behavior, and tests verify handler routing. Restart also retains the achieved best score before clearing the current run.

**Takeaway:** Define ownership of keyboard events explicitly to avoid duplicate native and game actions. State transitions should preserve durable session data while resetting run-specific data. Mocked handlers and focus objects help test intent, but native browser activation still requires browser validation.

## Limitations and future improvements

The current implementation is a single HTML file with shared state and some DOM coupling. It has no persisted best score, package-managed test runner, CI, benchmark suite, or automated browser coverage. The owner confirmed manual UAT on 2026-10-13. A project screenshot is included; its capture details and the final UAT browser/version and operating system remain unrecorded. The independently authored project has an MIT license; upstream documentation excerpts retain their own licensing. Its simplified physics, scoring, obstacle generation, and artwork intentionally differ from the source game.

Possible future work, not completed outcomes:

- Add an automated browser suite for keyboard default actions, focus transitions, sliders, visibility changes, and touch input.
- Preserve a reproducible test runner and evidence export with versioned source identities.
- Add seeded obstacle generation and broader collision geometry/property tests.
- Reduce idle rendering and introduce an explicit lifecycle/disposal API if the game becomes an embedded component.
- Review high-DPI rendering, assistive-technology support, and reduced-motion behavior.
- Consider persistent best scores as a separate feature with an explicit storage policy.
- Complete applicable upstream notices for the documentation excerpts and record screenshot and final UAT environment details before publication.

Any future move to elapsed-time acceleration should be an explicitly separate mode, because it would change the source-derived behavior preserved here.
