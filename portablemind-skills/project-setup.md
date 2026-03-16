---
name: Harmoniq Project Setup
description: How to create projects with WBS task hierarchies in Harmoniq via MCP CRUD tools, including lifecycle-based scaffolding with canonical duration distributions
type: knowledge
verified: 2026-03-15
test_projects:
  basic_wbs: 38 (deleted after verification)
  rup_canonical: 39 (active — full RUP layout with discipline descriptions)
---

# Harmoniq Project Setup

Knowledge for creating and structuring projects in Harmoniq (PortableMind) via the MCP CRUD tools.

## Prerequisites

- MCP server: `harmoniq` with `general_crud_tool`, `model_schema_tool`, and `apply_status_tool`
- Tenant context: `tenant_id: 22` (portablemind), `owner_party_id: 19` (Richard Koloski)
- Always use `model_schema_tool` first to discover fields and filters before CRUD operations

## Creating a Project

```
model_name: "Project"
action: "create"
attributes: {
  name: "Project Name",
  description: "Project description",
  tenant_id: 22
}
```

**Required fields:** `name`, `tenant_id`
**Optional fields:** `description`, `start_date`, `end_date`, `billable`, `owner_party_id`, `owning_team_id`, `custom_fields`

The API auto-assigns `owner_party_id` to the authenticated user if not specified.

**CRITICAL: Apply status immediately after creation.** Projects without a status are invisible in the UI. The frontend filters by status (e.g., `in_progress`), so a project with `current_status: null` won't appear.

```
apply_status_tool:
  model_name: "Project"
  id: <project_id>
  status: "in_progress"
```

Available project statuses (under parent `project_statuses`):
- `not_started`, `in_progress`, `completed`, `blocked`, `cancelled`, `review`, `approved`, `rejected`

## Creating Tasks (Flat)

```
model_name: "Task"
action: "create"
attributes: {
  name: "Task name",
  project_id: <project_id>,
  tenant_id: 22
}
```

**Required fields:** `tenant_id`
**Key optional fields:** `name`, `project_id`, `description`, `start_date`, `due_date`, `priority`, `parent_id`, `task_type_id`, `custom_fields`

Tasks created without `parent_id` are root-level (depth 0).

**Status for tasks:** Tasks also need status applied for full UI functionality. Apply after creation:

```
apply_status_tool:
  model_name: "Task"
  id: <task_id>
  status: "task_not_started"
```

Available task statuses (under parent `task_statuses`):
- `task_not_started`, `task_in_progress`, `task_review`, `task_completed`, `task_blocked`, `task_cancelled`, `task_on_hold`, `task_backlog`

Note: Task statuses use a different prefix (`task_`) than project statuses.

## WBS Hierarchy (awesome_nested_set)

Tasks use the `awesome_nested_set` gem for tree structure. The columns `lft`, `rgt`, `depth`, and `children_count` are **managed automatically** — never set them directly.

### Indent (Create as Child)

Set `parent_id` on create to make a task a child of another:

```
model_name: "Task"
action: "create"
attributes: {
  name: "Child task",
  project_id: <project_id>,
  tenant_id: 22,
  parent_id: <parent_task_id>
}
```

The nested set columns are computed automatically:
- `depth` = parent's depth + 1
- `lft`/`rgt` = inserted after parent's last child
- parent's `children_count` is incremented

### Indent (Move Existing Task Under a Parent)

Update `parent_id` to move an existing task under a new parent:

```
model_name: "Task"
action: "update"
id: <task_id>
attributes: {
  parent_id: <new_parent_task_id>
}
```

The nested set recalculates `lft`, `rgt`, and `depth` for the moved task and all its descendants.

### Outdent (Move to Root)

Set `parent_id` to `null` to make a task root-level:

```
model_name: "Task"
action: "update"
id: <task_id>
attributes: {
  parent_id: null
}
```

### Reading the WBS Tree

To get all tasks in WBS order, sort by `lft`:

```
model_name: "Task"
action: "list"
filters: { project_id: <project_id> }
limit: -1
sort: { column: "lft", direction: "asc" }
```

The `lft` column gives depth-first traversal order — the natural WBS reading order. Use `depth` to determine indentation level.

### Filtering by Level

```
# Root tasks only
filters: { project_id: <id>, parent_id: "root" }

# Children of a specific task
filters: { project_id: <id>, parent_id: <parent_id> }
```

## Key Rules

1. **Never manipulate `lft`, `rgt`, `depth`, or `children_count` directly** — these are managed by awesome_nested_set via `parent_id` changes
2. **Always sort by `lft` ascending** to get WBS order
3. **Use `depth` for indentation** — depth 0 = root, depth 1 = first-level child, etc.
4. **Moving a parent moves its subtree** — if you change a task's `parent_id`, all its descendants move with it
5. **`parent_id: null`** = root task (outdent to top level)
6. **Tasks are scoped to tenant** — the nested set operates within the tenant boundary

## Verified Operations

| Operation | Method | Verified |
|-----------|--------|----------|
| Create project | `create` with `name`, `tenant_id` | Yes |
| Apply project status | `apply_status_tool` with `in_progress` | Yes |
| Create root task | `create` without `parent_id` | Yes |
| Create child task (indent) | `create` with `parent_id` | Yes |
| Create grandchild (depth 2) | `create` with parent = child task | Yes |
| Move task under new parent (indent) | `update` `parent_id` to target | Yes |
| Move task to root (outdent) | `update` `parent_id` to `null` | Yes |
| Read WBS in order | `list` sorted by `lft` asc | Yes |
| Nested set auto-recalculation | All `lft`/`rgt`/`depth` updates | Yes |

## UI Verification

Verified in Harmoniq UI (2026-03-15):
- Project visible in Active Projects after status applied
- WBS hierarchy renders with correct indentation (depth 0, 1, 2)
- Parent tasks show expand/collapse controls
- Cross-parent move (1.3 from Inception to Elaboration) reflected correctly
- Gantt timeline picks up all tasks
- Tasks show "Set Status" until status is applied via `apply_status_tool`

## Lifecycle-Based Project Scaffolding

The SDLC lifecycle definitions (`lifecycles/*.md`) contain structured frontmatter with phase data, discipline intensity ratings, and completion_assertions. This data can drive automatic project scaffolding in Harmoniq.

### Canonical RUP Duration Distribution (Kruchten)

For a project of total duration D:

| Phase | % of Schedule | Typical |
|-------|--------------|---------|
| Inception | ~10% | 3 weeks for 6-month project |
| Elaboration | ~30% | 8 weeks |
| Construction | ~50% | 13 weeks |
| Transition | ~10% | 3 weeks |

### Canonical RUP Effort Distribution by Discipline

Each discipline appears in every phase but with varying intensity. The percentages show what fraction of that discipline's total effort falls in each phase:

| Discipline | Inception | Elaboration | Construction | Transition |
|-----------|-----------|-------------|--------------|------------|
| Business Modeling | ~28% | ~28% | ~28% | ~16% |
| Requirements | ~38% | ~18% | ~24% | ~20% |
| Analysis & Design | ~8% | ~36% | ~36% | ~20% |
| Implementation | ~4% | ~24% | ~56% | ~16% |
| Test | ~4% | ~12% | ~52% | ~32% |
| Deployment | ~4% | ~4% | ~24% | ~68% |
| Project Management | ~16% | ~28% | ~32% | ~24% |
| Environment | ~40% | ~20% | ~20% | ~20% |

### Scaffolding Pattern

1. **Create project** with `start_date` and `end_date`
2. **Apply status** (`in_progress`)
3. **Create phase tasks** (depth 0) with dates calculated from duration percentages
4. **Create discipline tasks** (depth 1) under each phase, with date spans reflecting intensity:
   - High-intensity disciplines span the full phase
   - Medium-intensity disciplines start or end partway through
   - Low-intensity disciplines get a short window at the appropriate point
5. **Create milestone tasks** (depth 1) as zero-duration tasks on the phase end date (LCO, LCA, IOC, GA)
6. **Add descriptions** explaining WHY each discipline appears in that phase — critical for teaching the methodology

### Description Pattern

Discipline descriptions should explain the non-obvious:
- **Why is Business Modeling in Transition?** User acceptance reveals "this isn't how we actually work" gaps
- **Why is Test in Inception?** Testing assumptions, not code — testability analysis
- **Why is Implementation in Inception?** Throwaway prototypes only for critical risk mitigation
- **Why is Deployment in Elaboration?** CI/CD pipeline setup, not production deployment
- **Why is Environment in Inception at 40%?** Tooling setup pays compound dividends — highest per-phase investment

Include effort percentages in descriptions (e.g., "56% of total implementation effort") to ground the duration in the canonical data.

## Test Artifacts

### Test 1: Basic WBS (deleted)
Project "SDLC Setup Test" (id: 38) verified CRUD, indent/outdent, and status visibility.

### Test 2: Canonical RUP Layout (active)
Project "RUP Canonical Layout" (id: 39) — 6-month project, Apr 1 – Oct 2, 2026.

```
Inception (Apr 1 – Apr 18, 3 wks)
  Business Modeling          Apr 1 – Apr 18   (full phase)
  Requirements               Apr 1 – Apr 18   (full phase)
  Analysis & Design          Apr 7 – Apr 18   (starts week 2)
  Implementation             Apr 14 – Apr 18  (last week only — prototypes)
  Test                       Apr 14 – Apr 18  (last week — testing assumptions)
  Deployment                 Apr 14 – Apr 18  (last week — planning only)
  Project Management         Apr 1 – Apr 18   (full phase)
  Environment                Apr 1 – Apr 18   (full phase — 40% of total env effort)
  LCO Milestone Review       Apr 18

Elaboration (Apr 21 – Jun 13, 8 wks)
  Business Modeling          Apr 21 – May 15  (first half — refine domain model)
  Requirements               Apr 21 – May 22  (front-loaded)
  Analysis & Design          Apr 21 – Jun 13  (full phase — architecture peak)
  Implementation             Apr 28 – Jun 13  (starts week 2 — executable architecture)
  Test                       May 4 – Jun 13   (starts week 3 — test the architecture)
  Deployment                 May 18 – Jun 5   (mid-phase — CI/CD pipeline)
  Project Management         Apr 21 – Jun 13  (full phase — 28% peak)
  Environment                Apr 21 – May 15  (first half — finalize tooling)
  LCA Milestone Review       Jun 13

Construction (Jun 16 – Sep 12, 13 wks)
  Business Modeling          Jun 16 – Jul 31  (first half — winding down)
  Requirements               Jun 16 – Aug 7   (two-thirds — refine as you build)
  Analysis & Design          Jun 16 – Aug 21  (three-quarters — detailed design)
  Implementation             Jun 16 – Sep 12  (full phase — 56% of total impl)
  Test                       Jun 23 – Sep 12  (near-full — 52% of total test)
  Deployment                 Jul 27 – Sep 12  (second half — staging deploys)
  Project Management         Jun 16 – Sep 12  (full phase — 32% steady)
  Environment                Jun 16 – Jul 17  (first third — maintenance only)
  IOC Milestone Review       Sep 12

Transition (Sep 15 – Oct 2, 3 wks)
  Business Modeling          Sep 15 – Sep 19  (first week — acceptance gaps)
  Requirements               Sep 15 – Sep 19  (first week — capture changes)
  Analysis & Design          Sep 15 – Sep 22  (first half — perf tuning)
  Implementation             Sep 15 – Sep 26  (two-thirds — defect fixes only)
  Test                       Sep 15 – Oct 2   (full phase — acceptance testing)
  Deployment                 Sep 15 – Oct 2   (full phase — 68% peak, production)
  Project Management         Sep 15 – Oct 2   (full phase — closeout)
  Environment                Sep 15 – Sep 22  (first half — prod hardening)
  GA Milestone Review        Oct 2
```

All 40 tasks include descriptions explaining why each discipline appears in each phase, with canonical effort percentages.

## Next Steps

- [x] Status management — `apply_status_tool` required for UI visibility
- [x] Lifecycle-based scaffolding — canonical RUP layout with duration distribution
- [x] Discipline descriptions — teaching-quality explanations per phase
- [ ] Resource assignments (assign parties/roles to tasks — enables effort vs duration)
- [ ] Task types (classify tasks by discipline, milestone, etc.)
- [ ] Template file attachments (LlmFile linked to tasks)
- [ ] Parameterized scaffolding (accept project duration, calculate all dates automatically)
- [ ] Multi-lifecycle support (scaffold from any lifecycle definition, not just RUP)
