document version: 1.0

# Codebase Naming pattern

This document defines the **Codebase Naming** pattern used at Keytiles.

When someone says “codebase naming pattern” (or “Codebase Naming”), this is what they mean: establish stable `CODEBASE_NAME` and per-package `PACKAGE_NAME` constants that identify the codebase and its packages. These names are the substrate for logging, faults, and related patterns (notably **Source Anchoring**).

## What this pattern covers (and does not)

**Covers:** creating and wiring `CODEBASE_NAME` and `PACKAGE_NAME` when missing, and how they relate.

**Does not cover:** `methodName`, log message shape, `WithSource` / call stacks, or type-level `fullyQualifiedName` — those belong to [Source Anchoring](golang-pattern-source-anchoring-v1.md).

## Name hierarchy

```
CODEBASE_NAME
  └── PACKAGE_NAME   (= CODEBASE_NAME + ".<package-suffix>")
```

## `CODEBASE_NAME`

- Define **once** for the whole codebase (library or service).
- Prefer the constant name **`CODEBASE_NAME`**, never `LIB_NAME`. Shared libraries and services use the same pattern; `LIB_NAME` only fits libraries and is obsolete.
- Place it in a small root package that other packages can import (commonly `pkg/common.go`).

```go
package pkg

const (
	// Used as a prefix for kt_errors.Fault sources and Logging
	CODEBASE_NAME = "keytiles.lib.messaging"
)
```

### Choosing the string value

First decide whether this codebase is a **library** (shared code) or a **service**. If that is not obvious from the repo / `go.mod`, **ask the user**.

Then take the identifying name from the Go module path in `go.mod` (the module’s short name for that library or service). Use one of these templates:

| Codebase kind | `CODEBASE_NAME` form | Example |
| --- | --- | --- |
| Library (shared code) | `keytiles.lib.<name-of-the-library>` | Module `…/lib-messaging-golang/v3` → `keytiles.lib.messaging` |
| Service | `service.<name-of-the-service>` | Module identifies service `foo` → `service.foo` |

- Use dotted lowercase segments only.
- Map the module path to `<name-of-the-library>` / `<name-of-the-service>` consistently with existing Keytiles repos (e.g. `lib-messaging-golang` → library name `messaging`). If that mapping is ambiguous, **ask the user** — do not invent a name.
- If `CODEBASE_NAME` already exists, keep it; do not rename casually.
- Prefer asking over guessing when library-vs-service or the short name is unclear.

## `PACKAGE_NAME`

- Each Go package that logs, builds faults, or otherwise needs a stable package origin defines its own `PACKAGE_NAME`.
- Compose it from `CODEBASE_NAME`; do not hard-code the full string in multiple places.

```go
package kt_messaging

import "…/pkg" // import path of the package that holds CODEBASE_NAME

const (
	PACKAGE_NAME = pkg.CODEBASE_NAME + ".kt_messaging"
)
```

Typically live in that package’s `common.go` (create the file if missing).

### Choosing the package suffix

- Suffix is a stable logical id for the package within the codebase (dotted segment(s) after `CODEBASE_NAME`).
- Often matches the Go package / module folder name, but **may differ** when the codebase already uses a clearer logical name (example: Go package `rocketmq` → suffix `kt_messaging_rocketmq`).
- Prefer consistency with existing `PACKAGE_NAME` / `WithSource` / logger names in the same repo.
- If unclear, **ask the user**.

## Bootstrap when constants / files are missing

When introducing Codebase Naming into a codebase that does not have it yet:

1. **Determine** library vs service (ask the user if unclear).
2. **Read** `go.mod` and derive the library/service short name; ask if the mapping is ambiguous.
3. **Find or create** the root holder for `CODEBASE_NAME` (usually `pkg/common.go`).
4. **Set** `CODEBASE_NAME` to `keytiles.lib.<name>` (library) or `service.<name>` (service).
5. For each package that needs an origin: **add** `PACKAGE_NAME = <import>.CODEBASE_NAME + ".<suffix>"` (usually in `common.go`).
6. **Wire imports** so packages reference the root `CODEBASE_NAME` constant — do not duplicate the codebase string.
7. If you find legacy `LIB_NAME`, migrate call sites to `CODEBASE_NAME` (same value unless the user directs a rename).
8. If you find bare string literals used as package origins (e.g. in `WithSource`), prefer replacing them with `PACKAGE_NAME` once it exists.

Do not invent `CODEBASE_NAME` or unusual suffixes silently when the choice is ambiguous — ask.

## Checklist

1. `CODEBASE_NAME` exists once at the codebase root package.
2. Constant is named `CODEBASE_NAME` (not `LIB_NAME`).
3. Each relevant package has `PACKAGE_NAME = CODEBASE_NAME + ".<suffix>"`.
4. No duplicated hard-coded codebase prefix across packages.
5. `CODEBASE_NAME` matches library (`keytiles.lib.<name>`) or service (`service.<name>`) rules from `go.mod` (or was confirmed with the user).
6. Values agreed / consistent with existing origins in the repo (or confirmed with the user).
