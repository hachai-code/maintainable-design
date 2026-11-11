# Code Craft (Hunt & Thomas, *The Pragmatic Programmer*; Martin, *Clean Code*)

## DRY — it's about knowledge, not text

Every piece of *knowledge* (a business rule, a format, a constant's meaning) must have a single authoritative representation. Two different rules that happen to produce similar code are NOT duplication — merging them couples things that change for different reasons. Conversely, the same rule expressed in code + a validation + a doc comment + a DB constraint IS duplication even though no text matches. Ask "if this rule changed, how many places must I edit?" — the answer must be one.

## Orthogonality

Unrelated things should not affect each other; a change in one component shouldn't require changes in another. Test: "If I dramatically change the requirements behind this module, how many other modules are affected?" Design decoupled: don't chain calls through several objects to reach data (Law of Demeter — talk to friends, not strangers); don't rely on global state; prefer passing what's needed.

## Reversibility & good-enough software

There are no final decisions — hide the ones likely to change (DB, framework, wire format) behind a boundary you control. But don't gold-plate: make quality a requirements question. Ship the thing that solves today's problem cleanly; resist both under-engineering (broken windows) and over-engineering (speculative flexibility).

**Broken windows:** one tolerated hack legitimizes the next. Fix small rot when you see it, or at minimum mark it (`TODO`, ticket) so it reads as *known* debt, not accepted style.

## Crash early, assert what can't happen

A dead program does less damage than a crippled one. Fail fast at the point of error instead of propagating corrupt state. Use assertions for "this can never happen"; validate at trust boundaries (user input, network, external APIs) — and only there. Don't code defensively against your own internal invariants.

## Naming (Clean Code, tempered)

- Names carry intent: `elapsedDays`, not `d`; `isEligibleForDiscount()`, not `check()`.
- Length proportional to scope: `i` is fine in a 3-line loop; a module-level export earns a full sentence-like name.
- No disinformation (a `list` that's a map), no noise words (`Data`, `Info`, `Manager`, `Helper`, `Util` — if the suffix could attach to anything, the name says nothing).
- One word per concept per codebase: don't mix `fetch`/`retrieve`/`get` for the same idea.
- Searchable: constants get names; magic numbers don't survive review.

## Functions (Clean Code, tempered by Ousterhout)

- A function does one thing at one level of abstraction; its body reads as a sequence of same-altitude steps.
- Few arguments; no boolean flag arguments (a flag = two functions fused — split them or pass an enum/options object at a real boundary).
- No hidden side effects: a function named as a query must not mutate (command–query separation).
- **Tempering:** "small functions" is a means, not a goal. Extract when the piece has a *better name than its code* and hides something; do NOT shred logic into 3-line fragments that force readers to hop files to follow one thought. A 40-line function with one clear job beats eight 5-line functions with entangled state. (See module-design.md: shallow modules, classitis.)

## Error handling

- Exceptions for exceptional paths, not control flow. Don't return null when you can return an empty collection or a Result the caller must inspect.
- Don't swallow: an empty catch block is a bug. Handle, translate to the caller's vocabulary, or let it propagate — never log-and-continue with corrupt state.
- Wrap third-party APIs at the boundary so their exception/return-style quirks don't leak through the codebase.

## Comments

Prefer making the code express it (rename, extract, type). Keep comments that carry what code can't: *why* (tradeoffs, links to issues/specs, chosen-over alternatives), warnings of consequences, regexes/formats, public interface contracts. Delete: restatements of the code, journal/changelog comments (git does that), commented-out code (delete it; git remembers).

## Tests as design pressure

Hard-to-test code is badly designed code — the test is the first "second caller" your interface meets. If the setup for a test is painful, the module has too many dependencies or hidden state; fix the design, don't build a heroic fixture. Tests pin behavior so refactoring is safe (see refactoring.md) — fast, deterministic, one assertion-of-concept per test.
