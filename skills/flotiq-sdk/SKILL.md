---
name: flotiq-sdk
description: Use when building an application that reads or writes Flotiq content — SDK setup, API client creation, fetching content, hydrated relations, caching, and frontend integration (including Next.js). Not for designing Content Types (see flotiq-schema) or populating content (see flotiq-content).
---

# Flotiq SDK

Build applications on top of Flotiq using the official SDK: setup, fetching, hydration, caching, and frontend integration. Assumes the schema and content already exist. Strongly favors the SDK and generated types over hand-rolled alternatives. Not for schema design (`flotiq-schema`) or populating content (`flotiq-content`).

## When to use

* Setting up an app (Next.js or otherwise) to consume a Flotiq project.
* Adding or reviewing a data-fetching layer, especially one using raw `fetch()` or handwritten types.
* Deciding a caching/revalidation strategy.

## Workflow

1. Install `@flotiq/flotiq-api-sdk` and build one shared client from environment-variable credentials. Avoid raw `fetch()` unless there's a specific, documented reason the SDK can't cover.
2. Confirm `npm exec flotiq-api-typegen` has run (see `flotiq-typegen`) before writing fetch code, and use the generated types as the source of truth — no duplicate handwritten interfaces.
3. Fetch through the SDK's typed methods, requesting hydrated relations for anything the UI renders, instead of issuing manual follow-up fetches.
4. Choose a caching/revalidation strategy deliberately per content type (webhook-driven revalidation, time-based, or none).
5. For Next.js, use `npx flotiq-nextjs-setup`'s generated draft-mode/revalidation routes rather than reimplementing them.
6. Verify end-to-end against real content — no `any`/manual casting papering over a type mismatch.

## Best practices

* Official SDK over raw `fetch()`, always.
* Generated types are the source of truth — a mismatch means regenerate (`flotiq-typegen`) or fix the schema (`flotiq-schema`), not hand-edit.
* Hydrate only the relations you actually render.
* Cache with intent, and make the reason explicit per content type.
* Draft/preview flows go through the SDK's built-in support, not a bespoke client.

## Common mistakes

* Raw `fetch()` instead of the SDK, losing typed responses and consistent auth handling.
* Handwritten interfaces duplicating (and drifting from) generated types.
* Hardcoded credentials instead of environment variables.
* N+1 manual fetches for relations that should have been hydrated.
* No caching strategy for rarely-changing content, or caching so aggressively that edits never show up.
* Writing fetch code before typegen has run.

## Completion checklist

* [ ] All Flotiq data access goes through the SDK; no unexplained raw `fetch()`.
* [ ] Credentials come from environment variables.
* [ ] Generated types are current and used directly, with no duplicate interfaces.
* [ ] Rendered relations are fetched hydrated.
* [ ] Caching/revalidation was a deliberate choice, verified against real content.
* [ ] (Next.js) Revalidation route is wired as a Flotiq webhook in the panel — publishing content invalidates the cache automatically.
* [ ] (Next.js) Draft mode toggles correctly: `/api/flotiq/draft?key=<FLOTIQ_CLIENT_AUTH_KEY>` shows unpublished content.

## References

* [`references/sdk-setup.md`](references/sdk-setup.md) — installing the SDK, client, environment variables.
* [`references/fetching-and-hydration.md`](references/fetching-and-hydration.md) — fetching, hydration, caching.
* [`references/nextjs-integration.md`](references/nextjs-integration.md) — Next.js setup, draft mode, revalidation.

## Official documentation

* [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/)
* [`@flotiq/flotiq-api-sdk` on npm](https://www.npmjs.com/package/@flotiq/flotiq-api-sdk)
* [Next.js integration](https://flotiq.com/docs/Universe/nextjs/nextjs-setup)
* [Flotiq Documentation](https://flotiq.com/docs/)
