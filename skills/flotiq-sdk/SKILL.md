---
name: flotiq-sdk
description: Use when building an application that reads or writes Flotiq content — SDK setup, the shared API client, fetching content, hydrated relations, relation links and filters, media URLs, caching, draft mode and revalidation (including Next.js). Not for designing Content Types (see flotiq-schema) or populating content (see flotiq-content).
---

# Flotiq SDK

Build applications on top of Flotiq using the official SDK (`@flotiq/flotiq-api-sdk`): the client, fetching, hydration, writes, caching, and frontend integration. Assumes the schema and content already exist. Strongly favors the SDK and generated types over hand-rolled alternatives. Not for schema design (`flotiq-schema`) or populating content (`flotiq-content`).

## When to use

* Setting up an app (Next.js or otherwise) to consume a Flotiq space.
* Adding or reviewing a data-fetching layer, especially one using raw `fetch()` or handwritten types.
* Saving entries from the app (a form, a vote, an upload).
* Checking or choosing how Flotiq content is cached and revalidated.

## Workflow

1. **Find the project's client first.** Most projects have one shared Flotiq client (`src/lib/flotiq-api-client.ts`, `src/flotiq-api-client.ts`…) and, in Next.js, `api/flotiq/draft` and `api/flotiq/revalidate` routes. Use them; never construct a client per route or component. No client yet: see [`references/sdk-setup.md`](references/sdk-setup.md) — in Next.js that's `npx flotiq-nextjs-setup`, which is interactive, so the user runs it themselves.
2. Confirm `flotiq-api.d.ts` has the types you need before writing fetch code — if one is missing, regenerate (`flotiq-typegen`). The generated types are the source of truth; no duplicate handwritten interfaces.
3. Fetch through the SDK's typed methods, requesting hydrated relations for anything the UI renders, instead of manual follow-up fetches.
4. Caching: if the project already has a strategy (the Next.js addon's middleware, a revalidation webhook), keep it — don't add per-route `revalidate`/`dynamic`/`cache` options on top. Otherwise choose one deliberately ([`references/fetching-and-hydration.md`](references/fetching-and-hydration.md#caching-and-revalidation)).
5. Writes run server-side only, with their own scoped key ([`references/writing.md`](references/writing.md)).
6. Verify end-to-end against real content — no `any`/manual casting papering over a type mismatch.

## Quick reference

Everything here is in the SDK README (`node_modules/@flotiq/flotiq-api-sdk/README.md`). Don't read the SDK's compiled sources or `.d.ts` files. `<ctd>` is the CTD **API name** (`blog_post`), typed once `flotiq-api.d.ts` has been generated.

```ts
import { flotiqApiClient } from "@/lib/flotiq-api-client"; // the project's shared client
import type { BlogPost } from "@flotiq/flotiq-api-sdk"; // generated types augment the SDK module

// List: returns { data, total_count, total_pages, current_page, count }
const { data: posts } = await flotiqApiClient.content.blog_post.list({
  hydrate: 1,                     // 0 | 1 | 2: inline related objects (1 = one level)
  limit: 10,
  page: 1,                        // 1-based
  orderBy: "internal.createdAt",
  orderDirection: "desc",
  filters: { slug: { type: "equals", filter: "hello-world" } },
});

// Get one by id
const post: BlogPost = await flotiqApiClient.content.blog_post.get(postId, { hydrate: 1 }); // ids are UUIDs

// Full-text search scoped to one CTD
const found = await flotiqApiClient.content.blog_post.search({ q: "release" });
const firstHit = found.data[0]?.item; // typed as BlogPost

// Media URL for next/image (media comes from a hydrated media relation)
const src = flotiqApiClient.helpers.getMediaUrl(post.image[0], { width: 800 });

// Narrow an untyped relation or search hit
if (flotiqApiClient.helpers.instanceOf(post.category[0], "category")) {
  // post.category[0] is now typed as Category
}

// Filter by a relation: its dataUrl path, the full dataUrl, `includes` — never `equals` with a bare id
await flotiqApiClient.content.blog_post.list({
  filters: { "category[*].dataUrl": { type: "includes", filter: "/api/v1/content/category/category-1" } },
});

// Writes: server-side only, with a write client (references/writing.md)
await flotiqWriteClient.content.blog_post.create({
  title: "New post",
  slug: "new-post",
  // A relation link: `type` is always "internal" — not the content type's name
  category: [{ type: "internal", dataUrl: "/api/v1/content/category/category-1" }],
});
await flotiqWriteClient.content.blog_post.patch(postId, { title: "Renamed" });
await flotiqWriteClient.content.blog_post.delete(postId);
```

The field names above (`slug`, `image`, `category`) are examples — use the ones in your generated types. Relation fields are arrays even when they hold a single item. Without `hydrate`, they contain `{ dataUrl, type }` links, not the related objects. A content type missing on `flotiqApiClient.content` means typegen hasn't run since it was created.

## Best practices

* Official SDK over raw `fetch()`, always — the one exception is media upload, which the SDK doesn't have.
* Generated types are the source of truth. A mismatch means regenerating (`flotiq-typegen`) or fixing the schema (`flotiq-schema`), never hand-editing.
* Hydrate only the relations you actually render.
* Cache with intent, in one place (the client's middleware), not per route.
* Draft/preview flows go through the existing draft route and the SDK's middleware, not a bespoke client.

## Common mistakes

* Raw `fetch()` instead of the SDK, losing typed responses and consistent auth handling.
* Handwritten interfaces duplicating (and drifting from) generated types.
* Hardcoded credentials instead of environment variables; a write-capable key reaching the browser.
* N+1 manual fetches for relations that should have been hydrated.
* Route-level `revalidate`/`dynamic` or fetch `cache` options on top of a client that already caches.
* Writing fetch code before `flotiq-api.d.ts` has the types it needs.
* A relation link with the content type's name as `type` (`{ type: "drug", dataUrl }`). It is always `"internal"`; Flotiq rejects anything else at runtime, and TypeScript doesn't catch it.
* Filtering a relation with `equals` and a bare id: it matches nothing, silently. Filter `<field>[*].dataUrl` with `includes` and the full `dataUrl`.
* An image or file from a `_media` object's `url` (`<img src={media.url}>`): it is relative (`/image/0x0/<id>.<ext>`), so the browser asks the app for it and gets a 404. Use `getMediaUrl()` (or the section library's `flotiq-media`), or prefix it: `https://api.flotiq.com${media.url}`.
* A `dateTime` value from `toISOString()`: Flotiq takes `YYYY-MM-DD` or `YYYY-MM-DDTHH:mm`.
* Running `flotiq-nextjs-setup` in a project that already has the client and routes — it would overwrite them.
* With Next.js Cache Components on (`cacheComponents: true`), relying on the middleware's caching: reads need `"use cache"` with `cacheTag("flotiq-content", …)` ([`references/nextjs-integration.md`](references/nextjs-integration.md#with-cache-components)).

## Completion checklist

* [ ] All Flotiq data access goes through the shared SDK client; no unexplained raw `fetch()`.
* [ ] Credentials come from environment variables; write keys only in server-side code.
* [ ] Generated types are current and used directly, with no duplicate interfaces.
* [ ] Rendered relations are fetched hydrated; media URLs are absolute.
* [ ] (Writes) Relation links are `{ type: "internal", dataUrl }`; relation filters use `<field>[*].dataUrl`.
* [ ] Caching/revalidation is a deliberate, single strategy.
* [ ] (Next.js) Draft mode toggles correctly: `/api/flotiq/draft?key=<FLOTIQ_CLIENT_AUTH_KEY>` shows unpublished content.

## References

* [`references/sdk-setup.md`](references/sdk-setup.md) — installing the SDK, the client, environment variables, keys.
* [`references/fetching-and-hydration.md`](references/fetching-and-hydration.md) — fetching, Draft & Public, hydration, caching.
* [`references/nextjs-integration.md`](references/nextjs-integration.md) — Next.js: the addon's middleware, RSC pattern, draft mode, revalidation webhook.
* [`references/writing.md`](references/writing.md) — saving entries: the write client, relation links, filtering by a relation, media upload, dates.

## Official documentation

* [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/)
* [`@flotiq/flotiq-api-sdk` on npm](https://www.npmjs.com/package/@flotiq/flotiq-api-sdk)
* [Next.js integration](https://flotiq.com/docs/Universe/nextjs/nextjs-setup)
* [Flotiq Documentation](https://flotiq.com/docs/)
