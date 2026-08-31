---
name: flotiq-ui-sections
description: Use when building a Flotiq page from Flowbite sections or adding a custom Flotiq section. Requires `@flotiq/flotiq-flowbite-next`.
---

# Flotiq UI Sections

Render Flotiq page sections with `SectionRender` from `@flotiq/flotiq-flowbite-next`. Use built-in Flowbite sections first; register custom ones through the same loop.

## When to use

* Render `page.sections` as components instead of raw JSON.
* Add or override a section in a Flotiq page.
* Connect generated Flotiq section types to `SectionRender`.

## Workflow

1. Install or confirm `@flotiq/flotiq-flowbite-next`.
2. Fetch pages with `hydrate: 1` or `hydrate: 2`.
3. Render every item in `page.sections` with `SectionRender`.
4. Add custom components through `customSections`, then verify rendering and TypeScript.

## Rules

* Keep one sections loop; do not duplicate it for custom content.
* Hydrate sections or `internal.contentType` will be missing.
* Keep custom components outside the page file.
* Regenerate generated types; never edit them manually.

## Completion checklist

* [ ] `page.sections` renders through `SectionRender`.
* [ ] The API request hydrates sections.
* [ ] Custom sections, if any, use `customSections`.
* [ ] The project type-checks.

## References

* [`references/flowbite-sections.md`](references/flowbite-sections.md) — render built-in Flowbite sections.
* [`references/custom-sections.md`](references/custom-sections.md) — create and register custom sections.

## Official documentation

* [Flotiq Docs](https://flotiq.com/docs/)