# Iterative development workflow

Normally we follow an iterative approach when we write code.

1. Understanding the current state of the feature / class / module - existing documentation helps a lot here. Please read [docs-rules-and-best-practices-v1.md](../docs/docs-rules-and-best-practices-v1.md)!
2. We plan the change and describe what and how we want to do. Please read [planning-documents-v1.md](planning-documents-v1.md) to understand how it works!
3. We do the implementation so the coding.
4. We document how the new feature or version of it works. Following same rules as in step #1 according to [docs-rules-and-best-practices-v1.md](../docs/docs-rules-and-best-practices-v1.md)!

# Rules to keep

## Comments

- Always add at least brief comments to classes / structs explaining what they do. Keep it short!
- Always add at least brief comments to methods explaining what they do.
- Also do it for internal helper methods.
- In longer methods provide inline brief comments so reader understands better which code block does what.
- When you are adding comment to a method itself above the function (godoc, javadoc, etc), please do not add the name of the method into the comment - totally pointless. We see the method...

## Unit test code

- Please keep BDD pattern (Gherkin language like) and organize test cases internals into GIVEN / WHEN / THEN segments - separated by comments.
  This improves test readability drastically.
- Always add a short comment to all test cases (typically methods) about what they are testing.
- Consider adding any inline comments which helps the reader to understand better what the test case does and make reverse engineering of the test case easier.

# Best practices to keep

Points documented here should be heavily considered when crafting code. Not mandatory, but strongly preferred. If we deviate from these please provide the reason why we do so.

## Internal helper methods.

We know clean code is cool and normally improves readability. But if we end up in a state we have 100 internal helper methods and 10 public facing real
methods that's bad because we will drown in number of methods quickly.

Please do NOT introduce internal helpers if:

- The method is just invoked from one place AND relatively short. Then better to keep the code inline and add a comment inline to the code block.

## Logging

- Keep logs consistent with existing style: `ecntx.LogWithLabels(logger).<Level>(...)`.
- Use local `methodName := "..."` and include it in log messages.
- Prefer lifecycle-oriented debug logs for internals: `started (...)` and `finished (...)` with compact, useful metrics.
- Avoid duplicate/redundant logs for the same step; one strong signal is better than two weak ones.
- Be explicit with key fields for large or sensitive payloads.
- `kt_utils.VarPrinter{TheVar: ...}` is a good alternative for small/compact structs when it improves readability and maintainability.
- Be careful with `VarPrinter` on large/deep objects: it can create noisy logs and unnecessary overhead.
- For shared helper code, pass logger through the method contract early, even if logging is minimal at first, so future diagnostics are easy to add.
