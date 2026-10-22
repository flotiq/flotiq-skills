---
name: flotiq-sections
description: Use when a website's pages are built from sections whose content lives in Flotiq — landing pages, company sites, product and service pages, marketing or campaign pages, "about us" and contact pages (navbar, hero, features, FAQ, team, testimonials, contact, footer…), including a site made from a design. Covers the section library's CLI (`@flotiq/flotiq-flowbite-next`), the content types and content, custom sections, and the routes, components and styles in a Next.js app. Not for app-like UIs or lists of entities (see flotiq-schema, flotiq-sdk).
---

# Flotiq sections

A website from sections follows one pattern; everything else is up to the app:

1. **A page** is a Flotiq `page` object: `slug`, SEO fields and `sections`, an ordered relation to section objects.
2. **A section** is an object of a `ff_section_<name>` content type. Everything a visitor sees in it — texts, images, links, repeated items — is in its fields, in Flotiq, not in the code.
3. **Its component** is `src/sections/<name>/<name>.tsx` (default export), typed with the generated `FfSection<Name>Hydrated`.
4. **Rendering**: `getPage` (`hydrate: 2`) → `src/rendering/page-render.tsx` → `src/rendering/sections/section-render.tsx`, which finds the component by the content type's name.
5. **Links to pages** are `internal_link` relations to a `page` (or `external_link` URLs).

The section library is a head start, not a constraint: its CLI copies a section's code into `src/sections/` and creates its content type, and from then on both are the app's — change them as freely as your own. A section of your own ([`references/custom-sections.md`](references/custom-sections.md)) is just as good a choice; take whichever gets to the result faster.

## When to use sections

Use them for **pages of a website**: a landing page, a company or product site, service, marketing, campaign, "about us" or contact pages — content an editor arranges from blocks.

Don't use them for **an app's own UI** (a quiz, a calculator, a dashboard, a form flow) or for **lists of entities** (blog posts, products, events): those get their own content types (`flotiq-schema`) and code (`flotiq-sdk`). A site can mix both: a company site from sections with a blog of its own content type, linked from the navbar.

## The section library's CLI

The library is `@flotiq/flotiq-flowbite-next` (1.1+): a CLI over a shadcn registry. It ships source files and content type definitions, not importable components — there is no `SectionRender` to import from the package.

<!-- private-registry -->
> **Private registry (temporary).** Until the package is published on npm, it lives in the cdwv GitLab registry. A user-level `~/.npmrc` (never the repo's) needs:
>
> ```ini
> @flotiq:registry=https://git.cdwv.pl/api/v4/projects/1115/packages/npm/
> //git.cdwv.pl/api/v4/projects/1115/packages/npm/:_authToken=<token with read_api, read_registry>
> ```
>
> Without it, `npx` answers 404 for the package: ask the user to set it up, don't look for a workaround.
<!-- /private-registry -->

The CLI reads `FLOTIQ_API_KEY` (read-only: compares CTDs, generates types) and `FLOTIQ_RW_API_KEY` (imports CTDs into the space) from the environment or `.env`; without them it prompts interactively.

```bash
npx -p @flotiq/flotiq-flowbite-next flotiq-flowbite init --yes --defaults   # once per app
npx -p @flotiq/flotiq-flowbite-next flotiq-flowbite add hero features footer --yes
```

(Installed as a dev dependency, the same commands are `pnpm flotiq-flowbite …` / `npx flotiq-flowbite …`.)

* `init` sets up shadcn (`components.json`, its theme in the global stylesheet, a font in the root layout), the page renderer (`src/rendering/`) and the `page` type, and regenerates the types. It also copies the library's own setup skill, `flotiq-flowbite-setup`, to `.github/skills/`; pass `--no-skill` if the project doesn't want it.
* `add` copies the sections and the components, hooks and lib files they use, creates their content types and regenerates `flotiq-api.d.ts`. Names can be `hero` or `ff_section_hero`; `--all` adds the whole registry.
* Inspect before changing anything: `add hero --dry-run`, `--diff` (also diffs the CTD against the space), `--view`. `--skip-typegen` skips type generation.
* Add only sections the app doesn't have: one already in `src/sections/<name>/` is the app's code — never add it again, and never pass `--overwrite` unless the user asked to replace their version.
* A content type that differs in the space is not overwritten (the default answer is No; "… Skipped"): don't work around it — it may be another app's type; say so in your report.

## First setup

If the project has the library's setup skill (`.github/skills/flotiq-flowbite-setup/SKILL.md`), read that file and follow it for the first setup (agents that load skills from `.claude/skills/` won't find it on their own): it is interactive and walks the user through `flotiq-nextjs-setup` and Content Preview.

Otherwise, after `init`:

1. The project needs a shared Flotiq client and the draft/revalidate routes (`flotiq-sdk`). If it has none, the user runs `npx flotiq-nextjs-setup` themselves — it is interactive and blocks an agent's terminal (answers: integration `Yes`, content type `Page`, example `/page/[slug]` page `No`, Content Preview `Yes`).
2. `tsconfig.json`: `compilerOptions.target` `"ES2018"` (the library's source needs it).
3. `next.config.*`: merge `{ protocol: "https", hostname: "api.flotiq.com", pathname: "/image/**" }` into `images.remotePatterns` (`flotiq-media` uses `next/image`).
4. Run `next typegen` before the first typecheck: `shadcn init` may use the generated `LayoutProps`/`PageProps`, which don't exist until then. Don't replace them with handwritten props.

## Content model and content (Flotiq)

1. Plan the pages and their sections. From a design (an export, screenshots), cut each page into sections, top to bottom. For each section take either a library section from [`references/sections-catalog.md`](references/sections-catalog.md) — when its fields fit or nearly fit, whatever it looks like now — or a type of your own. One section type serves many pages (two heroes are two `ff_section_hero` objects).
2. Add the library sections with the CLI (above): `init` once, then `add` with their names.
3. Shape each type to its section: add, remove or change fields of a library type, or create your own type (see `flotiq-schema`), then regenerate the types (`flotiq-typegen`). Every text, image, link and list the section shows gets a field. Before removing or retyping a field that already holds content, move that content.
4. Create the section objects with real content, then the `page` objects, their `sections` relations in page order (`flotiq-content`). The home page has an empty `slug`; others a path like `about` or `offer/web`.
5. Publish the pages with `cascade: true`, so their sections are published too.
6. When someone else writes the code, hand them: that the site is built from sections; each page's slug and its sections in order; for every section type that is new or whose fields changed, its component path and the fields the component must show; and the next step — "routes, components and styles per the `flotiq-sections` skill".

Images: media fields are optional. Without fitting media in the space leave them empty and list which images are needed (see `flotiq-content`).

## Code (routes, components, styles)

Once, after `init` (skip what's already there). Import the project's shared Flotiq client from wherever it lives (`flotiq-sdk`); `@/lib/flotiq-api-client` below is an example path.

1. `src/lib/pages.ts` — read a page with its sections:

   ```ts
   import type { PageHydratedTwice } from "@flotiq/flotiq-api-sdk";
   import { flotiqApiClient } from "@/lib/flotiq-api-client";

   /** The Flotiq `page` with this slug ("" is the home page), its sections hydrated. */
   export async function getPage(slug: string): Promise<PageHydratedTwice | undefined> {
     const response = await flotiqApiClient.content.page.list({
       limit: 1,
       hydrate: 2,
       filters: { slug: slug ? { type: "equals", filter: slug } : { type: "empty" } },
     });
     return response.data[0] as PageHydratedTwice | undefined;
   }
   ```

2. `src/app/[...slug]/page.tsx` — the pages from sections:

   ```tsx
   import type { Metadata } from "next";
   import { notFound } from "next/navigation";
   import { getPage } from "@/lib/pages";
   import PageRender from "@/rendering/page-render";

   export async function generateMetadata({ params }: PageProps<"/[...slug]">): Promise<Metadata> {
     const { slug } = await params;
     const page = await getPage(slug.join("/"));
     return { title: page?.seo_title, description: page?.seo_description };
   }

   export default async function SectionPage({ params }: PageProps<"/[...slug]">) {
     const { slug } = await params;
     const page = await getPage(slug.join("/"));
     if (!page) notFound();
     return <PageRender page={page} />;
   }
   ```

   The app's own routes (`/blog`, `/products`) still win: Next.js matches them before the catch-all.

   **With Cache Components** (`cacheComponents: true` in `next.config.*` — `create-next-app` 16.4+ turns it on), the build fails on this route ("uncached or runtime data during prerendering"). Cache the reads and list the slugs:

   ```ts
   // src/lib/pages.ts — getPage as above, plus three lines at its top
   import { cacheLife, cacheTag } from "next/cache";

   export async function getPage(slug: string): Promise<PageHydratedTwice | undefined> {
     "use cache";
     cacheTag("flotiq-content", "flotiq-content-page"); // the revalidate route's revalidateTag still clears it
     cacheLife("days");
     // …the same list() call
   }

   /** Every page's slug except the home page's. */
   export async function getPageSlugs(): Promise<string[]> {
     "use cache";
     cacheTag("flotiq-content", "flotiq-content-page");
     cacheLife("days");
     const response = await flotiqApiClient.content.page.list({ limit: 1000 });
     return response.data.map((page) => page.slug ?? "").filter(Boolean);
   }
   ```

   ```tsx
   // src/app/[...slug]/page.tsx — next to generateMetadata
   export async function generateStaticParams() {
     return (await getPageSlugs()).map((slug) => ({ slug: slug.split("/") }));
   }
   ```

   Pages published later still render (on demand). A missing slug shows the not-found page, with status 200: the response is already streaming.

3. `src/app/page.tsx` — the home page from sections, when the site's home page is a section page: the same as above with `getPage("")`. An app with a home page of its own keeps it, and its section pages live under the catch-all. If the build must work without `FLOTIQ_API_KEY`, guard both routes the way the project already does.

4. The global stylesheet — `init` replaced it with shadcn's theme. Change one line, so a section's dark mode follows a `dark` class (on a section or an ancestor, `<body>` for the whole site), not only its descendants: `@custom-variant dark (&:is(.dark *));` → `@custom-variant dark (&:where(.dark, .dark *));`. Replace the existing declaration; don't add a second one.

Then, for every new or changed section:

* **Components.** A new type gets its component per [`references/custom-sections.md`](references/custom-sections.md); a changed type gets its component updated. Every field of the type is shown — a field the component ignores is content an editor fills in and never sees.
* **Look.** Change a section's files under `src/sections/<name>/` as much as the design asks: markup, classes, variants, or a rewrite. The library's components (`@/components/ui/flotiq-link`, `flotiq-media`, `section-wrapper`…) are there to use, not to follow.
* **Styles.** The library's sections use shadcn's variables (`bg-primary`, `text-muted-foreground`, `border-border`…): put the design's colors into them, so those sections match at once. Add the design's own tokens and fonts alongside. Do this after `init`: it replaces the stylesheet and the font in the layout.
  * Change the values in the `:root` and `.dark` blocks `init` wrote; never add another block that sets them (`:root, .dark { … }` above or below): of two, the later one wins and the other's colors never show.
  * A design with one look (only dark, say) has no light and dark mode: its colors go into `:root` and the same into `.dark`. `<body>` and every section without a background of its own stand on `--background`, and their text is `--foreground`.
* **Server components.** Pages and sections are server components. A library that creates a React context when it loads (an icon set such as `@phosphor-icons/react`, many UI kits) fails there at runtime only, while lint, typecheck and build pass. Import its server build (`@phosphor-icons/react/ssr`) or use it in a `"use client"` component.

Then run the project's lint, typecheck and tests.

## Rules

* Content lives in Flotiq: no text, image or link a visitor sees is written into a section's code. Only decoration (icons, shapes, backgrounds that aren't content) is code.
* `section-render` turns a content type into a file: drop `ff_` and `section_`, `_` → `-`. `ff_section_price_list` → `src/sections/price-list/price-list.tsx`. "Section … not found" on a page means that folder or file is missing; any other error while loading a section is thrown, so the build fails with the real error.
* `components.json` stays, though nothing imports it: the CLI reads it on every `add`. `src/components/ui/button.tsx` and `src/lib/utils.ts` stay too: `flotiq-link` styles every section's buttons and links with the shadcn Button — restyle it there to change them all at once.
* Render pages only through `src/rendering/page-render.tsx` (default export `PageRender`) with `hydrate: 2`; it skips archived sections and shows draft-mode bars in draft mode.
* Draft mode's "Edit" links read `FLOTIQ_CLIENT_AUTH_KEY` and `SPACE_SLUG` (and `FLOTIQ_EDITOR_URL`, default `https://editor.flotiq.com/s`); a page renders without them. No setup command adds `SPACE_SLUG` — the user does.
* `flotiq-api.d.ts` is generated: never edit it. A section's props come from it — no handwritten copy.

## Completion checklist

* [ ] Every page opens at its slug and shows its sections, none "not found".
* [ ] Every field of every section type is shown by its component; no visible content is hardcoded.
* [ ] The catch-all (and the home page, if it is a section page) uses `getPage` with `hydrate: 2` and `notFound()` for a missing page; with Cache Components, `getPage` is `"use cache"` and the catch-all has `generateStaticParams`.
* [ ] `next build` passes.
* [ ] Stylesheet: the `dark` variant changed, the design's colors in shadcn's `:root` and `.dark` (each set once), a one-look design's in both.
* [ ] Lint, typecheck and tests pass.

## References

* [`references/sections-catalog.md`](references/sections-catalog.md) — the library's sections, variants and fields.
* [`references/custom-sections.md`](references/custom-sections.md) — a section of your own: content type and component.

## Official documentation

* [Flotiq Content Preview](https://flotiq.com/docs/panel/SpaceSettings/content-preview/)
* [Next.js integration](https://flotiq.com/docs/Universe/nextjs/nextjs-setup)
