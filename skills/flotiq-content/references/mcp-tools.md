# Flotiq MCP server tools

The Flotiq MCP server (`https://mcp.flotiq.com/mcp`) exposes Flotiq CMS operations as MCP tools. It covers **Content Objects only** — not Content Type Definitions (see the `flotiq-schema` skill's [`references/mcp-scope.md`](../../flotiq-schema/references/mcp-scope.md) for that boundary).

## Tools

| Tool | Signature | Use |
|---|---|---|
| List content types | `list_content_types()` | Discover Content Types and their `content_type_definition_id`s. The starting point for any content work — you need the ID for every other call. |
| List objects | `list_objects(content_type_definition_id)` | List existing objects of a type — check for duplicates, sample existing tone/style, find relation targets. |
| Get object | `get_object(content_type_definition_id, object_id)` | Fetch one object — resolve a relation target, or verify a create/update/publish call. |
| Create object | `create_object(content_type_definition_id, data)` | Create a new Content Object. |
| Update object | `update_object(content_type_definition_id, object_id, data)` | Update an existing Content Object. |
| Publish object | `publish_object(content_type_definition_id, object_id, cascade)` | Publish a draft object to public. Requires **Draft & Public** to be enabled on that Content Type. |

## `publish_object` details

* Only applies to Content Types with **Draft & Public** enabled. If a type doesn't have it enabled, objects are live immediately on create/update and there's nothing to publish separately.
* `cascade` controls whether publishing an object also publishes the draft objects it relates to. Set it deliberately: `true` when a related object (e.g. a newly created `Author`) must go live together with the object referencing it; omit/`false` when related objects should stay in draft independently.
* After publishing, call `get_object` to confirm the object's publish state actually changed rather than assuming the call succeeded.

## What's out of scope for the MCP server

* **Content Type Definitions** — no create/update/delete tool exists for CTDs. That's panel/API work (`flotiq-schema`).
* **Media upload** — there's no upload tool in this list. Upload files through the Media library API or panel, then reference the resulting media object's ID in `create_object`/`update_object` data.

## Authentication

The server uses OAuth 2.0 bearer token authentication: an MCP client authenticates once (the server exposes an OAuth discovery endpoint), and the resulting `Authorization` header is forwarded through to the Flotiq API on every tool call. This is connection setup, done once per client — not something to redo per task.
