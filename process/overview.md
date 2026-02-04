# SDLC Process Overview

## The Flow

```
Idea → Spec → Planning → Prompt → Implementation → Result → Chronicle
```

### 1. Idea
A feature, fix, or improvement is identified. Assign a deliverable ID: **D1, D2, ... Dnn**

### 2. Spec
Create `docs/current_work/specs/dNN_name_spec.md`
- Define the problem
- Specify requirements
- Design the approach
- Set success criteria

### 3. Planning
Create `docs/current_work/planning/dNN_name_instructions.md`
- Step-by-step implementation guide
- Specific files and functions to modify
- API signatures and patterns to follow

### 4. Prompt (optional)
Create `docs/current_work/prompts/dNN_name_prompt.md`
- Focused instructions for CC
- Context and references
- Explicit task list

### 5. Implementation
CC (or human) executes the work:
- Follow the planning document
- Create/modify code
- Run tests

### 6. Result
Create `docs/current_work/stepwise_results/dNN_name_COMPLETE.md`
- What was implemented
- Files changed
- Test outcomes
- Any deviations

### 7. Chronicle
Periodically move completed work to archives:
- `chronicle_by_concept/` — organized by domain
- `chronicle_by_step/` — organized chronologically

---

## Roles

### CD (Claude Director / Human)
- Defines what to build (specs)
- Reviews proposals and results
- Makes architectural decisions
- Guides priority and scope

### CC (Claude Code)
- Proposes approaches
- Implements features
- Asks clarifying questions
- Documents completion

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

## Deliverable IDs

- Sequential across entire project: D1, D2, D3, ...
- Never reused, even if work is abandoned
- Used in filenames: `d42_memory_formation_spec.md`
- Referenced in commits: "feat: implement D42 memory formation"

---

## When to Chronicle

Archive completed work when:
- A logical milestone is reached
- `current_work/` is getting cluttered (>20 active items)
- Starting a new phase of work
- Context refresh is needed

See `chronicle_organization.md` for detailed process.

---

## Key Principles

1. **Specs before code** — Define what before implementing how
2. **Explicit over implicit** — Document decisions and rationale
3. **Centralized by concept** — Related work belongs together
4. **Indexes for navigation** — `_index.md` enables targeted context loading
5. **Archives preserve memory** — Chronicles are long-term project memory
