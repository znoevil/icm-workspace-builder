# ICM Workspace Builder

A Claude Code skill that scaffolds structured, multi-stage AI workspaces using the Interpretable Context Methodology (ICM).

## What it does

ICM Workspace Builder turns workflow descriptions into numbered-folder workspaces where each stage has one job, loads only the context it needs, and passes work forward through manifest-based handoffs. No framework, no orchestration runtime -- just folders, markdown, and a context hierarchy.

## Five-Layer Context Architecture

Each workspace organises information into five layers. The agent loads only the layers relevant to the current stage -- typically 2,000-8,000 tokens instead of the entire workspace.

```
Layer 0  CLAUDE.md                  "Where am I?"          ~800 tokens
Layer 1  CONTEXT.md                 "Where do I go?"       ~300 tokens
Layer 2  stages/0N/CONTEXT.md       "What do I do?"        200-500 tokens
Layer 3  references/ _config/       "What rules apply?"    500-2k tokens    (stable)
Layer 4  stages/0N/output/          "What am I working     varies           (per run)
                                     with?"
```

Layers 0-2 are structural routing -- they don't change between runs. Layer 3 is reference material set at setup. Layer 4 is the working product of each run.

## How stages pass work forward

Stages form a one-way pipeline. Each stage reads the previous stage's manifest to discover exact filenames, does its job, writes output, and updates pipeline state.

```
                    manifest.md              manifest.md
 ┌──────────┐      ──────────>  ┌──────────┐ ──────────>  ┌──────────┐
 │ 01-ingest│                   │ 02-draft │                │ 03-review│
 │          │  reads _config/   │          │  reads 01/     │          │  reads 02/
 │  do work │  writes output/   │  do work │  writes output/│  do work │  writes output/
 └──────────┘                   └──────────┘                └──────────┘
```

Each stage writes `output/manifest.md` listing every artifact it produced. The next stage reads that manifest first to get exact filenames -- never hardcoded paths.

## Mode detection

```
User message
  │
  ├── Workspace exists at target path?
  │     ├── No  ── Describes a workflow? ── Yes ──> BUILD (scaffold full workspace)
  │     │                                   No ───> ADVISORY (answer with MWP patterns)
  │     │
  │     └── Yes ── Wants to add stages? ────────── > UPDATE (add stages, patch routing)
  │                Structural question? ──────────> ADVISORY
  │                Describes new workflow? ────────> Ask: new workspace or add stages?
```

## Parallel branches

When stages can run independently, they share the same number with letter suffixes. A merge stage at the next number synthesises both.

```
                    ┌─── 01a-security-scan ───┐
 00-ingest ────────>│                         ├──> 02-merge ──> 03-report
                    └─── 01b-compliance-check─┘
```

## What gets generated

A typical Build produces:

```
my-workspace/
├── CLAUDE.md                        # Layer 0 -- identity + routing
├── CONTEXT.md                       # Layer 1 -- task routing table
├── _config/
│   ├── workspace-config.md          # Layer 3 -- setup answers
│   └── pipeline-state.md            # Stage status tracker
├── shared/                          # Layer 3 -- cross-stage utilities
├── setup/
│   └── questionnaire.md             # One-time onboarding
└── stages/
    ├── 01-ingest/
    │   ├── CONTEXT.md               # Layer 2 -- stage contract
    │   ├── references/              # Layer 3 -- stage-specific refs
    │   └── output/                  # Layer 4 -- run artifacts
    ├── 02-draft/
    │   ├── CONTEXT.md
    │   ├── references/
    │   └── output/
    └── 03-review/
        ├── CONTEXT.md
        ├── references/
        └── output/
```

## Install

Copy the skill into your Claude Code skills directory:

```bash
cp -r icm-workspace-builder ~/.claude/skills/
```

Or clone and symlink:

```bash
git clone https://github.com/znoevil/icm-workspace-builder.git
ln -s "$(pwd)/icm-workspace-builder" ~/.claude/skills/icm-workspace-builder
```

## Usage

Inside Claude Code, invoke the skill by asking it to build a workspace:

> "Build me a workspace for weekly threat intelligence digests -- ingest raw clips, research context, draft a brief, review, and distribute."

Or ask structural questions:

> "Should this be one stage or two?"
> "Where does this config file belong?"

## Patterns

The skill implements 22 MWP patterns documented in `references/mwp-conventions.md`:

| # | Pattern | Purpose |
|---|---------|---------|
| 1 | Stage Contracts | Inputs/Process/Outputs structure for every stage |
| 2 | Stage Handoffs | Output folders + manifest-based file discovery |
| 3 | One-Way References | No back-references; downstream reads upstream only |
| 4 | Selective Section Routing | Reference specific sections, not whole files |
| 5 | Canonical Sources | Every fact has one home; no duplication |
| 6 | CONTEXT.md = Routing | Never contains actual content; routes to it |
| 7 | Tool Prerequisites | Scripts in `references/` or `shared/` |
| 8 | Questionnaire Design | Flat, all-at-once, derive what you can |
| 9 | Bundled Skills | Claude Code skills inside the workspace |
| 10 | Specs Are Contracts | WHAT and WHEN, not HOW |
| 11 | Checkpoints | Human steering between creative steps |
| 12 | Stage Audits | Self-check before writing to output |
| 13 | Value Validation | Agree on value types before creating |
| 14 | Docs Over Outputs | Reference docs are authoritative, not past outputs |
| 15 | Shared Constants | Colors, fonts, timing defined once |
| 16 | Stage Manifests | Every stage writes manifest.md; skips write "skipped" |
| 17 | Pipeline State | Central status table in `_config/` |
| 18 | Parallel Branches | Letter-suffixed stages + merge stage |
| 19 | Autonomy Flag | `guided` vs `autonomous` execution mode |
| 20 | Critique Stage | Evaluate output against codified criteria |
| 21 | Knowledge Base | Compiled wiki from raw intake documents |
| 22 | Deliverables Stage | Reshape markdown into docx/xlsx/pptx |

## Attribution

Based on the Interpretable Context Methodology (ICM) by Van Clief & McDermott, 2025.

## License

MIT
