# SDLC Framework

A reusable Software Development Lifecycle framework for AI-assisted development projects using Claude Code (CC).

## Overview

This framework defines:
- **Process:** How work flows from idea → spec → implementation → archive
- **Artifacts:** Standard document types and their purposes
- **Structure:** Directory organization for active work and chronicles
- **Collaboration:** How CD (Claude Director/human) and CC work together

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
├── process/                  # Process documentation
│   ├── overview.md           # High-level workflow
│   ├── deliverable_lifecycle.md
│   ├── chronicle_organization.md
│   ├── collaboration_model.md
│   ├── ad_hoc_reconciliation.md
│   └── compliance_audit.md
├── knowledge/                # Cross-project discipline knowledge
│   ├── data-modeling/        # UDM patterns, anti-patterns, assessments
│   └── testing/              # Tool patterns, gotchas, component strategies
├── disciplines/              # Discipline overviews and parking lots
├── templates/                # Document templates
│   ├── spec_template.md
│   ├── planning_template.md
│   ├── cc_prompt_template.md
│   ├── stepwise_result_template.md
│   ├── concept_index_template.md
│   └── compliance_audit_template.md
├── examples/                 # Filled-out examples
│   ├── spec_example.md
│   ├── planning_example.md
│   └── ...
└── skeleton/                 # Directory structure to copy
    ├── docs/
    │   ├── current_work/
    │   ├── chronicle_by_concept/
    │   ├── chronicle_by_step/
    │   ├── process/
    │   └── templates/
    └── CLAUDE.md             # Template project instructions
```

## Core Concepts

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
- `process/overview.md` — Complete process documentation
- `process/chronicle_organization.md` — How to archive completed work
- `process/compliance_audit.md` — How to audit project compliance
