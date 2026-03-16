---
name: RUP
ceremony: moderate-high
iteration_unit: iteration (2-6 weeks, within phases)
best_for: projects with significant technical risk, architecture-centric work, teams needing structured risk reduction
phases:
  inception:
    focus: scope, risk, business case
    typical_duration: 1 iteration (2-4 weeks)
    gate: LCO (Lifecycle Objectives)
    disciplines: { research: high, analysis: high, architecture: high, design: medium, coding: low, testing: low, data_modeling: medium }
    completion_assertions:
      - scope and vision agreed with stakeholders
      - key risks identified with mitigation strategies
      - initial cost/schedule estimate exists
      - architecturally significant use cases identified
      - stakeholders approve proceeding to elaboration
  elaboration:
    focus: architecture validation, risk reduction
    typical_duration: 1-3 iterations (4-12 weeks)
    gate: LCA (Lifecycle Architecture)
    disciplines: { architecture: high, design: high, coding: high, testing: medium, analysis: high, data_modeling: high }
    completion_assertions:
      - architecture validated through working code (executable architecture)
      - high-priority risks resolved or mitigated
      - construction plan credible based on elaboration actuals
      - integration points proven end-to-end
      - 80% of use cases identified, ~10% fully specified
  construction:
    focus: feature implementation against validated architecture
    typical_duration: 2-4+ iterations (bulk of project)
    gate: IOC (Initial Operational Capability)
    disciplines: { coding: high, testing: high, design: medium, deployment: medium, architecture: low }
    completion_assertions:
      - software is feature-complete for initial release
      - quality sufficient for beta/pilot deployment
      - user documentation and training materials ready
      - deployment plan finalized
  transition:
    focus: delivery, acceptance, cutover
    typical_duration: 1-2 iterations
    gate: GA (General Availability)
    disciplines: { deployment: high, testing: high, analysis: medium }
    completion_assertions:
      - system in production and stable
      - users trained and productive
      - support processes in place
      - lessons learned captured
scaffolding:
  structure_type: phases
  descriptions: portablemind-skills/rup-descriptions.md
  phase_durations:
    inception: 0.10
    elaboration: 0.30
    construction: 0.50
    transition: 0.10
  disciplines:
    - { key: business_modeling, label: "Business Modeling" }
    - { key: requirements, label: "Requirements" }
    - { key: analysis_design, label: "Analysis & Design" }
    - { key: implementation, label: "Implementation" }
    - { key: test, label: "Test" }
    - { key: deployment, label: "Deployment" }
    - { key: project_management, label: "Project Management" }
    - { key: environment, label: "Environment" }
  intensity_spans:
    high: [0.0, 1.0]
    medium: [0.25, 1.0]
    low: [0.67, 1.0]
  phase_disciplines:
    inception:
      business_modeling: high
      requirements: high
      analysis_design: medium
      implementation: low
      test: low
      deployment: low
      project_management: high
      environment: high
    elaboration:
      business_modeling: medium
      requirements: medium
      analysis_design: high
      implementation: high
      test: medium
      deployment: low
      project_management: high
      environment: medium
    construction:
      business_modeling: medium
      requirements: medium
      analysis_design: medium
      implementation: high
      test: high
      deployment: medium
      project_management: high
      environment: low
    transition:
      business_modeling: low
      requirements: low
      analysis_design: low
      implementation: medium
      test: high
      deployment: high
      project_management: high
      environment: low
  milestones:
    inception: "LCO Milestone Review"
    elaboration: "LCA Milestone Review"
    construction: "IOC Milestone Review"
    transition: "GA Milestone Review"
---

# RUP Lifecycle (Rational Unified Process)

**Ceremony level:** Moderate to high — phased iterations with milestone reviews
**Iteration unit:** Iteration (2-6 weeks, within phases)
**Best for:** Projects with significant technical risk, architecture-centric work, teams needing structured risk reduction

## Core Concepts

RUP organizes work into **four sequential phases**, each containing one or more **iterations**. The key insight — and the origin of the "hump chart" — is that disciplines don't map 1:1 to phases. Every iteration can include work from any discipline; the phases determine the *emphasis*.

RUP was designed to address the failures of waterfall: big-bang integration, late risk discovery, and requirements that change after they're "frozen." It does this through iterative development within a phased structure, with architecture as the central organizing principle.

### The Four Phases

```
Inception ──→ Elaboration ──→ Construction ──→ Transition
   │              │                │               │
   LCO            LCA              IOC             GA
   (Lifecycle     (Lifecycle       (Initial Op.    (General
    Objectives)    Architecture)    Capability)     Availability)
```

Each phase ends with a **milestone** that serves as a go/no-go decision point.

## Phase Structure

### Inception (1 iteration, typically 2-4 weeks)
**Purpose:** Establish the business case. Define scope. Identify key risks. Get stakeholder agreement to proceed.

**Dominant disciplines:** Product Research, Business Analysis, Architecture (initial)

**Key activities:**
- Define the vision and scope
- Identify key use cases (10-20% of the model, the architecturally significant ones)
- Estimate cost, schedule, and risk
- Build a proof-of-concept if needed to mitigate a critical risk
- Get stakeholder buy-in

**Milestone — Lifecycle Objectives (LCO):**
- [ ] Scope and vision are agreed upon
- [ ] Key risks are identified with mitigation strategies
- [ ] Initial cost/schedule estimate exists
- [ ] Stakeholders agree to proceed

### Elaboration (1-3 iterations, typically 4-12 weeks)
**Purpose:** Establish the architecture. Mitigate the highest risks. Build the architectural skeleton that will carry the system through construction.

**Dominant disciplines:** Architecture, Design, Coding (architectural spikes), Testing (architectural validation)

**Key activities:**
- Design and validate the architecture (the "executable architecture")
- Implement architecturally significant use cases end-to-end
- Refine the use case model (80% of use cases identified, ~10% fully specified)
- Resolve or mitigate high-risk items
- Establish the development environment, build system, and infrastructure

**Milestone — Lifecycle Architecture (LCA):**
- [ ] Architecture is stable and validated through code
- [ ] High risks are resolved or mitigated
- [ ] Plan for Construction is credible (based on Elaboration actuals)
- [ ] Stakeholders agree the architecture can support the full scope

**This is the most critical phase.** The number one predictor of RUP project failure is insufficient Elaboration — rushing to Construction with an unvalidated architecture.

### Construction (2-4+ iterations, the bulk of the project)
**Purpose:** Build the system. Implement remaining use cases against the established architecture. Emphasis shifts from risk reduction to feature completion.

**Dominant disciplines:** Coding, Testing, Design (detail), Deployment (preparation)

**Key activities:**
- Implement remaining functionality, iteration by iteration
- Each iteration produces a demonstrable, tested increment
- Continuous integration and testing
- Prepare user documentation and training materials
- Plan for deployment

**Milestone — Initial Operational Capability (IOC):**
- [ ] Software is feature-complete for the initial release
- [ ] Quality is sufficient for beta/pilot deployment
- [ ] User documentation and training materials are ready
- [ ] Deployment plan is finalized

### Transition (1-2 iterations)
**Purpose:** Deliver the system to users. Resolve defects. Tune performance. Train users. Transition from development to operations.

**Dominant disciplines:** Deployment, Testing (acceptance), Business Analysis (user feedback)

**Key activities:**
- Beta testing and user acceptance testing
- Defect resolution and performance tuning
- User training and documentation finalization
- Production deployment
- Post-deployment support and stabilization

**Milestone — General Availability (GA):**
- [ ] System is in production and stable
- [ ] Users are trained and productive
- [ ] Support processes are in place
- [ ] Lessons learned are captured

## Discipline Intensity Map (The Original Hump Chart)

```
                    Inception  Elaboration   Construction   Transition
                    ─────────  ───────────   ────────────   ──────────
Product Research    ████████   ███           █
Business Analysis   ████████   ████████      ██             █
Design              ███        ████████████  ████████       ██
Architecture        ████████   ████████████  ████           ██
Coding              █          ████████      ████████████   ████
Testing             █          ████          ████████████   ████████
Deployment                     █             ████           ████████████
Data Modeling       ██████     ████████      ██
Process Improvement ██         ██            ██             ██████
```

This is the signature pattern: overlapping humps, with each discipline peaking in a different phase but present across many phases. Architecture peaks in Elaboration. Coding peaks in Construction. Testing rises continuously and peaks in Transition.

## Rules of Goodness

**Doing RUP well:**
- Elaboration produces a working, tested architectural skeleton — not just documents
- Each iteration delivers an executable, demonstrable increment
- Risk is actively managed: the highest-risk items are addressed first, not last
- Architecture decisions are made in Elaboration, not deferred to Construction
- The milestone reviews are genuine go/no-go decisions, not rubber stamps
- The process is tailored to the project — RUP is a framework, not a prescription

**Common failure modes:**
- **Waterfall in disguise:** Using RUP phases as waterfall stages (all requirements in Inception, all design in Elaboration, all coding in Construction). This misses the entire point of iterative development within phases.
- **Ceremony overload:** The original Rational toolkit included hundreds of artifacts. Most projects need a small subset. Over-documenting is a common anti-pattern.
- **Skimping on Elaboration:** Rushing to Construction to "start building." The architecture is unstable, leading to expensive rework later. *Elaboration is not a phase to get through quickly — it's where you buy certainty.*
- **Architecture astronautics:** Over-engineering the architecture in Elaboration. The architecture should be validated through working code, not through increasingly detailed diagrams.
- **Milestone theater:** Reviews happen but don't actually gate progression. The team proceeds regardless of whether criteria are met.

## Configurations

### Minimal RUP (Small Team)
- Inception: 1 week (vision doc + key risks)
- Elaboration: 2-3 weeks (architectural spike, validated end-to-end)
- Construction: iterative, 2-week iterations
- Transition: 1 week (deploy + verify)
- Total artifacts: vision doc, architecture doc, iteration plans, test results

### Standard RUP
- Full four-phase structure with milestone reviews
- Use case driven requirements
- Architecture-centric design
- Risk-managed iteration planning

### RUP for Regulated Environments
- Adds traceability: requirements → design → code → tests
- Milestone reviews include compliance checkpoints
- Documentation artifacts expanded to satisfy audit requirements
- Phase gates require sign-off from designated authorities

## Mapping to Our SDLC

| Our artifact | RUP equivalent |
|-------------|----------------|
| Idea (D-number) | Use case or change request |
| Spec | Use case specification + supplementary requirements |
| Planning | Iteration plan |
| Prompt | Task assignment within an iteration |
| Implementation | Iteration development work |
| Validation | Iteration testing + phase milestone review |
| Deploy | Transition phase activities |
| Result | Iteration assessment + milestone review record |
| Chronicle | Project retrospective, lessons learned |

### Phase-to-flow mapping
Our native flow (Idea → Spec → Plan → Build → Validate → Deploy → Result) runs **within each iteration**, while the RUP phases determine the *character* of each iteration:
- **Inception iterations** are heavy on Idea + Spec, light on Build
- **Elaboration iterations** are heavy on Spec + Plan + Build (architectural), with Validation of the architecture
- **Construction iterations** run the full flow for feature work
- **Transition iterations** are heavy on Validation + Deploy

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Team is small, iterations feel heavy | → Scrum (simpler ceremonies) or Kanban |
| Need to scale across multiple teams | → SAFe (which borrowed heavily from RUP) |
| Architecture is stable, work is predictable | → Scrum (less architectural overhead) |
| Compliance/regulatory needs increase | → Waterfall (if iteration is not acceptable to regulators) |
