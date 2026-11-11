# Refactoring (Fowler, *Refactoring* 2nd ed.)

Refactoring = behavior-preserving structural change, applied as a **sequence of tiny steps, each leaving the code working**. Not "rewriting", not "cleanup while also fixing a bug".

## The discipline

1. **Tests first.** A solid, self-checking test suite is the precondition. No tests around the code you're changing? Write characterization tests first (pin current behavior, even bugs).
2. **Two hats.** You're either adding behavior or refactoring — never both in the same step (or commit). Notice when you swap hats.
3. **Small steps, run tests after each.** If a step breaks, revert the step — not a debugging session. Steps feel inefficiently small; that's what makes the compound fast.
4. Refactoring commits separate from behavior commits.

## When to refactor

- **Preparatory (most valuable):** before adding a feature, ask "what structure would make this change trivial?" — *make the change easy, then make the easy change.* If adding format N+1 means copy-pasting format N, refactor to make N+1 a one-liner first.
- **Rule of three:** first time, just do it. Second time, wince and duplicate. Third time, refactor.
- **Comprehension:** when you had to work hard to understand code, encode the understanding into the code (rename, extract) so the next reader doesn't.
- **Opportunistic:** leave code a little better than you found it — within the scope you're already touching.
- **When NOT to:** code that never changes and doesn't need to be understood; when a rewrite is genuinely cheaper; to satisfy aesthetics with no upcoming change. Don't refactor speculatively — refactor in response to a real change or real confusion.

## Smells → refactorings (the ones that matter most)

| Smell | What it looks like | Primary move |
|---|---|---|
| Duplicated code | Same knowledge/logic in 2+ places (identical text not required) | Extract Function; Pull Up Method; Form Template Method |
| Long function | Doing several things; needs comments to navigate; deep nesting | Extract Function (at each comment/blank-line seam); Decompose Conditional; Replace Temp with Query |
| Divergent change | One module edited for many unrelated reasons | Split Phase; Extract Class |
| Shotgun surgery | One change forces edits in many modules | Move Function/Field to gather the knowledge in one place; Inline Class |
| Feature envy | Function talks to another object's data more than its own | Move Function to where the data lives |
| Data clumps | Same 3 params travel together everywhere | Introduce Parameter Object; Extract Class |
| Primitive obsession | Strings/numbers where a type carries the rules (money, phone, range) | Replace Primitive with Object; Replace Type Code with Subclass |
| Repeated switch / else-if chains on the same key | Adding a variant means touching every switch | Replace Conditional with Polymorphism — or a lookup table/map (often simpler) |
| Middle man / lazy element | Class or method that only delegates | Remove Middle Man; Inline Function/Class |
| Speculative generality | Hooks, params, abstract classes "for later" with one user | Collapse Hierarchy; Inline; Remove Dead Code |
| Message chains | `a.getB().getC().doThing()` | Hide Delegate — or leave it; chains are often fine (weigh vs middle-man) |
| Mutable global / shared data | Anyone can write it; effects at a distance | Encapsulate Variable; narrow the scope |
| Long parameter list | Caller must marshal many values | Preserve Whole Object; Parameter Object; Remove Flag Argument |
| Comments as deodorant | Comment explaining a confusing block | Extract Function named for the comment; then keep only *why* comments |
| Loops doing several jobs | One loop filters + transforms + sums | Split Loop; Replace Loop with Pipeline (map/filter/reduce) |

## Key mechanics worth knowing by name

- **Extract Function / Inline Function** — the workhorses, in both directions. Extract when a fragment has a name better than its code; inline when the name adds nothing (indirection isn't free).
- **Rename** — cheapest, highest-value refactoring. If you had to think to understand a name, rename it now.
- **Split Phase** — one function computes then formats then sends → separate sequential phases with an explicit intermediate data structure.
- **Replace Conditional with Polymorphism / lookup map** — for repeated dispatch on the same type code.
- **Encapsulate Variable/Collection** — before moving widely-used data, route access through functions.
- **Separate Query from Modifier** — functions either return a value or have an effect, not both.

## Anti-patterns while refactoring

- Big-bang rewrite disguised as refactoring (branch alive for weeks).
- Refactoring without tests "because it's obviously safe".
- Changing behavior "while I'm in here" (hat violation).
- Extracting so hard the code becomes a maze of 3-line hops — extraction must *hide* something (see module-design.md: shallow modules).
