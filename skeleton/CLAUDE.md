# CLAUDE.md

This file provides guidance to Claude Code when working with this project.

## Project Purpose

[One paragraph describing what this project does and its goals]

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| Backend | [e.g., Phoenix/Elixir, Node.js, Python] |
| Frontend | [e.g., React, Vue, None] |
| Database | [e.g., PostgreSQL, MongoDB] |
| Other | [Any other key technologies] |

---

## Current Work

**Active deliverables are in:** `docs/current_work/`

| Location | Contents |
|----------|----------|
| `specs/` | What to build |
| `planning/` | How to build it |
| `prompts/` | CC instructions |
| `stepwise_results/` | Completion records |
| `issues/` | Blocked items |

**Completed work is archived in:** `docs/chronicle_by_concept/`

---

## Project Structure

```
project/
├── [source directories]
├── docs/
│   ├── current_work/
│   ├── chronicle_by_concept/
│   ├── chronicle_by_step/
│   ├── process/
│   └── templates/
└── [other directories]
```

---

## Running the Project

```bash
# Setup
[commands to set up the project]

# Run
[commands to run the project]

# Test
[commands to run tests]
```

---

## Key APIs / Patterns

[Document the main APIs, patterns, or conventions CC should know about]

### Example Pattern

```language
// Code example showing a key pattern
```

---

## Conventions

- **Deliverable IDs:** Sequential (D1, D2, ... Dnn)
- **Branch naming:** `feature/dNN-short-description`
- **Commit format:** `type: description` (feat, fix, docs, refactor)
- [Other project-specific conventions]

---

## SDLC Process

This project follows the SDLC framework from `~/src/SDLC/`.

See:
- `~/src/SDLC/process/overview.md` — Workflow
- `~/src/SDLC/process/chronicle_organization.md` — Archiving
- `docs/templates/` — Document templates
