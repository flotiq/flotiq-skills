# Typegen workflow

`flotiq-api-typegen` (from `@flotiq/flotiq-api-sdk`) reads the **live schema** of a Flotiq space — it isn't a static template — and writes TypeScript types that augment the `@flotiq/flotiq-api-sdk` module.

## Commands

| Purpose | Command |
|---|---|
| Generate once | `npm run typegen` if the project defines it, else `npx flotiq-api-typegen` |
| Regenerate on every schema change while iterating | `npx flotiq-api-typegen --watch` |
| Another output path | `--output src/flotiq-api.d.ts` (default `./flotiq-api.d.ts`) |
| Key not in the env | `--flotiq-key <read-only key>` (default: `FLOTIQ_API_KEY`, `.env` files are loaded) |

## What it produces

For each content type API name (`blog_post`): `BlogPost`, `BlogPostHydrated`, `BlogPostHydratedTwice`, `BlogPostBase`, filterable field paths (including relation paths like `category[*].dataUrl`), and a typed `flotiqApiClient.content.blog_post`. Import types from the SDK module: `import type { BlogPost } from "@flotiq/flotiq-api-sdk"`.

## When to run it

* Right after creating a Content Type, or adding, renaming, removing or retyping a field.
* Right after changing a relation's target type or multiplicity.
* Before fetch/render code is written against the new shape (the `flotiq-sdk` workflow).

## Committing vs. gitignoring

* **Commit by default.** The app then type-checks and builds without a live connection to Flotiq — in CI, offline, and in reviews of changes that touch content shapes.
* **Gitignore only deliberately**, when the build should always use the fresh schema. Then CI must run typegen before compiling, or builds fail or use stale local types.

## Keeping generated files untouched

Treat the output as read-only. Something wrong in it points to the Content Type Definition (`flotiq-schema`), not to the file — a manual edit is overwritten by the next run and hides the real schema issue.
