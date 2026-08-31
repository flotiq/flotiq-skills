# Relations and media fields

## Relations

* Relation fields describe a link from one Content Type to another. In the underlying schema, a relation is always an array — even when only one related object is allowed — so cardinality is controlled by the field's multiplicity setting, not the schema shape.
* **Model a relation when data is shared, reused, or has its own lifecycle** (an author edited independently of any one post, a category that exists before and after any given product). Model a **plain field** only when the value is genuinely intrinsic to that one object and never reused.
* Prefer a small number of reusable related types (`Author`, `Category`, `Tag`, `SEO`) over letting every content type grow its own copy of the same concept.
* When designing a relation, decide explicitly:
  * **Direction** — which type "owns" the relation field (usually the many side points at the one side, e.g. `BlogPost.author → Author`).
  * **Cardinality** — single related object vs. multiple (a post has one author but many tags).
  * **Required or optional** — can the object be saved without this relation set?
* Avoid circular required relations (A requires B which requires A) — they make it impossible to create the first object of either type.

## Media fields

* Media fields are relations to the built-in Media Content Type, not free-text URL or upload fields. This gets you the shared media library, reuse across objects, alt text, and Flotiq's image transformation API for free.
* Decide **single vs. multiple** deliberately: a hero image is typically a single-media relation; a gallery is a multiple-media relation. Get this right at design time — changing it later means migrating existing content.
* Don't create a separate custom Content Type to hold "an image plus some metadata" unless you need metadata beyond what the Media type already provides (alt text, title) — extend usage of Media first.
* Combined media file size and per-request upload limits are enforced by Flotiq; if a project needs large or unusual media handling, check the [media library docs](https://flotiq.com/docs/API/media-library/) rather than assuming.

## Verifying relation and media setup

After creating a CTD with relations or media fields, verify:

* The relation's target Content Type name matches an existing, correctly-spelled type.
* The multiplicity (single/multiple) matches the intended UI and content shape.
* Required relations don't create a chicken-and-egg dependency between two new types.

See [Content Type Definitions](https://flotiq.com/docs/API/content-types/) and [Media library](https://flotiq.com/docs/API/media-library/) for exact schema and API details.
