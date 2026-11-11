---
name: maintainable-design
description: Use when writing, extending, refactoring, or reviewing code, or making system-design/architecture decisions — adding a variant of something that exists (new format, case, endpoint), a change that touches many files (shotgun surgery), duplication or copy-paste, else-if/switch chains repeated on the same key, shallow wrapper/Manager classes, init-before-use ordering, formats or rules leaking across modules, god functions, code that's hard to test, or requests to "clean up", "refactor", "improve design".
---

# Maintainable Design

Distilled from Ousterhout (*A Philosophy of Software Design*), Fowler (*Refactoring*), Hunt & Thomas (*The Pragmatic Programmer*), Martin (*Clean Code*).

**The one criterion: does this change make the next change easier or harder?** (ETC — easier to change.) Complexity is incremental — it arrives as reasonable-looking shortcuts, one "just this once" at a time, and it compounds. Working code is not the goal; a simple design that works is.

## Entry points — pick by task

| Task | Path |
|---|---|
| Extend existing code (new variant/format/case/endpoint) | Core loop below |
| Bug fix, hotfix, incident | Smallest safe fix — see Fixing |
| Write a new module | Design rules + design it twice (sketch 2 designs, compare interfaces) |
| "Improve this design" / review | Diagnose → rank → fix — see Reviewing |

## The core loop (extending)

Before extending code, read it as a design: **"what structure would make my change trivial?"** If the honest answer is "copy the existing block and tweak it" — STOP. That is the signal to refactor first.

1. **Make the change easy, then make the easy change.** Preparatory refactoring in tiny behavior-preserving steps; tests after each. Then the feature lands as the small diff the new structure allows.
2. **No tests around the code? Pin behavior first** with a characterization test (capture current output, diff after).
3. **Two hats:** never refactor and change behavior in the same step or commit.
4. **Scope guardrail:** the preparatory refactor is the *smallest* structural change that makes your change trivial — not a redesign. Don't break the module's public surface during feature work unless every caller is in view and updated; otherwise note the deeper refactor in your report instead.

## Deadline pressure — the trap that catches everyone

"Ship today, keep it quick, just match the existing pattern" does NOT authorize copying a flawed pattern. Consistency with a bad pattern spreads the bad pattern. The preparatory refactor is usually minutes, makes YOUR diff smaller, and is in scope precisely because your change touches this code.

| Excuse | Reality |
|---|---|
| "Kept it additive/consistent with existing code" | You duplicated knowledge. The 4th copy is not consistency; it's compounding debt. |
| "Refactor would be a good follow-up" | Follow-ups don't happen. The moment you touch the code IS the follow-up. |
| "The brief said just make it work like X" | The brief defines behavior, not design. Design is your job. |
| "Refactoring is out of scope / too risky today" | Tiny extract-function steps behind a characterization test are lower risk than a divergent 4th copy. |
| "The lead prescribed the quick way / 'like last time'" | They specified outcome and urgency, not design. Centralize if it's minutes of work; if you truly can't, do the ask and name the debt in your report — silence is the only wrong option. |
| "It's still manageable at this size" | Your one-concept edit just touched N files. That IS shotgun surgery, already measured — and N only grows. |

Genuine emergency with zero test coverage and minutes to ship? Duplicate — but say so explicitly in your report and leave a `TODO` naming the refactor. Silent duplication is the failure.

## Fixing (bugs, hotfixes, incidents)

Smallest safe fix at the root cause, verified, shipped. Do NOT restructure, rename, or "improve while in here" — that widens the blast radius exactly when risk tolerance is lowest. Grep for sibling call sites so the fix covers every path, not just the reported one. Then *propose* the refactor you saw, separately, in your report.

## Design rules (the 20% that matters)

- **Deep modules:** simple interface, powerful implementation. A class whose interface is as complex as what it hides adds cost, not structure. Kill pass-through methods and do-nothing wrappers (`Manager`, `Helper`, `Util`, `Wrapper`).
- **One home per piece of knowledge (DRY):** a format, rule, or constant lives in exactly one module. Test: "if this changed, how many files know?" Two modules sharing a secret (a date format, an encoding, an ordering) = information leakage — move the knowledge to one owner. DRY is about knowledge, not text: two rules that coincidentally look alike must NOT be merged.
- **Pull complexity downward:** handle it inside the module with a good default instead of exporting config params, flags, and exceptions to every caller. Every config parameter is a decision you failed to make.
- **Define errors out of existence:** design APIs so the error case can't occur. `init()`-before-`use()` ordering is leaked internal state — construct ready-to-use values; use default parameters over mutable setup.
- **Somewhat general-purpose:** the interface shouldn't encode today's single caller — nor hypothetical future ones. Speculative generality (hooks, params, layers "for later") is the equal-and-opposite smell: delete it.
- **Repeated dispatch** (same else-if/switch key in several places) → one table/map or polymorphism, so a new variant is one entry, not N edits.
- **Names carry intent;** length proportional to scope; no noise suffixes. Extract a function only when its name says more than its body — small functions are a means, not a goal; a 40-line function with one clear job beats eight entangled 5-line hops.

## System-level signals

When the smell spans modules, think architecture, not line edits — see `references/architecture.md`:
- One conceptual change forces edits in many files (**shotgun surgery**) → gather the knowledge into one owner, expose a registry/interface.
- One file changes for many unrelated reasons (**divergent change**) → split along change reasons.
- Volatile decisions (DB, wire format, vendor API) reachable from everywhere → wrap in a boundary you own.
- Layers with near-identical interfaces, pass-through variables threaded through many signatures → a missing or fake abstraction.

## Reviewing / "improve this design"

Diagnose before editing: list concrete smells (`references/refactoring.md` has the smell → refactoring table), rank by **change amplification** ("what edit does this code get next, and how many places does it touch?"), fix the top items in small verified steps. Renames and comment tweaks alone are not a design improvement. Verify behavior preservation the same way as the core loop: characterization test before, diff after.

## References (load on demand)

- `references/refactoring.md` — Fowler: smells → refactorings table, mechanics, when (not) to refactor
- `references/module-design.md` — Ousterhout: deep modules, information hiding, pull complexity down, strategic programming, comments
- `references/architecture.md` — system/architecture: coupling, dependency direction, boundaries, orthogonality, reversibility
- `references/code-craft.md` — Pragmatic Programmer + Clean Code: DRY-as-knowledge, naming, functions, error handling, tests as design pressure

## When NOT to apply

One-off scripts, throwaway spikes, generated code, code that never changes and needs no understanding. Don't refactor speculatively with no upcoming change and no confusion. Never silently widen a narrow request into a redesign — do the asked change, then propose.
