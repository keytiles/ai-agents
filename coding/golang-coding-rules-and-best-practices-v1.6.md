document version: 1.7

# How to write Go code?

Please read [generic-coding-rules-and-best-practices-v1.md](generic-coding-rules-and-best-practices-v1.md)!
We need to keep all stadards and best practices described there.

On the top of them please also keep the following!

## Keytiles patterns

Also read and follow these Keytiles pattern documents (all of them):

- [golang-pattern-codebase-naming-v1.md](keytiles-patterns/golang-pattern-codebase-naming-v1.md) — **Codebase Naming** (`CODEBASE_NAME` / `PACKAGE_NAME`)
- [golang-pattern-source-anchoring-v1.md](keytiles-patterns/golang-pattern-source-anchoring-v1.md) — **Source Anchoring** (`methodName` / `fullyQualifiedName` for logs, faults, call stacks; depends on Codebase Naming)

When the user names a pattern (e.g. “source anchoring pattern”, “codebase naming pattern”), use the matching document above as the definition.

## Functional programming - ExecutionContext

We follow functional programming in many aspects when we write go code.

There is a type defined in module git.keytiles.com/keytiles-golang/lib-common-golang: kt_tracing.ExecutionContext.
This context is typically created on the public barrier (incoming message, incoming sync request) and then we pass it along on the call chain.
It also can encapsulate a standard Golang Context. And carries important information about the business transaction accross the call chain.

It is a good practice to take `ecntx *kt_tracing.ExecutionContext` parameter into all method signatures so method code is aware and can make logs better - see section "Logging" below!

## Unit test code

- We break Golang best practices and do NOT mix test cases/code with real code. Real code is kept under folder `/pkg` where sub folders breaking up code into packages. And we have `/tests` folder where we put test code. Here we keep the same sub folder structure as we have under `/pkg` preferably.
- This way we can not test package private methods or access package private fields. This is accepted. However what we can do - inspired by Java/Google lib @VisibleForTesting annotation - when we really need to test an internal method or internal anything, we can introduce methods with "VisibleForTesting\_" prefix in the code which can wrap internal logic making them visible for test cases.
- Prefer `t.Run("scenario name", ...)` to split **Scenarios** under one `Test_…` **Feature** (see generic unit-test rules). This gives clearer failures and supports `go test -run 'TestFoo/scenario'`. Keep `---- GIVEN` / `---- WHEN` / `---- THEN` inside each `t.Run`. A single tiny scenario does not require `t.Run` — use judgment.
- **Feature-level comment (required):** Every outer `func Test_…(t *testing.T)` must have a short comment immediately above it stating what behavior or area it covers (generic rules: Feature-level test comment). This is **mandatory** even when scenarios use `t.Run` — do not skip it because structure is already present.
  - **Good:** `// Verifies int container-option getters return -1 for nil/missing and cast stored values.`
  - **Bad:** no comment; or `// Test_Container_getIntOptions tests getIntOptions` (repeats the function name).
- Scenario-level (`t.Run`) comments: optional when the scenario name is already clear (see generic rules).

### Unit test checklist (before finishing)

- [ ] Each outer `func Test_…` has a short comment (what area/behavior — not repeating the function name)
- [ ] Multiple behaviors use `t.Run("scenario …")` with `---- GIVEN` / `---- WHEN` / `---- THEN` where helpful
- [ ] Generic unit-test rules (Feature / Scenario / steps) are satisfied

## Logging

Keep generic logging rules and on top of that keep these too:

- Keep logs consistent with existing style: `ecntx.LogWithLabels(logger).<Level>(...)` if `ecntx *kt_tracing.ExecutionContext` is available for the method defined
  in module git.keytiles.com/keytiles-golang/lib-common-golang.
- Use local `methodName := "..."` and include it in log messages.
- Prefer lifecycle-oriented debug logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- For shared helper code, pass logger through the method contract early, even if logging is minimal at first, so future diagnostics are easy to add.
- Be explicit with key fields for large or sensitive payloads.
- Prefer `kt_utils.VarPrinter{TheVar: ...}` in log lines when you want readable dumps **without eager string construction** — the dump is built only when the logger actually formats the `%s` argument (e.g. log level allows emission). Do **not** call `kt_utils.PrintVarS(...)` inline in log argument lists for the same reason.
- `VarPrinter` works well for small/compact structs when it improves readability and maintainability.
- For hiding secrets in a struct printed with `VarPrinter` please note `VarPrinter.Transform` - an optional `func(any) any` applied lazily when formatting. This is perfect in all cases a struct has secrets. Implementing the function stays with the service level. If you spot any suspiocious or known fields in a struct which are or can carry secrets use the Transformer or at least recommend this way to the user.
- Also note that `VarPrinter` is aware of interface `LogStringer` implementors. If a struct has a function `LogString() string` then that function is used during log printing. Know about this possibility and apply or recommend this option of implementing that function to the user.
- For **`[]byte` text payloads** (JSON, plain text, etc.) use lazy conversion via the flag — do **not** convert at the call site:
  - **Good:** `kt_utils.VarPrinter{TheVar: bytes, BytesAsString: true}`
  - **Bad:** `kt_utils.VarPrinter{TheVar: string(bytes)}` — `string(bytes)` runs eagerly before the logger decides to emit the line.
- `BytesAsString` is **opt-in** and only applies when `TheVar` is `[]byte`. For binary payloads where you want litter's native `[]uint8{...}` dump, pass raw `[]byte` without the flag: `kt_utils.VarPrinter{TheVar: bytes}`.
- Be careful with `VarPrinter` on large/deep objects: it can create noisy logs and unnecessary overhead.
- Feature details: `lib-utils-golang` docs `VarPrinting-v1.3.md` (or latest `VarPrinting-v*.md` in that repo).

### Fault logging ownership

`kt_errors.Fault` values that **bubble up unchanged** should not be logged with ERROR (or equivalent) on every hop (DAO → controller → HTTP/messaging).

- **Log once at the public barrier** that turns the Fault into an outward outcome — e.g. HTTP error mapping (`mapControllerError…` / `NewPublicFaultFromAnyError`) or the messaging consumer/producer that finishes the transaction.
- Intermediate layers (controllers, DAOs) should **return** the Fault (optionally `AddCallerToCallStack`) and avoid a second full Fault dump when the barrier will log it.
- If a layer **soft-handles / swallows** a Fault (continues the business transaction), log **at that handling site** (often WARN) — that site is then the barrier for that Fault.
- Metrics are not a substitute for this rule, and not an excuse to re-introduce deferred “log any Fault” helpers; bump counters next to where the condition is raised if needed.

### VarPrinter in logs (lib-utils-golang)

Use `kt_utils.VarPrinter` so formatting work is deferred until the log line is emitted.

| Case | Pattern |
|------|---------|
| Struct / general value | `VarPrinter{TheVar: value}` |
| `[]byte` as readable text | `VarPrinter{TheVar: bytes, BytesAsString: true}` |
| `[]byte` as byte dump | `VarPrinter{TheVar: bytes}` (no flag) |
| Avoid in log lines | `PrintVarS(...)` or `VarPrinter{TheVar: string(bytes)}` |

Requires `lib-utils-golang/v2` **2.1.0+** for `BytesAsString`.
