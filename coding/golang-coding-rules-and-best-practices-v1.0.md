document version: 1.0

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

## Unit test code

- We break Golang best practices and do NOT mix test cases/code with real code. Real code is kept under folder `/pkg` where sub folders breaking up code into packages. And we have `/tests` folder where we put test code. Here we keep the same sub folder structure as we have under `/pkg` preferably.
- This way we can not test package private methods or access package private fields. This is accepted. However what we can do - inspired by Java/Google lib @VisibleForTesting annotation - when we really need to test an internal method or internal anything, we can introduce methods with "VisibleForTesting\_" prefix in the code which can wrap internal logic making them visible for test cases.

## Logging

Keep generic logging rules and on top of that keep these too:

- Keep logs consistent with existing style: `ecntx.LogWithLabels(logger).<Level>(...)` if `ecntx *kt_tracing.ExecutionContext` is available for the method defined
  in module git.keytiles.com/keytiles-golang/lib-common-golang.
- Use local `methodName := "..."` and include it in log messages.
- Prefer lifecycle-oriented debug logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- Be explicit with key fields for large or sensitive payloads.
- `kt_utils.VarPrinter{TheVar: ...}` is a good alternative for small/compact structs when it improves readability and maintainability.
- Be careful with `VarPrinter` on large/deep objects: it can create noisy logs and unnecessary overhead.
- For shared helper code, pass logger through the method contract early, even if logging is minimal at first, so future diagnostics are easy to add.
