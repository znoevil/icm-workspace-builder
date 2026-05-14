# MWP Workspace Templates

Templates used by the icm-workspace-builder skill during Build Mode (Step 3).
Generate all files in one pass -- do not pause for confirmation between files.

---

## Root CLAUDE.md (Layer 0, ~800 tokens)

```markdown
# {Workspace Name}

{One paragraph: what this workspace does, what it produces, and who uses it.}

## Folder Map

{workspace-name}/
├── CLAUDE.md
├── CONTEXT.md
├── _config/
│   ├── workspace-config.md
│   └── pipeline-state.md
├── shared/
├── setup/
│   └── questionnaire.md
└── stages/
    ├── 01-{name}/
    ├── 02-{name}/
    └── ...

## Routing Table

- {user intent phrase} -> stages/01-{name}/CONTEXT.md
- {user intent phrase} -> stages/02-{name}/CONTEXT.md
- {continue for each stage}
- Read the full workspace spec -> CLAUDE.md

## Triggers

- `setup` -- Run onboarding in setup/questionnaire.md to populate _config/
- `status` -- Read _config/pipeline-state.md; show per-stage Status, Last Run, and Notes as an ASCII table
- `brief` -- Orientation snapshot. Read and present:
  1. `_config/session-log.md` -- last entry only (what was done, what's next)
  2. `_config/pipeline-state.md` -- all stages as a status table
  3. Each stage `output/manifest.md` that exists -- list artifacts from the most recent run
  Present as three labelled sections. If a section is empty, say so in one line.
```

---

## Root CONTEXT.md (Layer 1, ~300 tokens)

```markdown
# Task Routing

- {user intent phrase} -> stages/01-{name}/CONTEXT.md
- {user intent phrase} -> stages/02-{name}/CONTEXT.md
- {continue for each stage}
- Read the full workspace spec -> CLAUDE.md
```

---

## Stage CONTEXT.md (Layer 2, 200-500 tokens each)

```markdown
# {Stage Name}

{One sentence: what this stage does.}

## Inputs

| Source | File/Location | Section/Scope | Why |
|--------|--------------|---------------|-----|
| Layer 3 | ../../_config/{file}.md | {section} | {reason} |
| Layer 4 | ../{prev-stage}/output/manifest.md | full file | Exact filenames produced by previous stage |
| Layer 4 | ../{prev-stage}/output/{artifact from manifest} | full file | {reason} |

## Process

1. {Concrete action -- what the agent actually does, not just a label}
2. {Next action}
3. ...
N. Write `output/manifest.md` listing every artifact produced this stage (see Manifest rule below)
N+1. Write `completed` status to `../../_config/pipeline-state.md` for this stage

## Outputs

| Artifact | Location | Format |
|----------|----------|--------|
| YYYY-MM-DD-{topic-slug}-{artifact-type}.md | output/ | Markdown |
| manifest.md | output/ | Markdown |
```

### Stage CONTEXT.md rules

- Use real relative paths, not placeholders like `../previous/output/file.md`
- The first stage has no Layer 4 input from a prior stage; its working material is user-provided or from `_config/`
- Every stage after the first reads `manifest.md` from the previous stage's `output/` folder first to get exact filenames, then loads the listed files
- Keep under 80 lines
- The first stage (01-) must include a preflight check as step 1 of Process:

  ```
  1. Check that `../../_config/workspace-config.md` exists. If it does not, stop and tell
     the user: "Workspace not configured. Type `setup` to run onboarding before using the
     pipeline."
  ```

- **Manifest rule:** Every stage's second-to-last Process step writes `output/manifest.md`:

  ```markdown
  # {Stage Name} Output Manifest
  Run: {date}
  Status: complete
  Artifacts:
  - {exact filename of each artifact written to output/}
  ```

  If a stage exits early (conditional exit), it still writes a manifest with `Status: skipped` and a one-line reason.

- **Conditional exit rule:** Process steps may include inline conditions that cause early termination. Format: "If {condition}, write a `Status: skipped` manifest to `output/` with a one-line reason, update pipeline-state.md, and stop." Downstream stages that read a `Status: skipped` manifest must also exit cleanly without producing output.

- **Checkpoint rule:** If a Process step requires human approval before continuing, add `(Skip if execution_mode = autonomous in workspace-config.md)` to that step. This makes checkpoints optional for unattended runs without removing them for guided runs.

---

## setup/questionnaire.md

```markdown
# {Workspace Name} -- Setup

Answer these questions to configure your workspace. Answers are saved to _config/ and
remain stable across runs. You only need to do this once.

1. {Question} (default: {value})
2. {Question}
...
N-1. Stakeholder outputs: do you need formatted deliverables (docx assessment, xlsx
     scorecard, pptx executive deck) for presenting to leadership, peers, or external
     reviewers? (default: no -- if yes, a deliverables stage is added as the final
     pipeline stage per Pattern 22)

N. Execution mode: should pipeline stages pause for human review at checkpoints, or run
   unattended? (default: guided -- stages pause at review checkpoints; use autonomous for
   batch/unattended runs where checkpoints are skipped)
```

Questions are workspace-specific: brand, audience, output format, tone, language, naming conventions, etc. Do not ask about things that can be derived. Provide sensible defaults in parentheses.

If the stakeholder outputs question is answered "yes", add a deliverables stage as the
final pipeline stage. Follow Pattern 22 in `references/mwp-conventions.md` for the
CONTEXT.md structure, reference files, and shared script. Create:
- `stages/0N-deliverables/CONTEXT.md` (inputs from reporting + analysis stages)
- `stages/0N-deliverables/references/docx-structure.md`
- `stages/0N-deliverables/references/xlsx-structure.md`
- `stages/0N-deliverables/references/pptx-structure.md`
- `stages/0N-deliverables/references/templates/` (empty -- user provides styled templates)
- `stages/0N-deliverables/output/.gitkeep`

---

## _config/pipeline-state.md

```markdown
# Pipeline State
| Stage | Status | Last Run | Notes |
|-------|--------|----------|-------|
| 01-{name} | pending | -- | |
| 02-{name} | pending | -- | |
```

Add one row per stage. Stages write `complete`, `failed`, or `skipped` here as their final act.

---

## .claude/settings.json (SessionStart hook)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "[ -f _config/pipeline-state.md ] && (echo '--- Pipeline State ---' && cat _config/pipeline-state.md && echo '' && echo '--- Last Session ---' && tail -20 _config/session-log.md) || echo 'Workspace not configured. Run setup.'"
          }
        ]
      }
    ]
  }
}
```

---

## Files and folders to create (Build Mode checklist)

- `{workspace}/CLAUDE.md`
- `{workspace}/CONTEXT.md`
- `{workspace}/_config/pipeline-state.md` (from template above)
- `{workspace}/shared/.gitkeep`
- `{workspace}/setup/questionnaire.md`
- For each stage N: `{workspace}/stages/0N-{name}/CONTEXT.md`
- For each stage N: `{workspace}/stages/0N-{name}/references/.gitkeep`
- For each stage N: `{workspace}/stages/0N-{name}/output/.gitkeep`
- `{workspace}/.claude/settings.json` (SessionStart hook from template above)
