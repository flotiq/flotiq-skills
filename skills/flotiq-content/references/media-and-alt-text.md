# Media and alt text

## Media workflow

1. Flotiq's MCP server has no upload. A file you have (an image from a design export, say) goes into the Media library through the REST API: a multipart `POST https://api.flotiq.com/api/media` with `file` and `type` (`"image"`, or `"file"` for anything else, a video included) and the header `X-AUTH-TOKEN: <read-write key>`; the answer is the new `_media` object. Without the file, the Media library is filled in the Flotiq panel, where its title and alt text are set too.
2. Relate existing media: find it with `list_objects` on `_media`, and put its reference in the content object's media field (`create_object`/`update_object`), respecting whether that field allows a single asset or multiple (a gallery).
3. When the space has no fitting media, leave optional media fields empty, and say in your report which objects need which images. Don't invent placeholder URLs or point at unrelated assets.
4. Reuse one media object across content objects when the same asset applies, rather than asking for duplicates.

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
