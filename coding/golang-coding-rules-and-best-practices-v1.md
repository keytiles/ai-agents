# How to write Go code?

Please read [generic-coding-rules-and-best-practices-v1.md](generic-coding-rules-and-best-practices-v1.md)!
We need to keep all stadards and best practices described there.

On the top of them please also keep the following!

## Functional programming - ExecutionContext

We follow functional programming in many aspects when we write go code.

There is a type defined in module git.keytiles.com/keytiles-golang/lib-common-golang: kt_tracing.ExecutionContext.
This context is typically created on the public barrier (incoming message, incoming sync request) and then we pass it along on the call chain.
It also can encapsulate a standard Golang Context. And carries important information about the business transaction accross the call chain.

It is a good practice to take `ecntx *kt_tracing.ExecutionContext` parameter into all method signatures so method code is aware and can make logs better - see section "Logging" below!

## Logging

- Keep logs consistent with existing style: `ecntx.LogWithLabels(logger).<Level>(...)` if `ecntx *kt_tracing.ExecutionContext` is available for the method defined
  in module git.keytiles.com/keytiles-golang/lib-common-golang.
- Use local `methodName := "..."` and include it in log messages.
- Prefer lifecycle-oriented debug logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- Be explicit with key fields for large or sensitive payloads.
- `kt_utils.VarPrinter{TheVar: ...}` is a good alternative for small/compact structs when it improves readability and maintainability.
- Be careful with `VarPrinter` on large/deep objects: it can create noisy logs and unnecessary overhead.
- For shared helper code, pass logger through the method contract early, even if logging is minimal at first, so future diagnostics are easy to add.
