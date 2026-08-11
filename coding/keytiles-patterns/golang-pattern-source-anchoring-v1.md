document version: 1.2

# Source Anchoring pattern

This document defines the **Source Anchoring** pattern used at Keytiles.

When someone says “source anchoring pattern” (or “Source Anchoring”), this is what they mean: every log, fault, and call-stack entry is anchored to a clear origin via `methodName` and (for object methods) `receiver.fullyQualifiedName`, where that field is initialized from a package-level `FQN_<TypeName>` constant.

## Prerequisites — Codebase Naming

Source Anchoring assumes **[Codebase Naming](golang-pattern-codebase-naming-v1.md)** is already in place: a root `CODEBASE_NAME` and a per-package `PACKAGE_NAME`.

If those constants (or files) are missing, establish them using Codebase Naming **first**, then apply this pattern. Do not invent ad-hoc origin strings here.

Type-level origins build on that hierarchy:

```
CODEBASE_NAME                         // Codebase Naming
  └── PACKAGE_NAME                    // Codebase Naming
        ├── FQN_<TypeName>            // this pattern (= PACKAGE_NAME + ".<TypeName>", package-level const)
        └── receiver.fullyQualifiedName // set from FQN_<TypeName> in constructor
```

## Scope / prerequisites

- **`methodName` + log messages** — apply wherever Keytiles-style logging is used.
- **`WithSource` / `AddCallerToCallStack`** — apply only in codebases that use `kt_errors.Fault` from [lib-errorhandling-golang](https://github.com/keytiles/lib-errorhandling-golang) (`NewFaultBuilder`, etc.). If that library is not imported, still apply method naming and log anchoring; skip the Fault-builder parts.
- **`PACKAGE_NAME` / `CODEBASE_NAME`** — required via Codebase Naming (see above).

## methodName

At the start of each function/method that logs or builds faults, declare:

```go
methodName := "Foo()"
```

## FQN_<TypeName> constants

For each type whose methods log or build faults, declare a **package-level constant** in the same file as the type:

```go
const FQN_RelayEmailSender = PACKAGE_NAME + ".RelayEmailSender"
```

**Naming:** `FQN_<TypeName>` where `<TypeName>` is the Go type name (PascalCase), e.g. `FQN_RelayEmailSender`, `FQN_EmailMessageBuilder`.

**Rules:**

- One const per anchored type.
- Expression must be a **constant expression**: `PACKAGE_NAME + ".<TypeName>"` — both parts compile-time known.
- Keep `fullyQualifiedName string` on the struct for object methods.
- Do **not** build it in the constructor with a local `name := "..."` variable and string concatenation — that is a runtime concat and allocates on every constructor call.
- Initialize `fullyQualifiedName` from the constant in the constructor: `fullyQualifiedName: FQN_<TypeName>`.
- Use `receiver.fullyQualifiedName` in object methods (`WithSource`, `AddCallerToCallStack`) so methods are decoupled from package-level symbols.
- Reuse `FQN_<TypeName>` in constructors (e.g., for `kt_logging.GetLogger` and `fullyQualifiedName` assignment).

Go constant-folds `PACKAGE_NAME + ".TypeName"` at compile time; the resulting string lives once in the binary’s read-only data. This is preferred over constructor-time concatenation.

## Fault WithSource

- **Package-level function** (no receiver): `WithSource(PACKAGE_NAME, methodName)`
- **Object method** (has receiver): `WithSource(receiver.fullyQualifiedName, methodName)` — the field must come from `FQN_<TypeName>`

```go
// package-level
fault = kt_errors.NewFaultBuilder(...).
    WithSource(PACKAGE_NAME, methodName).
    // ...

// object method
fault = kt_errors.NewFaultBuilder(...).
    WithSource(instance.fullyQualifiedName, methodName).
    // ...
```

Do **not** use the old multi-segment form like `WithSource(PACKAGE_NAME, "TypeName", "Foo()")`.

`AddCallerToCallStack(...)` follows the same split: package-level uses `(PACKAGE_NAME, methodName)`; object methods use `(receiver.fullyQualifiedName, methodName)`.

Constructors that are package-level functions still use `PACKAGE_NAME` for their own `WithSource` / `AddCallerToCallStack` calls. In constructors, set `fullyQualifiedName: FQN_<TypeName>`.

## Logger

In constructors that create a logger for the type, pass the same package const:

```go
logger: kt_logging.GetLogger(FQN_RelayEmailSender),
```

Do **not** use ad-hoc concatenation such as `kt_logging.GetLogger(PACKAGE_NAME + "." + name)` or hardcoded prefixes like `kt_logging.GetLogger("keytiles.kt_email." + name)`.

## Examples

### Type with constructor and logger

```go
const FQN_RelayEmailSender = PACKAGE_NAME + ".RelayEmailSender"

type RelayEmailSender struct {
    logger *kt_logging.Logger
    name   string // keep if used for metrics etc.; not for source anchoring
    fullyQualifiedName string
    // ...
}

func NewRelayEmailSender(cfg EmailDeliveryConfig) *RelayEmailSender {
    return &RelayEmailSender{
        logger: kt_logging.GetLogger(FQN_RelayEmailSender),
        name:   "RelayEmailSender",
        fullyQualifiedName: FQN_RelayEmailSender,
        // ...
    }
}

func (sender *RelayEmailSender) Send(ecntx *kt_tracing.ExecutionContext, msg EmailMessage) kt_errors.Fault {
    methodName := "Send()"
    // ...
    fault = kt_errors.NewFaultBuilder(kt_errors.ValidationFault).
        WithSource(sender.fullyQualifiedName, methodName).
        WithMessageTemplate("...").
        Build()
    fault.AddCallerToCallStack(sender.fullyQualifiedName, methodName)
    // ...
}
```

### Type without logger in constructor

```go
const FQN_EmailMessageBuilder = PACKAGE_NAME + ".EmailMessageBuilder"

type EmailMessageBuilder struct {
    fullyQualifiedName string
    // ...
}

func NewEmailMessageBuilder() *EmailMessageBuilder {
    return &EmailMessageBuilder{
        fullyQualifiedName: FQN_EmailMessageBuilder,
    }
}

func (builder *EmailMessageBuilder) Build() (EmailMessage, kt_errors.Fault) {
    methodName := "Build()"
    return EmailMessage{}, kt_errors.NewFaultBuilder(kt_errors.ValidationFault).
        WithSource(builder.fullyQualifiedName, methodName).
        WithMessageTemplate("...").
        Build()
}
```

## Upgrading earlier applications

When you find code using constructor-time concat (`name := "..."`; `PACKAGE_NAME + "." + name`), upgrade it:

1. Add `const FQN_<TypeName> = PACKAGE_NAME + ".<TypeName>"` at package level in the type’s file.
2. Keep/add `fullyQualifiedName string` on the struct.
3. Set `fullyQualifiedName: FQN_<TypeName>` in the constructor.
4. Replace `kt_logging.GetLogger(PACKAGE_NAME + "." + name)` or other ad-hoc concat → `kt_logging.GetLogger(FQN_<TypeName>)`.
5. Ensure methods use `receiver.fullyQualifiedName` in `WithSource` and `AddCallerToCallStack`.
6. Remove constructor locals/concat code used to build the FQN dynamically.
7. Rename legacy variants (e.g. `fullyQualifiedName_emailMessageBuilder`) → `FQN_EmailMessageBuilder` for constants.

**Anti-patterns to remove:**

```go
// BAD — runtime concat in constructor
name := "RelayEmailSender"
fullyQualifiedName := PACKAGE_NAME + "." + name

// BAD — hardcoded prefix + runtime concat
kt_logging.GetLogger("keytiles.kt_email." + name)

// GOOD — constant plus struct field initialized from it
const FQN_RelayEmailSender = PACKAGE_NAME + ".RelayEmailSender"
fullyQualifiedName: FQN_RelayEmailSender
```

## Log messages: methodName first

Every log in a method that has `methodName` should include it. If the original message had no method name, **add** it. Put `methodName` at the **beginning** as `"%s: ..."`.

```go
// BAD — methodName missing entirely (must add it)
.Info("creating instance, creating Subscription...")

// BAD — methodName at the end
.Warn("not running - skipping %s", methodName)

// GOOD — methodName added at the beginning
.Info("%s: creating instance, creating Subscription...", methodName)

// GOOD — methodName moved to the beginning
.Warn("%s: not running - skipping", methodName)

// GOOD — lifecycle / success
.Info("%s: suspended!", methodName)

// GOOD — failure defer (methodName still leads)
.Error("%s failed: %s", methodName, kt_utils.VarPrinter{TheVar: fault})
```
