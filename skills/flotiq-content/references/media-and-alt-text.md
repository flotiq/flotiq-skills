# Media and alt text

## Upload workflow

1. Upload the file to the Media library first, via the Media API or the panel — the Flotiq MCP server has no upload tool, so this step happens outside MCP. Media is always uploaded as its own object, then related to content, never inlined.
2. Set the media object's own metadata (title, alt text) at upload time rather than leaving it for later.
3. Relate the uploaded media object's ID to the content object's media field (via `create_object`/`update_object` if you're using the MCP server for the content object itself), respecting whether that field allows a single asset or multiple (a gallery).
4. Reuse existing media objects when the same asset applies to multiple content objects, instead of re-uploading duplicates — `list_objects` against the Media type can help find them.

## Writing alt text

* Alt text describes what the image shows and why it's there — not the filename, not "image of X" boilerplate.
* Keep it concise (roughly one short sentence) and specific: "Red ceramic mug on a wooden table," not "Product photo" or "mug1.jpg."
* For purely decorative images that carry no content meaning, that's a signal to check whether the image field is the right choice at all — but when a value is required, still describe what's shown rather than leaving generic text.
* Don't reuse identical alt text across visually different images just to fill the field.

## Verifying media

After attaching media, confirm:

* The media relation resolves to the correct asset (not an unrelated or leftover upload).
* Alt text is present and specific for every image that conveys meaning.
* Single vs. multiple media fields hold the intended number of assets.

See [Media library](https://flotiq.com/docs/API/media-library/) for upload limits, supported formats, and transformation options.
