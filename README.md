# What is this repo?

We keep global .md / .mdc files in this repo which contain instructions for AI agents in several contexts.

# How is it used?

When we have a concrete code repository where we would like to use AI agents we simply create a local .md file containing instructions and they
refer to the global rules. So we use it basically as local -> global reference, a proxy so to speak.

Please check [how-to-develop-with-AI-agents.md](how-to-develop-with-AI-agents.md) doc!

That one describes a workflow how we use all of it - even with example videos attached.

# How we version these docs

This is a documentation / agent-instructions repo, not a single versioned library. We do **not** use Semantic Versioning for the repository as a whole.

Instead we use:

## Stable “head” files (what consumers should link)

- Each instruction family has a **head** file named with a **major** only, e.g. `coding/generic-coding-rules-and-best-practices-v1.md`.
- Point your local `/agents` proxies at these head files so links stay stable while content evolves.
- Inside the head file, the header `document version: <major>.<minor>` is the **current editorial revision** of that head (policy **B**).

## Snapshots before a change (history you can diff)

When we change a head file:

1. Copy the current head to a snapshot named with the **full** version being frozen, e.g. `…-v1.1.md` (same content as head had when it was `document version: 1.1`).
2. Edit the head and bump the minor in `document version:` (e.g. `1.1` → `1.2`).

That way you can always diff `…-v1.1.md` vs `…-v1.md` (or vs a later snapshot) and self-track evolution at any point in time.

Bump the **major** in the filename (`…-v2.md`) only for a **breaking** rewrite of that instruction family (when old guidance should no longer be the default ref).

## CHANGELOG

See [CHANGELOG.md](CHANGELOG.md) for a date-ordered digest of what changed (which heads, short summary). Git history remains the full source of truth; the changelog is the curated overview.
