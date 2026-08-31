# Fetching content and hydrated relations

## Fetching

* Use the SDK's typed list/get methods against the generated types produced by `flotiq-typegen` — don't construct raw query strings or endpoints by hand when the SDK exposes a typed method for it.
* Filter and paginate through the SDK's supported query options rather than over-fetching and filtering client-side.
* By default, only fetch published content for public-facing pages; fetching drafts is a deliberate choice tied to preview/draft mode, not the default path.

## Draft & Public

Draft & Public is a per-Content-Type setting in Flotiq that enables a two-state publish workflow (draft → public). Its effect on the SDK layer:

* When **disabled** (default), objects are live immediately on create/update — there's no draft state to filter on.
* When **enabled**, the SDK returns only published objects by default. To include drafts (e.g. for preview/draft mode), pass the appropriate draft parameter per the SDK docs — this should only happen inside an authenticated draft mode context, never on public-facing routes.
* If content is published in Flotiq but not appearing in the app, check whether Draft & Public is enabled and the object was explicitly published (not just saved). Use `get_object` via MCP or the panel to confirm publish state.

For Next.js draft mode integration, see [`nextjs-integration.md`](nextjs-integration.md).

## Hydrated relations

* A "hydrated" relation returns the related object's data inline instead of just an ID/reference — request this whenever the UI renders data from the related object (an author's name on a post listing, a category's label on a product card).
* Only hydrate relations you actually render. Requesting hydration for unused relations bloats the response for no benefit.
* If a relation isn't hydrated and the UI needs its data, that's a bug to fix at the fetch call (request hydration), not by issuing a second manual fetch per object.

## Caching and revalidation

* Pick a strategy per content type based on how often it changes:
  * **Rarely-changing content** (marketing pages, settings): cache aggressively, revalidate on webhook/publish events.
  * **Frequently-changing content** (a live listing, real-time data): shorter cache windows or no caching.
* If the app uses static generation, wire revalidation through Flotiq's webhook-triggered revalidation rather than relying solely on a fixed time interval — otherwise published edits won't appear until the next scheduled rebuild.
* Document the chosen strategy near the fetch code (which cache, what invalidates it) so it's clear why a given content type behaves the way it does.

See [Flotiq Documentation](https://flotiq.com/docs/) for exact SDK method signatures and query options, and the `flotiq-nextjs-setup` output for a working revalidation example.
