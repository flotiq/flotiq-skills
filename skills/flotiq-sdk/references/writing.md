# Writing content: relations, media, filters

For an app that saves entries (a form, a recording, a vote). Writes run server-side only, in a route handler, with a write-capable key — never in the browser.

## The write client

The shared client reads with the read-only `FLOTIQ_API_KEY`. Writes get one more client in its own module (e.g. `src/lib/flotiq-write-client.ts`), imported only by server-side code (route handlers, server actions), from a write-capable key scoped to the content types it writes — e.g. `FLOTIQ_WRITE_API_KEY`, listed in `.env.example`. Use the name the project already has, if any:

```ts
import { Flotiq } from "@flotiq/flotiq-api-sdk";

export const flotiqWriteClient = new Flotiq({ apiKey: process.env.FLOTIQ_WRITE_API_KEY });
```

## Relations: `type` is always `"internal"`

A relation field is an array of links, even when it holds one object. In every link `type` is the string `"internal"` — **not** the content type's name — and `dataUrl` names the object:

```ts
/** A relation link: `type` is always "internal"; the content type goes in dataUrl. */
const link = (ctd: string, id: string) => ({ type: "internal", dataUrl: `/api/v1/content/${ctd}/${id}` });

await flotiqWriteClient.content.recording.create({
  drug: [link("drug", drugId)],
  audio: [link("_media", mediaId)], // a media field is a relation to `_media`
  speakerId,
});
```

The SDK types `type` as a plain `string`, so TypeScript won't catch a wrong value; Flotiq rejects it at runtime with `Does not have a value in the enumeration ["internal","external"]`.

## Filtering by a relation

Filter on the relation's `dataUrl`, with the full `dataUrl` of the related object and `includes` (or `contains`, `overlaps`, `notContains`) — never `equals` with a bare id, which matches nothing and fails silently:

```ts
const { data } = await flotiqWriteClient.content.recording.list({
  limit: 1,
  filters: { "drug[*].dataUrl": { type: "includes", filter: `/api/v1/content/drug/${drugId}` } },
});
```

`flotiq-api.d.ts` lists these paths (`drug[*].dataUrl`) among each type's filterable fields.

## Uploading a file to the media library

The SDK has no upload, so this is the one documented raw `fetch`: a multipart `POST` to `https://api.flotiq.com/api/media` with `file` (the bytes) and `type` (`"image"` for images, `"file"` for anything else, e.g. audio). The answer is the new `_media` object; link it with `link("_media", media.id)`.

```ts
const form = new FormData();
form.append("file", blob, "recording.webm");
form.append("type", "file");
const response = await fetch("https://api.flotiq.com/api/media", {
  method: "POST",
  headers: { "X-AUTH-TOKEN": process.env.FLOTIQ_WRITE_API_KEY ?? "" },
  body: form,
});
if (!response.ok) throw new Error(`Flotiq media upload failed: ${response.status} ${await response.text()}`);
const media = (await response.json()) as { id: string };
```

## Dates

A `dateTime` field takes `YYYY-MM-DD` or `YYYY-MM-DDTHH:mm` — not a full `toISOString()` with seconds and `Z`: `new Date().toISOString().slice(0, 16)`.

See [Creating Content Objects](https://flotiq.com/docs/API/content-type/creating-co/), [Listing Content Objects](https://flotiq.com/docs/API/content-type/listing-co/) and [Media library](https://flotiq.com/docs/API/media-library/).
