document version: 1.0

# Source Anchoring pattern

This document defines the **Source Anchoring** pattern used at Keytiles.

When someone says “source anchoring pattern” (or “Source Anchoring”), this is what they mean: every log, fault, and call-stack entry is anchored to a clear origin via `methodName` and (for object methods) `fullyQualifiedName`.

## Scope / prerequisites

- **`methodName` + log messages** — apply wherever Keytiles-style logging is used.
- **`WithSource` / `AddCallerToCallStack`** — apply only in codebases that use `kt_errors.Fault` from [lib-errorhandling-golang](https://github.com/keytiles/lib-errorhandling-golang) (`NewFaultBuilder`, etc.). If that library is not imported, still apply method naming and log anchoring; skip the Fault-builder parts.

## methodName

At the start of each function/method that logs or builds faults, declare:

```go
methodName := "Foo()"
```

## Fault WithSource

- **Package-level function** (no receiver): `WithSource(PACKAGE_NAME, methodName)`
- **Object method** (has receiver with `fullyQualifiedName`): `WithSource(receiver.fullyQualifiedName, methodName)`

`fullyQualifiedName` is set in the constructor as `PACKAGE_NAME + ".TypeName"` and is also used for `kt_logging.GetLogger(...)`.

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
