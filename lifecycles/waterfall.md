---
name: Waterfall
ceremony: high
iteration_unit: none (each phase executes once)
best_for: regulated industries, fixed-scope contracts, hardware-dependent systems, stable requirements
phases:
  requirements:
    focus: define what the system must do
    gate: SRR (System Requirements Review)
    disciplines: { research: high, analysis: high, architecture: medium, data_modeling: high }
    completion_assertions:
      - requirements are complete, consistent, and traceable
      - stakeholders have reviewed and approved
      - feasibility confirmed
      - requirements baseline established
  design:
    focus: translate requirements into system and component design
    gates:
      - PDR (Preliminary Design Review)
      - DDR (Detailed Design Review)
    disciplines: { architecture: high, design: high, data_modeling: high, analysis: medium }
    completion_assertions:
      - architecture is sound and interfaces are defined (PDR)
      - component-level specs are implementable and traceable to requirements (DDR)
      - test plans drafted for each verification level
  implementation:
    focus: build the system according to detailed design
    gate: CDR (Code Review)
    disciplines: { coding: high, testing: medium }
    completion_assertions:
      - code implements the design
      - unit tests pass
      - coding standards met
      - code reviewed
  verification:
    focus: progressive testing — unit, integration, system, acceptance
    gate: TRR (Test Readiness Review)
    disciplines: { testing: high, analysis: medium }
    completion_assertions:
      - all test levels executed with documented results
      - requirements traceability matrix complete (every requirement tested)
      - defects resolved or dispositioned
      - acceptance criteria met
  deployment:
    focus: release to production, train users, transition to operations
    gates:
      - PRR (Production Readiness Review)
      - ORR (Operational Readiness Review)
    disciplines: { deployment: high, testing: medium, analysis: medium }
    completion_assertions:
      - all tests pass with documented results
      - user documentation complete
      - training conducted
      - operations team prepared to support
      - rollback plan exists and is tested
  maintenance:
    focus: ongoing support, defect resolution, change management
    gate: PIR (Post-Implementation Review)
    disciplines: { coding: medium, testing: medium, process_improvement: high }
    completion_assertions:
      - system stable in production
      - lessons learned captured
      - maintenance processes functioning
      - change control process established
---

# Waterfall Lifecycle

**Ceremony level:** High — sequential phases with formal stage gates and sign-offs
**Iteration unit:** None (each phase executes once)
**Best for:** Regulated industries, fixed-scope contracts, hardware-dependent systems, projects where requirements are well-understood and unlikely to change

## Core Concepts

Waterfall organizes work into **sequential phases**, each of which completes fully before the next begins. Each phase produces a defined set of artifacts that serve as input to the next phase. Phase transitions are controlled by **stage gates** — formal review and approval points.

Despite decades of criticism, waterfall remains the right choice when:
- Requirements are genuinely stable (embedded systems, regulatory compliance)
- External contracts or regulations require phase-gate documentation
- The cost of rework is extremely high (hardware, medical devices, safety-critical systems)
- The project is well-understood and has been done before

The core risk of waterfall is well-known: late integration and late validation. Problems discovered in testing that should have been caught in design are exponentially more expensive to fix. This is why RUP and agile methods exist — but understanding waterfall is essential both for historical context and for the domains where it's still appropriate.

## Phase Structure

```
Requirements → Design → Implementation → Verification → Deployment → Maintenance
     │            │           │               │              │            │
     ▼            ▼           ▼               ▼              ▼            ▼
  [SRR]        [PDR]       [CDR]           [TRR]          [PRR]       [PIR]
               [DDR]                                     [ORR]
```

### Phase Definitions

**1. Requirements**
Define what the system must do. Produce requirements documents (functional, non-functional, interface). All requirements are captured, reviewed, and baselined before proceeding.

**Gate — System Requirements Review (SRR):**
- [ ] Requirements are complete, consistent, and traceable
- [ ] Stakeholders have reviewed and approved
- [ ] Feasibility is confirmed
- [ ] Requirements baseline is established

**2. Design**
Translate requirements into a system design. Produce architecture documents, detailed design specifications, interface control documents. Two sub-phases are common:

*Preliminary Design:*
- System architecture and high-level component design
- **Gate — Preliminary Design Review (PDR):** Architecture is sound, interfaces are defined

*Detailed Design:*
- Component-level specifications, data models, algorithms
- **Gate — Detailed Design Review (DDR):** Design is implementable, complete, and traceable to requirements

**3. Implementation**
Build the system according to the detailed design. Produce source code, unit tests, build artifacts.

**Gate — Critical Design Review / Code Review (CDR):**
- [ ] Code implements the design
- [ ] Unit tests pass
- [ ] Coding standards are met
- [ ] Code is reviewed

**4. Verification (Testing)**
Verify the system meets its requirements through progressive testing levels:

| Level | Scope | Verifies against |
|-------|-------|-----------------|
| Unit testing | Individual components | Detailed design |
| Integration testing | Component interactions | Architecture / interface specs |
| System testing | Complete system | System requirements |
| Acceptance testing | User scenarios | User requirements / contract |

**Gate — Test Readiness Review (TRR):**
- [ ] Test plans and procedures are approved
- [ ] Test environment is ready
- [ ] All code is integrated and builds cleanly

**5. Deployment**
Release the system to production. Install, configure, train users, transition from development to operations.

**Gate — Production Readiness Review (PRR) / Operational Readiness Review (ORR):**
- [ ] All tests pass with documented results
- [ ] User documentation is complete
- [ ] Training is conducted
- [ ] Operations team is prepared to support
- [ ] Rollback plan exists

**6. Maintenance**
Ongoing support, defect resolution, and change management. Changes re-enter the lifecycle (often as mini-waterfalls).

**Gate — Post-Implementation Review (PIR):**
- [ ] System is stable in production
- [ ] Lessons learned are captured
- [ ] Maintenance processes are functioning

## Milestones & Stage Gates

Stage gates are the defining feature of waterfall. They serve three purposes:
1. **Quality assurance:** Work meets defined standards before proceeding
2. **Decision authority:** Stakeholders explicitly approve progression
3. **Traceability:** Each phase's outputs are documented and baselined

### Gate Review Process
1. **Entry criteria:** What must exist before the review can occur
2. **Review:** Designated reviewers evaluate the artifacts
3. **Exit criteria:** What must be true for the gate to pass
4. **Decision:** Go / No-Go / Conditional Go (with specific remediation)
5. **Record:** Gate decision is documented with rationale

## Discipline Intensity Map

Waterfall has the most extreme hump pattern — each discipline peaks in its corresponding phase:

```
                    Reqts    Design   Implement  Verify   Deploy   Maintain
                    ─────    ──────   ─────────  ──────   ──────   ───────
Product Research    ████████ █
Business Analysis   ████████████████  █
Design                       ████████████████    █
Architecture        ██       ████████████████    █                 █
Coding                       ██       ████████████████    █        ████
Testing                      █        ██         ████████████████  ██
Deployment                                       ██       ████████████
Data Modeling       ████████ ████████ █
Process Improvement                                                ████████
```

This is the opposite of kanban's flat pattern. Each discipline has one big hump, nearly zero presence in other phases. This is both waterfall's clarity (you know exactly what you're doing in each phase) and its weakness (late discovery of problems that cross phase boundaries).

## Rules of Goodness

**Doing waterfall well:**
- Requirements are genuinely complete and stable before design begins. If they're not, waterfall is the wrong lifecycle.
- Stage gates are real decision points with empowered reviewers, not rubber stamps.
- Traceability is maintained: every requirement traces forward to design, code, and test.
- Testing is planned during requirements and design, not an afterthought.
- Change control is rigorous but not paralytic — changes are evaluated for impact and either approved through the gate process or deferred.
- The team builds prototypes or proofs of concept *before* committing to waterfall — use the prototyping lifecycle to reduce uncertainty, then waterfall for disciplined execution.

**Common failure modes:**
- **Requirements aren't stable:** The most common failure. If requirements change frequently, waterfall's sequential structure means every change ripples through every subsequent phase.
- **Big-bang integration:** Components are built in isolation during Implementation, then integrated for the first time during Verification. Integration problems cascade.
- **Testing phase as defect discovery:** Testing finds fundamental design problems, but the budget and schedule for fixing them was spent in earlier phases.
- **Document-driven, not value-driven:** Producing the required artifacts becomes the goal. Massive requirements documents are produced that nobody reads carefully. Gates pass because documents exist, not because they're correct.
- **No feedback loops:** The sequential structure discourages revisiting earlier phases. "We already did requirements" becomes a barrier to incorporating learning from later phases.
- **Death march to the gate:** Teams rush to meet gate deadlines with incomplete work, knowing the gate will pass anyway. Accumulated quality debt explodes during verification.

## Configurations

### V-Model
An extension of waterfall that explicitly pairs each development phase with a corresponding verification phase:

```
Requirements  ←──────────────→  Acceptance Testing
    Design    ←──────────────→  System Testing
      Detailed Design  ←─────→  Integration Testing
          Implementation ←───→  Unit Testing
```

Test planning for each level happens during its paired development phase, not during Verification. This is a significant improvement over basic waterfall — it forces testability to be considered early.

### Incremental Waterfall
The system is divided into increments, each delivered through a full waterfall cycle. Later increments build on earlier ones. This adds iteration at the system level while preserving waterfall discipline within each increment.

### Spiral Model (Boehm)
Adds explicit risk analysis loops to the waterfall structure. Each cycle of the spiral includes: objectives, risk analysis, development/test, and planning for the next cycle. The spiral model was a direct precursor to RUP.

### Regulatory Waterfall
Adds compliance artifacts and third-party review gates:
- Independent Verification & Validation (IV&V)
- Regulatory submission artifacts
- Audit trail requirements
- Formal methods for safety-critical components

## Mapping to Our SDLC

| Our artifact | Waterfall equivalent |
|-------------|---------------------|
| Idea (D-number) | Change request or project initiation |
| Spec | Requirements specification (SRS) + Design specification |
| Planning | Project plan (Gantt chart, resource allocation, phase schedule) |
| Prompt | Work package assignment |
| Implementation | Coding phase deliverables |
| Validation | Verification phase (all test levels) |
| Deploy | Deployment phase |
| Result | Phase completion reports, test reports, gate review records |
| Chronicle | Project closeout report, lessons learned, maintenance handoff |

### Key difference from our native process
Our native process is iterative and deliverable-centric. Waterfall is sequential and phase-centric. In waterfall, *all* requirements are specified before *any* design begins. In our native process, a single deliverable flows through all phases. Mapping between the two requires either:
1. Treating each waterfall phase as consuming/producing our artifacts in bulk, or
2. Using incremental waterfall, where each increment maps to a set of our deliverables

## Transition Triggers

| Signal | Direction |
|--------|-----------|
| Requirements are changing frequently | → RUP (adds iteration) or Scrum (drops phases) |
| Team needs faster feedback | → Any iterative lifecycle |
| Project is well-understood, team is small | → Scrum or even Kanban |
| Regulation allows iterative delivery | → RUP with compliance extensions |
| Current waterfall is producing documents nobody reads | → Diagnose: is it the lifecycle or the culture? |
