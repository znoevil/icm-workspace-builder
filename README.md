# ICM Workspace Builder

A Claude Code skill that scaffolds structured, multi-stage AI workspaces using the Interpretable Context Methodology (ICM).

## What it does

ICM Workspace Builder turns workflow descriptions into numbered-folder workspaces where each stage has one job, loads only the context it needs, and passes work forward through manifest-based handoffs. No framework, no orchestration runtime -- just folders, markdown, and a context hierarchy.

Each workspace follows a five-layer context architecture:

| Layer | File | Purpose |
|-------|------|---------|
| 0 | `CLAUDE.md` | Workspace identity and routing |
| 1 | `CONTEXT.md` | Task routing table |
| 2 | `stages/0N-name/CONTEXT.md` | Stage contract (inputs, process, outputs) |
| 3 | `references/`, `_config/`, `shared/` | Stable reference material |
| 4 | `stages/0N-name/output/` | Working artifacts per run |

## Modes

| Mode | When | What happens |
|------|------|--------------|
| **Build** | You describe a workflow, no workspace exists | Scaffolds the full workspace |
| **Update** | Workspace exists, you want more stages | Adds stages, patches routing |
| **Advisory** | You have a structural question | Answers citing MWP patterns |

## Install

Copy the skill into your Claude Code skills directory:

```bash
cp -r icm-workspace-builder ~/.claude/skills/
```

Or clone and symlink:

```bash
git clone https://github.com/caliroo7s/icm-workspace-builder.git
ln -s "$(pwd)/icm-workspace-builder" ~/.claude/skills/icm-workspace-builder
```

## Usage

Inside Claude Code, invoke the skill by asking it to build a workspace:

> "Build me a workspace for weekly threat intelligence digests -- ingest raw clips, research context, draft a brief, review, and distribute."

Or ask structural questions:

> "Should this be one stage or two?"
> "Where does this config file belong?"

## What gets generated

A typical Build produces:

```
my-workspace/
├── CLAUDE.md
├── CONTEXT.md
├── _config/
│   ├── workspace-config.md
│   └── pipeline-state.md
├── shared/
├── setup/
│   └── questionnaire.md
└── stages/
    ├── 01-ingest/
    │   ├── CONTEXT.md
    │   ├── references/
    │   └── output/
    ├── 02-draft/
    │   ├── CONTEXT.md
    │   ├── references/
    │   └── output/
    └── 03-review/
        ├── CONTEXT.md
        ├── references/
        └── output/
```

## Key concepts

- **Single-purpose stages** -- each stage reads one input type, does one job, writes one output
- **Manifest-based handoffs** -- every stage writes `output/manifest.md`; downstream reads it first
- **One-way references** -- Stage N+1 reads Stage N, never the reverse
- **Context engineering** -- each stage loads 2,000-8,000 tokens, not the entire workspace
- **Parallel branches** -- letter-suffixed stages (`01a-`, `01b-`) with a merge stage at the next number

## Patterns

The skill implements 22 MWP patterns documented in `references/mwp-conventions.md`, covering stage contracts, handoffs, critique stages, knowledge bases, deliverables formatting, and more.

## Attribution

Based on the Interpretable Context Methodology (ICM) by Van Clief & McDermott, 2025.

## License

MIT
