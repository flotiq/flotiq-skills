# Field type cheat sheet

Quick reference for picking a field type when designing a Content Type Definition. This is a summary for decision-making, not the full field/validation spec — see [Content Type Definitions](https://flotiq.com/docs/API/content-types/) for the exhaustive list and JSON Schema details.

| Need | Field type | Notes |
|---|---|---|
| Short label, name, single-line value | Text | Use for titles, slugs, short codes. |
| Long-form copy with formatting | Rich text | Use for body copy, descriptions with links/lists. |
| Structured, repeatable content blocks | Block | Use for flexible page/article bodies made of distinct block types (header, image, quote, etc). |
| Number (integer or decimal) | Number | Set min/max where a negative or out-of-range value would be invalid. |
| True/false toggle | Checkbox | Prefer this over a two-option select for genuine booleans. |
| Fixed set of choices | Select / Radio | Use for enums an editor picks from (status, category tier) that don't need their own Content Type. |
| Date or date+time | Datetime | Use for `publishedAt`, event dates, embargo dates. |
| Geographic coordinates | Geo | Use for location-based content (store locator, event venue). |
| Link to another object of a known type | Relation | Always an array in the schema, even for a single related object — cap it with the field's multiplicity setting. |
| Image / file / gallery | Media (relation to `_media`) | Configure `multiple` for single-asset vs. gallery fields. |
| Email address | Email | Gets basic format validation for free. |

## Decision rules

* **If the value is reused elsewhere or has its own identity** (an author, a category, a product brand), it's a relation to its own Content Type — not a text or select field.
* **If the value is a closed, small set of options that never need editor-added entries**, a Select/Radio field is fine. If editors need to add new options without a schema change, make it a relation to a Content Type instead.
* **If it's an image, file, or set of files**, it's always a Media relation — never a plain text URL field. This is what unlocks the media library, alt text, and Flotiq's image transformations.
* **If content needs a flexible, editor-composed layout** (mixed text/image/quote sections), use a Block field rather than one giant rich text field.

See also: [`relations-and-media.md`](relations-and-media.md) for how to configure relation and media fields, and [`naming-conventions.md`](naming-conventions.md) for naming fields once you've picked their type.
