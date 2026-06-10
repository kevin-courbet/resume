---
name: pdf-visual-qa
description: Visually verify and pixel-check rendered PDF artifacts (resume themes, print CSS). Use when changing templates/resume.html.j2 tokens, palettes, or layout, or whenever a claim about how a generated PDF looks must be verified rather than assumed.
---

# PDF Visual QA

Loop for any visual change to generated resume artifacts: **diagnose from source → rebuild → render → eyeball → pixel-verify → record**.

## Tools

| Tool | Use | Command |
|---|---|---|
| poppler `pdftoppm` | PDF page → PNG | `pdftoppm -png -r 80 dist/<file>.pdf /tmp/<prefix>` (page range: `-f 1 -l 1`) |
| poppler `pdfinfo` | Page count sanity (layout overflow adds pages) | `pdfinfo dist/<file>.pdf \| grep Pages` |
| Read tool on PNG | Visual inspection | Read `/tmp/<prefix>-1.png` |
| pillow via uv | Pixel ground truth | `uv run --with pillow python -c "..."` (no install needed) |
| `task build` | Regenerate artifacts | `task build` or `task build:no-docx-pdf` without LibreOffice |

Install poppler once: `sudo apt-get install -y poppler-utils`.

## Method

1. **Diagnose from values before rendering.** Read the CSS token layer first. Most issues are computable: contrast ratio (body text on dark: target ~10:1, floor 7:1; muted floor 4.5:1; ~17:1 near-white = halation), hue cast in grays (R≠G≠B by >4 = tinted), synthetic font weights (Arial/Liberation only ships 400/700 — anything above 700 is fake).
2. **Render before AND after** with distinct prefixes (`/tmp/old-*.png`, `/tmp/new-*.png`) so both stay comparable.
3. **Eyeball every page**, not just page 1.
4. **Never trust the image preview alone.** Previews can misrender (a dark page once previewed as white). Confirm with pixels:

```bash
uv run --with pillow python -c "
from PIL import Image
im = Image.open('/tmp/new-2.png').convert('RGB')
w,h = im.size
print('center:', im.getpixel((w//2,h//2)))   # expect page-bg token value
g = im.convert('L'); px = list(g.getdata())
print('dark px:', sum(p<80 for p in px)/len(px))  # dark theme ≈ 0.9
"
```

Center pixel must equal the `--page-bg` token; dark/light pixel ratio must match the theme. Disagreement between preview and pixels → pixels win.

5. **Check page count** vs expected — a 1px border or margin change can silently overflow A4 and add pages.
6. **Record**: palette/typography rules go in `DESIGN_SYSTEM.md` (tokens mirrored in template `:root` blocks); append a row to `DESIGN_DECISIONS.md` (`date|actor|scope|decision|rationale|code_refs`).

## Design constraints (see DESIGN_SYSTEM.md for full rules)

- Dark ≠ inverted light: brightness ceiling `#ececef` (h1 only), neutral zinc surfaces, accents at ~half chroma, borders over shadows.
- Hierarchy = size + color + spacing; max font-weight 700.
- Everything must survive WeasyPrint A4 print rendering.
