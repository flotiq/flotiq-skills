# SDK setup

## Install

```
npm install @flotiq/flotiq-api-sdk
```

## Create a client

Construct a single API client instance from the SDK, configured with the project's API key, and reuse it across the app rather than instantiating a new client per call. Keep the client construction in one shared module so every part of the app talks to Flotiq the same way.

## Environment variables

* Store the Flotiq API key in an environment variable (e.g. `FLOTIQ_API_KEY`), never in source.
* Next.js projects set up via `npx flotiq-nextjs-setup` get `FLOTIQ_CLIENT_AUTH_KEY` auto-generated in `.env.local` — this powers draft mode and cache revalidation. Don't rename or hand-roll this if the framework integration already provides it.
* Check for an existing `.env.local` / `.env.example` before introducing a new variable name — match the project's existing convention.
* Never commit real API keys or auth keys; commit an `.env.example` with placeholder values if the project doesn't already have one.

## Read vs. write keys

Use a read-only or appropriately scoped API key for applications that only display content. Reserve write-capable keys for server-side code that legitimately creates/updates content (e.g. a form handler), and never expose a write-capable key to the client/browser.

See [Flotiq SDK for Node.js/TypeScript](https://flotiq.com/docs/API/generate-package/sdk-nodejs/) and [API key management](https://flotiq.com/docs/) for exact key scoping options.
