# Knowledge Stores — Cross-Project

Deep, structured knowledge organized by discipline. Each subdirectory contains patterns, anti-patterns, gotchas, and assessment rubrics that apply across all projects using the SDLC framework.

## Structure

```
knowledge/
├── README.md                  ← This file
├── data-modeling/             ← UDM patterns, anti-patterns, assessment templates
├── design/                    ← UX modeling methodology, ASCII conventions
├── product-research/          ← Competitive analysis methodology, dimension catalog
└── testing/                   ← Tool patterns, component strategies, gotchas, timing
```

Other disciplines (architecture, coding, deployment, etc.) will add directories here as their knowledge matures beyond the parking-lot stage in `disciplines/`.

## Relationship to Other Directories

| Directory | Purpose |
|-----------|---------|
| `disciplines/` | Overviews — *what* each discipline covers, when to engage it |
| `knowledge/` | Deep content — *how* to apply the discipline (patterns, rubrics, gotchas) |
| `skills/` | Automation — thin wrappers that orchestrate knowledge into Claude Skills |

## How This Gets Used

1. A discipline overview (`disciplines/testing.md`) points here for deep knowledge
2. Claude Skills (`skills/model-assess/SKILL.md`) reference knowledge files via relative paths
3. Project-specific knowledge lives in each project's docs (e.g., `rockcut/docs/testing/knowledge/`)

Cross-project knowledge accumulates here; project-specific knowledge stays local.

## Adding a New Discipline

1. Create `knowledge/<discipline-name>/` with a `README.md`
2. Add structured files (YAML preferred for AI-parseable content)
3. Update the discipline overview in `disciplines/<discipline-name>.md` to point here
