# Naming conventions

Consistent naming keeps generated types readable and content editors unconfused. Check the project's existing schema first and match it if it already has a convention.

## Content Type names

A Content Type has two names:

* **API name** — `a-z` and `_` only, singular: `blog_post`, `product_category`. Anything else is refused, and it can't be changed later, so pick it carefully. It is the SDK method (`flotiqApiClient.content.blog_post`), the `content_type_definition_name` of every MCP call, part of every relation's `dataUrl` (`/api/v1/content/blog_post/<id>`), and the source of the generated TypeScript name (`blog_post` → `BlogPost`, `BlogPostHydrated`).
* **Label** — what editors see in the panel: `Blog Post`, `Product Category`. It can be changed any time. Group related types with a prefix, as the section library does (`FF Section/Hero`).

For both:

* Singular, not plural: `blog_post`, not `blog_posts`. A Content Type describes one object; the collection is implicit.
* Name the entity, not its storage or usage: `author`, not `authors_table`; `hero_section`, not `homepage_block_1`.
* Avoid near-duplicates for conceptually the same thing (`category` vs `product_category` vs `blog_category`) unless they really are distinct entities with different fields — otherwise one shared `category` type with a relation.

## Field names

* `snake_case` by default — it's what the section library uses (`cta_buttons`, `seo_title`, `internal_link`). If the project's types already use another style, match it within the project.
* Name for meaning, not type: `price`, not `number_field`; `cover_image`, not `media_field_1`.
* Booleans read as a yes/no question: `is_featured`, `has_video`, not `featured` or `flag1`.
* Dates end in a clear suffix: `published_at`, `starts_at`.
* Relation fields are named for what they point to, singular or plural to match cardinality: `author` (one), `tags` (many).

## Slugs

* Any Content Type rendered at a stable, public URL has an explicit `slug` field — don't route on the internal object ID.
* Keep slugs URL-safe (lowercase, hyphen-separated) and unique in practice for that type, even if the schema doesn't enforce it.
* Generate slugs from the title ("Hello World" → `hello-world`) with the CTD's auto-slug rule (`slugs` in `create_content_type`) and let editors override them.

## Title fields

* Every Content Type needs a title field that's useful in a relation picker or content list — usually a text field like `name` or `title`.
* Never leave the title field pointing at an internal ID, timestamp, or other non-human-readable value.

## Sources of truth

If a project has an existing naming scheme (even an imperfect one), match it rather than introducing a second convention. Consistency inside a project beats a "more correct" convention applied inconsistently.
