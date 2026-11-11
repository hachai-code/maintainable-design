# System Design & Architecture (across all four books)

Architecture is the set of decisions that are expensive to reverse. The job is not predicting the future — it's making the future cheap to accommodate: **ETC, easier to change**, applied at module boundaries. There are no final decisions (Pragmatic Programmer: *reversibility*); good systems hide the ones most likely to flip.

## Coupling & cohesion, measured by change

Forget abstract definitions — use change vectors:
- **Cohesion:** things that change together live together. If every "add a variant" edit touches files A, B, C, the variant knowledge is scattered; gather it.
- **Coupling:** if requirements behind module X change dramatically, how many other modules must change? (Orthogonality test.) Unrelated capabilities must not interact: swapping the storage engine shouldn't touch the report renderer.

The two cross-module smells (Fowler):
| Smell | Shape | Move |
|---|---|---|
| **Shotgun surgery** | One conceptual change → edits in many modules | Move Function/Field to gather knowledge under one owner; expose a registry or interface others consume |
| **Divergent change** | One module edited for many unrelated reasons | Split along change reasons (Split Phase, Extract Class/Module) |

## One owner per piece of knowledge

System-scale DRY: every fact — a wire format, an enum of variants, a routing rule, a rounding policy — has exactly one owning module; everything else consumes it through an interface. When N modules each hold a `switch` on the same key, the variant set has N owners: adding a variant is N edits and one forgotten default-case bug. Replace with one table/registry owning the variant knowledge (per-variant data in one place; channels/consumers keep only what is genuinely theirs).

Corollary (Ousterhout: *information leakage*): two modules that both "know" a format share a secret coupling no import graph shows. The comment "this must match X over there" is the tell — replace the comment with a shared function or constant so the compiler enforces what prose only hoped.

## Boundaries you own

Wrap volatile decisions — vendor SDK, database, wire format, framework — behind an adapter whose interface speaks YOUR domain vocabulary, not the vendor's (Clean Code: *boundaries*; Pragmatic: *reversibility*). Rules:
- Third-party types don't leak past the adapter (no vendor exceptions, response shapes, or client handles in core signatures).
- The adapter is where retries, quirks, and error translation live — callers see domain results (pull complexity downward).
- One adapter per decision, not per call site.

Don't wrap stable things (stdlib, the language) — a boundary is for volatility, and every layer must earn its cost.

## Layers & dependency direction

- **Different layer, different abstraction** (Ousterhout). A layer whose interface mirrors the layer below is a fake layer — delete it. Pass-through variables threaded down many signatures signal a missing context object or misplaced responsibility.
- **Special-purpose code at the top, general-purpose at the bottom.** Lower layers shouldn't know about specific callers or features; upper layers shouldn't know about mechanisms.
- **Depend toward stability.** Core domain logic depends on nothing volatile; volatile edges depend on the core. No dependency cycles between modules — a cycle makes two modules one.
- **Law of Demeter:** talk to immediate collaborators, not `a.getB().getC().doThing()` — each link in the chain is a structural assumption that will break.

## Structure around knowledge, not time

Temporal decomposition — one module per execution step (read, then parse, then write) — scatters shared knowledge across steps (file format in both reader and writer). Organize modules around *what each one knows*; the execution order is an implementation detail of the orchestrator. When a computation has genuinely distinct stages, make the pipeline explicit: **Split Phase** with a typed intermediate structure between stages, so each stage can change independently.

## Shared mutable state is system glue

A global (or module-level) mutable value couples every reader and writer at a distance — effects with no visible call path. Encapsulate it behind functions, narrow its scope, pass what's needed explicitly. Configuration counts: a grab-bag settings object every module reads is shotgun surgery waiting to happen.

## Prove the architecture with a tracer bullet

Validate seams by driving one thin end-to-end feature through all layers early (Pragmatic: *tracer bullets*) — real interfaces, real deploy, minimal behavior — then flesh out. A tracer bullet is production code and stays; a **prototype** is a disposable experiment (cardboard, not steel) and must never be promoted to production. Decide which one you're writing before you start.

## Strategy & investment

- **Design it twice** applies to system shape more than anywhere: sketch two radically different decompositions; the comparison is hours, the wrong architecture is months.
- **Strategic over tactical** (Ousterhout): invest 10–20% continually in design as you go; when a change is awkward, fix the structure rather than coding around it. Big-bang rewrites are how tactical debt gets paid at the worst exchange rate.
- **No broken windows** (Pragmatic): one tolerated hack legitimizes the next; fix or explicitly mark rot (`TODO` + ticket) so it reads as known debt, not house style.
- **Good-enough software:** architecture quality is a requirements question. A one-team CRUD app does not need hexagonal ceremony; a system with volatile vendors and many change vectors does. Match investment to the real change pressure.
