# Flotiq's MCP tools for schema work

Content types are created and changed through Flotiq's MCP server (`https://mcp.flotiq.com/mcp`), or by hand in the Flotiq panel. The server takes a read-write API key of the space (header `Authorization: Apikey <key>`) or an OAuth sign-in (it advertises OAuth discovery, so MCP clients that support it log in themselves). The key never goes into the repo.

The names below are the server's own. An environment may expose them under its own names (`flotiq_create_object`, say) — the parameters are the same, and its tools take precedence.

| Tool | Use |
|---|---|
| `data_efficient_list_ctd` | List all CTDs by name/label, without schema details. Use it to check what already exists before designing a new type. |
| `get_content_type(name)` | Get one full CTD (`schemaDefinition`, `metaDefinition`). Use it before changing a type and to verify a type after creating or updating it. |
| `create_content_type(name, label, properties, properties_config, required, order, draft_public, slugs)` | Create a new CTD. The schema boilerplate (`allOf`/`$ref`, `additionalProperties`…) is added automatically; you pass the fields. `draft_public` turns on Draft & Public; `slugs` adds auto-slug rules (`[{"source": "title", "target": "slug", "active": true}]`). The tool's description has the exact format of every field type. |
| `update_content_type(name, properties, properties_config, required, order, label, draft_public, slugs, allow_property_removal)` | **Replace** the field set of an existing CTD. Omitted `required`, `order`, `label`, `draft_public`, `slugs` keep their current values. |

Content Object tools (`list_objects`, `create_object`, `update_object`, `publish_object`) belong to the `flotiq-content` skill.

## What this means for schema work

* `properties` is the JSON Schema of each field; `properties_config` is its editor config (input type, label, validation, `order`, relation targets). Every field needs an entry in both. See `field-types.md` for which input type to use.
* Mark the title field(s) with `isTitlePart: true` in `properties_config` — that's what relation pickers show. Field names `id`, `_id`, `objectType`, `internal` (and other `_`-prefixed search names) are reserved.
* API names are `a-z` and `_` only (`quiz_topic`, not `quizTopic`); anything else is refused, and the name can't be changed later.
* A relation needs its target type to exist: create the related type first, one call at a time.
* `update_content_type` replaces the whole field set, it does not merge. Always start from `get_content_type(name)`, change the returned fields, and pass the complete set back. Omitting a field would remove it; the tool refuses that unless `allow_property_removal` is true. Pass that flag only for a removal or rename that was asked for — it deletes that field's data in every object — never to get past the refusal.
* After `create_content_type`/`update_content_type`, call `get_content_type(name)` and check that fields, required flags, relations and field order match the design.
* A failed call returns Flotiq's error message: read it and fix that field.
* There is no delete tool for CTDs. A type created by mistake is removed in the Flotiq panel — say so in your report.
* After the last change, regenerate the types (`flotiq-typegen`).
