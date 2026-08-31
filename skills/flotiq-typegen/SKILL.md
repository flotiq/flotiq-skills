---
name: flotiq-typegen
description: Use when TypeScript types for Flotiq content need to be generated or refreshed after a Content Type change, via `npm exec flotiq-api-typegen`. Not for schema design (see flotiq-schema) or SDK/fetch code (see flotiq-sdk).
---

# Flotiq Typegen

Generate and maintain TypeScript definitions for Flotiq content. Three rules — kept deliberately small. Not for schema design (`flotiq-schema`) or fetch/render code (`flotiq-sdk`).

## When to use

After any Content Type is created or changed, before code depends on the new shape.

## Workflow

1. Run `npm exec flotiq-api-typegen` from the project root (`--watch` while actively iterating on a schema).
2. Never hand-edit the output — if a generated type looks wrong, fix the CTD (`flotiq-schema`) and regenerate.
3. Commit the generated types by default, so builds don't need a live Flotiq connection. Only gitignore them if CI is set up to run typegen before build.

## Common mistakes

* Forgetting to regenerate after a schema change, so code compiles against a stale shape.
* Hand-editing the generated file, or hand-writing a parallel interface instead of trusting it.
* Gitignoring generated types without CI actually regenerating them before build.

## Completion checklist

* [ ] `FLOTIQ_API_KEY` is set in the environment before running typegen.
* [ ] `npm exec flotiq-api-typegen` ran successfully after the most recent schema change.
* [ ] The generated file has not been hand-edited.
* [ ] Generated types are committed (or CI is confirmed to regenerate them before build).
* [ ] No duplicate handwritten interfaces exist alongside the generated types.

## References

* [`references/typegen-workflow.md`](references/typegen-workflow.md) — command reference and the commit-vs-gitignore tradeoff.

## Official documentation

* Type generation: `npm exec flotiq-api-typegen`
* [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/)
