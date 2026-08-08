# Changelog

This repository is a union of documentation / AI-agent instruction files — not a single versioned code artifact — so we do **not** apply Semantic Versioning to the repo as a whole.

We track evolution in two complementary ways:

1. **This file** — date-ordered digest of meaningful changes (what / which files / why in short).
2. **Per-doc snapshots** — before editing a stable **head** file (`…-v1.md`), we copy it to `…-v<major>.<minor>.md` freezing that revision; then we bump `document version:` in the head. Diff head vs snapshot anytime to see exact edits.

Full versioning / linking policy: see [README.md](README.md) (“How we version these docs”).

---

# 2026-08-08

- **Versioning policy** — documented head + snapshot + date changelog approach in README; this CHANGELOG introduced.
- **coding/generic-coding-rules-and-best-practices-v1.md** (`document version: 1.1`; previous freeze: `…-v1.0.md`)
  - Clarified BDD test structure: Feature → Scenario → steps
  - Prefer language subtests / nested cases for Scenarios when available
  - Keep `---- GIVEN` / `---- WHEN` / `---- THEN` inside each Scenario
  - Scenario comments: do not repeat the scenario name; comment only when the name is not enough
  - Clarified that the mandatory short test comment applies to Feature-level tests
- **coding/golang-coding-rules-and-best-practices-v1.md** (`document version: 1.1`; snapshot of previous head: `…-v1.0.md`)
  - Prefer `t.Run("scenario name", …)` to split Scenarios under one `Test_…`
  - Keep GIVEN/WHEN/THEN inside each `t.Run`; optional for a single tiny case
