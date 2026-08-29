document version: 1.0

# How to write Java 8 code?

Please read [generic-coding-rules-and-best-practices-v1.md](generic-coding-rules-and-best-practices-v1.md)!
We need to keep all stadards and best practices described there.

On the top of them please also keep the following!

The generic rules (workflow, comments, BDD tests, logging levels, helpers) do **not** change just because the language is Java. This file exists mainly because **Java 8 is a hard language/API ceiling**: agents otherwise emit Java 9–21 features that will not compile. Design-wise, Java 8 vs later Java is close; the delta is syntax and JDK APIs, not a different coding philosophy.

## Language baseline (Java 8)

Write code that compiles with **source/target 1.8**. Do not use language features or JDK APIs introduced after Java 8 — even if they look cleaner.

If you are unsure whether something is Java 8, assume it is not and check. Match the project's existing compiler settings; do not bump them.

**Do not use** (common agent defaults that are *not* Java 8):

- `var` (local-variable type inference)
- `List.of` / `Set.of` / `Map.of` / `Map.ofEntries`
- `stream.toList()` — use `.collect(Collectors.toList())`
- records, text blocks (`"""`), sealed classes, pattern matching
- switch expressions / arrow `switch`
- `module-info.java` / JPMS
- private methods on interfaces (default methods **are** Java 8 and OK)
- `Optional.isEmpty()` — use `!optional.isPresent()`
- `String.isBlank` / `strip` / `repeat` / `lines`
- `Files.readString` / `writeString`, `InputStream.transferTo`, `Predicate.not`

**Prefer Java 8 idioms** over pre-8 leftovers in *new* code:

- lambdas and method references instead of one-off anonymous classes
- `java.time` (`Instant`, `LocalDate`, `ZonedDateTime`, …) instead of `Date` / `Calendar` — convert at the boundary if an old API still needs `Date`
- `Optional` as a **return type** for “value may be absent” — not as a field, not as a method parameter, not to wrap collections (empty collection is better)
- diamond operator: `new ArrayList<>()`
- `Arrays.asList(...)` or an existing Guava immutable type if the project already depends on Guava — do not add Guava (or Lombok, or anything else) just to get factory methods

Streams: use them when they make the code clearer. A simple `for` loop is better than a long, side-effecting stream chain. Do not use `parallelStream()` unless there is a proven need.

Do not introduce Lombok, records-via-library, or other “make it look like modern Java” shortcuts unless the repository already uses them.

## Comments (Javadoc)

Keep the generic comment rules. In Java that means Javadoc on types and methods (including package-private / private helpers).

- Do **not** start the comment with the type or method name.
  - Bad: `/** ensureMap allocates the backing map if it is missing. */`
  - Good: `/** Allocates the backing map if it is missing. */`
- `@param` / `@return` / `@throws` only when they add information that names and types do not already make obvious.
- `@deprecated` on the Javadoc plus `@Deprecated` on the symbol — and point to the replacement (see generic deprecation rules).

## Unit test code

Keep the generic BDD structure (Feature → Scenario → `---- GIVEN` / `---- WHEN` / `---- THEN`).

Keytiles Java is **JUnit 4** (`org.junit.Test`, `org.junit.Assert`). Stay on that. Do **not** introduce JUnit 5, `@Nested`, `org.junit.jupiter`, or Hamcrest/AssertJ unless the file you are editing already uses them.

JUnit 4 has no `t.Run` / `@Nested`. The house substitute is **numbered Scenario blocks inside one `@Test` method**.

- **Feature** → one `@Test` method (this is the Feature-level test from the generic rules — same role as Go `func Test_…`). Name it after the area / behavior (`userGETByAnotherUserTests`, `fieldValueInheritanceTest`). A test *class* is just a container for related Features of one production type.
- **Feature-level comment (required):** every `@Test` method must have a short comment stating what it covers (Javadoc above the method, or a brief `//` at the start of the body). This is **mandatory** even when Scenario blocks exist inside — do not skip it because structure is already present.
  - **Good:** `// If a field is NULL in the updated resource, the value is inherited from the current resource.`
  - **Bad:** no comment; or `// fieldValueInheritanceTest tests field value inheritance` (repeats the method name).
- Class-level Javadoc is optional. Add it when the whole class needs orientation (wiki link, algorithm overview) — it does **not** replace the per-`@Test` Feature comment.
- **Scenario** → a numbered block *inside* the `@Test` method. Match the banner style already used in that test file (hash banners or a `/* Scenario … */` block). `"Scenario 1"` is not a behavior name, so add a short line under the header saying what the scenario is (who acts, what is special). Do **not** add a comment that only repeats `"Scenario 1"`.
  ```
  // #############################################
  // Scenario 1
  // #############################################
  // Another normal user is querying the resource
  ```
- Keep `---- GIVEN` / `---- WHEN` / `---- THEN` inside each Scenario. A single tiny case (or several tiny WHEN/THEN pairs that share setup) does not need Scenario banners — use judgment.
- If a Scenario is large and standalone (e.g. wiki-numbered cases), it **may** be its own `@Test` method (`buildQueryPlanForPeriod_Scenario_1`) so it can be run in isolation. That is the exception, not the default.
- Assertions: `org.junit.Assert` with JUnit 4 argument order `(message, expected, actual)`.
- Expected exceptions: capture with try/catch into a variable, then assert type / reason / message in THEN. Do **not** use `@Test(expected = …)` (cannot assert details) and do **not** switch to JUnit 5 `assertThrows`.
- Mocks: Mockito, the same way neighbouring tests in the module already do. Reuse existing test helpers / builders; do not invent a parallel harness.
- Layout: `src/test/java` mirroring `src/main/java` packages. Same-package tests may use package-private members — prefer that over widening visibility.

### Unit test checklist (before finishing)

- [ ] Each `@Test` method has a short comment (what area/behavior — not repeating the method name)
- [ ] Multiple behaviors use numbered Scenario blocks (or a dedicated `@Test` if the scenario is large/standalone) with `---- GIVEN` / `---- WHEN` / `---- THEN`
- [ ] Scenario headers have a behavior line when the number alone is not enough
- [ ] Generic unit-test rules (Feature / Scenario / steps) are satisfied


## Logging

Keep generic logging rules and on top of that keep these too:

- Use the project's existing facade (typically SLF4J). Logger identity = the class: `LoggerFactory.getLogger(TheClass.class)` (or the equivalent already used in the repo).
- Do not use `System.out` / `System.err` / `printStackTrace()` as logging.
- Prefer **parameterized** messages so argument formatting is skipped when the level is off (same idea as not eagerly building strings for logs):
  - **Good:** `log.debug("started (size={})", size)`
  - **Bad:** `log.debug("started (size=" + size + ")")`
- Prefer lifecycle-oriented DEBUG logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- For shared or static helper code, pass the logger (or a logger name) through the method contract early if the surrounding code already does that, even if logging is minimal at first.
