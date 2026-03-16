---
name: SAFe
ceremony: high
iteration_unit: iteration (2 weeks) within program increments (8-12 weeks)
best_for: large organizations (50-125+ developers) needing coordinated delivery across multiple teams
levels:
  team: { unit: iteration, cadence: 2 weeks }
  program: { unit: program_increment, cadence: 8-12 weeks }
  large_solution: { unit: solution_increment, cadence: aligned to PIs }
  portfolio: { unit: strategic_themes, cadence: quarterly }
phases:
  pi_planning:
    focus: align all teams on PI objectives, identify dependencies
    typical_duration: 2 days
    disciplines: { research: high, analysis: high, architecture: high, design: high }
    completion_assertions:
      - all teams have committed PI Objectives
      - cross-team dependencies identified and owned
      - confidence vote averages 3/5 or higher
      - architectural runway assessed and enabler work planned
  iterations:
    focus: team-level development within PI context
    typical_duration: 4 iterations × 2 weeks
    disciplines: { coding: high, testing: high, design: high, architecture: medium, deployment: medium }
    completion_assertions:
      - working tested increment at each iteration boundary
      - system demo conducted per iteration
      - PI objectives on track or risks escalated
  ip_iteration:
    focus: innovation, technical debt, education, next PI prep
    typical_duration: 2 weeks
    disciplines: { process_improvement: high, research: high, architecture: high }
    completion_assertions:
      - innovation/hackathon work demonstrated
      - technical debt items addressed
      - next PI planning preparation complete
  inspect_and_adapt:
    focus: quantitative review and problem-solving
    typical_duration: half day
    disciplines: { process_improvement: high, testing: medium }
    completion_assertions:
      - PI objectives assessed quantitatively
      - root cause analysis on missed objectives
      - improvement backlog items created with owners
events:
  pi_planning: { duration: "2 days", cadence: "start of PI" }
  iteration_planning: { duration: "4 hours per team", cadence: "per iteration" }
  system_demo: { duration: "1-2 hours", cadence: "per iteration" }
  pi_system_demo: { duration: "2-3 hours", cadence: "end of PI" }
  inspect_and_adapt: { duration: "half day", cadence: "end of PI" }
configurations:
  essential: "1 ART, team + program levels only (recommended starting point)"
  large_solution: "multiple ARTs, adds solution train coordination"
  portfolio: "adds strategic alignment, lean portfolio management, epic kanban"
  full: "all four levels active (hundreds of practitioners)"
---

# SAFe Lifecycle (Scaled Agile Framework)

**Ceremony level:** High — multi-level cadences with extensive coordination events
**Iteration unit:** Iteration (2 weeks) within Program Increments (8-12 weeks)
**Best for:** Large organizations (50-125+ developers) needing coordinated delivery across multiple teams

## Core Concepts

SAFe scales agile practices beyond a single team by adding layers of coordination. The fundamental unit is the **Program Increment (PI)** — a timebox (typically 10 weeks: 4 development iterations + 1 Innovation & Planning iteration) during which an **Agile Release Train (ART)** of 5-12 teams delivers an integrated increment.

SAFe is explicitly built on RUP's foundation (iterative, architecture-centric, risk-driven) combined with Lean and agile principles. Where RUP phases manage risk for a single project, SAFe manages risk and alignment across a portfolio of projects.

### Key Abstractions

| Level | Unit | Cadence | Key role |
|-------|------|---------|----------|
| **Team** | Iteration (sprint) | 2 weeks | Scrum Master, Product Owner |
| **Program (ART)** | Program Increment | 8-12 weeks | Release Train Engineer (RTE), System Architect |
| **Large Solution** | Solution Increment | Aligned to PIs | Solution Train Engineer |
| **Portfolio** | Strategic themes | Quarterly | Lean Portfolio Management |

## Phase Structure

### Program Increment (PI) — The Core Cycle

```
┌──────────────── Program Increment (10 weeks) ─────────────────┐
│                                                                │
│  PI Planning → Iter 1 → Iter 2 → Iter 3 → Iter 4 → IP Iter   │
│  (2 days)     (2 wks)  (2 wks)  (2 wks)  (2 wks)  (2 wks)   │
│                                                                │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ Team-level sprints run inside each iteration           │    │
│  │ System Demo at end of each iteration                   │    │
│  │ PI System Demo + Inspect & Adapt at PI end             │    │
│  └────────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

### PI Events

| Event | Duration | Purpose |
|-------|----------|---------|
| **PI Planning** | 2 days (all teams, co-located or virtual) | Align all teams on PI objectives, identify dependencies, commit to deliverables |
| **Iteration Planning** | 4 hours per team per iteration | Standard sprint planning within the PI context |
| **System Demo** | Per iteration | Integrated demo of all teams' work |
| **PI System Demo** | End of PI | Full integrated demo for stakeholders |
| **Inspect & Adapt** | Half day, end of PI | Quantitative measurement + problem-solving workshop |
| **Innovation & Planning (IP) Iteration** | 2 weeks | Hackathons, education, technical debt, next PI planning prep |

### Within Each Iteration
Teams run standard scrum or kanban (SAFe supports both). The iteration is the team-level cycle; the PI is the program-level cycle.

## Milestones & Stage Gates

### PI-Level Milestones

| Milestone | Criteria |
|-----------|----------|
| **PI Planning exit** | All teams have committed PI Objectives; dependencies are identified and owned; confidence vote ≥ 3/5 |
| **Iteration boundary** | Working, tested, integrated increment; System Demo conducted |
| **PI boundary** | PI Objectives met; quantitative metrics reviewed; improvement items identified |

### Portfolio-Level Gates

| Gate | Purpose |
|------|---------|
| **Lean Business Case** | Strategic alignment and economic justification for epics |
| **Epic approval** | Portfolio Kanban: epics move from Funnel → Analyzing → Implementing |
| **Solution intent** | For large solutions: fixed vs. variable requirements are distinguished |

## Discipline Intensity Map

SAFe has a **nested pattern**: the PI-level hump wraps team-level sprint humps.

**PI-level pattern:**
```
                    PI Planning  Iterations 1-4    IP Iteration
                    ───────────  ──────────────    ────────────
Product Research    ████████     ██                ████████
Business Analysis   ████████████ ████              ████
Design              ████████     ████████████      ████
Architecture        ████████████ ████████          ████████
Coding                           ████████████████
Testing                          ██████████████████████
Deployment                       ██     ████████  ██
Data Modeling       ████████     ██
Process Improvement                               ████████████
```

**Key observations:**
- PI Planning front-loads analysis, architecture, and research disciplines
- The IP Iteration creates space for process improvement and research that would otherwise be squeezed out
- Architecture has two peaks: PI Planning (alignment) and IP Iteration (technical debt, enablers)
- Testing rises continuously through the PI as integration work accumulates

## Rules of Goodness

**Doing SAFe well:**
- PI Planning produces genuine alignment, not theater. Teams leave knowing what they'll build and why.
- The confidence vote is honest. A low vote triggers replanning, not pressure to vote higher.
- The IP Iteration is protected. It doesn't become overflow capacity for unfinished features.
- Architectural Runway is maintained: enabler work is funded and prioritized, not deferred.
- Cross-team dependencies are managed actively, not discovered late.
- The ART demos a truly integrated increment each iteration, not team-by-team slide decks.

**Common failure modes:**
- **SAFe as waterfall at scale:** PI Planning becomes a big upfront design session. Teams are told what to build, not empowered to plan.
- **PI Planning as theater:** Two days of ceremony that produces plans nobody follows. The real work is negotiated in hallways afterward.
- **IP Iteration raided:** Innovation time is consistently consumed by feature work. Technical debt grows. Teams burn out.
- **Dependency management as bureaucracy:** Dependency boards and risk rosters become artifacts to fill out, not tools for managing real coordination challenges.
- **Over-configuration:** SAFe offers many optional components. Teams adopt all of them at once, drowning in process before getting value from the basics.
- **Ignoring the team level:** SAFe adds program-level coordination but doesn't replace the need for healthy team-level agile practices. If the teams can't do scrum/kanban well, SAFe won't save them.

## Configurations

### Essential SAFe
The minimal configuration. One ART (5-12 teams). Team and Program levels only. No portfolio or large solution layers. This is the recommended starting point.

**Key elements:** PI Planning, System Demo, Inspect & Adapt, ART backlog, PI Objectives.

### Large Solution SAFe
Multiple ARTs building a single large solution. Adds the Solution level:
- Solution Train coordinates multiple ARTs
- Solution Demo integrates across ARTs
- Solution Intent captures fixed and variable requirements
- Pre- and Post-PI Planning synchronizes across ARTs

### Portfolio SAFe
Adds strategic alignment and funding governance:
- Lean Portfolio Management
- Strategic Themes and OKRs
- Epic Kanban (Funnel → Analyzing → Implementing → Done)
- Lean Business Cases for significant investments
- Participatory budgeting

### Full SAFe
All four levels active. Portfolio, Large Solution, Program, Team. Appropriate for very large enterprises (hundreds of practitioners) with multiple value streams.

## Mapping to Our SDLC

| Our artifact | SAFe equivalent |
|-------------|----------------|
| Idea (D-number) | Feature or Story (depending on size) |
| Spec | Feature acceptance criteria + Enabler specifications |
| Planning | PI Planning → PI Objectives → Iteration Planning |
| Prompt | Story-level task description |
| Implementation | Iteration development work |
| Validation | Iteration System Demo + PI System Demo |
| Deploy | Release on Demand (continuous delivery pipeline) |
| Result | PI Objectives assessment, Inspect & Adapt quantitative review |
| Chronicle | PI retrospective, solution metrics, portfolio reporting |

### Scaling our disciplines
In SAFe, discipline ownership is distributed:
- **Team-level disciplines:** Coding, Testing, Design (detail) — owned by each team
- **Program-level disciplines:** Architecture, Business Analysis — coordinated across teams by System Architect and Product Management
- **Portfolio-level disciplines:** Product Research, Process Improvement — driven by Lean Portfolio Management

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Only 1-2 teams, coordination overhead isn't justified | → Scrum or Kanban |
| Need stricter phase gates and compliance traceability | → RUP within each ART's PI |
| Teams are struggling with basic agile practices | → Fix team-level Scrum first, then scale |
| Organization is ready to decentralize further | → Portfolio Kanban without the full SAFe framework |
