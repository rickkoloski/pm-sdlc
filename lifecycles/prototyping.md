---
name: Prototyping
ceremony: none
iteration_unit: none
best_for: solo developers, vibe coding, spikes, exploratory work, early-stage ideas, breakout from structured lifecycles
modes:
  functional: { focus: "validate the idea and workflow", disciplines: { research: high, analysis: high, design: medium } }
  technical: { focus: "validate feasibility and architecture", disciplines: { architecture: high, coding: high, testing: medium } }
  ui: { focus: "validate the experience and interaction", disciplines: { design: high, research: medium, analysis: medium } }
moments:
  spark:
    focus: start building
    disciplines: { coding: high }
    completion_assertions: []
  friction:
    focus: something isn't working or isn't clear
    disciplines: { research: medium, analysis: medium, design: medium, architecture: medium, testing: medium }
    completion_assertions:
      - the specific friction point is identified
      - a discipline has been applied to address it
  traction:
    focus: the work is clearly going somewhere
    disciplines: { architecture: medium, design: medium, testing: medium }
    completion_assertions:
      - can articulate what is being built and why
      - core concept is demonstrable
  commitment:
    focus: the prototype is becoming the real thing
    disciplines: { architecture: high, testing: high, design: high, analysis: high, deployment: high }
    completion_assertions:
      - retroactive spec exists or is in progress
      - test coverage on core paths
      - deployment strategy identified
      - target lifecycle for ongoing work is chosen
  reentry:
    focus: bringing breakout work back into a structured lifecycle
    disciplines: { analysis: high, architecture: medium }
    completion_assertions:
      - findings documented and communicated to the parent lifecycle
      - artifacts mapped to the target lifecycle's expectations
      - decision made on what to keep vs. discard
scaffolding:
  structure_type: moments
  descriptions: portablemind-skills/prototyping-descriptions.md
  moments:
    - { key: spark, label: "Spark" }
    - { key: friction, label: "Friction" }
    - { key: traction, label: "Traction" }
    - { key: commitment, label: "Commitment" }
    - { key: reentry, label: "Reentry" }
---

# Prototyping Lifecycle

**Ceremony level:** None (by design)
**Iteration unit:** None — work flows as needed
**Best for:** Solo developers, vibe coding, spikes, exploratory work, early-stage ideas, breakout from structured lifecycles

## Core Concepts

Prototyping is not the absence of a lifecycle — it's a lifecycle where structure is **emergent rather than prescribed**. Disciplines are available on demand but never required. The work itself signals when more structure would help, and the practitioner decides when to add it.

This makes prototyping the **on-ramp to every other lifecycle**. Work starts here and graduates into more structured approaches when the work warrants it. It also serves as the **breakout lane** — when a team working in scrum or SAFe needs to temporarily step outside the lifecycle's guardrails to explore, spike, or answer a question by building something.

### Two Entry Patterns

**1. Greenfield / Vibe Coding**
Start building. No spec, no plan, no ceremony. The disciplines are a toolkit you reach for when you feel friction:
- "I keep changing my mind about what this should do" → reach for **product research** or **business analysis**
- "I'm building the same thing three ways" → reach for **architecture**
- "I don't know if this actually works" → reach for **testing**
- "I can't explain what I built" → time to write a spec *retroactively*

**2. Breakout from a Structured Lifecycle**
A team working in scrum or RUP hits a question that can't be answered within the current iteration's constraints. They need to prototype, explore, or spike. The work temporarily exits the lifecycle:
- Declare the breakout: "We're spiking X outside the sprint"
- Do the work in prototyping mode — build, test, learn
- Bring findings back: the spike becomes input to the next planning cycle
- Optionally, retroactively structure what was built (spec, result doc) if the prototype is being kept

## Phase Structure

Prototyping has no phases. It has **recognizable moments** that naturally occur as work progresses:

| Moment | What happens | Discipline trigger |
|--------|-------------|-------------------|
| **Spark** | An idea takes shape — you start building | (none needed) |
| **Friction** | Something isn't working or isn't clear | Whichever discipline addresses the friction |
| **Traction** | The work is clearly going somewhere | Consider retroactive spec, lightweight architecture |
| **Commitment** | The prototype is becoming the real thing | Adopt structure: write a spec, set up tests, plan deployment |
| **Reentry** | Bringing breakout work back into a structured lifecycle | Map what was built to the lifecycle's next planning cycle |

These moments are not gates. You don't need permission to pass through them. They're patterns you'll recognize in hindsight, and awareness of them helps you add structure at the right time rather than too early (overhead) or too late (technical debt).

## Milestones & Stage Gates

None are required. The following are **optional checkpoints** that signal the work may be outgrowing prototyping:

- **"This is getting complicated"** — More than ~3 interconnected components, or you can't hold the design in your head. Consider a lightweight architecture doc.
- **"Someone else needs to understand this"** — Another person (or future you) will work on this. Write a spec or at least a README.
- **"This needs to not break"** — The work has users or dependencies. Add tests.
- **"This needs to ship"** — Deployment, security, and ops concerns apply. Time for a deployment plan.
- **"We're keeping this"** — The prototype graduated. Retroactively create the artifacts that the target lifecycle expects.

## Discipline Intensity Map

In prototyping, discipline intensity is **demand-driven**, not scheduled. The hump chart is flat until friction creates a spike:

```
                    Spark     Friction    Traction    Commitment
                    ─────     ────────    ────────    ──────────
Product Research                ████                  ██
Business Analysis               ████                  ████
Design                          ████        ██        ████
Architecture                    ██          ████      ████████
Coding              ████████████████████████████████████████████
Testing                         ██          ████      ████████
Deployment                                  █         ████████
Data Modeling                   ██          ██        ████
Process Improvement                                   ██
```

Coding is continuous because that's what prototyping *is*. Everything else activates in response to need.

## Rules of Goodness

**Doing prototyping well:**
- You add structure when friction demands it, not because a process says to
- You can articulate *why* you're not using more structure ("this is a throwaway prototype" vs. "I don't want to bother")
- When the work gains traction, you recognize it and consciously choose: stay in prototyping or adopt structure
- Breakout spikes have a clear question they're answering and a defined reentry point
- Retroactive structuring is a normal activity, not a punishment for "doing it wrong"

**Common failure modes:**
- **Permanent prototyping:** The prototype ships to production without tests, docs, or deployment strategy. Every change is risky. This isn't agile — it's avoidance.
- **Premature structure:** Adding specs, plans, and ceremonies to a 2-hour prototype. The overhead kills the exploratory value of prototyping.
- **Breakout without reentry:** The spike answers the question but findings never flow back into the structured lifecycle. The team loses the learning.
- **Guilt-driven process:** Adding structure because "we should" rather than because friction demands it. Process without purpose is just overhead.

## Prototyping Modes

The word "prototyping" carries different meanings depending on what question you're trying to answer. Each mode emphasizes different disciplines and produces different outputs:

### Functional Prototyping
**Question:** "What should this do? What problem does it solve? Does the workflow make sense?"

Build enough to validate the *idea* — the user-facing behavior, the business logic, the flow. Fidelity is secondary to function. The output might be a working script, a CLI tool, a notebook, or a rough app that proves the concept.

**Dominant disciplines:** Product Research, Business Analysis, Design (interaction, not visual)

**What "done" looks like:** You can demonstrate the concept to a stakeholder and have a meaningful conversation about whether it solves the right problem. The code may be throwaway.

### Technical Prototyping
**Question:** "Can we build this? What are the hard parts? What are the risks?"

Build enough to validate the *architecture* — the integration points, the performance characteristics, the feasibility. This is the classic "spike" — answering a technical question by writing code rather than debating in a meeting.

**Dominant disciplines:** Architecture, Coding, Testing (focused on the technical question)

**What "done" looks like:** You can answer the technical question with evidence. "Yes, the API supports real-time updates at our scale" or "No, the library can't handle our data volume — here's what we measured." The code is often throwaway, but the findings feed into architecture decisions.

### UI Prototyping
**Question:** "What should this look and feel like? How will users interact with it?"

Build enough to validate the *experience* — the layout, the interactions, the visual language. This ranges from static wireframes to interactive mockups to high-fidelity prototypes with realistic data.

**Dominant disciplines:** Design (visual + interaction), Product Research (user feedback), Business Analysis (workflow validation)

**What "done" looks like:** You can put the prototype in front of users (or stakeholders acting as users) and observe whether they can accomplish the intended tasks. The implementation may be completely disposable — the value is in the design decisions it validates.

### Mixed Mode
In practice, prototyping often blends modes. A functional prototype reveals a technical question, which spawns a technical spike, which surfaces a UI design challenge. The modes aren't phases — they're lenses that clarify what question you're currently answering.

## Postures

Prototyping postures describe the social context — who's involved and how they coordinate:

### Solo Exploration
One person, one idea. No coordination overhead. Structure emerges from personal friction ("I keep losing track of what I've tried").

### Paired Spike
Two people (or human + AI) exploring together. Minimal coordination: shared understanding of the question being answered, verbal agreement on when to stop.

### Team Breakout
A subset of a structured team temporarily works in prototyping mode. Requires: declared scope, timebox (even if informal), reentry plan. The rest of the team continues in the primary lifecycle.

## Mapping to Our SDLC

Our native SDLC flow maps onto prototyping **retroactively**:

| Our artifact | When it appears in prototyping |
|-------------|--------------------------|
| Idea (D-number) | When you decide the work is worth tracking |
| Spec | When you need to explain what you built (or plan to keep building) |
| Planning | Rarely — ad hoc work is its own plan |
| Prompt | When handing off to CC for a specific task within the larger exploration |
| Result | When the work reaches a stopping point worth documenting |
| Chronicle | When the ad hoc work completes or gets absorbed into a structured lifecycle |

The key insight: in structured lifecycles, artifacts are created *before* the work. In prototyping, they're created *after* — or not at all. Both are valid. The discipline toolbox ensures that when you do create structure, you know how.

## Reentry: Reconciling Prototyping Work

When prototyping work needs to rejoin a structured lifecycle, the reconciliation process bridges the gap between unstructured exploration and organized process.

### Triggers

Any of these signal it's time to reconcile:
- "Let's catalog our ad hoc work"
- "What have we done since D[last]?"
- "Let's rejoin the process"
- The Commitment or Reentry moment has arrived

### Reconciliation Process

**1. Discovery** — Identify the boundary since last formal checkpoint:
- Find the last deliverable-tagged commit or chronicled deliverable
- List commits since then
- Categorize each: bug fix, UI tweak, missed requirement, edge case, iteration, trivial

**2. Choose a reconciliation option per item:**

| Option | When to use |
|--------|-------------|
| **Absorb** into existing deliverable | Work is related to a recent D-number. Update its COMPLETE doc and optionally its spec. |
| **Lightweight record** | Standalone work that doesn't warrant a full spec. Create `stepwise_results/adhoc_YYYYMMDD_brief_description.md` |
| **Batch into polish deliverable** | Group related tweaks into a single `dNN_polish_spec.md` |
| **Skip tracking** | Truly trivial work (typos, formatting). Acknowledge it happened, move on. |

**3. Spec maintenance** — If prototyping revealed spec gaps, update the specs or create tasks to do so.

**4. Clean git state** — Commit pending changes. Optional reconciliation commit: `docs: reconcile prototyping work [date range]`

**5. Path forward:**
- Resume formal process (start next D-number with a spec)
- Continue prototyping (reconcile again later)
- Chronicle and pause

### Anti-Patterns
- Creating full spec/planning/prompt for a 2-line fix
- Feeling guilty about prototyping work
- Letting prototyping work accumulate for weeks without reconciliation
- Abandoning process entirely because "it's too heavy"

### For CC: Handling Prototyping Requests

When CD asks for quick changes without a deliverable ID:
1. **Do the work** — Don't block on process
2. **Note it** — Remember this is prototyping
3. **Offer reconciliation** — After several items, suggest: "We've done several small changes. Want to catalog them?"

When CD explicitly says "just do X, skip the process": comply, don't lecture, offer to reconcile later.

## Transition Triggers

Signs that prototyping work should graduate to a structured lifecycle:

| Signal | Suggested transition |
|--------|---------------------|
| Multiple people need to coordinate | → Kanban (if flow-based) or Scrum (if time-boxed) |
| Stakeholders need predictability | → Scrum or RUP |
| Regulatory or compliance requirements | → RUP or Waterfall |
| The prototype is becoming the product | → Whatever the team's standard lifecycle is |
| You keep discovering the same friction | → Add the specific ceremony that addresses it, even if you stay mostly in prototyping |
