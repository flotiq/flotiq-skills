# What the Flotiq MCP server covers for schema work

The Flotiq MCP server (`https://mcp.flotiq.com/mcp`) exposes exactly these tools:

* `list_content_types()`
* `list_objects(content_type_definition_id)`
* `get_object(content_type_definition_id, object_id)`
* `create_object(content_type_definition_id, data)`
* `update_object(content_type_definition_id, object_id, data)`
* `publish_object(content_type_definition_id, object_id, cascade)`

All but `list_content_types` operate on **Content Objects**. There is no `create_content_type` / `update_content_type` tool — the MCP server cannot create or modify a Content Type Definition.

## What this means for schema work

* Use `list_content_types()` to discover existing Content Types and their IDs — this is the right way to check what already exists (workflow step 2) and to verify a CTD after it's created (step 9).
* Do **not** attempt to create or edit a CTD through the MCP server — it has no tool for that. CTD creation/editing happens through the Flotiq panel's content type builder or a direct call to the Content API (see [Creating new Content Types](https://flotiq.com/docs/API/content-type/creating-ctd/)).
* Once a CTD exists, populating it with Content Objects *is* fully MCP-covered — that's the `flotiq-content` skill's job, using `create_object`/`update_object`/`publish_object`.

## Authentication

The MCP server uses OAuth 2.0 bearer token authentication (with an OAuth discovery endpoint for MCP clients) and forwards the `Authorization` header through to the Flotiq API. This is a one-time client/connection setup concern, not something to reconfigure per schema task.
