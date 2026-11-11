# Module & System Design (Ousterhout, *A Philosophy of Software Design*)

Complexity is anything that makes software hard to understand or modify. It shows up as three symptoms: **change amplification** (one change touches many places), **cognitive load** (how much you must know to change it), and **unknown unknowns** (it's not obvious what must change). Complexity is incremental — it accumulates from many small "this one hack is fine" decisions, and it's far easier to add than remove.

## Deep vs shallow modules

A module = interface + implementation. **Deep module**: simple interface, powerful implementation (Unix file I/O: 5 calls hide buffering, caching, devices, permissions). **Shallow module**: interface nearly as complex as what it hides — it adds interface area without hiding anything.

Signs of shallow modules:
- **Pass-through methods**: a method that does little but call another method with a similar signature. The class adds no abstraction — remove the layer or give it real responsibility.
- A class you can't describe without describing its internals.
- Wrappers that exist "for structure" or "so callers don't instantiate X directly."

Classitis: "many small classes" is not a virtue. Interfaces are a cost paid by every reader; implementations are a cost paid once. Prefer fewer, deeper modules.

## Information hiding & leakage

Each module should encapsulate design decisions (a file format, an algorithm, a protocol) so they can change without touching other modules.

- **Information leakage**: the same knowledge (a format, an encoding, an ordering) appears in multiple modules. The smell: "if we changed X, how many files would know?" If two modules both understand a format, merge that knowledge into one place.
- **Temporal decomposition**: structuring code around *when* things happen (read-then-parse-then-write as three classes) instead of around knowledge. Order of execution leaks into structure; the same knowledge ends up split across steps. Structure around *what each module knows*, not when it runs.
- Avoid exposing internal state or requiring callers to know call order (`init()` before `use()` is leaked internal state — see "define errors out of existence").

## Somewhat general-purpose

Make a module's interface *somewhat* more general than today's single caller needs — general enough that the interface doesn't encode the caller's specifics, no more. Test: "what is the simplest interface that covers all current needs?" and "would this method be awkward for a second, different caller?" Special-purpose code belongs at the edges (top layer); lower layers should be general.

## Pull complexity downward

It is better for a module to have a complex implementation and a simple interface than the reverse. Don't punt complexity to callers via config parameters, flags, or thrown exceptions when the module could handle it internally with a sensible default. Every configuration parameter is a decision you failed to make.

## Define errors out of existence

Exceptions are interface complexity. Best: design the API so error cases can't occur (deleting a nonexistent file = success; `substring` out of range = clamp). Next best: handle internally (retry, mask). Only surface exceptions the caller can genuinely act on. Aggregate error handling in one place rather than scattering try/catch.

## Different layer, different abstraction

Adjacent layers with similar interfaces signal a missing or fake abstraction (decorator piles, pass-through variables threaded through many signatures). If a layer restates the layer below, delete it.

## Strategic vs tactical programming

Tactical: fastest path to working code; each shortcut adds a little complexity; the sum is a big ball of mud. Strategic: working code isn't the goal — a great *design* that also works is. Invest 10–20% continually: fix the design when a change is awkward, don't code around it. "Tactical tornado" output looks fast and costs the team more than it saves.

## Comments (Ousterhout, contra folklore)

"Good code documents itself" is a myth at the interface level. Comments should capture what the code *cannot*: interface contracts (what, units, invariants, edge behavior — written *before* implementation as a design tool) and the *why* of non-obvious implementation. Never restate the code. If an interface needs a long comment to explain, the interface is too complex — redesign, don't document harder.

## Design it twice

For any non-trivial module, sketch two or three radically different designs and compare (interface simplicity, generality, performance). The first design is rarely the best; the comparison is cheap and the winner is usually a hybrid.
