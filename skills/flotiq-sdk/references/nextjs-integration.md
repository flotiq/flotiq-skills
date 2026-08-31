# Next.js integration

## Setup

1. Create the Next.js app (Node.js 20+): `npx create-next-app@latest <path>`
2. Run the Flotiq setup wizard inside the project: `npx flotiq-nextjs-setup`. It authenticates with your Flotiq account, lets you pick a space and content types, and scaffolds the SDK client, generated types, example pages, and API routes.

Prefer this scaffolding over manually wiring the SDK into a Next.js app from scratch — it sets up draft mode and revalidation routes correctly out of the box.

## What the wizard generates

* An SDK client configured from environment variables.
* `FLOTIQ_CLIENT_AUTH_KEY` in `.env.local`, used for draft mode and revalidation auth.
* Example pages per selected content type (e.g. `app/blog_post/[slug]`) showing the fetch → render pattern for that type.
* API routes for draft mode and cache revalidation.

## Shared client module

The wizard generates a single shared client. If you're wiring the SDK manually, create one module (e.g. `lib/flotiq.ts`) and import it everywhere — never construct a new client per route or component:

```ts
// lib/flotiq.ts
import { Flotiq } from '@flotiq/flotiq-api-sdk'

// Reads FLOTIQ_API_KEY from process.env automatically
export const flotiqClient = new Flotiq()
```

## RSC fetch pattern

In React Server Components, import the shared client and use the SDK's typed list/get methods. The content type method name (e.g. `blogPost`) matches the API name from the Content Type Definition — check the generated client after running `npm exec flotiq-api-typegen`. Generated types in `flotiq-api.d.ts` are picked up by TypeScript automatically.

```ts
// app/blog/[slug]/page.tsx
import { flotiqClient } from '@/lib/flotiq'

export default async function BlogPostPage({
  params,
}: {
  params: { slug: string }
}) {
  const result = await flotiqClient.content.blogPost.list({
    filters: { slug: { type: 'equals', filter: params.slug } },
    hydrate: 1,
  })
  const post = result.data[0] // typed via generated flotiq-api.d.ts
  // render post…
}
```

If a content type method is missing on `flotiqClient.content`, typegen hasn't been run yet or the content type wasn't selected during `npx flotiq-nextjs-setup`.

## Draft mode

Draft mode is entered via the generated route, e.g.:

```
http://localhost:3000/api/flotiq/draft?key=<FLOTIQ_CLIENT_AUTH_KEY>
```

Use the framework's draft-mode-aware fetch (published-only outside draft mode, drafts included inside it) rather than branching manually on an env flag.

## Revalidation

Cache revalidation is triggered via a `POST` to the generated revalidate route with an `x-editor-key` header matching `FLOTIQ_CLIENT_AUTH_KEY`. Wire this as a Flotiq webhook (see the panel's webhook configuration) so publishing content in Flotiq invalidates the right Next.js cache automatically, instead of relying on a fixed revalidation interval alone.

## After schema changes

Any Content Type change upstream means the generated types and example pages may be stale — re-run `npm exec flotiq-api-typegen` (see the `flotiq-typegen` skill) before trusting the shapes used in fetch code.

See [Next.js integration](https://flotiq.com/docs/Universe/nextjs/nextjs-setup) for the full setup reference.
