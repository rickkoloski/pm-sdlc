# SDLC Framework

A reusable Software Development Lifecycle framework for AI-assisted development projects using Claude Code (CC).

## Overview

This framework is built on three orthogonal dimensions:

- **Lifecycles** — How work is organized over time (prototyping → kanban → scrum → RUP → SAFe → waterfall)
- **Disciplines** — What capabilities are applied (research, analysis, design, architecture, coding, testing, deployment, ...)
- **Operations** — The infrastructure that supports both (conventions, roles, artifact management, archival, compliance)

## Quick Start

To apply this framework to an existing project:

1. Open the project in Claude Code
2. Add this directory to your workspace: `~/src/ops/sdlc`
3. Say: **"Let's implement the SDLC process from ~/src/ops/sdlc"**

CC will:
- Analyze your existing documentation
- Propose a categorization and structure
- Create the directory structure in your project
- Copy relevant templates
- Customize for your project

## Directory Structure

```
SDLC/
├── README.md                 # This file
├── BOOTSTRAP.md              # Instructions for CC when initializing projects
├── lifecycles/               # How work is organized over time
│   ├── README.md             # Mental model, lifecycle spectrum
│   ├── native.md             # Our default: deliverable-driven with lightweight gates
│   ├── prototyping.md        # Zero-ceremony, disciplines on demand
│   ├── kanban.md             # Continuous flow, WIP limits
│   ├── scrum.md              # Time-boxed sprints, ceremonies
│   ├── rup.md                # Phased iterations, milestones
│   ├── safe.md               # Scaled iterations, program increments
│   └── waterfall.md          # Sequential phases, hard stage gates
├── disciplines/              # What capabilities are applied
│   ├── README.md             # Toolbox philosophy, discipline overview
│   ├── testing.md            # Most developed
│   ├── architecture.md
│   ├── design.md
│   ├── coding.md
│   ├── business-analysis.md
│   ├── product-research.md
│   ├── data-modeling.md
│   ├── deployment.md
│   └── process-improvement.md
├── operations/               # Infrastructure supporting both dimensions
│   ├── conventions.md        # Deliverable IDs, file suffixes, working locations, principles
│   ├── collaboration_model.md # CD/CC roles, communication patterns, decision authority
│   ├── deliverable_lifecycle.md # Artifact state machine (Draft → Complete → Archived)
│   ├── chronicle_organization.md # How to archive completed work
│   ├── compliance_audit.md   # How to audit project compliance
│   └── sdlc_changelog.md    # Living record of process evolution
├── knowledge/                # Cross-project discipline knowledge
│   ├── architecture/
│   ├── data-modeling/
│   ├── design/
│   ├── product-research/
│   └── testing/
├── templates/                # Document templates
│   ├── spec_template.md
│   ├── planning_template.md
│   ├── cc_prompt_template.md
│   ├── stepwise_result_template.md
│   ├── concept_index_template.md
│   └── compliance_audit_template.md
├── examples/                 # Filled-out examples
├── improvement-ideas/        # Active design work
└── skeleton/                 # Directory structure to copy into projects
    ├── docs/
    └── CLAUDE.md             # Template project instructions
```

## Core Concepts

### Three Dimensions

| Dimension | Question it answers | Directory |
|-----------|-------------------|-----------|
| **Lifecycles** | How is work organized over time? | `lifecycles/` |
| **Disciplines** | What capabilities are applied? | `disciplines/` |
| **Operations** | What conventions and infrastructure support the work? | `operations/` |

### Deliverable IDs
Sequential identifiers (D1, D2, ... Dnn) that track work across the project lifetime.

### Current Work vs Chronicles
- `current_work/` — Active deliverables being worked on
- `chronicle_by_concept/` — Completed work organized by domain
- `chronicle_by_step/` — Completed work organized chronologically

### Artifact Types
| Type | Location | Purpose |
|------|----------|---------|
| Spec | `specs/` | Define what to build |
| Planning | `planning/` | Define how to build it |
| Prompt | `prompts/` | Instructions for CC |
| Result | `stepwise_results/` | Record of completion |
| Issue | `issues/` | Blocked items |

## See Also

- `BOOTSTRAP.md` — Detailed initialization instructions for CC
- `lifecycles/native.md` — Our default lifecycle (the deliverable-driven flow)
- `lifecycles/README.md` — How lifecycles relate to disciplines
- `operations/conventions.md` — Deliverable IDs, file suffixes, working locations
- `operations/collaboration_model.md` — CD/CC roles and interaction patterns
- `operations/compliance_audit.md` — How to audit project compliance
