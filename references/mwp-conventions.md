# MWP Conventions Reference

Source: Interpretable Context Methodology (ICM) — Van Clief & McDermott, 2025.
Reference repo: github.com/RinDig/Interpreted-Context-Methdology

---

## Architecture Framing

MWP implements the **Sequential pattern**: a digital assembly line where the output of one stage becomes the direct input for the next. Each stage is a specialist with one job; stages do not overlap. This is a named agentic design pattern -- cite it when explaining MWP to someone unfamiliar with it.

Each stage's Inputs table enforces **context engineering**: the active selection, packaging, and management of only the information the model needs for that specific step. This is the mechanism that keeps each stage focused and prevents context overload. The five-layer hierarchy (Layers 0-4) is the structural expression of context engineering -- each layer adds exactly the right information at the right level of specificity.

When to cite these terms in Advisory mode:
- "Why does MWP use numbered folders instead of one big prompt?" -- Sequential pattern; each stage is a specialist
- "Why do I have to list inputs so carefully?" -- context engineering; the model performs better with a curated, minimal context than with everything available
- "What's the difference between Layer 3 and Layer 4?" -- Layer 3 is stable reference (constraints); Layer 4 is working material (the product of this run)

---

## Five-Layer Context Hierarchy

| Layer | File/Folder | Question answered | Token budget |
|-------|-------------|-------------------|--------------|
| 0 | `CLAUDE.md` (workspace root) | Where am I? | ~800 |
| 1 | `CONTEXT.md` (workspace root) | Where do I go? | ~300 |
| 2 | `stages/0N-name/CONTEXT.md` | What do I do? | 200-500 |
| 3 | `references/`, `_config/`, `shared/` | What rules apply? | 500-2k |
| 4 | `stages/0N-name/output/` | What am I working with? | varies |

Layers 0-2 are structural routing (stable across runs).
Layer 3 is reference material — the factory (stable across runs).
Layer 4 is working artifacts — the product (changes each run).

Total context per stage: typically 2,000-8,000 tokens.

---

## Patterns

The full pattern catalogue (Patterns 1-24) lives in `mwp-patterns.md`.

---

## Naming Conventions

- Folders and files: `lowercase-with-hyphens`
- Stage folders: zero-padded prefix: `01-`, `02-`, `03-`
- Placeholders: `{{SCREAMING_SNAKE_CASE}}`
- Output artifacts: `{topic-slug}-{artifact-type}.md`
- No spaces anywhere in file or folder names

## Quality Guardrails

- CONTEXT.md files: under 80 lines
- Reference files: under 200 lines (split if longer)
- Plain English; no jargon
- No em dashes anywhere in the repo
- Empty persistent folders get `.gitkeep`
- Every markdown file readable by someone with markdown/git basics but no engineering background

---

## Trigger Keywords

- `setup` — Starts onboarding; agent reads `setup/questionnaire.md`, populates `_config/`
- `status` — Reads `_config/pipeline-state.md`; shows per-stage Status, Last Run, and Notes as an ASCII table

## Layer 3 vs Layer 4

| | Layer 3 (Reference) | Layer 4 (Working) |
|-|--------------------|--------------------|
| Changes between runs | No | Yes |
| Example files | voice.md, conventions.md | research-output.md, script-draft.md |
| Model treats it as | Constraints to internalise | Input to process |
| Configured during | Workspace setup (once) | Pipeline execution (each run) |
| Folder location | `references/`, `_config/`, `shared/` | `output/` |
