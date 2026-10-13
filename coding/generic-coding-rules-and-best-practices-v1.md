document version: 1.4

# Iterative development workflow

Normally we follow an iterative approach when we write code.

1. Understanding the current state of the feature / class / module - existing documentation helps a lot here. Please read [docs-rules-and-best-practices-v1.md](../docs/docs-rules-and-best-practices-v1.md)!
2. We plan the change and describe what and how we want to do. Please read [planning-documents-v1.md](planning-documents-v1.md) to understand how it works!
3. We do the implementation so the coding.
4. We document how the new feature or version of it works. Following same rules as in step #1 according to [docs-rules-and-best-practices-v1.md](../docs/docs-rules-and-best-practices-v1.md)!

# Rules to keep

In general please read the file [karpathy-guidelines.mdc](../3rd-party/karpathy-guidelines.mdc) when we are crafting code together.

On the top of that also keep the following we define in this document!

## Deprecated APIs and symbols

Treat deprecation as a hard constraint when writing or changing code.

- **Do not use** symbols marked `Deprecated` / `@deprecated` / similar (constants, types, functions, methods, packages).
- Before using a constant, type, or API from a dependency (especially error codes, builders, helpers), **check its godoc / docs for a deprecation note**.
- If deprecated, use the **replacement named in the deprecation comment**. Prefer the correctly spelled / current symbol even when an old alias still compiles.
- If you find deprecated usage in **existing** code you are touching: warn the user and prefer migrating that call site in the same change when it is cheap and in scope.
- Do **not** treat “it still compiles / same underlying value” as permission to keep using the deprecated alias.

## Comments

- Always add at least brief comments to classes / structs explaining what they do. Keep it short!
- Always add at least brief comments to methods explaining what they do.
- Also do it for internal helper methods.
- **Feature-level test functions** (e.g. outer `func Test_…` in Go) count as methods here — see [Unit test code](#unit-test-code) (mandatory short comment on what the Feature covers).
- In longer methods provide inline brief comments so reader understands better which code block does what.
- Do not repeat or add the name of the method to method comments. When you are adding comment to a method itself above the function (godoc, javadoc, etc) - totally pointless. We see the method...
  example: if method name is "ensureMap()" then
  - Bad: `// ensureMap allocates...`
  - Good: `// Allocates the backing map if...`

## Unit test code

Please keep BDD pattern (Gherkin language like). Prefer this structure:

- **Feature / area under test** → one test function / test method (with a short comment on what it covers).
- **Scenario** → when the language supports subtests / nested cases, prefer splitting multiple scenarios that way
  (in Go: `t.Run("scenario name", func(t *testing.T) { ... })`). This maps cleanly to Gherkin Scenarios and improves failure output / selective runs.
- **Steps inside a Scenario** → organize with `"---- GIVEN"` / `"---- WHEN"` / `"---- THEN"` comment segments.
  This improves test readability drastically. The `"----"` characters in the comments improve readability for human eye more.
- Scenario names should read like behavior (e.g. `"named types are accepted"`), not like implementation details.
- Do **not** add a Scenario comment that only repeats the scenario name. Prefer a strong name; add a short comment above the Scenario only when the name is not enough (why / non-obvious edge / important constraint).
- A single tiny case does not require a nested scenario wrapper — use judgment; prefer scenarios when a feature has several distinct behaviors.
- Always add a short comment to Feature-level test cases (typically the outer test method/function) about what they are testing. In Go this is the comment above `func Test_…(t *testing.T)` — **required**, not optional when scenarios exist inside.
- Consider adding any inline comments which helps the reader to understand better what the test case does and make reverse engineering of the test case easier.

### Unit test checklist (before finishing)

- [ ] Each Feature-level test (outer test method / `func Test_…` in Go) has a short comment stating what it covers
- [ ] Scenarios have clear behavior-oriented names; `---- GIVEN` / `---- WHEN` / `---- THEN` where helpful
- [ ] No comment that only duplicates a scenario name

# Best practices to keep

Points documented here should be heavily considered when crafting code. Not mandatory, but strongly preferred. If we deviate from these please provide the reason why we do so.

## Internal helper methods / shared helpers

We know clean code is cool and normally improves readability. But if we end up in a state we have 100 internal helper methods and 10 public facing real
methods that's bad because we will drown in number of methods quickly.

Please do NOT introduce an internal or package-level helper if:

- The method is just invoked from **one live (production) call site** AND is relatively short. Prefer inline code with a brief comment on that block.
- The method is a **pass-through / alias**: it only forwards to another function (especially across packages) with no added logic, validation, or policy.
  Example of what to avoid: `func defaultX() T { return helper.DefaultX() }` — call `helper.DefaultX()` at the call sites instead.
- The only reason for the helper is **reuse from tests**. Tests may duplicate a tiny fixture, assert fields directly, or use a `VisibleForTesting_*` (or language equivalent) seam for a real private method — do not invent a shared helper for test convenience.

### Where a helper may live

- **Private in the owning type/package** when the logic belongs to one use-case (one controller, one DAO/parse path, etc.), even if tests need a VisibleForTesting wrapper.
- **Shared package** (`helper`, `model`, …) only when there are **multiple live call sites in different packages or barriers** (e.g. HTTP + persistence), and the helper encodes a real shared rule — not because “we might need it elsewhere”.

A shared helper that has a single live caller misleads readers into thinking the rule is reused across the system.

### When scanning for compliance

Flag and prefer removing:

1. One-line / pass-through wrappers
2. Exported helpers with ≤1 live call site outside tests
3. Helpers used heavily in tests but barely (or never) in production code

## Single ownership of a domain default / policy

When the same default value, normalization, or business policy can be applied in more than one layer (e.g. persistence soft-recovery on missing/corrupt data **and** a controller normalizing an omitted request field), prefer **one clear owner**.

- Put materialization of that default/policy in **one** layer (or one shared domain helper used by that owner).
- Other layers should either **rely on that owner** (e.g. always read through the DAO that normalizes) or have an **explicit, different barrier reason** documented at the call site — not a second silent copy of the same rule “just in case”.
- Duplicating the same “apply default X” in controller + DAO (or similar) without a distinct reason smells: readers cannot tell which layer is authoritative, and the copies drift.

Shared factories for the *value* (e.g. `DefaultFoo()`) are fine; scattering the *decision when to apply it* across layers is what to avoid.

## Logging

- Log levels: never use Critical/Fatal log levels! Valid log levels are:
  - ERROR: business transaction failed, no recovery possible - we raise an exception/error for caller
  - WARN: something went wrong but we have recovery plan, business transaction can continue
  - INFO: Default log level setting in Production. To answer question "why the code did what it did?" INFO level logs should be sufficient to have.
  - DEBUG: Extra information to enable way more insights in terms of what the code did.
- We use hierarchical logging. A Logger created with `GetLogger(<name>)` factory method should be identified with package + class/struct/block name. So we know which code piece
  created the log event.
- We use structured logging. We decorate each log event with meta data. This possibility can be considered while crafting logs.
- Prefer lifecycle-oriented DEBUG logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- For shared or static helper code, pass logger through the method contract early, even if logging is minimal at first, so future diagnostics are easy to add.
