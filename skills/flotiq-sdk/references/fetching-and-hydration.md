# Fetching content and hydrated relations

## Fetching

* Use the SDK's typed list/get methods against the generated types (`flotiq-typegen`) — don't construct raw query strings or endpoints by hand.
  ```ts
  const { data } = await flotiqApiClient.content.<ctd>.list({ hydrate: 1, limit: 10, filters: { slug: { type: "equals", filter: slug } } });
  const one = await flotiqApiClient.content.<ctd>.get(id, { hydrate: 1 });
  ```
* Filter and paginate through the SDK's query options rather than over-fetching and filtering client-side. Relation filters: [`writing.md`](writing.md#filtering-by-a-relation).
* Public pages fetch published content only; drafts are a deliberate choice tied to draft mode.

## Draft & Public

Draft & Public is a per-Content-Type setting that enables a two-state publish workflow (draft → public):

* **Disabled** (default): objects are live immediately on create/update — there's no draft state.
* **Enabled**: public reads return only published objects. Drafts are read only inside an authenticated draft-mode context, never on public routes. In Next.js, `@flotiq/nextjs-addon`'s middleware switches this for you ([`nextjs-integration.md`](nextjs-integration.md)).
* Content published in Flotiq but missing in the app: check whether Draft & Public is on and the object was actually published, not just saved (MCP `get_object` or the panel shows its state), and whether a cache is still serving the old response.

## Hydrated relations

* A hydrated relation returns the related object's data inline instead of a `{ dataUrl, type }` link — request it whenever the UI renders data from the related object (an author's name on a post card). `hydrate: 2` goes two levels deep (a page → its sections → their relations).
* Only hydrate what you render; unused hydration bloats the response.
* A relation the UI needs but didn't hydrate is a bug at the fetch call, not a reason for a second fetch per object.

## Caching and revalidation

If the project already has a strategy, keep it. A route-level `revalidate`, `dynamic = "force-dynamic"` or a fetch `cache` option overrides the client's caching for the whole page — and either hides edits or throws the cache away.

Without one, choose deliberately:

* **Next.js:** `@flotiq/nextjs-addon`'s middleware on the shared client — responses tagged `flotiq-content` and `flotiq-content-<ctd>`, cached for a set time, invalidated by a Flotiq webhook calling the revalidate route on every change ([`nextjs-integration.md`](nextjs-integration.md#revalidation)). Previews or staging that the webhook doesn't reach should not cache (`revalidateTime: 0`), or they never see edits.
* **Rarely-changing content** (marketing pages, settings): cache long, invalidate on webhook.
* **Frequently-changing content** (live listings): short windows or no cache.

Write the chosen strategy next to the client (which cache, what invalidates it), so the next person doesn't add a second one.

The full option list is in the SDK README (`node_modules/@flotiq/flotiq-api-sdk/README.md`).
