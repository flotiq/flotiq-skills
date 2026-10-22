# Flotiq's MCP tools for content

Content objects are created and changed through Flotiq's MCP server (`https://mcp.flotiq.com/mcp`), or by hand in the Flotiq panel. Connecting (API key or OAuth) and tool naming are as in [`schema-tools.md`](../../flotiq-schema/references/schema-tools.md). Creating and changing CTDs is the `flotiq-schema` skill's job.

## Tools

| Tool | Use |
|---|---|
| `data_efficient_list_ctd` | Discover Content Types by name, without schema details. The starting point for any content work: the CTD's API name is the `content_type_definition_name` of every object call. |
| `get_content_type(name)` | Read a type's fields, required flags and relations before writing objects — never guess field names. |
| `list_objects(content_type_definition_name, limit, page, offset, filters, sort, include_drafts, hydrate, status)` | List existing objects of a type — check for duplicates, sample existing tone/style, find relation targets, verify a batch. Up to 1000 per page; `filters` take the same shape as the SDK's (`{"slug": {"type": "equals", "filter": "about"}}`, relations via `"<field>[*].dataUrl"` with `includes`); `sort` like `"-internal.createdAt"`; `status` `"public"`/`"draft"`. |
| `get_object(content_type_definition_name, object_id)` | Fetch one object — resolve a relation target, or verify a create/update/publish call. |
| `create_object(content_type_definition_name, data)` | Create a new Content Object. |
| `update_object(content_type_definition_name, object_id, data)` | Update an existing Content Object. |
| `publish_object(content_type_definition_name, object_id, cascade)` | Publish a draft object. Requires **Draft & Public** to be enabled on that Content Type. It answers with a success message, not the object. |

A relation in `data` is a list: `[{"type": "internal", "dataUrl": "/api/v1/content/<type>/<id>"}]`.

## `publish_object` details

* Only applies to Content Types with **Draft & Public** enabled. If a type doesn't have it, objects are live immediately on create/update and there's nothing to publish separately.
* `cascade` controls whether publishing an object also publishes the draft objects it relates to. Set it deliberately: `true` when a related object (e.g. a newly created `Author`) must go live together with the object referencing it; omit/`false` when related objects should stay in draft.
* After publishing, check the object's state with `get_object` rather than assuming the call succeeded.

## Out of scope

* **Changing Content Type Definitions** — that's `flotiq-schema`. Don't add fields to a type just to fit content.
* **Media upload** — the MCP server has none; see [`media-and-alt-text.md`](media-and-alt-text.md).
* **Deleting** content types or objects — there is no delete tool. Something created by mistake is removed in the panel; say so in your report.
