---
name: flotiq-schema
description: Use when designing, creating, or evolving Flotiq Content Type Definitions (CTDs) — content modeling, field selection, naming, relations, and schema evolution. Not for populating content (see flotiq-content) or building frontends (see flotiq-sdk).
---

# Flotiq Schema

Design, create, and evolve Flotiq Content Type Definitions (CTDs) — from identifying business entities through creating and verifying the CTDs. Not for populating content (`flotiq-content`) or frontend/SDK work (`flotiq-sdk`).

**MCP scope:** the MCP server can only `list_content_types()` — there's no tool to create or update a CTD. Create/edit CTDs via the panel or Content API. See [`references/mcp-scope.md`](references/mcp-scope.md).

## When to use

* Designing a new content model, adding a Content Type, or changing fields/relations on an existing one.
* Reviewing whether a proposed schema is well-modeled before creating it.

## Workflow

1. List the business entities involved — not fields yet. Check `list_content_types()` first so you reuse or extend an existing type instead of duplicating one.
2. For each entity, pick a clear name, a genuinely human-readable **title field**, and a field list where every field has a purpose (see `references/field-types.md`).
3. Model shared or reused data as a **relation** to its own Content Type, not duplicated fields (`references/relations-and-media.md`). Apply naming conventions consistently (`references/naming-conventions.md`).
4. Set validation where correctness matters, and order fields for the editor filling the form, not alphabetically.
5. Create or update the CTD via the panel or Content API — not MCP — then re-fetch it (`list_content_types()`, the panel, or the API) to confirm it matches the design.
6. Flag that `flotiq-typegen` needs to run before any frontend code depends on the new shape.

## Best practices

* Relations over duplication; a small set of reusable types (`Author`, `Media`, `SEO`) beats many narrow, single-use ones.
* Names describe meaning, not type or storage (`publishedAt`, not `dateField1`).
* User-facing types get an explicit `slug` field; every type gets a real title field, never an ID.
* Media fields are relations to the built-in Media type, not URL/text fields.
* Schema evolution is additive by default — add fields rather than rename/remove, which breaks existing content and generated types.

## Common mistakes

* Copying data (an author's name, a category's label) onto every object instead of relating to it.
* A technical or missing title field (`id`, `createdAt`) instead of something human-readable.
* Renaming or deleting fields in place instead of adding new ones.
* A near-duplicate Content Type created instead of extending an existing one.
* Trying to create or edit a CTD through the MCP server — it only lists them.
* Not re-verifying the CTD after creation.

## Completion checklist

* [ ] Entities map to Content Types with no missing types or near-duplicates.
* [ ] Every type has a real title field and meaningfully named fields.
* [ ] Shared data uses relations; media fields relate to Media.
* [ ] Required validation is set; user-facing types have slugs.
* [ ] The CTD was created/updated outside MCP and re-fetched to confirm it matches the design.
* [ ] Any breaking change (rename/remove) was deliberate, not a default.

## References

* [`references/field-types.md`](references/field-types.md) — field type cheat sheet.
* [`references/naming-conventions.md`](references/naming-conventions.md) — naming rules for types, fields, and slugs.
* [`references/relations-and-media.md`](references/relations-and-media.md) — modeling relations and media fields.
* [`references/mcp-scope.md`](references/mcp-scope.md) — what the MCP server can and can't do for schema work.

## Official documentation

* [Content Type Definitions](https://flotiq.com/docs/API/content-types/)
* [Creating new Content Types](https://flotiq.com/docs/API/content-type/creating-ctd/)
* [Content types in the panel](https://flotiq.com/docs/panel/content-types/)
* [Media library](https://flotiq.com/docs/API/media-library/)
* [Flotiq MCP Server](https://mcp.flotiq.com/mcp)
