# Typegen workflow

## Commands

| Purpose | Command |
|---|---|
| Generate types once | `npm exec flotiq-api-typegen` |
| Regenerate continuously while editing the schema | `npm exec flotiq-api-typegen --watch` |

Run these from the project root. Generation reads the **live schema** — it isn't a static template. `FLOTIQ_API_KEY` (or the env variable your project uses for the Flotiq API key) must be set in the environment before running; without it the command fails to reach the schema. Check `.env.local` or `.env.example` for the expected variable name.

## When to run it

* Right after creating a new Content Type.
* Right after adding, renaming, or removing a field on an existing Content Type.
* Right after changing a relation's target type or multiplicity.
* Before writing or reviewing any fetch/render code that depends on a recently changed type (part of the `flotiq-sdk` workflow, not this skill).

## Committing vs. gitignoring

* **Commit by default.** Committed generated types mean the app type-checks and builds without a live connection to Flotiq — useful for CI, offline work, and reviewing PRs that touch content shapes.
* **Gitignore only deliberately**, when a project specifically wants types always regenerated fresh at build time (e.g. to catch schema drift automatically). If chosen, the build/CI pipeline must run `npm exec flotiq-api-typegen` before compiling — otherwise builds will fail or use stale local types.

## Keeping generated files untouched

Treat the generated output as read-only. If something about it looks wrong, that's a signal to check the Content Type Definition (see the `flotiq-schema` skill) rather than to patch the generated file — a manual edit will simply be overwritten by the next regeneration and masks the real schema issue.
