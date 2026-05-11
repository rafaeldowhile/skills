---
name: engineering-principles
description: Apply core software-engineering principles whenever writing, reviewing, or refactoring code. Study the code first (Chesterton's Fence), pick one canonical source per fact (Single Source of Truth), keep code that operates on the same data close together (Locality of Behavior), drop speculative complexity (YAGNI), favor simple over clever (KISS), narrow types at boundaries (Make Illegal States Unrepresentable + Parse Don't Validate), leave the area cleaner than you found it (Boy Scout Rule). Use on every code task — from writing new modules to consolidating overengineered flows. Triggers: any code-writing or code-review work, especially when the user mentions "overengineered", "consistent code", "one source of truth", "let's clean this up", or shows signs of fallback chains, merge spreads, `any` casts, or scattered duplicated lookups.
---

# Engineering Principles

A reference for **how to write and refactor code**. The skill is the principles + the discipline of applying them every time, not just during big refactors.

## Mindset

You don't strip first. You **own** the code by understanding it. The decision to keep / consolidate / simplify / strip / rewrite follows the audit, not the other way around.

The verb is "own" via understanding. Apply principles deliberately every time you touch code.

---

## Core Principles

Cited by their well-known names so you and the user share vocabulary.

### 1. Chesterton's Fence
**Don't remove what you don't understand.**
Before changing or deleting code, find out why it exists. The "weird" code usually has a reason — sometimes legitimate (an edge case, a workaround for a real bug), sometimes legacy. You can't tell without reading it. If a layer's purpose is unclear, find the commit/PR that introduced it.

### 2. Single Source of Truth (SSOT)
**Each fact lives in one canonical location.**
Multiple producers of the same fact create drift. Pick the canonical one; the rest become candidates for consolidation. After landing this, you should be able to point at one place and say "this is where X is defined."

### 3. Locality of Behavior
**Code that operates on the same data should live close together.**
Reads of the same fact scattered across N files = anti-pattern. One typed helper, called everywhere, is better. When reading a feature, you shouldn't have to hop through 5 files to understand what it does.

### 4. YAGNI (You Aren't Gonna Need It)
**Remove speculative complexity.**
Fallback chains "in case X" for X that never happened. Generic merge utilities used in exactly one place. Optionality nobody exercises. Hooks for futures that haven't arrived. If it's not used today, it's not paid for today.

### 5. KISS (Keep It Simple, Stupid)
**Favor simple over clever.**
Merge spreads, precedence chains, type gymnastics, and abstraction towers are often clever. Clever costs reading time forever. Simple costs writing time once. Choose simple unless complexity is genuinely required by the domain.

### 6. Make Illegal States Unrepresentable (Yaron Minsky) + Parse, Don't Validate (Alexis King)
**Narrow types at boundaries; trust within.**
Type the entry of a module narrowly so wrong combinations can't compile. Inside the module, don't re-validate — the type carries the proof. `any` at module boundaries is the inverse of this principle; replace with a narrow interface.

### 7. Boy Scout Rule
**Leave the area cleaner than you found it.**
Touched a file with stale comments, dead imports, or a small adjacent mess? Fix it in the same diff. Don't expand scope into a parallel refactor, but don't ignore obvious decay either.

---

## Supporting Principles

These reinforce the core seven and apply selectively:

- **Separation of Concerns** — distinct responsibilities live in distinct modules; don't entangle them.
- **Fail Fast** — reject invalid state at the boundary, not deep in the call chain.
- **Simple Made Easy (Rich Hickey)** — "simple" (one fold, decomplected) is not the same as "easy" (familiar). Prefer simple.
- **Pit of Success (Brad Abrams)** — make the right thing the easy thing; the easy path should be the safe path.
- **Type-Driven Development** — let the type system shape the design; if the types are awkward, the design is awkward.
- **Refactoring discipline (Fowler)** — code is a living thing; small, continuous improvements beat big-bang rewrites.

---

## Design Generators

The core principles tell you what "good" looks like. These tell you how to **find** the good design — they generate the option space so you don't anchor on the first idea.

### Enumerate before choosing
Before picking, list the design axes — what concerns this unit bundles, who owns each, how they compose. Each axis is a potential split. If you have ≤ 2 options on the table, you probably haven't generated the full space; apply Separation of Concerns as a *generator* and regenerate.

### Question the contract, not just the location
A function's return type advertises its responsibility. If you can't name that responsibility in one phrase without "and", it's bundling jobs — split it. "Where to put the code" is the easy question; "what does it return and what does it own" is the hard one.

### Resist anchoring on the existing shape
Existing APIs are one option, never the default. When refactoring, "what would I design from scratch?" is a peer option you must put on the list. If a proposal preserves the existing return shape unquestioned, flag it.

---

## When to Use This Skill

**Always**, on every code task. The framing is "apply principles deliberately", not "trigger only on refactors".

Specific situations where the wins are most visible:
- New code: design with SSOT + Locality + narrow types from the start
- Refactor: Chesterton's Fence first, then YAGNI + KISS + SSOT
- Code review: check each principle as a lens
- Bug fix: ask whether the bug exists because a principle was violated (it often does)
- User says: "overengineered", "consistent code", "one source of truth", "let's clean this up", "this is messy"
- You see: fallback chains (`a ?? b ?? c ?? d` across unrelated sources), merge spreads (`{ ...x, ...y }`), helpers named `mergeFoo` + `fooFromMerged` + `pickFoo`, `any` casts at module boundaries, scattered duplicated lookups, "for back-compat" comments past their grace period

## When NOT to Strip

The skill is "own", not "strip". Don't strip when:
- The code's purpose isn't yet clear (apply Chesterton's Fence — investigate first)
- Layers are intentional (e.g. user prefs override admin defaults override system defaults — that's deliberate precedence)
- Out of scope for the current task (note it, surface it, don't expand scope unilaterally)

---

## Process

### 1. Study (Chesterton's Fence)
Read the relevant code before judging or proposing changes.
- `grep` writers/producers — where is this data set?
- `grep` readers/consumers — where is it read?
- Note the shape at each boundary
- Read each layer's purpose, even the weird ones — there might be a real reason
- If purpose is unclear, find the introducing commit/PR

### 2. Map the structure
Sketch the topology: `producer → middleman1 → middleman2 → reader` for flows, or list modules + responsibilities for non-flow code. Mark which parts do real work vs ceremony.

### 3. Apply principles as lenses
Ask deliberately:
- **SSOT**: is this fact written in more than one place? Which is canonical?
- **Locality**: are reads of this fact scattered? Can they collapse to one helper?
- **YAGNI**: are there fallbacks/options nobody exercises?
- **KISS**: is there a simpler shape that still meets requirements?
- **Types**: are the boundaries narrow? Any `any` to remove?
- **Boy Scout**: small adjacent messes worth fixing in this diff?

### 4. Restate before executing
Write back to the user in 4-5 sentences:
- What's currently there (briefly)
- What changes
- Where the canonical version lives after
- What gets preserved / consolidated / stripped

**Wait for confirmation.** Pushback at this stage exposes misunderstandings cheaply.

### 5. Plan execution order
Each step must leave the codebase compilable.
- Adding fields: producer first, then readers
- Dropping fields: readers first, then producer
- Swapping shape: introduce new alongside old, switch readers, drop old last

### 6. Execute in the same diff
- Apply SSOT: collapse multi-source readers to one
- Apply Locality: move scattered reads into one typed helper
- Apply YAGNI: drop unused fallbacks, speculative options, dead exports
- Apply narrow types: replace `any` with interfaces at boundaries
- Apply Boy Scout: fix the smaller adjacent messes (stale comments, dead imports)
- Don't expand scope unilaterally — surface adjacent issues but ask before chasing them

### 7. Lint + typecheck as part of the work
- Run typecheck after each meaningful edit
- Fix errors immediately, not after
- Zero new `eslint-disable` / `as any` in the diff
- The diff is incomplete until clean

### 8. Verify end-to-end
- For data flows: add env-gated probes at producer + reader, exercise the real path, confirm data lands at the named location, strip probes
- For UI/feature code: manually verify behavior in the running app
- For pure logic: type-level proof + targeted unit test
- Don't declare done from typecheck alone

---

## Anti-patterns and the Principle They Violate

| Anti-pattern | Violates | Fix direction |
|---|---|---|
| `{ ...src1, ...src2, ...src3 }` | SSOT | Pick one source, drop the others |
| `a ?? b ?? c ?? d` across unrelated sources | SSOT | Same |
| `as any` / `as unknown as X` at module boundaries | Make Illegal States Unrepresentable | Type the boundary with a narrow interface |
| `mergeFoo` + `fooFromMerged` + `pickFoo` exports doing variants of the same lookup | Locality + KISS | Collapse to one typed reader |
| "Legacy path for back-compat" comments past their grace period | YAGNI | Finish the migration or scope it explicitly |
| Reader tries `runtime.context.x` AND `runtime.configurable.x` AND `runtime.config.configurable.x` | SSOT + KISS | Commit to one channel |
| Optional fields nobody sets, fallbacks nobody exercises | YAGNI | Drop |
| Same lookup duplicated in 4 modules | Locality | Extract |
| Stale comments mentioning old architecture | Boy Scout | Strip in the same diff |
| Multi-paragraph JSDoc describing what the code does | KISS | The code should be the documentation |
| Speculative abstraction with one implementation | YAGNI | Inline until there are 3 callers |
| Validation repeated inside a function that takes a typed parameter | Parse Don't Validate | Trust the type |
| Return type bundles a primary value with a formatted error for one surface | Separation of Concerns | Return resolution-only; caller formats |
| Refactor preserves existing return shape without asking whether it should | Anti-Anchoring | Put "from scratch" as a peer option |
| Trade-off is only about file location, not contract | Anti-Anchoring | Decide the contract first |

---

## Communication patterns

- **Before code**: restate the change in 4-5 sentences. Wait for sign-off. Most assumption mismatches surface here.
- **During execution**: announce each step briefly. One sentence per major change. Don't narrate line-by-line.
- **When the user pushes back**: stop. Re-state. The pushback almost always exposes an assumption you didn't surface.
- **At end**: summarize what changed, what got consolidated, what got stripped, what was preserved. Cite file paths.

---

## What "owning" means

- **You can explain every layer's purpose** in one sentence each
- **One producer per fact** (or intentional, documented alternates)
- **One typed reader** — narrow input, narrow output
- **Zero accidental complexity** in the hot path (merge/spread/precedence chains across unrelated sources)
- **Zero `any`** at module boundaries
- **Documented invariant** somewhere durable: "X is set at A, read at B"

## Don't declare done without

- [ ] Typecheck green on every touched file
- [ ] Lint green (no new disables)
- [ ] No new `any` in the diff
- [ ] One canonical reader where SSOT applies
- [ ] No "for back-compat" leftover unless explicitly scoped to a tracked follow-up
- [ ] If user-facing flow: live-tested through the real path, not just types
- [ ] Boy-scout: stale adjacent comments/imports cleaned

---

## Examples where these principles produce wins

- Auth / identity / tenancy propagation across proxy → service → tools
- Feature flag resolution across env → config → runtime override → user override
- Theme / preferences propagation across server → client → component
- Error envelope normalization across N HTTP clients
- Cache key construction across producers and invalidators
- Event payload schemas across publisher and N consumers
- Request context (locale, timezone, A/B variant) across layers
- State machine transitions where state lives in multiple places
- API client design (one client, narrow types, clear error envelope)
- New module design (define the canonical interface first, then the implementation)

Any code where data flows through layers, behavior is duplicated, or types are loose at boundaries — these principles produce wins. Apply them every time.
