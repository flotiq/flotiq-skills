# Testing the skills in a real project

A path for checking the skills "in the field": a real agent, a real Flotiq space and a real Next.js app. The stages follow the dependency chain: **schema → typegen → content → sdk → sections**. `plugins` is independent and can be tested on its own.

Every stage has three checks. Record the result in the [results log](#results-log) at the end.

| Check | Question | How to check |
|---|---|---|
| **T: triggered** | Did the agent load the right skill on its own? | The transcript has a `Skill(flotiq-…)` call, and the prompt didn't name the skill. |
| **F: followed** | Did the agent do what the skill requires? | The "Check" items under each stage (taken from the *Common mistakes* and *Completion checklist* sections of SKILL.md). |
| **W: works** | Does the result actually work? | Something measurable: a CTD re-fetch, `flotiq-api.d.ts`, `next build`, a page opening at its slug. |

Write prompts naturally ("design the types for a blog"), **never** "use flotiq-schema". Otherwise check T measures nothing. Run every stage in a new session (or after `/clear`), so the agent doesn't rely on the previous stage's context.

---

## Stage 0: Setup

The tests are only meaningful if this stage is done right.

### 0.1 A separate Flotiq space

Create a new, empty space just for testing. The skills create and replace CTDs, and the MCP server has no delete tool. Don't test on a space with real data.

- [ ] The test space exists and is empty (apart from `_media`).
- [ ] You have two keys: read-only (`FLOTIQ_API_KEY`) and read-write (`FLOTIQ_RW_API_KEY`).

### 0.2 A test project outside this repo

An agent started inside `flotiq-skills` reads SKILL.md straight from `skills/`, which hides whether a skill triggers on its own.

```sh
cd ~/projects
npx create-next-app@latest flotiq-skills-test --ts --eslint --tailwind --app --src-dir --import-alias "@/*" --use-npm
cd flotiq-skills-test
git log --oneline   # create-next-app makes the first commit itself
```

- [ ] The project builds (`npm run build`) and its starting state is committed, so `git diff` shows every change the agent makes. Add `.claude/` and `.env` to `.gitignore`.

### 0.3 Skills from this branch, not from GitHub

`npx skills add flotiq/flotiq-skills` pulls `main`, so it does **not** test the current branch. Install the skills from the working tree of the branch under test (`git pull` it first):

```sh
SKILLS_DIR=~/projects/flotiq-skills/skills
mkdir -p .claude/skills
for s in "$SKILLS_DIR"/*/; do ln -sfn "$s" ".claude/skills/$(basename "$s")"; done
ls -l .claude/skills
```

With symlinks, a fix in the repo takes effect in the test project right away. After changing a SKILL.md, start a new agent session.

- [ ] `.claude/skills/` holds 6 skills pointing at the repo.
- [ ] `grep -r flowbite .claude/skills/` finds nothing (the skills are the renamed version).
- [ ] At the start of a Claude Code session the `flotiq-*` skills are visible (`/skills`, or ask "what skills do you have?").

> Before a release, run the whole test once more with the skills installed the way a user installs them (`npx skills add flotiq/flotiq-skills --agent claude-code --skill '*' --yes` after merging to `main`). That also tests the installation itself.

### 0.4 Flotiq MCP and keys

```sh
cat > .env <<'EOF'
FLOTIQ_API_KEY=<read-only key>
FLOTIQ_RW_API_KEY=<read-write key>
EOF
claude mcp add --transport http flotiq https://mcp.flotiq.com/mcp --header "X-AUTH-TOKEN: <read-write key>"
```

(Check the Flotiq MCP docs for the header it expects, or use OAuth.)

- [ ] `claude mcp list` shows `flotiq` as connected.
- [ ] The prompt "how many content types are in my Flotiq space?" calls `data_efficient_list_ctd` and returns the right answer.

---

## Stage 1: flotiq-schema

> **Not run on 2026-10-28:** the Flotiq MCP server (`mcp.flotiq.com/mcp`) returned 503 all day. The whole stage depends on it.

**Prompt 1a:**
> I'm building a company blog. Posts have authors (name, bio, photo) and categories. Design and create the content types in Flotiq.

Check:
- [ ] **T:** `flotiq-schema` loaded.
- [ ] It called `data_efficient_list_ctd` first (looking for existing types).
- [ ] Author and category are separate types, and the post relates to them (no copied `author_name` fields).
- [ ] Every type has a human-readable title field, and the post has a `slug`.
- [ ] The photo is a relation to `_media`, not a URL or text field.
- [ ] Creation order: the types others relate to first, then the post.
- [ ] After creating, it re-fetched the types with `get_content_type`.
- [ ] At the end it mentioned regenerating the types (`flotiq-typegen`).
- [ ] **W:** in the Flotiq panel the types look the way the agent described them.

**Prompt 1b (the riskiest step):**
> Add a "reading time in minutes" field to the blog post.

- [ ] It started from `get_content_type` and sent the **complete** field set (`update_content_type` replaces all fields).
- [ ] It didn't use `allow_property_removal`.
- [ ] **W:** `get_content_type` after the change has **all** the old fields plus the new one.

**Prompt 1c (a breaking change):**
> Rename the post's `title` field to `headline`.

- [ ] The agent warns that this is a breaking change (existing content, generated types) and suggests adding a field instead of renaming, or asks before doing it.

---

## Stage 2: flotiq-typegen

**Prompt 2:**
> Generate TypeScript types for the Flotiq content.

Check:
- [ ] **T:** `flotiq-typegen` loaded.
- [ ] It ran `npx flotiq-api-typegen` (or the project's script) and didn't write types by hand.
- [ ] It suggested keeping `flotiq-api.d.ts` out of lint (`globalIgnores`).
- [ ] It suggested committing the generated file.
- [ ] **W:** `flotiq-api.d.ts` has the types from stage 1, including the reading-time field.

**Negative test:** temporarily remove `FLOTIQ_API_KEY` from `.env` and repeat the prompt.
- [ ] The agent says the key is missing and does **not** write the types by hand.

---

## Stage 3: flotiq-content

> **Not run on 2026-10-28:** the Flotiq MCP server (`mcp.flotiq.com/mcp`) returned 503 all day. The whole stage depends on it.

**Prompt 3:**
> Add sample content to the blog: 2 authors, 3 categories and 5 posts. Publish them.

Check:
- [ ] **T:** `flotiq-content` loaded.
- [ ] It read the schema before creating objects and checked `list_objects` (no duplicates).
- [ ] Authors and categories first, then posts with relations to **existing** IDs.
- [ ] The content is realistic and varied (no "Sample Post 1", no lorem ipsum).
- [ ] Media: it used existing media, or left the fields empty and listed the missing images in its report (no invented URLs).
- [ ] Publishing was deliberate (`cascade`), and the result was re-fetched.
- [ ] **W:** in the panel every relation works and the objects are published.

**Prompt 3b (a second run):**
> Add 2 more posts in the existing categories.

- [ ] It reused the existing categories and authors (no new duplicates).

---

## Stage 4: flotiq-sdk

> **Not run on 2026-10-28:** the Flotiq MCP server (`mcp.flotiq.com/mcp`) returned 503 all day. Run against the existing `step` type instead of a blog: 4a and 4b. Not run: 4c (`step` has no relations), 4d (needs a new type through MCP), 4e (`step` has no Draft & Public).

**Prompt 4a (setup):**
> Connect this Next.js app to Flotiq.

- [ ] **T:** `flotiq-sdk` loaded.
- [ ] The agent does **not** run `npx flotiq-nextjs-setup` itself; it asks you to (the tool is interactive). Run it with `! npx flotiq-nextjs-setup`.
- [ ] After setup there is one shared client (`src/lib/flotiq-api-client.ts` or similar) and the `api/flotiq/draft` and `api/flotiq/revalidate` routes.

**Prompt 4b (reading):**
> Make a /blog page listing the posts (title, author, category, image) and a /blog/[slug] page with the post's details.

- [ ] It uses the shared client and the SDK, no raw `fetch()`.
- [ ] It uses `hydrate`, with no follow-up fetches for author and category (no N+1).
- [ ] Types come from `flotiq-api.d.ts`: no handwritten interfaces, `any` or casts.
- [ ] Images go through `getMediaUrl()` (not `media.url`), and `next.config` has `remotePatterns` for `api.flotiq.com`.
- [ ] No per-route `revalidate`, `dynamic` or `cache` on top of the client's caching.
- [ ] **W:** `npm run build` passes; with `npm run dev`, `/blog` and `/blog/<slug>` show the content with images.

**Prompt 4c (filtering by a relation):**
> Add a /blog/category/[slug] page with the posts in that category.

- [ ] The filter is `"category[*].dataUrl": { type: "includes", filter: "/api/v1/content/…" }`, not `equals` with a bare ID.
- [ ] **W:** the category page shows the right posts (not an empty list).

**Prompt 4d (writing):**
> Under each post, add a "suggest a topic" form that saves the suggestion in Flotiq with a relation to the post.

- [ ] The agent creates the type first (`flotiq-schema`) and regenerates the types.
- [ ] The write runs server-side only (Server Action / route), with its own key that never reaches the browser.
- [ ] The relation link is `{ type: "internal", dataUrl }`.
- [ ] Dates are `YYYY-MM-DD` or `YYYY-MM-DDTHH:mm` (not `toISOString()`).
- [ ] **W:** after submitting the form, the object shows up in the panel with a working relation. `.next/static` doesn't contain the read-write key (`grep -r "<start of the key>" .next/static` finds nothing).

**Check 4e (draft mode):**
- [ ] **W:** change a post in the panel without publishing. `/api/flotiq/draft?key=<FLOTIQ_CLIENT_AUTH_KEY>` shows the draft; without draft mode you see the published version.

---

## Stage 5: flotiq-sections

> **Not run on 2026-10-28:** the Flotiq MCP server (`mcp.flotiq.com/mcp`) returned 503 all day. Run: 5a without content objects and pages (the CLI creates the types through REST) and 5c. Not run: 5b (a new type needs MCP) and the content part of 5a.

Preferably on a **second**, fresh project (repeat steps 0.2–0.4), to test the first setup from scratch. Optionally use the project from stage 4, which adds a test of mixing sections with the blog.

**Prompt 5a:**
> Build a company website for the architecture studio "Linia": a home page (menu, hero, services, client testimonials, FAQ, footer) and an "About us" page (hero, team, map, footer).

Check:
- [ ] **T:** `flotiq-sections` loaded (not just `flotiq-schema`).
- [ ] `npx -p @flotiq/flotiq-sections-next flotiq-sections init --yes --defaults`, then `add` with only the sections needed (`navbar hero features testimonials faq footer team map`).
- [ ] After `init`, the agent found and read `.github/skills/flotiq-sections-setup/SKILL.md`.
- [ ] It left `flotiq-nextjs-setup` to you, with the answers given (Yes / Page / No / Yes).
- [ ] `tsconfig` has `target: "ES2018"`, `next.config` has `remotePatterns` for `/image/**`, and `next typegen` ran before the typecheck.
- [ ] One navbar object and one footer object, used on both pages.
- [ ] The home page has an empty `slug`; "About us" has `about` (or similar).
- [ ] Pages published with `cascade: true`.
- [ ] Code: `src/lib/pages.ts` with `getPage` (`hydrate: 2`), `src/app/[...slug]/page.tsx` with `notFound()`, and `src/app/page.tsx`.
- [ ] Stylesheet: the line `@custom-variant dark (&:where(.dark, .dark *));` **replaced** the old one, not added as a second.
- [ ] With `cacheComponents: true` (the default in a new create-next-app): `getPage` has `"use cache"` and `cacheTag`, and the catch-all has `generateStaticParams`.
- [ ] No visible text, image or link is written into a section's code.
- [ ] **W:** `npm run build` passes. `/` and `/about` show every section, none "Section … not found". The menu link to "About us" works.

**Prompt 5b (a section of your own):**
> Add a "Projects" section to the home page: a grid of projects with an image, name, year and city.

- [ ] The agent chooses between a library section and its own (`ff_section_<name>`) per `custom-sections.md`.
- [ ] The component is `src/sections/<name>/<name>.tsx`, typed with `FfSection<Name>Hydrated`, and shows **every** field of the type.
- [ ] **W:** the section renders on `/`.

**Prompt 5c (the look from a design):**
> Brand colors: background #0f1115, text #e8e6e1, accent #c8a96a. The site has a dark look only.

- [ ] The values went into the existing `:root` **and** `.dark` blocks (no additional block).
- [ ] **W:** in the browser the library sections pick up the new colors right away.

---

## Stage 6: flotiq-plugins (independent)

**Prompt 6:**
> Build a Flotiq panel plugin: a "Copy link" button on the blog post list that copies the post's public URL (`https://my-site.com/blog/<slug>`). The domain should be set in the plugin's settings.

Check:
- [ ] **T:** `flotiq-plugins` loaded.
- [ ] Events come from `references/events-and-ui.md`, no invented names.
- [ ] `::render`/`::add` return synchronously (not `async`) and return `null` for other types.
- [ ] The settings schema has the full CTD shape (`allOf`, `metaDefinition.order`, `propertiesConfig`).
- [ ] The manifest asks only for the permissions it needs (no `'*'`).
- [ ] No keys in the code, `.env` or the manifest.
- [ ] **W:** `FlotiqPlugins.loadPlugin(...)` in the panel's console loads the plugin, the button shows only on blog posts, and copying works.

---

## Stage 7: Boundaries between skills (negative tests)

Each prompt in a new session. Check **T** matters most here.

| Prompt | Expected | Should not load |
|---|---|---|
| "Build a loan calculator in this app, keep the rates in Flotiq" | `flotiq-schema`, `flotiq-sdk` | `flotiq-sections` |
| "Add a blog to the site built from sections" | `flotiq-schema`, `flotiq-sdk` | `flotiq-sections` as the blog's data model |
| "Add a hero section" (with `src/sections/hero/` already there) | no second `add`, no `--overwrite` | — |
| `add` of a section whose type in the space differs from the library's | the CLI skips the type ("Skipped"); the agent reports it and doesn't work around it | — |
| "Fix the typo in the title of post X" | `flotiq-content`, only that field changed | `flotiq-schema` |
| "Fetch the posts from the Flotiq API with fetch" | `flotiq-sdk`, switching to the SDK (or explaining why not) | — |

---

## Results log

One row per run. Turn every ❌ into a concrete SKILL.md change in the "What to fix" column, then rerun that stage.

Legend: ✅ pass · ⚠️ partial · ❌ fail · ⏭ not run · ⏸ needs a human (browser).

| Date | Agent / model | Stage / prompt | T | F | W | What went wrong | What to fix in the skill |
|---|---|---|---|---|---|---|---|
| 2026-10-28 | Claude Code / Opus 5.5 | 0 setup | — | — | ✅ | MCP 503; `flotiq-nextjs-setup` failed at the last step (saving Content Preview: Cloudflare block on flotiq.com) | — (Flotiq side) |
| 2026-10-28 | Claude Code / Opus 5.5 | 1 schema | ⏭ | ⏭ | ⏭ | not run: no MCP | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 2 typegen | ✅ | ✅ | ✅ | — | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 2 typegen without the key | ✅ | ✅ | ✅ | — | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 3 content | ⏭ | ⏭ | ⏭ | not run: no MCP | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 4a sdk setup | ✅ | ✅ | ✅ | — | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 4b sdk read (`step`) | ✅ | ✅ | ✅ | filter applied in the app rather than in the query (a deliberate, explained choice) | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 4c–4e | ⏭ | ⏭ | ⏭ | not run: no relations / no MCP / no Draft & Public | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 5a sections (code + types) | ✅ | ✅ | ✅ | Cache Components + zero pages: an empty `generateStaticParams` fails the build; the agent invented a `_placeholder` slug, and `/` was cached as not-found for a day | `flotiq-sections`: say what to do before any page exists |
| 2026-10-28 | Claude Code / Opus 5.5 | 5b custom section | ⏭ | ⏭ | ⏭ | not run: no MCP | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 5c colors | ✅ | ✅ | ✅ | — | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 6 plugin | ✅ | ⚠️ | ⏸ | guessed the CTD API name `krok` (it's `step`) instead of checking; not loaded in the panel | `flotiq-plugins`: check the CTD's API name before using it |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.1 calculator | ✅ | ✅ | — | schema + sdk, no sections | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.2 blog on a sections site | ✅ | ✅ | — | schema + sdk, no sections | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.3 hero already exists | ✅ | ✅ | — | no `add`/`--overwrite`; plans content only | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.4 CTD differs in the space | ⏭ | ⏭ | ⏭ | not run | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.5 typo in an entry | ✅ | ✅ | — | content only; found no typo and asked | — |
| 2026-10-28 | Claude Code / Opus 5.5 | 7.6 raw fetch requested | ✅ | ✅ | — | did what was asked, noting that the SDK is preferred | — |

---

## Other agents

The same scenario can be repeated in Copilot (VS Code) or Cursor: link the skills into their directory (`.github/skills/`, `.cursor/skills/`; the paths are in the [Skills CLI agent list](https://github.com/vercel-labs/skills#supported-agents)). Check T matters most, because each agent picks skills from `description` a little differently.

## Automation (optional, later)

Check T can be automated, without clicking through:

```sh
claude -p "I'm building a company blog… (prompt 1a)" --output-format stream-json --verbose \
  | grep -o '"skill":"flotiq-[a-z]*"' | sort -u
```

A bigger step is an eval suite (`claude plugin eval`) that runs the prompts from stages 1–7 on every skill change and checks which skills load. It pays off once the manual path stops finding new problems.
