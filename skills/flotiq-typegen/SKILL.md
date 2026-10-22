---
name: flotiq-typegen
description: Use when TypeScript types for Flotiq content need to be generated or refreshed after a Content Type change — `flotiq-api-typegen` (often `npm run typegen`) writes `flotiq-api.d.ts`. Not for schema design (see flotiq-schema) or SDK/fetch code (see flotiq-sdk).
---

# Flotiq Typegen

Generate and maintain the TypeScript definitions for Flotiq content in `flotiq-api.d.ts`. Three rules — kept deliberately small. Not for schema design (`flotiq-schema`) or fetch/render code (`flotiq-sdk`).

## When to use

After any Content Type is created or changed, before code depends on the new shape.

## Workflow

1. Run the project's typegen script if it has one (`npm run typegen`); otherwise `npx flotiq-api-typegen` (the bin ships with `@flotiq/flotiq-api-sdk`). It reads the live schema of the space of `FLOTIQ_API_KEY` (from the environment or `.env`; a read-only key is enough) and writes `flotiq-api.d.ts` in the project root. Without the key it fails: say so instead of writing types by hand.
2. Never hand-edit the output — if a generated type looks wrong, fix the CTD (`flotiq-schema`) and regenerate. Keep `flotiq-api.d.ts` out of lint (e.g. ESLint `globalIgnores`), so its style never needs fixing or suppressing.
3. Commit the generated file with the change, so builds don't need a live Flotiq connection.

Tools that create content types for you (the section library's `add`, an environment's own tool) usually regenerate the types themselves; run typegen after your own schema changes.

## Common mistakes

* Forgetting to regenerate after a schema change, so code compiles against a stale shape.
* Hand-editing the generated file, or hand-writing a parallel interface instead of trusting it.
* Gitignoring the generated file without CI regenerating it before build.

## Completion checklist

* [ ] Typegen ran successfully after the most recent schema change.
* [ ] The generated file has not been hand-edited.
* [ ] It is committed (or CI is confirmed to regenerate it before build).
* [ ] No duplicate handwritten interfaces exist alongside the generated types.

## References

* [`references/typegen-workflow.md`](references/typegen-workflow.md) — options, when to regenerate, the commit-vs-gitignore tradeoff.

## Official documentation

* [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/)
