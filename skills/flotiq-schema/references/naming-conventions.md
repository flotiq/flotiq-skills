# Naming conventions

Consistent naming keeps generated types readable and content editors unconfused. These are conventions to apply, not hard platform rules — check the project's existing schema first and match it if it already diverges.

## Content Type names

* Singular, not plural: `BlogPost`, not `BlogPosts`. A Content Type describes one object; the collection is implicit.
* PascalCase for the display name (`BlogPost`, `ProductCategory`), which also drives the generated TypeScript type name via `flotiq-typegen`.
* Name the entity, not its storage or usage: `Author`, not `AuthorsTable`; `HeroSection`, not `HomepageBlock1`.
* Avoid near-duplicate names for conceptually different things (`Category` vs `ProductCategory` vs `BlogCategory`) unless they really are distinct entities with different fields — otherwise prefer one shared `Category` type with a relation.

## Field names

* camelCase, matching typical JS/TS consumption: `publishedAt`, `heroImage`, `isFeatured`.
* Name for meaning, not type: `price`, not `numberField`; `coverImage`, not `mediaField1`.
* Boolean fields read as a yes/no question: `isPublished`, `hasVideo`, not `published` (ambiguous verb/adjective) or `flag1`.
* Date fields end in a clear suffix: `publishedAt`, `startsAt`, `updatedAt`.
* Relation fields are named for what they point to, singular or plural to match cardinality: `author` (single relation), `tags` (multiple relation).

## Slugs

* Any Content Type rendered at a stable, public URL should have an explicit `slug` field — don't route on the internal object ID.
* Keep slugs URL-safe (lowercase, hyphen-separated) and treat them as unique in practice for that content type, even if the schema doesn't enforce global uniqueness.
* Generate slugs from the title by convention (e.g. title-cased "Hello World" → `hello-world`) and let editors override them, rather than requiring manual entry every time.

## Title fields

* Every Content Type needs a title field that's genuinely useful in a relation picker or content list — usually a `text` field like `name` or `title`.
* Never leave the title field pointing at an internal ID, timestamp, or other non-human-readable value.

## Sources of truth

If a project has an existing naming scheme (even an imperfect one), match it rather than introducing a second convention. Consistency inside a project beats a "more correct" convention applied inconsistently.
