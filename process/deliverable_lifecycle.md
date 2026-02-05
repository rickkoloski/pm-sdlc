# Deliverable Lifecycle

## States

```
[Draft] → [Ready] → [In Progress] → [Complete] → [Archived]
                 ↘ [Blocked] ↗
```

### Draft
Spec exists but is incomplete or under review.
- File: `dNN_name_spec.md`
- Status marker in file: `**Status:** Draft`

### Ready
Spec is approved and has planning/instructions.
- Files: spec + planning + optional prompt
- Status marker: `**Status:** Ready`

### In Progress
Implementation is underway.
- CC is actively working on it
- Status marker: `**Status:** In Progress`

### Blocked
Work cannot proceed.
- File: `dNN_name_BLOCKED.md` in `issues/`
- Contains: what's blocked, why, what's needed

### Complete
Implementation is finished and verified.
- File: `dNN_name_COMPLETE.md` in `stepwise_results/`
- Status marker: `**Status:** Complete`

### Archived
Moved to chronicles for long-term reference.
- Files copied to `chronicle_by_concept/` and `chronicle_by_step/`
- Removed from `current_work/`

---

## Transitions

### Draft → Ready
- Spec is reviewed and approved
- Planning document is written
- Optional: CC prompt is prepared

### Ready → In Progress
- CC begins implementation
- Or: human begins implementation

### In Progress → Complete
- All tasks finished
- Tests pass
- Result document created

### In Progress → Blocked
- Obstacle encountered
- Issue document created
- Work pauses until resolved

### Blocked → In Progress
- Blocker resolved
- Issue document updated or removed
- Work resumes

### Complete → Archived
- Periodic chronicle organization
- Files moved to appropriate concept/step
- `_index.md` updated

---

## File Naming

| State | File Pattern |
|-------|--------------|
| Draft/Ready | `dNN_name_spec.md` |
| Planning | `dNN_name_plan.md` |
| Prompt | `dNN_name_prompt.md` |
| Complete | `dNN_name_COMPLETE.md` |
| Blocked | `dNN_name_BLOCKED.md` |

---

## Status Markers

Every spec should have a status block at the top:

```markdown
# DNN: Feature Name — Specification

**Status:** Draft | Ready | In Progress | Complete
**Created:** YYYY-MM-DD
**Updated:** YYYY-MM-DD
**Depends On:** D1, D5 (if any)
```
