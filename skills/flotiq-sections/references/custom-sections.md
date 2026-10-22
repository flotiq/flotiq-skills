# Custom sections

A section of the app's own: when no library section in [`sections-catalog.md`](sections-catalog.md) has fitting fields, or when reshaping one would mean rewriting it anyway. It takes two parts that must match `src/rendering/sections/section-render.tsx`'s naming.

## The content type

* API name `ff_section_<name>`, snake case: `ff_section_price_list`.
* Label `FF Section/<Name>`, so it sits with the other sections in the panel.
* A field for everything the section shows. Where a field means the same as in the library, give it the library's shape, so editors and the library's components know it: `heading`, `description`, `cta_buttons` {label, external_link, internal_link}, media relations to `_media`.
* Simple repeated items (slides, tiles, entries) are a list field of the section (`inputType: "object"`), like the library's `features`, `faq` or `logos` — `slides` {image, video, heading, description, cta_buttons}. A content type of their own only for things used in more places than this section.
* Video is a `_media` relation too: the library's `flotiq-media` renders a video file as `<video>`.
* A `variant` select only when the section has more than one look.
* Then regenerate the types (`flotiq-typegen`), and the section objects go on pages like any other section.

## The component

`section-render` turns the content type into a folder and file: drop `ff_`, drop `section_`, `_` → `-`, lowercase. `ff_section_price_list` → `src/sections/price-list/price-list.tsx`, with a default export typed by the generated type:

```tsx
import type { FfSectionPriceListHydrated } from "@flotiq/flotiq-api-sdk";

export default function PriceList({ heading, items }: Readonly<FfSectionPriceListHydrated>) {
  return (
    <section>
      <h2>{heading}</h2>
      <ul>
        {items?.map((item) => (
          <li key={item.name}>
            {item.name} — {item.price}
          </li>
        ))}
      </ul>
    </section>
  );
}
```

With a `variant`, the default export switches on it and each look lives in `variants/<name>-<variant>.tsx`, like the library's sections.

Markup and styles follow the design. For links and media use the library's `@/components/ui/flotiq-link` (an `internal_link` becomes the page's slug) and `@/components/ui/flotiq-media` (image or video from `_media`), if the app has them; the rest of the library's components are optional.

## Verification

* A page with the new section shows it, not "Section ff_section_… not found".
* Every field of the type is shown; no visible content is hardcoded.
* Typecheck passes with the generated `FfSection…Hydrated` type — no handwritten copy of it.
