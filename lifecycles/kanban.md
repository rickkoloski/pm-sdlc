---
name: Kanban
ceremony: low
iteration_unit: none (continuous flow with cadences)
best_for: support/ops teams, mature products with steady demand, teams transitioning from prototyping
columns:
  backlog:
    focus: intake and prioritization
    disciplines: { research: low, analysis: medium }
    completion_assertions:
      - acceptance criteria defined
      - no unresolved blockers
  ready:
    focus: prepared for work
    disciplines: { analysis: medium, design: low }
    completion_assertions:
      - item meets Definition of Ready
      - WIP limit not exceeded in next column
  in_progress:
    focus: active development
    disciplines: { coding: high, design: medium, architecture: medium, testing: medium }
    completion_assertions:
      - implementation complete
      - unit tests pass
  review:
    focus: quality verification
    disciplines: { testing: high, architecture: low }
    completion_assertions:
      - code reviewed and feedback addressed
      - integration tests pass
  validate:
    focus: acceptance verification
    disciplines: { testing: high, analysis: medium }
    completion_assertions:
      - acceptance criteria met
      - no regressions in existing functionality
  done:
    focus: delivered
    disciplines: { deployment: medium }
    completion_assertions:
      - deployed or ready to deploy
      - stakeholder accepts the result
cadences:
  standup: { frequency: daily, focus: "walk the board, surface blockers" }
  replenishment: { frequency: weekly, focus: "pull new items into ready" }
  delivery_review: { frequency: biweekly, focus: "flow metrics, cycle time, throughput" }
  risk_review: { frequency: monthly, focus: "blockers, dependencies, capacity" }
---

# Kanban Lifecycle

**Ceremony level:** Low — flow-based with cadences, not time-boxes
**Iteration unit:** None (continuous flow); cadences for review and planning
**Best for:** Support/ops teams, mature products with steady demand, teams transitioning from prototyping

## Core Concepts

Kanban organizes work as a **continuous flow** through defined stages, governed by **work-in-progress (WIP) limits** rather than time-boxed iterations. There are no sprints, no fixed planning cycles — work is pulled when capacity is available.

The power of kanban is visibility. The board makes bottlenecks, overload, and blocked work immediately obvious. Discipline application is **continuous and even** rather than front-loaded (waterfall) or sprint-shaped (scrum).

### Key Principles

1. **Visualize the workflow** — every work item has a visible state
2. **Limit WIP** — the single most important rule; prevents overload and exposes bottlenecks
3. **Manage flow** — optimize for throughput and lead time, not utilization
4. **Make policies explicit** — Definition of Done, WIP limits, pull criteria are visible
5. **Improve collaboratively** — use metrics (lead time, throughput) to drive improvement

## Phase Structure

Kanban doesn't have phases in the traditional sense. It has **columns** (workflow stages) that work flows through:

```
Backlog → Ready → In Progress → Review → Validate → Done
           WIP:3    WIP:3        WIP:2    WIP:2
```

Columns are customized to the team's actual workflow. Common patterns:

**Development-focused:**
```
Backlog → Analysis → Design → Build → Test → Deploy → Done
```

**Support-focused:**
```
Triage → Investigate → Fix → Verify → Release → Done
```

**Mixed (feature + support):**
```
Backlog → Spec → Build → Review → Validate → Deploy → Done
                   ↑
Incidents → Triage → Fix → Verify ──────────────────→ Done
```

### Swim Lanes

Work types can be separated into horizontal swim lanes with independent WIP limits:
- **Expedite lane:** Urgent items that bypass normal WIP limits (use sparingly)
- **Standard lane:** Normal feature/maintenance work
- **Improvement lane:** Technical debt, process improvements, tooling

## Milestones & Stage Gates

Kanban uses **pull criteria** instead of stage gates — work is pulled into the next column only when it meets the criteria and capacity exists:

| Transition | Pull criteria |
|-----------|--------------|
| Backlog → Ready | Acceptance criteria defined, no blockers |
| Ready → In Progress | WIP limit not exceeded, developer available |
| In Progress → Review | Implementation complete, tests pass |
| Review → Validate | Code reviewed, feedback addressed |
| Validate → Done | Acceptance criteria met, deployed (or ready to deploy) |

### Cadences (not ceremonies)

Kanban replaces fixed ceremonies with **cadences** — regular but flexible meetings:

| Cadence | Frequency | Purpose |
|---------|-----------|---------|
| **Standup** | Daily | Walk the board right-to-left; focus on blocked items |
| **Replenishment** | Weekly or as needed | Pull new items from backlog into Ready |
| **Delivery planning** | As needed | Coordinate releases and deployments |
| **Service delivery review** | Bi-weekly/monthly | Review flow metrics, identify improvements |
| **Risk review** | Monthly | Assess blockers, dependencies, capacity risks |

## Discipline Intensity Map

In kanban, discipline intensity is **continuous and proportional** rather than phased. Every discipline is always active at a baseline level, with intensity driven by the current mix of work:

```
                    Continuous flow →
                    ───────────────────────────────────
Product Research    ██  ██  ██  ██  ██  ██  ██  ██
Business Analysis   ███ ███ ███ ███ ███ ███ ███ ███
Design              ███ ███ ███ ███ ███ ███ ███ ███
Architecture        ██  ██  ██  ██  ██  ██  ██  ██
Coding              ████████████████████████████████
Testing             ████████████████████████████████
Deployment          ██  ██  ██  ██  ██  ██  ██  ██
Data Modeling       █   █   █   █   █   █   █   █
Process Improvement █   █   ██  █   █   █   ██  █
```

The pattern is flat with small pulses — no big humps. This is the visual signature of kanban: disciplines are always "on," tuned by the current work item, not the lifecycle phase.

## Rules of Goodness

**Doing kanban well:**
- WIP limits are enforced, not aspirational. When a column is full, you help finish existing work before starting new work.
- The board reflects reality. Every active work item is visible, in the correct column.
- Lead time (idea to done) is measured and trending downward (or stable).
- Blocked items are escalated immediately, not left to age.
- The team pulls work; managers don't push it.
- Policies (DoD, pull criteria) are explicit and posted on or near the board.

**Common failure modes:**
- **WIP limits ignored:** "We'll just make an exception" becomes the norm. The board becomes a to-do list, not a flow system. Bottlenecks hide.
- **Push, not pull:** Work is assigned rather than pulled. Developers are overloaded. WIP limits exist on paper but not in practice.
- **No cadences:** Without regular review, the board drifts from reality. Blocked items rot. Metrics aren't collected.
- **Treating kanban as "scrum without sprints":** Kanban is a fundamentally different model. It's not about removing ceremonies — it's about managing flow.
- **Expedite abuse:** Too many items in the expedite lane. If everything is urgent, nothing is. The expedite lane should be used for genuine emergencies only.

## Configurations

### Personal Kanban
Solo practitioner. 3-column board (To Do, Doing, Done). WIP limit of 2-3. No cadences — just the board and the discipline to respect WIP limits.

### Team Kanban
5-7 column board with swim lanes. Daily standup, weekly replenishment. Flow metrics tracked. Pull criteria explicit.

### Kanban for Portfolio
Multiple teams' boards feed into a portfolio-level board. Each team board is a "cell" in the larger flow. WIP limits cascade: portfolio limits constrain how many initiatives are in flight, which constrains team-level intake.

## Mapping to Our SDLC

| Our artifact | Kanban equivalent |
|-------------|-------------------|
| Idea (D-number) | Card created in Backlog |
| Spec | Written when card enters Ready (pull criterion) |
| Planning | Implicit in card breakdown and acceptance criteria |
| Implementation | Card moves through Build column |
| Validation | Card in Validate column; testing discipline active |
| Deploy | Card transitions to Done (or separate Deploy column) |
| Result | Card completion record; may be lightweight if card is small |
| Chronicle | Periodic archival of Done cards with context |

### Key difference from our native process
Our native process is **deliverable-centric** (one flow per D-number). Kanban is **card-centric** — a deliverable might be one card or several. The mapping is flexible: small work items are single cards, larger features decompose into multiple cards that flow independently.

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Need for time-boxed predictability (stakeholder commitments) | → Scrum |
| Solo work, exploratory | → Prototyping |
| Regulatory requirements, fixed milestones | → RUP or Waterfall |
| Multiple teams need coordination | → SAFe (or portfolio kanban) |
