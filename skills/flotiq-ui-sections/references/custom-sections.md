# Custom Sections

Create a custom Flotiq section or replace a built-in Flowbite section.

## Use this for

* A new section type should be added to the page sections loop.
* A built-in section from `@flotiq/flotiq-flowbite-next` should be overridden.
* A custom Flotiq CTD should render as a page section.

## Decide first

1. What should the section contain?
2. Is it static, or does it need fields from Flotiq?
3. If it is dynamic, does the CTD already exist?

If the CTD does not exist yet, define it in Flotiq first and regenerate types before writing the final component.

## Type generation

Generated types come from:

```bash
npx flotiq-api-typegen
```

Never edit generated API types manually. If a field is missing or wrong, fix the CTD in Flotiq and regenerate.

The generated hydrated type follows the content type name:

* `ff_section_banner` -> `FfSectionBannerHydrated`

## Build the component

Place custom section components in the project's section/components area, following the existing app structure.

### Static example

```tsx
export default function SectionBanner() {
  return (
    <section className="bg-gray-50 py-16 text-center">
      <h2 className="text-3xl font-bold">Banner</h2>
    </section>
  );
}
```

### Dynamic example

```tsx
import { FfSectionBannerHydrated } from "@/flotiq-api";

export default function SectionBanner({
  id,
  heading,
  variant,
}: Readonly<FfSectionBannerHydrated>) {
  return (
    <section id={id} data-variant={variant} className="bg-gray-50 py-16 text-center">
      <h2 className="text-3xl font-bold">{heading}</h2>
    </section>
  );
}
```

## Register it

Add the component through `customSections` in the main sections loop:

```tsx
const customSections = [
  { contentType: "ff_section_banner", component: SectionBanner },
];

{page.sections?.map((section) => (
  <SectionRender
    key={section.id}
    sectionData={section as SectionData}
    customSections={customSections}
  />
))}
```

You can also override a built-in content type by registering the same `contentType` with a custom component.

## CTD conventions

* API name: `ff_section_[name]`
* Label: `Section/[Name]`

## Verification

1. Confirm the CTD exists and generated types are current.
2. Confirm the component renders through `customSections`.
3. Run the project's TypeScript check.