# Architecture Discipline

**Status**: Parking lot — capturing ideas as they emerge from other work

## Scope

System design, component boundaries, integration patterns, technology choices, cross-project patterns.

## Parking Lot

### From Testing (2026-02-22)

- **Layer 0 (upstream SDLC context) is an architectural function.** The architect sees code being written and flags concerns (race conditions, state management bugs, fragile patterns) that are invisible from the UI. These become targeted test probes. No traditional testing framework has this channel. It's unique to the AI-in-the-loop approach where the same agent participates in both building and testing.

- **Component-scoped knowledge travels with the component.** Test strategies for datagrid-extended, Gantt charts, flowcharts should live with the component, not the project. When Harmoniq or vNext uses the same component, it inherits the test knowledge. Architecture implication: shared components need a `testing/` directory alongside their source.

- **Two-tier knowledge architecture.** Cross-project knowledge (framework patterns, MUI behaviors) lives in `ops/sdlc/testing-knowledge/`. Project-specific knowledge (routes, data, business logic) lives in the project. This pattern generalizes beyond testing to all disciplines.

- **Token economics as an architectural constraint.** The 4x token reduction from PW CLI vs MCP, and the potential 10-15x from self-improving knowledge, are architectural decisions — they determine what's feasible in an AI-assisted workflow. Architecture should treat token budgets like performance budgets.
