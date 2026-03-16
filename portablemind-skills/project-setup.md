---
name: Harmoniq Project Setup
description: How to create projects with WBS task hierarchies in Harmoniq via MCP CRUD tools
type: knowledge
verified: 2026-03-15
test_project_id: 38
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

## Test Artifacts

Project "SDLC Setup Test" (id: 38) was created to verify these operations. The resulting WBS:

```
1. Inception (id:782, depth:0)
  1.1 Define scope and vision (id:785, depth:1)
    1.1.1 Draft vision document (id:788, depth:2)
  1.2 Identify key risks (id:786, depth:1)
2. Elaboration (id:783, depth:0)
  1.3 Stakeholder alignment (id:787, depth:1)  ← moved from Inception via parent_id update
3. Construction (id:784, depth:0)
```

## Next Steps

- [x] Status management — `apply_status_tool` required for UI visibility (projects and likely tasks)
- [ ] Resource assignments (assign parties to tasks)
- [ ] Task types (classify tasks by type)
- [ ] Template file attachments (LlmFile linked to tasks)
- [ ] Bulk WBS creation pattern (create a full hierarchy from a lifecycle definition)
