# SDK setup

## A project that already has a client

Look for it before anything else: a module exporting a `Flotiq` instance (often `flotiqApiClient` in `src/lib/flotiq-api-client.ts` or `src/flotiq-api-client.ts`), and in Next.js the routes under `src/app/api/flotiq/`. Import that client everywhere and keep the routes as they are. Don't run `flotiq-nextjs-setup` again — it would overwrite them.

## Next.js without a client

`npx flotiq-nextjs-setup` creates the client (with `@flotiq/nextjs-addon`'s middleware), the draft-mode and revalidation routes, `FLOTIQ_CLIENT_AUTH_KEY` in the env file, the generated types and, optionally, Content Preview settings in the space. It is **interactive** (prompts, a content type picker): an agent's terminal blocks on it. Ask the user to run it in their own terminal (in Claude Code: `! npx flotiq-nextjs-setup`) and wait for them to confirm it finished. Don't ask them to paste keys.

## Any other app

```bash
npm install @flotiq/flotiq-api-sdk
```

```ts
// lib/flotiq.ts — the one shared client
import { Flotiq } from "@flotiq/flotiq-api-sdk";

export const flotiqApiClient = new Flotiq(); // reads process.env.FLOTIQ_API_KEY
```

Pass `{ apiKey }` explicitly only when the key lives under another name. Then generate the types (`flotiq-typegen`).

## Environment variables

* `FLOTIQ_API_KEY` — a read-only key of the space, used by the app and by typegen. `FLOTIQ_CLIENT_AUTH_KEY` (Next.js) — a random secret guarding the draft and revalidate routes (`openssl rand -hex 32`).
* Code reads them from `process.env` only; never print, commit or rename them. Check `.env.example` (or `.env.local`) for the names the project already uses, and list any new variable there with a comment and a placeholder; its value comes from whoever owns it, never invented.

## Read vs. write keys

* The app's `FLOTIQ_API_KEY` is read-only. An app that must write (a form that saves entries) gets its own write-capable key, scoped to those content types, used only in server-side code ([`writing.md`](writing.md)) — never in the browser.
* The read-write key used to build the content model (MCP, the section library's CLI — often `FLOTIQ_RW_API_KEY`) is a developer tool: keep it in the local env only, never in the app's runtime environment.

See [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/) and [Flotiq Documentation](https://flotiq.com/docs/) for key scoping options.
