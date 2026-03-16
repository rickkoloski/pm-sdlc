# SDLC Conventions

Lifecycle-independent conventions that apply regardless of which lifecycle is in use.

---

## Deliverable IDs

- Sequential across entire project: D1, D2, D3, ...
- Never reused, even if work is abandoned
- Used in filenames: `d42_memory_formation_spec.md`
- Referenced in commits: "feat: implement D42 memory formation"

---

## File Suffix Convention

All documentation files carry a type suffix so that a file's purpose is self-evident regardless of where it lives.

| Type | Suffix | Example |
|------|--------|---------|
| Specification | `_spec.md` | `d42_memory_formation_spec.md` |
| Planning/Instructions | `_plan.md` | `d42_memory_formation_plan.md` |
| Prompt (CC instructions) | `_prompt.md` | `d42_memory_formation_prompt.md` |
| Completion record | `_COMPLETE.md` | `d42_memory_formation_COMPLETE.md` |
| Issue/blocker | `_BLOCKED.md` | `d42_memory_formation_BLOCKED.md` |
| Reference/archived | `_ref.md` | `project_overview_ref.md` |
| Roadmap | `_roadmap.md` | `next_steps_roadmap.md` |

**Why this matters:**
1. **Context-free identification** — A file named `tpv_research_spec.md` is unambiguously a spec in a directory listing, git log, grep result, or cross-agent chat
2. **Multi-agent workflows** — When CD and multiple CC instances share file references, the suffix eliminates "which file do you mean?"
3. **Search and automation** — `find . -name "*_COMPLETE.md"` finds all completion records; `*_spec.md` finds all specs
4. **Directory structure is additive** — The directory (`specs/`, `planning/`) still provides organization, but the suffix ensures the file is self-describing even without it

---

## Working Locations

### `current_work/`
Active deliverables in progress:
```
current_work/
├── specs/           # What to build
├── planning/        # How to build it
├── prompts/         # CC instructions
├── stepwise_results/ # Completion records
└── issues/          # Blocked items
```

### `chronicle_by_concept/`
Completed work by domain:
```
chronicle_by_concept/
├── 01_authentication/
│   ├── _index.md    # Navigation and context
│   ├── specs/
│   ├── planning/
│   └── results/
├── 02_data_model/
└── ...
```

### `chronicle_by_step/`
Completed work by time:
```
chronicle_by_step/
├── step_01_foundation/
├── step_02_core_features/
└── ...
```

---

## When to Chronicle

Archive completed work when:
- A logical milestone is reached
- `current_work/` is getting cluttered (>20 active items)
- Starting a new phase of work
- Context refresh is needed

See `operations/chronicle_organization.md` for detailed process.

---

## Roles

See `operations/collaboration_model.md` for the full CD/CC collaboration model.

**Quick reference:**

| Role | Responsibilities |
|------|-----------------|
| **CD** (Claude Director / Human) | Sets direction, approves specs, makes architectural decisions, reviews results |
| **CC** (Claude Code) | Proposes approaches, implements features, asks clarifying questions, documents work |

---

## Key Principles

1. **Specs before code** — Define what before implementing how
2. **Explicit over implicit** — Document decisions and rationale
3. **Centralized by concept** — Related work belongs together
4. **Indexes for navigation** — `_index.md` enables targeted context loading
5. **Archives preserve memory** — Chronicles are long-term project memory
6. **Process accommodates reality** — Prototyping work is legitimate; reconcile periodically
7. **Capture testing knowledge** — Tester navigation paths and learnings compound across runs
8. **Validation before deployment** — Tests must pass before code ships
9. **Deploy is a process step** — Not an afterthought; gated by validation, verified by smoke tests

---

## Updating the SDLC

When the team discovers process improvements, gaps, or new conventions during real work:

**Trigger:** Say **"Let's update the SDLC"**

CC will:
- Read the current SDLC changelog at `operations/sdlc_changelog.md`
- Discuss the proposed addition or change
- Update the relevant canonical files
- Append to the changelog with date, description, and rationale
- Wait for approval before committing

See `operations/sdlc_changelog.md` for the change history.

---

## Compliance Auditing

Periodically verify the project follows SDLC standards:

**Trigger:** Say **"Let's run an SDLC compliance audit"**

See `operations/compliance_audit.md` for the full process.
