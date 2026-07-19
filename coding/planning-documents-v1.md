# What are planning documents?

When we develop a feature / class / module we create a development plan document which

- Describes the business requirements
- Describes the change
- Documents decisions we made
- Optionally contains implementation steps

## Where do we keep these documents?

Each code repository has a folder `/development-plans`. We keep these files there.

Subfolders are allowed for grouping. A common pattern is **one subfolder per artifact release version** (e.g. `/development-plans/v2.0.0/`), so maintainers can see which plans belong to which target release. Feature-based subfolders are also fine.

## Format

We keep these planning documents in Markdown .md files.

## File naming pattern

The name of files following this pattern: "<name of the thing>-v<major>.<minor>-plan.md"

## Companion documents

The feature / class / module documents - please read [docs-rules-and-best-practices-v1.md](../docs/docs-rules-and-best-practices-v1.md)

Very often the planning document is a further development of a specific version of a documented feature / class / module.
In this case the version of the plan.md file (see file name pattern above) is derived from the referred version of the feature visible from the docs file name pattern.

## Rules to keep

Please keep the following rules when working with plan.md files:

- Always add a header which contains
  - The date when we created/modified the file.
- Blocks of the file:
  1.  Header - as stated above
  2.  Documentation references.
      If we do the change based on feature / class / module documentation files because we evolve it further, refer them in here! See "Companion documents" above.
  3.  "why are we doing this?" - Business requirements.
      If we know the business requirements so "why" we are doing this new version, write that down into this block. Be short and concise! Use bulletted list if applicable.
  4.  "what will be changed?" - this block is summarizing the changes. Be short and concise here too! Use bulletted list if applicable.
  5.  Decisions we made.
      Just summarize them in concise form. Use bulletted list if applicable.
  6.  Implementation steps
      If the change is bigger it really make sense to come up with a step-by-step plan what do we do in which step.
      This block is documenting these steps and also their state.
      In the beginning all steps are "planned" but as we make progress with the implementation they get into "implemented" state.
  7.  Anything else can come here which you think worth mentioning.
      For example, how can we test the implementation? But can contain other useful info too - you can be creative :-)
- If something is unclear regarding any of the above blocks please always ask the user for more information! As this is the only way to keep highest achievable quality.
