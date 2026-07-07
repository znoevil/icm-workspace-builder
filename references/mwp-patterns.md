# MWP Patterns Reference

Source: Interpretable Context Methodology (ICM) -- Van Clief & McDermott, 2025.
Reference repo: github.com/RinDig/Interpreted-Context-Methdology
Conventions, layers, and guardrails: `mwp-conventions.md`.

---

## The 24 Patterns

**Pattern 1: Stage Contracts**
Every stage CONTEXT.md has three sections: Inputs (table), Process (numbered steps), Outputs (table). This is the contract between stages.

**Pattern 2: Stage Handoffs via Output Folders**
Stage N writes to `stages/0N-name/output/`. Stage N+1 reads from `../0N-name/output/`. File naming: `{topic-slug}-{artifact-type}.md`.

**Pattern 3: One-Way Cross-References**
Folders point outward only. No back-references. Stage 3 can reference Stage 2's output; Stage 2 must not reference Stage 3.

**Pattern 4: Selective Section Routing**
CONTEXT.md Inputs tables reference specific sections of files, not whole files. Format: `| file.md | "Voice Rules" through "What the Voice Is NOT" | Tone guidance |`

**Pattern 5: Canonical Sources**
Every piece of information has one home. Other files point to it; they do not duplicate it.

**Pattern 6: CONTEXT.md = Routing, Not Content**
CONTEXT.md answers: What is this folder? What do I load? What is the process? Never contains actual reference material. Target: 25-80 lines.

**Pattern 7: Tool Prerequisites**
Setup guides for external tools (Node.js, ffmpeg, scripts) live in `references/` of the stage that uses them. Multi-stage tools go in `shared/`. Use local scripts for deterministic work that does not need AI: fetching data, moving files, formatting output, sending emails.

**Pattern 8: Questionnaire Design**
`setup/questionnaire.md` is flat, all-at-once, system-level only. Derive what can be derived. Sensible defaults. Ask once, never again.

**Pattern 9: Bundled Skills**
Workspaces can include a `skills/` folder with Claude Code skills. Stage CONTEXT.md Inputs tables reference them. Replaces custom reference docs when an official skill covers the ground.

**Pattern 10: Specs Are Contracts**
Spec stages define WHAT and WHEN, not HOW. No implementation details in specs.

**Pattern 11: Checkpoints**
Creative stages pause for human steering between process steps. Table format: `| After Step | Agent Presents | Human Decides |`

**Pattern 12: Stage Audits**
Creative/build stages run a checklist after completing work but before writing to `output/`. If any check fails, agent revises before saving.

**Pattern 13: Value Validation**
Content stages define value types (NOVEL, USABLE, QUESTION-GENERATING) and agree with human on which types the piece will hit before main creative work begins.

**Pattern 14: Docs Over Outputs**
Reference docs are authoritative for how to build. Previous outputs in `output/` are artifacts, not templates. Agents must not read other outputs to learn patterns.

**Pattern 15: Shared Constants**
Code-producing workspaces define shared constant files (colors, fonts, timing, layout) populated once during onboarding.

**Pattern 16: Stage Manifests**
Every stage writes `output/manifest.md` as its final step, listing exact filenames produced. Downstream stages read the manifest before loading any artifact. A skipped stage writes `Status: skipped` with a one-line reason instead of artifacts. This makes handoffs deterministic and skips detectable.

**Pattern 17: Pipeline State**
`_config/pipeline-state.md` is a table (Stage, Status, Last Run, Notes) that stages update on completion (`complete`, `failed`, or `skipped`). The `status` trigger reads this file. Recovery from failure is a human responsibility; this pattern provides detection, not retry.

**Pattern 18: Parallel Branches**
Parallel stages share the same number with a letter suffix (`01a-`, `01b-`). A merge stage at the next number (`02-merge`) lists both in its Inputs table. Both branch stages appear in `pipeline-state.md` and the root Folder Map. Branches run as separate agent invocations; the merge stage synthesises them.

Recommended invocation: use `/branch` (or `claude --resume <id> --fork-session`) to fork the current session before running each parallel stage. The forked session inherits workspace context and diverges from that point forward. The original session remains authoritative for the merge stage. The merge stage must check that both branch rows in `pipeline-state.md` show `complete` before proceeding; if either is `skipped` or `failed`, it exits with a `Status: skipped` manifest noting which branch was incomplete.

**Stage insertion (amendment).** Letter suffixes also serve sequential insertion. Default when adding a stage mid-pipeline is to renumber downstream stages; when renumbering is too disruptive, insert with a letter suffix on the preceding stage number (e.g. `01b-validation` between `01-` and `02-`). Disambiguation rule: parallel branches always form a complete letter set starting at `a` (`01a-`, `01b-`) with a merge stage at the next number; a lone suffixed folder with no sibling, whose Inputs read the base stage's output, is a sequential insertion. An insertion must also repoint the immediately-downstream stage's Inputs rows from the base stage's output to the inserted stage's output -- this repointing is part of the insertion and is exempt from the Update-mode rule against touching existing stage files (change only those Inputs rows, nothing else).

**Pattern 19: Autonomy Flag**
`workspace-config.md` includes `execution_mode: guided | autonomous`. Stages with human-review checkpoints append `(Skip if execution_mode = autonomous)` to those steps. Default is `guided`. This makes checkpoints optional for unattended runs without removing them from guided runs.

**Pattern 20: Critique Stage**
A critique stage evaluates a previous stage's primary artifact against codified criteria and surfaces issues before the pipeline continues. It is a distinct stage, not a review step inside an existing stage.

Use a dedicated critique stage when all three conditions hold:
1. The previous stage produces open-ended output (draft, analysis, report) whose quality is uncertain
2. The evaluation criteria can be written as a checklist ahead of time (not a judgment call)
3. Re-running the previous stage is worthwhile if issues are found

Use a checkpoint (Pattern 11) inside a stage instead when: the review is a single human judgment call that cannot be codified, or when the criteria only become clear during the run.

Structure:
- **Inputs:** Previous stage's primary artifact + a criteria file from `references/` (Layer 3 -- stable, not run-specific)
- **Process:** Evaluate artifact against each criterion; classify each finding as Blocking or Advisory; write critique artifact; if Blocking issues found, add a checkpoint (Pattern 11) before proceeding
- **Outputs:** `YYYY-MM-DD-{topic}-critique.md` (findings table: criterion, result, severity, note) + manifest

Key rules:
- A critique stage reads evaluation criteria from Layer 3 (`references/criteria.md`), not from the previous stage's output
- It does NOT re-run the previous stage -- if Blocking issues are found, the pipeline pauses at a checkpoint and the human decides: fix and re-run the prior stage, or accept and proceed
- Status is always `complete` (the critique ran); Blocking issues are surfaced via checkpoint, not via `failed` status
- Add a `0N-critique` row to `pipeline-state.md`; human records the checkpoint outcome in the Notes column

**Pattern 21: Knowledge Base Layer**
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

Deviation from standard MWP: Stage 02-compile writes directly to `wiki/` in place (not to a per-run `output/` folder). This is intentional -- wiki pages are cumulative artifacts, not run-specific outputs. Document this explicitly in the workspace CLAUDE.md.

Consuming workspaces: use `--add-dir <wiki_path>` at session start; reference specific wiki pages in stage Inputs tables (Pattern 4: Selective Section Routing).

Webclipper integration: Obsidian Webclipper saves clips as markdown with frontmatter (title, URL, date, tags) to the configured raw_source_path. Stage 01-ingest reads the processed-log to skip already-compiled clips, processes only new ones.

Reference workspace: `~/Documents/workspaces/knowledge-base/`

**Pattern 22: Deliverables Stage**
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

Reference implementation: `~/Documents/workspaces/cybersecurity/stages/05-deliverables/` and `~/Documents/workspaces/cybersecurity/shared/deliverables.py`.

**Pattern 23: In-Stage Worker Fan-Out**
Use when a single stage contains many independent, homogeneous work items -- claims to verify, sources to sweep, files to convert -- where each item's raw material is bulky but its result is small.

The stage owner (the session running the stage) extracts the work list, dispatches one sub-agent per item or item-group, and synthesises the returned results. Workers are stateless and do not inherit workspace context: every dispatch prompt must be self-contained (the work item, the relevant config excerpt, the required return format).

Rules:
- Only the stage owner writes to `output/`, `manifest.md`, or `pipeline-state.md`. Workers return results to the owner; they never write stage artifacts. This keeps handoffs deterministic and the stage gate intact.
- Every dispatch names an explicit model, at least one tier below the session model (mechanical verification: sonnet or haiku; open research: sonnet). Exception: verification-critical stages (Pattern 24) may run workers at session tier (opus) -- subtle verdict calls on legal and policy language justify the spend, and workers are the only ones who read the sources.
- In `guided` mode, state the worker count and model before dispatching -- fan-out is a spending decision.
- For verification work, frame workers adversarially ("try to refute this claim") -- independence from the authoring context is the point.
- Judgment, filtering, synthesis, and gating stay with the stage owner. Work that needs whole-pipeline context is not a fan-out candidate.

Relation to Pattern 18: `/branch` runs whole stages in parallel sessions; Pattern 23 parallelises work items within one stage. A workflow can use both.

**Pattern 24: Validation Stage**
Use when a stage produces output whose factual claims are load-bearing for downstream decisions -- intelligence briefs feeding architecture analysis, vendor assessments feeding risk ratings, compliance claims feeding reports.

Distinct from Pattern 20 (Critique): critique evaluates an artifact against internal criteria from Layer 3; validation verifies factual claims against external primary sources. A workflow can have both.

Structure:
- **Inputs:** previous stage's manifest + its primary artifact
- **Process:** (1) extract every factual claim (data handling, policies, roles, platform behaviours, dates, technical values); (2) verify each claim against official primary sources -- fan out per Pattern 23, one worker per claim group, adversarial framing; (3) assign verdicts: VERIFIED / PARTIALLY ACCURATE / INCORRECT / UNVERIFIED; (4) the stage owner synthesises a claim verification report and a corrected artifact with inline `[Source: URL]` citations
- **Outputs:** `YYYY-MM-DD-{topic}-claim-verification-report.md` + `YYYY-MM-DD-{topic}-verified.md` + manifest. The verified artifact -- not the original -- feeds the next stage.

Key rules:
- Zero tolerance for INCORRECT: fix before the manifest is marked complete
- PARTIALLY ACCURATE claims are rewritten in the source's exact language, not paraphrased
- UNVERIFIED claims are flagged inline (`[UNVERIFIED -- confirm with vendor]`) so downstream stages can assess whether decisions depend on them
- Never generate the corrected artifact from memory: read the source, then write

Reference implementation: `~/Documents/workspaces/cybersecurity/stages/01b-validation/`
