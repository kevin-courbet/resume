---
name: pdf-visual-qa
description: Visually verify and pixel-check rendered PDF artifacts, resume themes, print CSS, and generated document layouts. Use when changing templates, design tokens, palettes, typography, spacing, or when any claim about how a generated PDF looks must be verified rather than assumed.
---

# PDF Visual QA

Use this loop for visual changes to generated PDF artifacts: **diagnose from source -> rebuild -> render -> inspect -> pixel-verify -> record**.

## Tools

| Tool | Use | Command |
|---|---|---|
| poppler `pdftoppm` | PDF page -> PNG | `pdftoppm -png -r 80 dist/<file>.pdf /tmp/<prefix>` (page range: `-f 1 -l 1`) |
| poppler `pdfinfo` | Page count sanity; layout overflow adds pages | `pdfinfo dist/<file>.pdf` |
| image-capable file reader/viewer | Visual inspection | Inspect `/tmp/<prefix>-1.png` |
| Pillow via project Python runner | Pixel ground truth | `uv run --with pillow python -c "..."` |
| project build command | Regenerate artifacts | Prefer the repo's documented task, for this project `task build` or `task build:no-docx-pdf` |

Install poppler once if missing: `sudo apt-get install -y poppler-utils`.

## Method

1. **Diagnose from values before rendering.** Read the CSS token layer first. Most issues are computable: contrast ratio (body text on dark: target ~10:1, floor 7:1; muted floor 4.5:1; ~17:1 near-white = halation), hue cast in grays (R!=G!=B by >4 = tinted), synthetic font weights (Arial/Liberation only ships 400/700; anything above 700 is fake).
2. **Render before and after** with distinct prefixes (`/tmp/old-*.png`, `/tmp/new-*.png`) so both stay comparable.
3. **Inspect every page**, not just page 1.
4. **Never trust the image preview alone.** Previews can misrender (a dark page once previewed as white). Confirm with pixels:

```bash
uv run --with pillow python -c "
from PIL import Image
im = Image.open('/tmp/new-2.png').convert('RGB')
w,h = im.size
print('center:', im.getpixel((w//2,h//2)))   # expect page-bg token value
g = im.convert('L'); px = list(g.getdata())
print('dark px:', sum(p<80 for p in px)/len(px))  # dark theme approx 0.9
"
```

Center pixel must equal the `--page-bg` token; dark/light pixel ratio must match the theme. Disagreement between preview and pixels means pixels win.

5. **Check page count** vs expected; a 1px border or margin change can silently overflow A4 and add pages.
6. **Record durable decisions** in repo docs: palette/typography rules go in `DESIGN_SYSTEM.md`; append decision rows to `DESIGN_DECISIONS.md` when behavior or design intent changes.

## This Project

- Design system source: `DESIGN_SYSTEM.md`.
- Template tokens: `templates/resume.html.j2` `:root` blocks.
- Build commands: `task build` for full artifacts, `task build:no-docx-pdf` when LibreOffice is unavailable.
- Expected output path: `dist/`.

## Design Constraints

- Dark mode is not inverted light mode: brightness ceiling `#ececef` for h1 only, neutral zinc surfaces, accents at roughly half chroma, borders over shadows.
- Hierarchy comes from size, color, and spacing; max font-weight 700.
- Output must survive WeasyPrint A4 print rendering.
