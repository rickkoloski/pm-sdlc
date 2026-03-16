---
name: Native
ceremony: low-moderate
iteration_unit: deliverable (variable duration)
best_for: small teams, human+AI pairs, projects using the SDLC framework directly
phases:
  idea:
    focus: identify the work, assign deliverable ID
    disciplines: { research: medium, analysis: medium }
    completion_assertions:
      - work item is identified and named
      - deliverable ID assigned (D-number)
  spec:
    focus: define the problem, requirements, approach, success criteria
    disciplines: { research: high, analysis: high, architecture: high, design: medium, data_modeling: medium }
    completion_assertions:
      - problem is clearly stated
      - requirements are specified
      - approach is designed
      - success criteria are defined
      - spec reviewed and approved by CD
  planning:
    focus: step-by-step implementation guide
    disciplines: { architecture: high, coding: medium, testing: low }
    completion_assertions:
      - specific files and functions to modify are identified
      - API signatures and patterns are defined
      - implementation steps are ordered
  prompt:
    focus: focused instructions for CC (optional)
    disciplines: { coding: medium }
    completion_assertions:
      - context and references are provided
      - explicit task list is defined
  implementation:
    focus: execute the work
    disciplines: { coding: high, testing: medium, design: medium }
    completion_assertions:
      - code compiles and runs
      - implementation follows the planning document
      - unit/integration tests pass
  validation:
    focus: verify the work meets requirements
    gate: validation_gate
    disciplines: { testing: high }
    completion_assertions:
      - CCP exploratory testing passes (Phase 1)
      - test specs written in markdown (Phase 2)
      - Playwright tests generated and passing (Phases 3-4)
      - testing knowledge files updated
  deploy:
    focus: release to staging/production
    gate: deploy_gate
    disciplines: { deployment: high, testing: medium }
    completion_assertions:
      - all Playwright tests pass (Phase 4 green)
      - deployed to target environment
      - post-deploy smoke tests pass (health checks, auth, key endpoints)
  result:
    focus: document what was done
    disciplines: { process_improvement: medium }
    completion_assertions:
      - completion record created
      - files changed are listed
      - test outcomes documented
      - deviations from spec noted
  chronicle:
    focus: archive completed work
    disciplines: { process_improvement: low }
    completion_assertions:
      - work moved to chronicle_by_concept and/or chronicle_by_step
      - _index.md updated
      - current_work cleaned up
---

# Native Lifecycle

**Ceremony level:** Low to moderate — deliverable-driven with lightweight gates
**Iteration unit:** Deliverable (one D-number flows through all phases)
**Best for:** Small teams, human+AI pairs (CD/CC), projects using this SDLC framework directly

## Core Concepts

The native lifecycle is the default working mode for this SDLC framework. It organizes work around **deliverables** — individually tracked units of work (D1, D2, ...) that flow through a defined sequence of phases. Each deliverable moves through the full flow independently.

This lifecycle sits between Prototyping and Kanban on the ceremony spectrum:

```
Prototyping ── Native ── Kanban ──── Scrum ──── RUP ──── SAFe ──── Waterfall
     │            │          │          │         │         │           │
   zero       per-item    flow +     time-boxed  phased   scaled     sequential
 ceremony     gates      WIP limits  iterations  iters    phased     phases +
                                                 + gates  iters      hard gates
```

It's more structured than prototyping (specs before code, validation before deploy) but less ceremonial than kanban (no WIP limits, no cadences, no board). The human (CD) and AI (CC) collaborate directly on one deliverable at a time.

## The Flow

```
Idea → Spec → Planning → Prompt → Implementation → Validation → Deploy → Result → Chronicle
                          (opt)
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
Create `docs/current_work/planning/dNN_name_plan.md`
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

### 6. Validation (stage gate)
A deliverable is not deployable until it passes validation. Validation follows the **5-phase hybrid testing workflow**:

| Phase | Tool | Output |
|-------|------|--------|
| **1. Explore** | CCP (Chrome Plugin) | Exploratory testing — verify features work, discover issues |
| **2. Specify** | CC | Test specs in markdown format (`docs/testing/specs/dNN_name.spec.md`) |
| **3. Generate** | CC / PW Test Agents | Executable Playwright `.spec.ts` files (`e2e/` directory) |
| **4. Execute** | Playwright CLI | Deterministic test run — all tests must pass |
| **5. Heal** | CC / PW Healer Agent | Auto-repair failing tests when UI changes (future runs) |

**Minimum bar:** Phases 1-4 must complete with all tests passing before proceeding to Deploy. Phase 5 applies to subsequent runs when existing tests break due to new changes.

**Knowledge capture:** Each validation cycle updates project-specific testing knowledge:
- `docs/testing/knowledge/app-map.yaml` — new routes, navigation changes
- `docs/testing/knowledge/element-catalog.yaml` — new page locators
- `docs/testing/knowledge/layer0-upstream.yaml` — new risk areas, acceptance criteria
- `docs/testing/knowledge/timing-profile.yaml` — measured waits for new components

Cross-project knowledge updates go to `~/src/ops/sdlc/knowledge/testing/`.

**When to skip phases:** Work done in the Prototyping lifecycle (bug fixes, UI tweaks <30 min) may skip Phases 2-4 if Phase 1 (exploratory CCP) provides sufficient confidence. Document the rationale in the result doc.

### 7. Deploy (stage gate)
Deployment is gated by validation:
- **Prerequisite:** All Playwright tests pass (Phase 4 green)
- **Action:** Deploy to staging/production per project deployment docs
- **Verification:** Post-deploy smoke test (health checks, auth, key endpoints)
- **Parallel option:** CD may authorize early deployment for human review while Playwright tests run in parallel, but the deliverable is not marked Complete until both gates pass

A deliverable deployed without passing validation remains **In Progress**, not Complete.

### 8. Result
Create `docs/current_work/stepwise_results/dNN_name_COMPLETE.md`
- What was implemented
- Files changed
- Test outcomes (CCP + Playwright results)
- Deploy verification
- Any deviations

### 9. Chronicle
Periodically move completed work to archives:
- `chronicle_by_concept/` — organized by domain
- `chronicle_by_step/` — organized chronologically

See `operations/chronicle_organization.md` for detailed process.

## Discipline Intensity Map

```
                    Idea  Spec    Planning  Impl     Validate  Deploy  Result
                    ────  ────    ────────  ────     ────────  ──────  ──────
Product Research    ████  ████    █
Business Analysis   ████  ████████  ██      █
Design              █     ████████  ████    ██████   ██
Architecture        ██    ████████  ████████  ████   ██
Coding                             ████     ████████████████   ██
Testing                   ██       ██       ████     ████████████████
Deployment                                  █        ██       ████████
Data Modeling       ██    ████████  ██
Process Improvement                                                    ████████
```

## Rules of Goodness

**Doing the native lifecycle well:**
- Specs exist before implementation starts for any non-trivial work
- Validation is a real gate — "all tests pass" means the UI actually works, not just that APIs return 200
- Results document what actually happened, including deviations
- Chronicles keep current_work clean and project memory alive
- The CD/CC collaboration model (see `operations/collaboration_model.md`) is followed

**Common failure modes:**
- **Spec theater:** Specs are written but not reviewed. CD approves without reading. Specs don't reflect what actually gets built.
- **Validation shortcuts:** Declaring victory when API tests pass without verifying the UI end-to-end. See the testing discipline's persistence rule.
- **Chronicle neglect:** current_work accumulates dozens of completed deliverables. Context loading becomes expensive. Navigation is lost.
- **Over-specification:** Writing detailed specs for 10-minute tasks. Use the Prototyping lifecycle for small work and reconcile later.

## Mapping to Other Lifecycles

The native lifecycle maps cleanly because it IS the mapping target — each lifecycle file in `lifecycles/` includes a "Mapping to Our SDLC" section showing how its phases correspond to this flow.

| Native phase | Is analogous to |
|-------------|----------------|
| Idea | Backlog item, use case, PBI, change request |
| Spec | Requirements, acceptance criteria, feature spec |
| Planning | Iteration plan, sprint backlog, task breakdown |
| Implementation | Development, coding phase |
| Validation | Testing, verification, DoD check |
| Deploy | Release, transition, deployment phase |
| Result | Sprint review, completion record, gate review |
| Chronicle | Retrospective, archival, project closeout |

## Tester Knowledge Capture

When a tester agent (human or AI) completes a test pass, the test results document should include a **Navigation & Learnings** section that captures reusable knowledge:

**Navigation Paths** — How to reach each test area:
- Exact URLs, click sequences, scroll directions needed
- UI quirks (e.g., "must scroll right to see Actions column")
- Which icons/buttons to click and where they are

**Lessons Learned** — What future testers should know:
- Common gotchas encountered during testing
- Data prerequisites (e.g., "need at least one assigned user to test unassign")
- Timing issues (e.g., "wait for toast to dismiss before re-opening flyout")
- Environment-specific notes (e.g., "dev has no email data populated")

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Work is exploratory, specs feel premature | → Prototyping (reconcile back later) |
| Multiple deliverables need flow control | → Kanban (add WIP limits and board) |
| Team needs predictable cadence for stakeholders | → Scrum (add sprints and ceremonies) |
| Significant architectural risk needs early resolution | → RUP (add phase milestones) |
