# Lifecycles

## The Second Dimension

Disciplines describe **what capabilities** are available. Lifecycles describe **how those capabilities are organized over time** — the cadence, stage gates, milestones, and iteration patterns that shape when and how intensely each discipline is applied.

This is the insight from the Rational Unified Process "hump chart": disciplines don't live inside phases — they *cross* phases, with varying intensity. The lifecycle determines the shape of those humps.

```
                        Lifecycle (time →)
                        ┌─────────────────────────────────────────┐
                        │  Phase 1    Phase 2    Phase 3   Phase N│
           ┌────────────┼─────────────────────────────────────────┤
Discipline │ Research   │  ████████   ███        █               │
           │ Analysis   │  ████████   ████████   ██        █     │
           │ Design     │  ███        ████████   ██████    ██    │
           │ Architect  │  ██████     ████████   ████      ██    │
           │ Coding     │             ███        █████████ ██    │
           │ Testing    │  █          ██         ████      ██████│
           │ Deploy     │                        █         █████ │
           └────────────┼─────────────────────────────────────────┤
                        └─────────────────────────────────────────┘
```

Different lifecycles reshape these humps. Waterfall compresses each discipline into its own phase. Scrum spreads all disciplines across every sprint. Kanban makes them continuous. Prototyping lets them emerge on demand.

## The Lifecycle Spectrum

The lifecycles form a spectrum from least to most ceremony:

```
Prototyping ── Kanban ──── Scrum ──── RUP ──── SAFe ──── Waterfall
     │            │          │         │         │           │
   zero        flow +     time-boxed  phased   scaled     sequential
 ceremony     WIP limits  iterations  iters    phased     phases +
                                      + gates  iters      hard gates
```

No lifecycle is inherently better. The right choice depends on team size, regulatory environment, organizational culture, and the nature of the work. A solo developer exploring an idea is well-served by prototyping. A medical device team shipping to production needs the gates of waterfall or RUP. A mature product team iterating on known problems fits scrum or kanban.

## Relationship to Our SDLC

Our native process (`process/overview.md`) is a lightweight iterative lifecycle:

```
Idea → Spec → Planning → Prompt → Implementation → Validation → Deploy → Result → Chronicle
```

This is not a competing lifecycle — it's our **default implementation** of the lifecycle concept. Each lifecycle file in this directory describes how to adapt the discipline toolbox to that methodology's structure, and how our native artifacts (specs, plans, results, chronicles) map onto that lifecycle's phases and ceremonies.

Teams already using a methodology can adopt our disciplines without changing their lifecycle. Teams without a methodology start with our native process or prototyping, and can adopt more structure as needed.

## Structure

Each lifecycle file covers:

1. **Core concepts** — the iteration unit, cadence, key events
2. **Phase/stage structure** — what the lifecycle calls its temporal divisions
3. **Milestones & stage gates** — what must be true to proceed
4. **Discipline intensity map** — how our disciplines map to this lifecycle's phases
5. **Rules of goodness** — what "doing this well" looks like vs. common failure modes
6. **Configurations** — variants and scaling options (where applicable)
7. **Mapping to our SDLC** — how our native artifacts and flow map onto this lifecycle

## Files

```
lifecycles/
├── README.md        ← This file (mental model + structure)
├── prototyping.md   ← Zero-ceremony; disciplines on demand; the on-ramp
├── kanban.md        ← Continuous flow, WIP limits, pull-based cadences
├── scrum.md         ← Time-boxed sprints, ceremonies, Definition of Done
├── rup.md           ← Phased iterations, milestones, discipline humps (the original)
├── safe.md          ← Scaled iterations, program increments, ARTs
└── waterfall.md     ← Sequential phases, hard stage gates, document-driven
```

## How to Use

**Choosing a lifecycle:** Start with the team's current reality. If they already follow scrum, use `scrum.md` to map disciplines onto their existing ceremonies. If they have no methodology, start with `prototyping.md` and let structure emerge.

**Transitioning between lifecycles:** Work naturally moves along the spectrum. A prototype starts in prototyping mode, gains enough traction to need kanban-style flow control, and eventually gets folded into a team's sprint cadence. The lifecycle files describe these transition points.

**Lifecycle as context, not constraint:** The lifecycle tells you what ceremonies and gates are expected — it doesn't prevent you from using any discipline at any time. A scrum team can still break out into prototyping mode for a spike. A waterfall project can still iterate within a phase. The discipline toolbox is always available.
