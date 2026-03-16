---
name: Scaffold Project
description: Parameterized project scaffolding from SDLC lifecycle definitions — reads frontmatter, calculates dates, creates Harmoniq project with WBS hierarchy
type: skill
depends_on: project-setup.md
verified: false
---

# Scaffold Project

Generate a fully structured Harmoniq project from an SDLC lifecycle definition and three parameters.

## Input Parameters

| Parameter | Required | Format | Example |
|-----------|----------|--------|---------|
| `lifecycle` | Yes | Lifecycle filename (without .md) | `rup` |
| `name` | Yes | Project name string | `"Alpha Release"` |
| `start_date` | Yes | ISO date | `2026-04-01` |
| `duration_weeks` | Yes* | Integer | `26` |
| `end_date` | Alt* | ISO date | `2026-10-02` |

*Provide either `duration_weeks` or `end_date`. If both given, `end_date` wins.

## Algorithm

### Step 1: Load Lifecycle Data

Read `lifecycles/{lifecycle}.md` and parse YAML frontmatter. Extract the `scaffolding` key. If no `scaffolding` key exists, stop and report that the lifecycle doesn't support parameterized scaffolding yet.

Required scaffolding fields:
- `structure_type` — currently only `phases` is implemented
- `phase_durations` — map of phase key → decimal (must sum to 1.0)
- `disciplines` — ordered list of `{ key, label }` objects
- `intensity_spans` — map of intensity level → `[start_pct, end_pct]`
- `phase_disciplines` — map of phase key → map of discipline key → intensity level
- `milestones` — map of phase key → milestone task name

Optional scaffolding fields:
- `descriptions` — relative path to a descriptions file (e.g., `portablemind-skills/rup-descriptions.md`)

### Step 1b: Load Descriptions

If `scaffolding.descriptions` is set, read the referenced file. Descriptions are keyed by markdown headings in the format `### {phase}.{discipline}` (e.g., `### inception.requirements`). The text below each heading is the task description.

Special keys:
- `{phase}.phase` — description for the phase task itself (depth 0)
- `{phase}.milestone` — description for the milestone task

If no descriptions file exists, tasks are created without descriptions.

### Step 2: Calculate Project Dates

```
If end_date provided:
  total_days = end_date - start_date (calendar days)
Else:
  total_days = duration_weeks * 7
  end_date = start_date + total_days

total_weeks = total_days / 7  (keep as float)
```

### Step 3: Calculate Phase Weeks

Convert percentages to whole weeks. The last phase absorbs rounding remainder.

```
for each phase except last:
  phase_weeks[phase] = max(1, round(total_weeks * phase_durations[phase]))

used_weeks = sum(phase_weeks for all except last)
# Last phase gets whatever remains (measured by actual dates, not weeks)
```

### Step 4: Calculate Phase Boundaries

Work in weeks, aligned to Monday–Friday boundaries:

```
cursor = start_date

for each phase in order:
  phase_start = cursor
  if last phase:
    phase_end = project end_date
  else:
    first_friday = next_friday_on_or_after(phase_start)
    phase_end = first_friday + (phase_weeks[phase] - 1) * 7 days
  store phase_start, phase_end
  cursor = next_monday_after(phase_end)
```

This produces clean Mon–Fri boundaries for all phases except possibly the first (which starts on the project start date, whatever day that is).

### Step 5: Create Project

```
general_crud_tool:
  model_name: "Project"
  action: "create"
  attributes:
    name: {name}
    description: "Scaffolded from {lifecycle} lifecycle, {duration_weeks} weeks"
    tenant_id: 22
    start_date: {start_date}
    end_date: {end_date}

apply_status_tool:
  model_name: "Project"
  id: {project_id}
  status: "in_progress"
```

### Step 6: Create Phase Tasks (Depth 0)

For each phase, create a root-level task:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "{Phase Name}"   # capitalize phase key
    project_id: {project_id}
    tenant_id: 22
    start_date: {phase_start}
    due_date: {phase_end}
    description: "{descriptions[phase.phase]}"  # from descriptions file, or phase.focus as fallback

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

### Step 7: Create Discipline Tasks (Depth 1)

For each discipline in the phase (from `phase_disciplines`), calculate the date span using `intensity_spans`:

```
intensity = phase_disciplines[phase][discipline]
[start_pct, end_pct] = intensity_spans[intensity]
phase_days = (phase_end - phase_start).days
discipline_start = phase_start + round(phase_days * start_pct)
discipline_end = phase_start + round(phase_days * end_pct)

# Weekend snap: start → forward to Monday, end → back to Friday
if discipline_start falls on Sat/Sun: snap forward to Monday
if discipline_end falls on Sat: snap back to Friday
if discipline_end falls on Sun: snap back to Friday
```

Then create the task:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "{discipline.label}"
    project_id: {project_id}
    parent_id: {phase_task_id}
    tenant_id: 22
    start_date: {discipline_start}
    due_date: {discipline_end}
    description: "{descriptions[phase.discipline_key]}"  # from descriptions file

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

### Step 8: Create Milestone Tasks (Depth 1)

For each phase with a milestone, create a zero-duration task on the phase end date:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "{milestones[phase]}"
    project_id: {project_id}
    parent_id: {phase_task_id}
    tenant_id: 22
    start_date: {phase_end}
    due_date: {phase_end}
    description: "{descriptions[phase.milestone]}"  # from descriptions file

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

## Date Rules

1. **Week-based calculation:** Phase durations are converted to whole weeks (rounded). This produces clean Mon–Fri boundaries naturally.
2. **Phase end = Friday:** Each phase ends on a Friday. The end is calculated as: first Friday on or after phase start + (weeks - 1) * 7 days.
3. **Phase start = Monday:** Each phase (except possibly the first) starts on the Monday after the previous phase's Friday.
4. **First phase:** Starts on the project start date, regardless of day of week.
5. **Last phase:** Ends on the project end date (absorbs rounding from week conversion).
6. **Minimum duration:** Every phase gets at least 1 week, regardless of percentage.
7. **Discipline weekend snap:** Start dates snap forward to Monday if they fall on Sat/Sun. End dates snap back to Friday if they fall on Sat/Sun.
8. **Tolerance:** The algorithm produces dates within ~1 day of hand-calculated layouts. This is acceptable for the basic parameterization.

## Execution Notes

### MCP Tools Required

- `general_crud_tool` — CRUD operations on Project and Task models
- `apply_status_tool` — status application (required for UI visibility)
- `model_schema_tool` — optional, for discovering field names if needed

### Tenant Context

- `tenant_id: 22` (portablemind)
- `owner_party_id: 19` (Richard Koloski) — auto-assigned if not specified

### Task Creation Order

Create tasks in this exact order to ensure correct WBS (lft/rgt) ordering:

1. Project
2. Phase 1 (depth 0)
3. Phase 1 disciplines (depth 1, in discipline list order)
4. Phase 1 milestone (depth 1, last child)
5. Phase 2 (depth 0)
6. Phase 2 disciplines...
7. ...continue for all phases

The awesome_nested_set gem assigns `lft` values based on creation order within a parent, so creating tasks in WBS order produces the correct tree.

### Error Handling

- If `phase_durations` don't sum to 1.0 (within 0.01 tolerance), warn and normalize
- If a discipline key in `phase_disciplines` isn't in the `disciplines` list, skip it with a warning
- If MCP tool calls fail, report the error and the last successfully created task so work can resume

## Extending to Other Lifecycles

To add scaffolding support to a new lifecycle:

1. Add a `scaffolding` key to the lifecycle's YAML frontmatter
2. Set `structure_type` (currently only `phases` supported)
3. Define `phase_durations` — must sum to 1.0
4. Define `disciplines` — the task labels to create under each phase
5. Define `intensity_spans` — can reuse `{ high: [0,1], medium: [0.25,1], low: [0.67,1] }` or define custom spans
6. Define `phase_disciplines` — which disciplines appear in which phases at what intensity
7. Define `milestones` — phase-end milestone names (optional)

### Structure Types (Future)

- `phases` — sequential temporal divisions with duration percentages (RUP, waterfall, native)
- `sprints` — repeating timebox pattern (scrum) — not yet implemented
- `columns` — flow-based, no temporal divisions (kanban) — not yet implemented
- `moments` — non-linear progression (prototyping) — not yet implemented

## Validation Checklist

After scaffolding a project, verify in the Harmoniq UI:

- [ ] Project visible in Active Projects
- [ ] Phase tasks at depth 0 with correct date spans
- [ ] Discipline tasks at depth 1 under each phase
- [ ] Milestone tasks at depth 1, zero-duration, on phase end dates
- [ ] All tasks show status (not "Set Status")
- [ ] WBS order matches expected structure (phases in order, disciplines in order within phases)
- [ ] Gantt timeline shows reasonable date distribution
- [ ] Phase boundaries don't overlap (each phase starts day after previous ends)
