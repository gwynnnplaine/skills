# Vocabulary

Shared engineering idioms for **fe-indie-next**. Use these terms exactly when designing, reviewing, and explaining code — don't substitute "component," "service," "API," or "boundary." Consistent language is the whole point.

This is the single source of truth for the terms below. Skills that need them link here rather than redefining them: `codebase-design`, `review`, `improve-codebase-architecture`, `type-design`, `type-review`.

Stack-specific note: typed failures use `better-result` (`Result<T, E>`, `TaggedError`, `matchError`); runtime parsing uses zod 4; secrets never enter errors/logs/Sentry. See [AGENTS.md](../../AGENTS.md), [result-errors guide](../guides/result-errors.md), [typescript guide](../guides/typescript.md).

## Architecture

**Module** — anything with an interface and an implementation. Deliberately scale-agnostic: a function, class, package, or tier-spanning slice. _Avoid_: unit, component, service.

**Interface** — everything a caller must know to use the module correctly: the type signature, but also invariants, ordering constraints, error modes, required configuration, and performance characteristics. _Avoid_: API, signature (too narrow — they refer only to the type-level surface).

**Implementation** — what's inside a module, its body of code. Distinct from **Adapter**: a thing can be a small adapter with a large implementation (a `fetch`-backed API client) or a large adapter with a small implementation (an in-memory fake). Reach for "adapter" when the seam is the topic; "implementation" otherwise.

**Depth** — leverage at the interface: the amount of behaviour a caller (or test) can exercise per unit of interface they have to learn. A module is **deep** when a large amount of behaviour sits behind a small interface, **shallow** when the interface is nearly as complex as the implementation.

**Seam** _(Michael Feathers)_ — a place where you can alter behaviour without editing in that place; the *location* at which a module's interface lives. Where to put the seam is its own design decision, distinct from what goes behind it. _Avoid_: boundary (overloaded with DDD's bounded context).

**Adapter** — a concrete thing that satisfies an interface at a seam. Describes *role* (what slot it fills), not substance (what's inside).

**Leverage** — what callers get from depth: more capability per unit of interface they learn. One implementation pays back across N call sites and M tests.

**Locality** — what maintainers get from depth: change, bugs, knowledge, and verification concentrate in one place rather than spreading across callers. Fix once, fixed everywhere.

### Module relationships

- A **Module** has exactly one **Interface** (the surface it presents to callers and tests).
- **Depth** is a property of a **Module**, measured against its **Interface**.
- A **Seam** is where a **Module**'s **Interface** lives.
- An **Adapter** sits at a **Seam** and satisfies the **Interface**.
- **Depth** produces **Leverage** for callers and **Locality** for maintainers.

**Adapter Reuse Audit** — the search before creating a new adapter/service: reuse an existing one through a narrow dependency type if it already provides the behaviour; extend it if the new operation is cohesive and changes for the same reason; create new only when reuse/extension would force bad coupling. One adapter usually means a hypothetical seam; two adapters means a real one.

**Narrow Dependency Interface** — a dependency type stating only the behaviour a module actually uses, named for the consuming capability (e.g. `UsersForPasswordReset`). Wider concrete adapters satisfy it structurally. _Avoid_: depending on a whole repository for one operation, one-method-interface confetti, ad-hoc `deps` bags.

## Failure & observability

**Expected Failure** — a normal-operation failure that belongs in the return type: domain, parsing, authorization, integration, I/O, persistence. In this repo it travels through `better-result` as `Result<T, E>`, where `E` is a `TaggedError` variant. _Avoid_ calling it a recoverable exception, and _avoid_ throwing it.

**Unrecoverable Defect** — a programmer error or catastrophic condition where continuing is invalid: a violated invariant, an impossible branch, startup misconfiguration, or a temporary unimplemented path. Throwing is acceptable here.

**Not Found Failure** — the typed Expected Failure for a *required* get/find/read that finds nothing. Model it as a `TaggedError` variant in the operation's error union, not as a silent `null`. Optional lookup semantics (where absence is normal and expected) should be explicit in the operation's name/contract instead.

**Precise Error Union** — the explicit, narrow set of Expected Failures a caller can handle semantically, discriminated with `matchError`. Keep unions tight at module seams; broad app-level unions belong near entrypoints, orchestration, rendering, and logging.

**Redacted Value** — a secret wrapped so it can't be accidentally logged, inspected, or serialized. Wrap at the seam, unwrap only where the raw value is genuinely needed. Corollary: secrets never enter error context, logs, traces, or Sentry payloads; report failures through the established Sentry reporters (`reportFetchFailure`, `reportZodValidationFailure`) with safe fields only.

**Unknown Catch Cause** — a caught thrown value treated as `unknown` until classified; JavaScript can throw anything. Adapters catching exception-style dependencies classify `unknown`, preserve useful causes, and translate into the local typed failure contract.

## Parsing & data modeling

**Parse, Don't Validate** — convert unknown or less-structured input into a refined/domain value at the earliest seam, then pass that value inward. Core/application code receives precise types, never `unknown`, `any`, `Record<string, unknown>`, or raw DTOs.

**Serialized Data Trust Cast** — a direct cast from decoded serialized data to a domain type, e.g. `JSON.parse(text) as CreateUserInput`. Banned: a parser or smart constructor must establish the invariant, and the refined value must be the one used. (See the repo's `as any`/`as unknown` ban.)

**Smart Constructor** — the single sanctioned way to build a refined value. Naming convention: `parseX` for untrusted/less-structured input, `makeX`/`createX` for construction from already-typed pieces, `isX` only for predicates/type-guards. _Avoid_ `validateX` when the function actually returns a refined value.

**Branded Type** — a nominal type distinguishing a parsed domain value from its primitive representation, constructed only through a parser/smart constructor.

**Value Class** — an immutable domain value class created only through parsers/smart constructors, with cohesive methods, no hidden I/O or dependencies, and explicit projections when crossing seams.

**Correct by Construction** — APIs and types that make invalid states impossible (or hard) to construct rather than relying on caller memory. See [Make invalid states unrepresentable](../guides/typescript.md).

**State Machine** — a tagged-union (or value-class) lifecycle model where each state carries only its legal fields and transitions. Prefer over independent booleans/nullables for mutually exclusive lifecycle states.

**Boolean Blindness** — loss of meaning from raw booleans used for modes, policies, or lifecycle distinctions. Reach for a discriminated union or named type instead.

**Protocol Projection** — a named conversion from a domain/application value to an HTTP/RPC/public DTO, owned by the protocol adapter (`toPublicJson`). Domain modules do not own protocol DTO shapes.

**Mutating Parser** — a parser for a mutating command/request body. It rejects unknown fields (zod `.strict()`) unless the external contract is explicitly extensible, so silent extra data can't slip through a write path.
