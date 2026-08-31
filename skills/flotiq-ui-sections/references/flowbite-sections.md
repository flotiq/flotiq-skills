# Flowbite Sections

Render built-in Flotiq Flowbite sections with `SectionRender`.

## Package

`@flotiq/flotiq-flowbite-next` exports `SectionRender` and the built-in section components.

## `SectionRender` requirements

* `sectionData` must include `id` and `internal.contentType`.
* Built-in content types map automatically to components.
* `customSections` can override a built-in component or add a new content type.
* The Flotiq API request must use `hydrate: 1` or `hydrate: 2`.

## Built-in sections

| contentType | Component |
|---|---|
| `ff_section_hero` | `Hero` |
| `ff_section_content_block` | `ContentBlock` |
| `ff_section_features` | `Features` |
| `ff_section_faq` | `Faq` |
| `ff_section_footer` | `Footer` |
| `ff_section_logotypes` | `Logotypes` |
| `ff_section_navbar` | `Navbar` |
| `ff_section_testimonials` | `Testimonials` |

## Implementation

### 1. Hydrate the API response

```ts
flotiqApiClient.content.page.list({ hydrate: 1, ... })
```

### 2. Import the renderer

```ts
import { SectionRender } from "@flotiq/flotiq-flowbite-next";
```

### 3. Bridge generated types

`SectionRender` expects `Record<string, unknown> & { id; internal.contentType }`, while generated Flotiq types are narrower. Bridge them locally:

```ts
type SectionData = {
  id: string;
  internal: { contentType: string };
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  [key: string]: any;
};
```

### 4. Render sections

```tsx
{page.sections?.map((section) => (
  <SectionRender
    key={section.id}
    sectionData={section as SectionData}
  />
))}
```

## TypeScript notes

* Flotiq section types such as `FfSectionHeroHydrated` come from the generated API types.
* `Page.sections` is typically a union array, so each item is cast to the local `SectionData` bridge.
* The index signature is intentional because `SectionRender` expects a record-like object.

## Verification

1. Run the local app and confirm sections render as components, not JSON.
2. Check that there are no `Section ... not found` warnings in the console.
3. Run the project's TypeScript check.