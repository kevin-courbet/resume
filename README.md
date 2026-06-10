# Kevin Courbet CV Builder

Local pipeline for regenerating CV artifacts from structured YAML content.

## Outputs

`task build` writes:

```text
dist/Kevin_Courbet_CV_2026_with_threadmill.docx
dist/Kevin_Courbet_CV_2026_with_threadmill_docx.pdf
dist/Kevin_Courbet_CV_2026_with_threadmill.html
dist/Kevin_Courbet_CV_2026_with_threadmill_html.pdf
```

## Setup

```bash
uv sync
```

Install LibreOffice for DOCX -> PDF generation:

```bash
sudo apt install libreoffice
```

## Generate

Full pipeline:

```bash
task build
```

When LibreOffice is not installed, generate everything except DOCX-PDF:

```bash
task build:no-docx-pdf
```

Direct command:

```bash
uv run cv-build --data data/resume.yml --template-dir templates --out-dir dist
```

## Edit

- Content: `data/resume.yml`
- HTML presentation: `templates/resume.html.j2`
- DOCX presentation and pipeline behavior: `src/cv_builder/build.py`
