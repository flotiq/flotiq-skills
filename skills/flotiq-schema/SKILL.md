---
name: flotiq-schema
description: Use when designing, creating, or evolving Flotiq Content Type Definitions (CTDs) — content modeling, field selection, naming, relations, and schema evolution. Not for populating content (see flotiq-content) or building frontends (see flotiq-sdk).
---

# Flotiq Schema

Design, create, and evolve Flotiq Content Type Definitions (CTDs) — from identifying business entities through creating and verifying the CTDs. Not for populating content (`flotiq-content`) or frontend/SDK work (`flotiq-sdk`).

**Tools:** Flotiq's MCP server lists, reads, creates and updates CTDs with `data_efficient_list_ctd`, `get_content_type`, `create_content_type` and `update_content_type`. `update_content_type` replaces the whole field set. There is no delete tool. See [`references/schema-tools.md`](references/schema-tools.md).

## When to use

* Designing a new content model, adding a Content Type, or changing fields/relations on an existing one.
* Reviewing whether a proposed schema is well-modeled before creating it.

## Workflow

Website pages built from sections (landing, company, product pages) follow `flotiq-sections`: the `page` type and the library's `ff_section_*` types come with the section library's CLI, and each is the app's to change like any other type. Design here the site's own entities (blog posts, products, events) and the fields of a section type you add or change.

1. List the business entities involved — not fields yet. Check `data_efficient_list_ctd` first so you reuse or extend an existing type instead of duplicating one.
2. For each entity, pick a clear name, a genuinely human-readable **title field**, and a field list where every field has a purpose (see `references/field-types.md`).
3. Model shared or reused data as a **relation** to its own Content Type, not duplicated fields (`references/relations-and-media.md`). Apply naming conventions consistently (`references/naming-conventions.md`).
4. Set validation where correctness matters, and order fields for the editor filling the form, not alphabetically.
5. Create the CTD with `create_content_type`, or update it with `update_content_type`, starting from `get_content_type` and passing the complete field set back. Create one type at a time, and a type that others relate to first. Then re-fetch it with `get_content_type` to confirm it matches the design.
6. Regenerate the types (`flotiq-typegen`) before any code uses the new shape.

## Best practices

* Relations over duplication; a small set of reusable types (`author`, `category`, `seo`) beats many narrow, single-use ones.
* Names describe meaning, not type or storage (`published_at`, not `date_field_1`).
* User-facing types get an explicit `slug` field; every type gets a real title field, never an ID.
* Media fields are relations to the built-in Media type, not URL/text fields.
* Schema evolution is additive by default — add fields rather than rename/remove, which breaks existing content and generated types.

## Common mistakes

* Copying data (an author's name, a category's label) onto every object instead of relating to it.
* A technical or missing title field (`id`, `createdAt`) instead of something human-readable.
* Renaming or deleting fields in place instead of adding new ones.
* A near-duplicate Content Type created instead of extending an existing one.
* Calling `update_content_type` with only the new or changed fields. It replaces the field set, so start from `get_content_type` and pass every field.
* Setting `allow_property_removal=True` just to get past the refusal instead of treating it as a breaking change.
* Not re-verifying the CTD after creation.

## Completion checklist

* [ ] Entities map to Content Types with no missing types or near-duplicates.
* [ ] Every type has a real title field and meaningfully named fields.
* [ ] Shared data uses relations; media fields relate to Media.
* [ ] Required validation is set; user-facing types have slugs.
* [ ] The CTD was re-fetched with `get_content_type` after create/update and matches the design.
* [ ] Any breaking change (rename/remove) was deliberate, not a default.

## References

* [`references/field-types.md`](references/field-types.md) — field type cheat sheet.
* [`references/naming-conventions.md`](references/naming-conventions.md) — naming rules for types, fields, and slugs.
* [`references/relations-and-media.md`](references/relations-and-media.md) — modeling relations and media fields.
* [`references/schema-tools.md`](references/schema-tools.md) — what Flotiq's MCP tools can and can't do for schema work.

## Official documentation

* [Content Type Definitions](https://flotiq.com/docs/API/content-types/)
* [Creating new Content Types](https://flotiq.com/docs/API/content-type/creating-ctd/)
* [Content types in the panel](https://flotiq.com/docs/panel/content-types/)
* [Media library](https://flotiq.com/docs/API/media-library/)
* [Flotiq MCP Server](https://mcp.flotiq.com/mcp)
