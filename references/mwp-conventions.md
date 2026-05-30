# MWP Conventions Reference

Source: Interpretable Context Methodology (ICM) -- Van Clief & McDermott, 2025.
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
Layer 3 is reference material -- the factory (stable across runs).
Layer 4 is working artifacts -- the product (changes each run).

Total context per stage: typically 2,000-8,000 tokens.

---

## 16 Core Conventions

These conventions apply to every MWP workspace. They are the grammar of the methodology.

**Convention 1: Stage Contracts**
Every stage CONTEXT.md has three sections: Inputs (table), Process (numbered steps), Outputs (table). This is the contract between stages.

**Convention 2: Stage Handoffs**
Stage N writes to `stages/0N-name/output/`. Stage N+1 reads from `../0N-name/output/`. Every stage writes `output/manifest.md` as its final step listing exact filenames produced; downstream stages read the manifest before loading any artifact. A skipped stage writes a manifest with `Status: skipped` and a one-line reason instead of artifacts. File naming: `YYYY-MM-DD-{topic-slug}-{artifact-type}.md`.

**Convention 3: One-Way Cross-References**
Folders point outward only. No back-references. Stage 3 can reference Stage 2's output; Stage 2 must not reference Stage 3.

**Convention 4: Selective Section Routing**
CONTEXT.md Inputs tables reference specific sections of files, not whole files. Format: `| file.md | "Voice Rules" through "What the Voice Is NOT" | Tone guidance |`

**Convention 5: Canonical Sources**
Every piece of information has one home. Other files point to it; they do not duplicate it. For code-producing workspaces, this extends to shared constant files (colors, fonts, timing, layout) in `shared/`, populated once during onboarding.

**Convention 6: CONTEXT.md = Routing, Not Content**
CONTEXT.md answers: What is this folder? What do I load? What is the process? Never contains actual reference material. Target: 25-80 lines.

**Convention 7: Tool Prerequisites**
Setup guides for external tools (Node.js, ffmpeg, scripts) live in `references/` of the stage that uses them. Multi-stage tools go in `shared/`. Use local scripts for deterministic work that does not need AI: fetching data, moving files, formatting output, sending emails.

**Convention 8: Questionnaire Design**
`setup/questionnaire.md` is flat, all-at-once, system-level only. Derive what can be derived. Sensible defaults. Ask once, never again.

**Convention 9: Bundled Skills**
Workspaces can include a `skills/` folder with Claude Code skills. Stage CONTEXT.md Inputs tables reference them. Replaces custom reference docs when an official skill covers the ground.

**Convention 10: Specs Are Contracts**
Spec stages define WHAT and WHEN, not HOW. No implementation details in specs.

**Convention 11: Checkpoints**
Creative stages pause for human steering between process steps. Table format: `| After Step | Agent Presents | Human Decides |`. Autonomy: `workspace-config.md` includes `execution_mode: guided | autonomous`. Checkpoint steps append `(Skip if execution_mode = autonomous)` to make them optional for unattended runs without removing them from guided runs. Default is `guided`.

**Convention 12: Stage Audits**
Creative/build stages run a checklist after completing work but before writing to `output/`. If any check fails, agent revises before saving.

**Convention 13: Value Validation**
Content stages define value types (NOVEL, USABLE, QUESTION-GENERATING) and agree with human on which types the piece will hit before main creative work begins.

**Convention 14: Docs Over Outputs**
Reference docs are authoritative for how to build. Previous outputs in `output/` are artifacts, not templates. Agents must not read other outputs to learn patterns.

**Convention 15: Pipeline State**
`_config/pipeline-state.md` is a table (Stage, Status, Last Run, Notes) that stages update on completion (`complete`, `failed`, or `skipped`). The `status` trigger reads this file. Recovery from failure is a human responsibility; this convention provides detection, not retry.

**Convention 16: Parallel Branches**
Parallel stages share the same number with a letter suffix (`01a-`, `01b-`). A merge stage at the next number (`02-merge`) lists both in its Inputs table. Both branch stages appear in `pipeline-state.md` and the root Folder Map. Branches run as separate agent invocations; the merge stage synthesises them.

Recommended invocation: use `/branch` (or `claude --resume <id> --fork-session`) to fork the current session before running each parallel stage. The forked session inherits workspace context and diverges from that point forward. The original session remains authoritative for the merge stage. The merge stage must check that both branch rows in `pipeline-state.md` show `complete` before proceeding; if either is `skipped` or `failed`, it exits with a `Status: skipped` manifest noting which branch was incomplete.

---

## Naming Conventions

- Folders and files: `lowercase-with-hyphens`
- Stage folders: zero-padded prefix: `01-`, `02-`, `03-`
- Placeholders: `{{SCREAMING_SNAKE_CASE}}`
- Output artifacts: `YYYY-MM-DD-{topic-slug}-{artifact-type}.md`
- No spaces anywhere in file or folder names

## Quality Guardrails

- CONTEXT.md files: under 80 lines
- Reference files: under 200 lines (split if longer)
- Plain English; no jargon
- No em dashes anywhere in the repo
- Empty persistent folders get `.gitkeep`
- Every markdown file readable by someone with markdown/git basics but no engineering background

---

## Archetypes

Archetypes are reusable workspace patterns that apply when triggered by specific workflow needs. Unlike core conventions (which apply to every workspace), archetypes introduce their own internal structure and may extend or override core conventions. Each archetype documents which conventions it deviates from.

**Archetype A: Critique Stage**

A critique stage evaluates a previous stage's primary artifact against codified criteria and surfaces issues before the pipeline continues. It is a distinct stage, not a review step inside an existing stage.

Use a dedicated critique stage when all three conditions hold:
1. The previous stage produces open-ended output (draft, analysis, report) whose quality is uncertain
2. The evaluation criteria can be written as a checklist ahead of time (not a judgment call)
3. Re-running the previous stage is worthwhile if issues are found

Use a checkpoint (Convention 11) inside a stage instead when: the review is a single human judgment call that cannot be codified, or when the criteria only become clear during the run.

Structure:
- **Inputs:** Previous stage's primary artifact + a criteria file from `references/` (Layer 3 -- stable, not run-specific)
- **Process:** Evaluate artifact against each criterion; classify each finding as Blocking or Advisory; write critique artifact; if Blocking issues found, add a checkpoint (Convention 11) before proceeding
- **Outputs:** `YYYY-MM-DD-{topic}-critique.md` (findings table: criterion, result, severity, note) + manifest

Key rules:
- A critique stage reads evaluation criteria from Layer 3 (`references/criteria.md`), not from the previous stage's output
- It does NOT re-run the previous stage -- if Blocking issues are found, the pipeline pauses at a checkpoint and the human decides: fix and re-run the prior stage, or accept and proceed
- Status is always `complete` (the critique ran); Blocking issues are surfaced via checkpoint, not via `failed` status
- Add a `0N-critique` row to `pipeline-state.md`; human records the checkpoint outcome in the Notes column

**Archetype B: Knowledge Base Layer**

Use when a workspace's Layer 3 references need to grow and stay current from a stream of incoming source documents, rather than being set once at setup.

Core concept: separate immutable intake (`raw/`) from compiled structured knowledge (`wiki/`). An LLM-driven pipeline ingests source documents and compiles them into an incrementally growing wiki of interlinked markdown pages. Knowledge is compiled once on ingest, not rediscovered on every query -- unlike per-query RAG.

Structure:
- `raw/` (Obsidian Webclipper inbox): immutable intake. LLM reads; never writes. Shared across all domains.
- `wiki/` (Obsidian vault, one folder per domain): compiled knowledge base. Persistent, cumulative. LLM reads and writes.
- `wiki/index.md`: content catalog listing all pages.
- `wiki/processed-log.md`: append-only record of which raw files have been compiled. One per domain wiki -- the same source clip can be compiled into multiple domain wikis with different extracted knowledge.
- Three stages: `01-ingest` (extract summaries from new clips), `02-compile` (update wiki pages), `03-lint` (health check).

Multi-domain configuration: a single knowledge-base workspace can serve multiple domains from one shared inbox. Split `_config/` into a shared config (raw_source_path, staleness_threshold, execution_mode) and one domain config per domain (domain, domain_slug, wiki_path, topic_focus). Each domain run reads the shared config and its own domain config. Stages write to per-domain output subfolders (`output/{domain_slug}/`) to allow parallel runs without write conflicts.

Parallel execution: when multiple domains share the same inbox, run all domain ingests in parallel using a shell script (`shared/run-ingest.sh`) that fires one `claude -p` process per domain, each with its own `--add-dir {wiki_path}`. Set `execution_mode: autonomous` in the shared config so stages skip checkpoints and complete without interaction. Pipeline state tracks per-domain status with a Domain column: `| Stage | Domain | Status | Last Run | Notes |`.

Convention deviations: Stage 02-compile writes directly to `wiki/` in place, not to a per-run `output/` folder. This overrides Convention 2 (Stage Handoffs). Wiki pages are cumulative artifacts, not run-specific outputs. Document this explicitly in the workspace CLAUDE.md.

Consuming workspaces: use `--add-dir <wiki_path>` at session start; reference specific wiki pages in stage Inputs tables (Convention 4: Selective Section Routing).

Webclipper integration: Obsidian Webclipper saves clips as markdown with frontmatter (title, URL, date, tags) to the configured raw_source_path. Stage 01-ingest reads the processed-log to skip already-compiled clips, processes only new ones.

**Archetype C: Deliverables Stage**

Use when a workflow produces reports, assessments, or analysis that must be presented to stakeholders in formatted documents (docx, xlsx, pptx) beyond markdown and PDF.

Core concept: a final stage that reads upstream stage outputs and reshapes them into presentation-ready formats. It creates no new content -- it formats what stages before it already produced. This separates content production (analysis, documentation, reporting) from content packaging (formatted deliverables).

Structure:
- `stages/0N-deliverables/CONTEXT.md`: inputs from 2+ upstream stages; process uses python-docx/openpyxl/python-pptx
- `stages/0N-deliverables/references/`: structure definitions per format (docx-structure.md, xlsx-structure.md, pptx-structure.md) describing section maps, column specs, slide layouts, and content-source mappings
- `stages/0N-deliverables/references/templates/`: pre-styled binary templates (.docx, .xlsx, .pptx) with visual design baked in
- `shared/deliverables.py`: generator script; reads upstream manifests and markdown, parses into structured data, populates templates or generates from scratch

Three deliverable types:
- **Docx** (security assessment / report): domain-oriented prose document with focused tables. Structure follows the assessment domain, not the pipeline stages. Flowing paragraphs, not markdown dumps.
- **Xlsx** (scorecard / workbook): multi-sheet workbook with Document Control, Links (hyperlinked source URLs), domain assessment (Sheet A), and assessment-specific breakdown (Sheet B). FLAG columns with dropdown validation and conditional formatting.
- **Pptx** (executive deck): slide deck with comparison cards, per-channel/domain summaries, mitigation columns, and embedded diagram PNGs.

Sheet B adapts to the assessment type -- channels for a multi-channel vendor assessment, controls for a hardening assessment, articles for a compliance assessment. The template defines the variant; the generator populates rows.

When to suggest: if the user's workflow description mentions "presenting to leadership", "peer review documents", "formatted reports", "Excel scorecards", or "PowerPoint decks", suggest a deliverables stage as the final pipeline stage. It always reads from the reporting stage and at least one upstream analysis stage.

Dependencies: `python-docx`, `openpyxl`, `python-pptx`, `pillow`. Install in a workspace-local venv.

---

## Trigger Keywords

- `setup` -- Starts onboarding; agent reads `setup/questionnaire.md`, populates `_config/`
- `status` -- Reads `_config/pipeline-state.md`; shows per-stage Status, Last Run, and Notes as an ASCII table

## Layer 3 vs Layer 4

| | Layer 3 (Reference) | Layer 4 (Working) |
|-|--------------------|--------------------|
| Changes between runs | No | Yes |
| Example files | voice.md, conventions.md | research-output.md, script-draft.md |
| Model treats it as | Constraints to internalise | Input to process |
| Configured during | Workspace setup (once) | Pipeline execution (each run) |
| Folder location | `references/`, `_config/`, `shared/` | `output/` |
