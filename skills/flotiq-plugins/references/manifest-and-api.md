# Manifest, permissions, and API access

## Manifest

`plugin-manifest.json` describes the plugin and is typically also bundled/copied by the build. Keep these fields aligned with the implementation:

* `id` — globally unique, stable; prefix with the organization/company name.
* `name` — user-facing name.
* `version` — semantic version used to identify updates.
* `url` — full URL of the built JavaScript file.
* `description`, `repository` — optional metadata that should stay accurate.
* `permissions` — only the API capabilities the current implementation needs.

The manifest has no event list — event handlers are registered in code.

## Permissions

Permission entries use `type: 'CO' | 'CTD'`, a `ctdName` (`'*'` means all), and operation flags. Use the API reference names as canonical: `canRead`, `canCreate`, `canUpdate`, `canDelete`. (One getting-started example also shows `canWrite` — don't infer its behavior; verify current support before using a field absent from the `PluginPermission` API reference.)

Examples of least privilege:

* Create imported `product` objects: `{ "type": "CO", "ctdName": "product", "canCreate": true }`.
* Also read `product` objects before deduplicating: add `canRead: true` to that entry.
* Read the `product` schema: a separate `{ "type": "CTD", "ctdName": "product", "canRead": true }` entry.

Don't request wildcard access merely because it's convenient during development.

## Provided plugin client

Use the dynamic methods on `client[ctdName]`:

* `get(id)`, `list(params)`, `getVersions(id)`, `getVersion(id, version)`
* `post(object)`, `put(id, object)`, `patch(id, partialObject)`, `delete(id)`
* `getContentType()`, `putContentType(object)`
* top-level `getContentTypes(params)` and `getMediaUrl(mediaData, height, width)`

The client enforces manifest permissions. Response shapes can vary between local code/examples (`{ body, ok }` wrappers) and parts of the generated API reference (resolved domain objects) — follow whatever shape the surrounding code already uses, confirm current behavior when adding a new method, and don't silently mix the two.

Cache stable or repeated read promises where useful (relation/media lookups) to avoid duplicate parallel requests.

## Direct Flotiq REST access

Official plugin guidance is to use the provided client, not a plugin's own API key, and not direct Flotiq API calls — because a plugin's JavaScript runs in the browser, so any key it holds is exposed to whoever can load the page.

If a needed operation is genuinely missing from `client`, treat that as a capability gap to report rather than a reason to work around it with a bundled or user-supplied key. Where an application still needs the operation and no backend exists to broker it, prefer adding a small backend proxy that holds the credential server-side over having the plugin call the Flotiq API directly with a browser-held key.

If a project deliberately accepts this trade-off anyway (a scoped, revocable key, an accepted risk for that specific project), keep that decision inside the project it applies to — it's the project's tradeoff to own — and implement it carefully:

* Read the key from parsed plugin settings at request time, never from source, `.env`, the manifest, or a build-time constant.
* Use `globals.getApiUrl()` instead of hard-coding the Flotiq API origin.
* Consult current REST endpoint documentation for the exact path, body, auth mechanism, response shape, and limits — don't guess them.
* Use HTTPS, and never send the key to any origin other than the expected Flotiq API origin.
* Never log the key, include it in thrown/user-visible errors, put it in URLs unless the API contract requires it, or persist it outside plugin settings.
* Handle non-2xx responses, timeouts, cancellation, validation errors, rate limits, and partial batch success.
* Don't auto-retry non-idempotent operations unless the endpoint provides an idempotency mechanism or the duplicate-write risk is otherwise controlled.

## External APIs

Calls to import sources or third-party providers are separate from Flotiq manifest permissions — OpenAI, Google Maps, and similar credentials don't require Flotiq `CO`/`CTD` permissions; add those only for Flotiq data operations. Check CORS, the provider's browser-use policy, rate limits, data sensitivity, and whether the credential can be safely restricted by origin, API, or quota. Route an unrestricted secret through a backend instead of the browser.
