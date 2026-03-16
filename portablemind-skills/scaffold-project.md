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

Required scaffolding fields vary by `structure_type`:

**For `phases` (e.g., RUP, Waterfall):**
- `phase_durations` — map of phase key → decimal (must sum to 1.0)
- `disciplines` — ordered list of `{ key, label }` objects
- `intensity_spans` — map of intensity level → `[start_pct, end_pct]`
- `phase_disciplines` — map of phase key → map of discipline key → intensity level
- `milestones` — map of phase key → milestone task name

**For `sprints` (e.g., Scrum):**
- `sprint_length_weeks` — integer (1-4, typically 2)
- `events` — ordered list of `{ key, label, timing: [start_pct, end_pct] }` objects

**Common optional fields:**
- `descriptions` — relative path to a descriptions file

### Step 1b: Load Descriptions

If `scaffolding.descriptions` is set, read the referenced file. Descriptions are keyed by markdown headings in the format `### {context}.{key}`.

**For phases:** `### {phase}.{discipline}` (e.g., `### inception.requirements`)
- `{phase}.phase` — description for the phase task itself (depth 0)
- `{phase}.milestone` — description for the milestone task

**For sprints:** `### sprint.{event}` (e.g., `### sprint.sprint_planning`)
- `sprint.phase` — description for each Sprint N task (depth 0, reused across all sprints)
- `sprint.{event_key}` — description for each event task (depth 1, reused across all sprints)

**For moments:** `### moment.{key}` (e.g., `### moment.friction`)
- Each moment gets its own description explaining what it feels like and when to add structure

**For program_increments:** `### pi.{element}` (e.g., `### pi.pi_planning`)
- `pi.phase` — description for each PI N task (depth 0, reused across all PIs)
- `pi.iteration` — description for each Iteration N task (depth 1, reused)
- `pi.{event_key}` — description for PI Planning, IP Iteration, Inspect & Adapt

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

**If `structure_type` is `phases`, continue with Steps 3-8. If `sprints`, skip to Steps 3s-6s. If `program_increments`, skip to Steps 3p-7p. If `moments`, skip to Steps 3m-4m.**

---

## Phases Algorithm (RUP, Waterfall, etc.)

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

---

## Sprints Algorithm (Scrum)

### Step 3s: Calculate Sprint Count

```
sprint_length = scaffolding.sprint_length_weeks
num_full_sprints = floor(total_weeks / sprint_length)
remainder_weeks = total_weeks - (num_full_sprints * sprint_length)

if remainder_weeks >= 1:
  num_sprints = num_full_sprints + 1  # shortened final sprint
  warn: "Final sprint is {remainder_weeks} weeks (shorter than standard {sprint_length})"
else:
  num_sprints = num_full_sprints

if num_sprints == 0: num_sprints = 1  # minimum 1 sprint
```

### Step 4s: Calculate Sprint Boundaries

```
cursor = start_date

for i in 1..num_sprints:
  sprint_start = cursor
  if i == num_sprints:
    sprint_end = project end_date  # last sprint absorbs remainder
  else:
    first_friday = next_friday_on_or_after(sprint_start)
    sprint_end = first_friday + (sprint_length - 1) * 7 days
  store sprint_start, sprint_end
  cursor = next_monday_after(sprint_end)
```

### Step 5s: Create Sprint Tasks (Depth 0)

For each sprint, create a root-level task:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "Sprint {i}"
    project_id: {project_id}
    tenant_id: 22
    start_date: {sprint_start}
    due_date: {sprint_end}
    description: "{descriptions[sprint.phase]}"

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

### Step 6s: Create Event Tasks (Depth 1)

For each event in the `events` list, calculate dates from the `timing` field:

```
[start_pct, end_pct] = event.timing
sprint_days = (sprint_end - sprint_start).days
event_start = sprint_start + round(sprint_days * start_pct)
event_end = sprint_start + round(sprint_days * end_pct)

# Weekend snap
if event_start falls on Sat/Sun: snap forward to Monday
if event_end falls on Sat: snap back to Friday
if event_end falls on Sun: snap back to Friday

# Single-day events: when start_pct == end_pct, both dates are the same day
```

Then create the task:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "{event.label}"
    project_id: {project_id}
    parent_id: {sprint_task_id}
    tenant_id: 22
    start_date: {event_start}
    due_date: {event_end}
    description: "{descriptions[sprint.event_key]}"

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

Repeat for all sprints. Each sprint gets the same set of events with the same descriptions — the pattern repeats.

---

## Program Increments Algorithm (Essential SAFe)

### Step 3p: Calculate PI Count and Structure

```
pi_length = scaffolding.pi_length_weeks          # typically 10
iter_length = scaffolding.iteration_length_weeks  # typically 2
ip_weeks = scaffolding.ip_iteration_weeks         # typically 2

num_pis = max(1, round(total_weeks / pi_length))
```

For each PI, calculate the internal structure. PI Planning and Inspect & Adapt are **overlapping events** — they share dates with the first iteration and IP iteration respectively, matching how SAFe actually works:

```
dev_weeks = pi_actual_weeks - ip_weeks
num_iterations = max(1, floor(dev_weeks / iter_length))

# IP iteration gets the final ip_weeks (shortened if PI is short)
ip_actual_weeks = min(ip_weeks, max(1, pi_actual_weeks - num_iterations * iter_length))
```

If the project is shorter than `pi_length`, create a single shortened PI. Reduce IP to 1 week if needed, then derive iteration count from remaining time.

### Step 4p: Calculate PI Boundaries

```
cursor = start_date

for i in 1..num_pis:
  pi_start = cursor
  if i == num_pis:
    pi_end = project end_date
  else:
    first_friday = next_friday_on_or_after(pi_start)
    pi_end = first_friday + (pi_length - 1) * 7 days
  store pi_start, pi_end
  cursor = next_monday_after(pi_end)
```

### Step 5p: Create PI Tasks (Depth 0)

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "PI {i}"
    project_id: {project_id}
    tenant_id: 22
    start_date: {pi_start}
    due_date: {pi_end}
    description: "{descriptions[pi.phase]}"

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

### Step 6p: Create PI Internal Structure (Depth 1)

Create child tasks in this order for correct WBS:

**1. PI Planning** — first 2 business days of the PI (overlaps with Iteration 1):
```
name: "PI Planning"
start_date: {pi_start}
due_date: {pi_start + 1 business day}
description: "{descriptions[pi.pi_planning]}"
```

**2. Development Iterations** — 2 weeks each, starting from PI start (Iteration 1 shares its first week with PI Planning):
```
iter_cursor = pi_start

for j in 1..num_iterations:
  iter_start = iter_cursor
  first_friday = next_friday_on_or_after(iter_start)
  iter_end = first_friday + (iter_length - 1) * 7 days

  name: "Iteration {j}"
  start_date: {iter_start}
  due_date: {iter_end}
  description: "{descriptions[pi.iteration]}"

  iter_cursor = next_monday_after(iter_end)
```

**3. IP Iteration** — starts after last dev iteration, runs to PI end:
```
name: "IP Iteration"
start_date: {iter_cursor}
due_date: {pi_end}
description: "{descriptions[pi.ip_iteration]}"
```

**4. Inspect & Adapt** — last business day of the PI (overlaps with IP end):
```
name: "Inspect & Adapt"
start_date: {pi_end}
due_date: {pi_end}
description: "{descriptions[pi.inspect_and_adapt]}"
```

Apply `task_not_started` status to all tasks after creation.

Repeat for all PIs. Each PI gets the same internal structure; iteration count may vary for shortened PIs.

---

## Moments Algorithm (Prototyping)

The lightest scaffolding. No date subdivision — moments are states you recognize, not time blocks. All moment tasks span the full project duration. Teams advance them through statuses as they occur.

### Step 3m: Create Moment Tasks (Depth 0)

For each moment in the `moments` list:

```
general_crud_tool:
  model_name: "Task"
  action: "create"
  attributes:
    name: "{moment.label}"
    project_id: {project_id}
    tenant_id: 22
    start_date: {project_start}
    due_date: {project_end}
    description: "{descriptions[moment.key]}"

apply_status_tool:
  model_name: "Task"
  id: {task_id}
  status: "task_not_started"
```

### Step 4m: Usage Guidance

After creating the tasks, inform the user:

- Move each moment to **In Progress** when you recognize you're in that state
- Move to **Completed** when the moment has passed
- Moments are not sequential — you may revisit Friction multiple times
- When **Commitment** is reached, choose a target lifecycle for ongoing work
- **Reentry** only applies to breakout prototypes that need to rejoin a structured lifecycle

No date calculation needed. Total tasks = number of moments (typically 5).

---

## Date Rules

1. **Week-based calculation:** Durations (phase or sprint) are in whole weeks. This produces clean Mon–Fri boundaries naturally.
2. **Division end = Friday:** Each phase or sprint ends on a Friday. Calculated as: first Friday on or after start + (weeks - 1) * 7 days.
3. **Division start = Monday:** Each division (except possibly the first) starts on the Monday after the previous division's Friday.
4. **First division:** Starts on the project start date, regardless of day of week.
5. **Last division:** Ends on the project end date (absorbs rounding).
6. **Minimum duration:** Every phase gets at least 1 week. Every sprint gets at least 1 week (shortened final sprint).
7. **Child task weekend snap:** Start dates snap forward to Monday if they fall on Sat/Sun. End dates snap back to Friday if they fall on Sat/Sun.
8. **Single-day events:** When timing `[x, x]` (start_pct == end_pct), both start and end are the same day. This is common for Scrum ceremonies (Sprint Planning, Review, Retro).
9. **Tolerance:** The algorithm produces dates within ~1 day of hand-calculated layouts.

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

### Structure Types

- `phases` — sequential temporal divisions with duration percentages (RUP, Waterfall, Native)
- `sprints` — repeating timebox pattern (Scrum)
- `program_increments` — repeating timeboxes with internal structure (SAFe)
- `columns` — flow-based, no temporal divisions (Kanban) — not yet implemented
- `moments` — non-linear progression, no date subdivision (Prototyping)

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
