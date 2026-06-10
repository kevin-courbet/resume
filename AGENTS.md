# Agent Instructions

## Skills

- PDF visual QA lives at `.agents/skills/pdf-visual-qa/SKILL.md`.
- Use `pdf-visual-qa` when changing `templates/resume.html.j2`, PDF/HTML layout, design tokens, palettes, typography, spacing, or when verifying generated PDF appearance.
- OpenCode loads project skills from `.agents/skills` via `opencode.json`; Codex-style agents should discover this skill through this `AGENTS.md` pointer.

## Project Notes

- Use `uv` and `task`.
- Build full artifacts with `task build`.
- If LibreOffice is unavailable, use `task build:no-docx-pdf`.
- Record durable product, visual, architecture, or workflow decisions in `DESIGN_DECISIONS.md` as `date|actor|scope|decision|rationale|code_refs`.
