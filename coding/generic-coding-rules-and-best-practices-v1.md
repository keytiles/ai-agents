document version: 1.1

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
- Always add a short comment to Feature-level test cases (typically the outer test method/function) about what they are testing.
- Consider adding any inline comments which helps the reader to understand better what the test case does and make reverse engineering of the test case easier.

# Best practices to keep

Points documented here should be heavily considered when crafting code. Not mandatory, but strongly preferred. If we deviate from these please provide the reason why we do so.

## Internal helper methods.

We know clean code is cool and normally improves readability. But if we end up in a state we have 100 internal helper methods and 10 public facing real
methods that's bad because we will drown in number of methods quickly.

Please do NOT introduce internal helpers if:

- The method is just invoked from one place AND relatively short. Then better to keep the code inline and add a comment inline to the code block.

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
