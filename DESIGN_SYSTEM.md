# Resume Design System

Single source of truth for the visual language of all HTML/PDF resume artifacts.
The CSS token layer in `templates/resume.html.j2` implements this file — change values here first, then mirror them in the template. DOCX constants in `src/cv_builder/build.py` follow the **light** palette.

## Principles

1. **One content source, two presentations.** Light and dark are the same layout with different token values. No theme-specific layout or markup.
2. **Dark is not inverted light.** Dark mode is rebuilt from rules (below), never by flipping light values.
3. **Surfaces are neutral, accents carry chroma.** Grays must have no visible hue cast (zinc scale). Color appears only through `accent` / `accent-2`.
4. **Hierarchy = size + color + spacing, not weight.** Arial/Liberation Sans only ships 400 and 700; intermediate weights are synthesized and unreliable in WeasyPrint. Never use weights above 700.
5. **A4 print parity.** Everything must survive WeasyPrint PDF rendering: explicit `print-color-adjust`, mm-based spacing, no viewport-relative units.

## Color tokens

| Token | Role | Light | Dark |
|---|---|---|---|
| `--body-bg` | Canvas behind the page | `#e8ebf0` | `#0a0a0b` |
| `--page-bg` | Page surface | `#ffffff` | `#18181b` |
| `--panel-bg` | Cards (client, project, edu) | `#f8fafc` | `#1e1e22` |
| `--panel-bg-strong` | Expertise grid panel | `#f7f9fc` | `#1c1c20` |
| `--heading` | h1 only | `#172033` | `#ececef` |
| `--ink` | Body text, strong labels | `#172033` | `#d4d4d8` |
| `--summary` | Profile / expertise body | `#243047` | `#c2c2c9` |
| `--card-title` | Card headings | `#1d293e` | `#e4e4e8` |
| `--muted` | Context lines, metadata | `#5a6578` | `#94949c` |
| `--footer` | Page footer note | `#8b94a5` | `#71717a` |
| `--accent` | Brand: section titles, dates, headline, stripe | `#244f8f` (blue) | `#56b8a9` (muted teal) |
| `--accent-2` | Secondary: stripe gradient end, alt cards | `#0f766e` | `#3f9d8f` |
| `--line` | Hairlines, card borders | `#d9e0ea` | `#2c2c31` |
| `--page-edge` | 1px page outline | `transparent` | `#2c2c31` |
| `--page-shadow` | Page elevation | blue-gray 18% | black 50% |

## Dark mode rules

These produced the dark column and govern any future tuning:

- **Text brightness ceiling.** No text brighter than `#ececef`, and only the h1 gets that. Body text targets ~10:1 contrast on `--page-bg` (zinc-300 range), never ~17:1 near-white — over-bright small text halates and is *harder* to read.
- **Contrast floors.** Body ≥ 7:1. Muted/metadata ≥ 4.5:1. Accent on page ≥ 4.5:1.
- **Accent desaturation.** Dark accents are the light brand hue family at roughly half the chroma (muted teal, not teal-300 neon). If an accent looks fine on a white mockup, it is too bright for dark.
- **Neutral grays.** All surface and text grays come from a zinc scale (slight cool, zero green/blue cast). Hue lives only in accents.
- **Borders over shadows.** Elevation on dark surfaces uses `--line` borders and the `--page-edge` outline; shadows are near-invisible on dark and only ground the page on the canvas.
- **Surface elevation order:** `body-bg < panel-bg-strong < panel-bg`-on-`page-bg` steps stay within ~3% lightness of each other — separation comes from borders.

## Typography

| Element | Size | Weight | Color |
|---|---|---|---|
| h1 (name) | 29pt / 20pt (page 2) | 700 | `--heading` |
| Headline | 10.7pt | 700 | `--accent` |
| Section title | 8.6pt, uppercase, +1.25pt tracking | 700 | `--accent` |
| Role title | 10.6pt | 700 | `--ink` |
| Card/project/edu title | 9.1–9.3pt | 700 | `--card-title` |
| Dates | 8.35pt | 700 | `--accent` |
| Body / bullets | 9.1pt, line-height 1.32 | 400 | `--ink` |
| Summary/profile | 9.25pt | 400 | `--summary` |
| Context / metadata | 8.35–8.45pt | 400–600 | `--muted` |
| Footer note | 7.7pt | 400 | `--footer` |

- Stack: `Arial, Helvetica, sans-serif` (Liberation Sans in CI/WSL). Available weights: 400, 700. **Max weight 700.**
- Negative tracking only on the h1 (−0.7pt). Positive tracking only on uppercase section titles.

## Spacing & shape

- Page: A4, padding `12mm 13mm 11mm`; brand stripe 4.5mm gradient (`accent → accent-2`) on left edge.
- Section gap 4.5mm (compact 3.5mm), role gap 3.3mm, card padding ~2.2mm × 3mm.
- Radius: 7px panels/cards, `0 6px 6px 0` for left-accent client cards.
- Hairlines 1px `--line`; client-card accent bar 2.4px.

## Adding a token

Add it to both palettes here, then to both `:root` blocks in the template. A token must name a **role** (what it colors), never a value (`--teal-300` is wrong, `--accent` is right).
