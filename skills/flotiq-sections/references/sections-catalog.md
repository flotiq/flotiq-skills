# Sections catalog

The sections of the Flotiq section library (`@flotiq/flotiq-flowbite-next` 1.1). The name is what the CLI's `add` takes; the content type is `ff_section_<name>` with `-` as `_`. Every section has a required `variant` (the first one listed is the default). `*` marks a required field.

Shared field shapes:

* `cta_buttons` — a list of `{label, external_link, internal_link}`; `internal_link` is a relation to a `page` (its slug becomes the link), `external_link` a URL, `mailto:` or `tel:`.
* `heading`, `description` — the section's title and lead text (`description` is a textarea; simple HTML is allowed).
* Media fields (`images`, `image`, `logo`, list items' `image`, `hover_image`, `person_avatar`, `icon`) are relations to `_media`; all optional.

| Name | For | Variants | Fields |
|---|---|---|---|
| `navbar` | The site's top menu with logo, links (with dropdowns) and buttons | `default` | `logo`, `items` {label, external_link, internal_link, sub_items {label, category, icon, icon_hover, external_link, internal_link}}, `languages` {label, url_prefix}, `cta_buttons` |
| `hero` | The first, big block of a page: headline, lead, buttons, image | `default`, `illustration` | `heading*`, `description`, `images`, `cta_buttons` |
| `content-block` | Text with images and buttons, anywhere on a page | `hero`, `only-images`, `two-columns`, `content-only` | `heading`, `description`, `images`, `cta_buttons` |
| `features` | Benefits, services or product features as a list | `cards`, `tabs`, `icon-boxes`, `illustration` | `heading`, `description`, `features` {image, hover_image, heading, description, external_link, internal_link}, `image`, `cta_buttons` |
| `logotypes` | Clients', partners' or technologies' logos | `carousel`, `cards`, `grid` | `heading`, `description`, `logos` {image, hover_image, external_link, internal_link}, `cta_buttons` |
| `testimonials` | Quotes from customers | `default`, `cards` | `heading`, `description`, `testimonials` {image, hover_image, heading, description, person_name, person_position, person_avatar, external_link, internal_link} |
| `team` | People with photo, role and links | `carousel` | `heading`, `description`, `people` {image, name, position, testimonial, socials}, `cta_buttons` |
| `faq` | Questions and answers | `accordion` | `heading`, `description`, `faq` {question, answer} |
| `table` | A comparison or price table | `default` | `heading`, `description`, `column_headings` (list of strings), `rows` {row_heading, row_data} |
| `contact-form` | A contact form from an external form script | `default` | `heading`, `description`, `form_script_url` |
| `map` | A map of a place | `default` | `heading`, `description`, `location*` |
| `footer` | The site's footer: logo, link columns, socials, copyright | `default` | `logo`, `items` {label, sub_items}, `copyright`, `socials` {icon, hover_icon, external_link} |

The components also implement two variants their content types don't offer yet: `two-columns` for `features` and `cards` for `team`. To use one, add it to the type's `variant` options (`flotiq-schema`).

A typical landing page: `navbar`, `hero`, `features`, `logotypes` or `testimonials`, `faq`, `content-block` (a closing call to action), `footer`. The navbar and footer are objects like any other: create each once and put the same object on every page.

The exact field formats (list item fields, helps) are in the content type: `get_content_type` after `add`.
