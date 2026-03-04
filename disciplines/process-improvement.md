# Process Improvement Discipline

**Status**: Parking lot — meta-discipline for improving the SDLC itself

## Scope

Improving the SDLC framework, process maturity, tooling evolution, skill development.

## Foundational Principles

### Toolbox, not recipe (2026-02-23)

The single most important principle governing all process improvement work. Both RUP and SAFe were considered heavy not because the authors intended them to be — both included "tailor the methodology" as a first activity — but because inexperienced practitioners treated them as prescriptions to follow from alpha to omega, and the ecosystems (tooling, certification, consulting) reinforced that behavior.

Our framework must function as a toolbox: helpful, available prescriptions for common needs, with no overhead that isn't needed or requested. This means:

- **Process is pulled, not pushed.** Teams reach for a discipline when they need it. Nobody mandates all eight disciplines for every project.
- **Ad hoc is legitimate.** Vibe coding, exploratory prototyping, and "just build it" are valid approaches. The process accommodates them (see ad hoc reconciliation) rather than fighting them.
- **Overhead earns its place.** Every process element must demonstrate value before it becomes standard. "We should have done X" is the trigger, not "the process says we must do X."
- **Skills inherit this principle.** When disciplines become Claude Code skills, each skill should be independently invocable and immediately useful — not part of a mandatory pipeline.
- **Maturity is not ceremony.** Moving from CMMI Level 1 to Level 2 means the process is *documented and repeatable when invoked*, not that it's always invoked.

This principle is the governor on all other process improvement. If an improvement makes things heavier without making them better, it's wrong.

## Parking Lot

### From Testing (2026-02-22)

- **CMMI maturity progression is our roadmap.** Level 1 (ad hoc) → Level 2 (managed, documented) → Level 3 (standardized across projects) → Level 4 (measured) → Level 5 (self-improving). We're building Level 2 with the architecture to support 3-5. Each level is earned by demonstrating it works at the current level.

- **Disciplines → Skills progression.** Each discipline parking lot is raw material for a future Claude Code skill. The pattern: manual process → documented steps → structured knowledge → skill definition → skill suite. Testing is the pioneer discipline; others follow the same path.

- **The self-improving loop generalizes.** The testing knowledge store (Layers 0-6) with its capture-accumulate-feed-forward cycle applies to every discipline. Design knowledge, architecture knowledge, BA knowledge all benefit from the same structure. Consider a generic "discipline knowledge store" template.

- **Tester captures, architect prescribes — generalized.** The remediation flow (tester writes factual observations → knowledge store → architect polls at planning boundaries → triages into backlog) applies to any discipline pair. Designer captures usability concerns → product research polls them. BA captures requirement gaps → architect polls them for system impact.

- **Disciplines-as-skills orchestration.** When individual discipline skills exist, the SDLC becomes an orchestrator that invokes discipline skills at appropriate phases with appropriate intensity. This is the "SDLC skill suite" vision from D10.

### Process Maturity Tracker

| Discipline | Current level | Evidence | Next level target |
|-----------|---------------|----------|--------------------|
| Testing | 2 (Managed) | Documented process, knowledge store, spec format, tooling verified | 3 (Standardized) — apply to second project |
| Design | 1 (Initial) | Ad hoc, some insights captured in parking lot | 2 — document patterns from shared component library |
| Coding | 1-2 | Conventions in CLAUDE.md, some patterns documented | 2 — formalize coding patterns library |
| Architecture | 1-2 | Decisions documented in deliverables, Layer 0 concept | 2 — formalize architecture decision records |
| Business Analysis | 1 | Requirements in specs, ad hoc | 2 — structured requirements templates |
| Product Research | 1 | Not yet started formally | 1 — capture initial questions |
| Deployment | 2 | Documented in Rockcut (Fly.io), bootstrap guide exists | 2-3 — generalize across projects |
| Process Improvement | 2 | This framework exists, changelog maintained | 3 — self-improving via discipline feedback |
