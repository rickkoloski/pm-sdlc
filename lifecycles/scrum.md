---
name: Scrum
ceremony: moderate
iteration_unit: sprint (1-4 weeks, typically 2)
best_for: cross-functional teams building products iteratively, organizations wanting regular delivery cadence
phases:
  sprint_planning:
    focus: select work, define sprint goal, create sprint backlog
    disciplines: { analysis: high, architecture: high, design: medium, research: medium }
    completion_assertions:
      - sprint goal is a single clear sentence
      - sprint backlog is committed and understood by the team
      - no item in the sprint lacks acceptance criteria
  daily_work:
    focus: implement, test, integrate
    disciplines: { coding: high, testing: high, design: high, architecture: medium }
    completion_assertions:
      - each item meets Definition of Done before being marked complete
      - impediments surfaced at standup are actively being resolved
  sprint_review:
    focus: demonstrate increment, gather feedback
    disciplines: { analysis: medium, research: medium }
    completion_assertions:
      - working increment demonstrated to stakeholders
      - feedback captured and added to backlog
  sprint_retrospective:
    focus: inspect process, identify improvements
    disciplines: { process_improvement: high }
    completion_assertions:
      - at least one actionable improvement identified
      - improvement is assigned and will be tracked next sprint
events:
  sprint_planning: { duration: "4 hours max", cadence: "start of sprint" }
  daily_standup: { duration: "15 minutes", cadence: daily }
  sprint_review: { duration: "2 hours max", cadence: "end of sprint" }
  sprint_retrospective: { duration: "1.5 hours max", cadence: "end of sprint" }
  backlog_refinement: { duration: "~10% of sprint capacity", cadence: ongoing }
gates:
  definition_of_ready:
    - item has acceptance criteria
    - item is estimated
    - no unresolved dependencies
  definition_of_done:
    - code complete and reviewed
    - tests pass
    - documented
    - potentially shippable
---

# Scrum Lifecycle

**Ceremony level:** Moderate — time-boxed iterations with defined ceremonies
**Iteration unit:** Sprint (1-4 weeks, typically 2)
**Best for:** Cross-functional teams building products iteratively, organizations wanting regular delivery cadence

## Core Concepts

Scrum organizes work into **sprints** — fixed-length iterations where a cross-functional team commits to delivering a potentially shippable increment. The lifecycle is defined by its ceremonies (events), roles, and artifacts, all designed to create a regular inspect-and-adapt cycle.

### The Scrum Pillars
1. **Transparency** — work, progress, and impediments are visible to all
2. **Inspection** — frequent checkpoints to detect variance
3. **Adaptation** — adjust process, product, or plan based on what's learned

### Roles
- **Product Owner** — owns the backlog, prioritizes work, defines acceptance criteria
- **Scrum Master** — facilitates the process, removes impediments, coaches the team
- **Development Team** — self-organizing, cross-functional, delivers the increment

In our context (human + AI), CD typically fills Product Owner + Scrum Master roles; CC fills the Development Team role. The disciplines provide the cross-functional capabilities.

## Phase Structure

Scrum's phases repeat every sprint:

```
┌─────────────────── Sprint (2 weeks) ────────────────────┐
│                                                          │
│  Planning → Daily Work → Review → Retrospective          │
│     ↑                                        │          │
│     └────────────────────────────────────────┘          │
│                                                          │
│  ┌─────────┐  ┌──────────────────────┐  ┌───────────┐  │
│  │ Sprint  │  │   Development        │  │  Sprint   │  │
│  │ Planning│→ │   (daily standups)   │→ │  Review   │  │
│  │ (4 hrs) │  │                      │  │  (2 hrs)  │  │
│  └─────────┘  └──────────────────────┘  └───────────┘  │
│                                          ┌───────────┐  │
│                                          │  Sprint   │  │
│                                          │  Retro    │  │
│                                          │  (1.5 hrs)│  │
│                                          └───────────┘  │
└──────────────────────────────────────────────────────────┘
```

### Sprint Events

| Event | Duration (2-wk sprint) | Purpose |
|-------|----------------------|---------|
| **Sprint Planning** | 4 hours max | Select backlog items, define Sprint Goal, create Sprint Backlog |
| **Daily Standup** | 15 minutes | Sync on progress, surface impediments |
| **Sprint Review** | 2 hours max | Demo increment to stakeholders, gather feedback |
| **Sprint Retrospective** | 1.5 hours max | Inspect process, identify improvements |
| **Backlog Refinement** | ~10% of sprint capacity | Ongoing: clarify, estimate, decompose upcoming items |

## Milestones & Stage Gates

### Sprint-Level Gates

| Gate | Criteria |
|------|----------|
| **Sprint Planning exit** | Sprint Goal defined, Sprint Backlog committed, team understands the work |
| **Definition of Ready (DoR)** | Item has acceptance criteria, is estimated, has no unresolved dependencies |
| **Definition of Done (DoD)** | Code complete, tests pass, reviewed, documented, potentially shippable |
| **Sprint Review exit** | Increment demonstrated, stakeholder feedback captured |

### Product-Level Milestones

Scrum itself doesn't define product milestones — it delivers incrementally. Common overlays:
- **Release:** A set of sprints culminating in a production release
- **MVP:** Enough increments to validate core value proposition
- **Feature complete:** All planned features in the increment

## Discipline Intensity Map

In scrum, all disciplines are active within every sprint, but with a characteristic shape — front-loaded analysis/design, peak coding in the middle, testing rising toward the end:

```
                    Sprint Planning  Daily Work        Review+Retro
                    ──────────────  ──────────────────  ────────────
Product Research    ████             █                   ██
Business Analysis   ████████         ██                  ██
Design              ██████           ████████            █
Architecture        ████████         ████                █
Coding                               ████████████████
Testing             █                ██████   ████████   █
Deployment                                    ████       █
Data Modeling       ████             ██
Process Improvement                                      ████████
```

This pattern repeats every sprint. Unlike waterfall (one big hump per discipline) or kanban (flat line), scrum produces a **regular wave** — the same shape, sprint after sprint, with discipline mix varying based on the sprint's content.

### Refinement: The Hidden Discipline Window

Backlog refinement (happening continuously, ~10% of capacity) is where disciplines do quiet, preparatory work for future sprints:
- Product research informs upcoming feature priorities
- Business analysis clarifies acceptance criteria
- Architecture identifies technical dependencies
- Design produces wireframes or prototypes for upcoming items

## Rules of Goodness

**Doing scrum well:**
- Sprints have a clear, single-sentence Sprint Goal that the team can explain
- The team commits to what they can actually finish, not what they hope to finish
- The Definition of Done is non-negotiable — partial work doesn't count as done
- Retrospectives produce actionable changes, not just complaints
- The Product Owner is available and decisive throughout the sprint
- Velocity stabilizes after 3-4 sprints and is used for forecasting, not performance measurement

**Common failure modes:**
- **Sprint stuffing:** Committing to more than the team can deliver. Velocity becomes fiction. Carryover becomes the norm.
- **Zombie ceremonies:** Events happen on schedule but without engagement. Standups become status reports. Retros produce no changes.
- **No Sprint Goal:** Work is a disconnected bag of tickets. The team can't explain what the sprint is *about*.
- **Scrummerfall:** Sprints become mini-waterfalls — all design in week 1, all coding in week 2, panic testing at the end.
- **Velocity as a weapon:** Using velocity to compare teams, set targets, or measure individual performance. Velocity is a planning tool, not a productivity metric.
- **Skipping retros:** The adaptation pillar dies. The same problems repeat sprint after sprint.

## Configurations

### Short Sprints (1 week)
Higher ceremony overhead per unit of work, but faster feedback loops. Good for teams in early product discovery or handling high uncertainty.

### Standard Sprints (2 weeks)
The most common cadence. Balances ceremony overhead with enough time to deliver meaningful increments.

### Long Sprints (3-4 weeks)
Lower ceremony overhead per unit of work, but slower feedback. Appropriate for teams doing work with longer lead times (hardware integration, complex infrastructure).

### Scrum of Scrums
Multiple scrum teams coordinate via a regular cross-team sync. Each team runs its own sprint; the Scrum of Scrums aligns dependencies and shared deliverables. This is the entry point to SAFe-like coordination without the full SAFe framework.

## Mapping to Our SDLC

| Our artifact | Scrum equivalent |
|-------------|-----------------|
| Idea (D-number) | Product Backlog Item (PBI) |
| Spec | PBI acceptance criteria + any linked design docs |
| Planning | Sprint Planning → Sprint Backlog |
| Prompt | Task breakdown within a PBI |
| Implementation | Daily work within the sprint |
| Validation | Part of DoD; testing discipline within the sprint |
| Deploy | Potentially shippable increment; may align with release cadence |
| Result | Sprint Review demonstration + increment documentation |
| Chronicle | Release notes, sprint summaries archived |

### Key difference from our native process
Our native process flows per-deliverable (each D-number has its own Spec → Plan → Build → Result lifecycle). Scrum bundles multiple deliverables into a sprint — the sprint is the organizational unit, not the deliverable. A large D-number might span multiple sprints; a small sprint might complete several D-numbers.

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Team needs more flexibility, ceremonies feel heavy | → Kanban (keep WIP limits, drop time-boxes) |
| Multiple teams need coordination | → SAFe or Scrum of Scrums |
| Solo work, no stakeholder cadence needed | → Prototyping or Kanban |
| Regulatory hard gates required | → RUP (adds phase milestones) |
