# Architecture Discipline

**Status**: Active — backend capability assessment

## Scope

System design, component boundaries, integration patterns, technology choices, cross-project patterns. Assessing what a backend currently supports and estimating the cost of adding capabilities that unlock desired feature scope.

## Backend Capability Assessment

**When to invoke:** After competitive analysis produces a scoped feature definition with open questions, and before UX modeling finalizes the design. The backend assessment bridges "what should we build?" with "what CAN we build?"

**What it produces:**
- Backend inventory summary (API surface, database, services, auth, existing patterns)
- Capability matrix mapping feature dimensions to support levels (Supported / Partial / New / Infrastructure)
- Cost estimates with T-shirt sizing and dependency chains
- Scope recommendation organized by effort tier (ship now, quick wins, significant investment, revisit later)

**How to use the output:** Map each effort tier back to the competitive analysis scoping questions. This gives the product owner cost-aware information to adjust scope decisions.

**Knowledge store:** `knowledge/architecture/` — methodology in `backend-capability-assessment.yaml`, stack-specific patterns in `technology-patterns.yaml`

**Pipeline position:**
```
/feature-compare → scope questions → product owner picks target scope
  ↓
/backend-assess → capability matrix, cost estimates    ← THIS
  ↓
/ux-model → IA, wireframes, specs (informed by what's feasible)
  ↓
spec → plan → implement
```

## Parking Lot

### From Testing (2026-02-22)

- **Layer 0 (upstream SDLC context) is an architectural function.** The architect sees code being written and flags concerns (race conditions, state management bugs, fragile patterns) that are invisible from the UI. These become targeted test probes. No traditional testing framework has this channel. It's unique to the AI-in-the-loop approach where the same agent participates in both building and testing.

- **Component-scoped knowledge travels with the component.** Test strategies for datagrid-extended, Gantt charts, flowcharts should live with the component, not the project. When Harmoniq or vNext uses the same component, it inherits the test knowledge. Architecture implication: shared components need a `testing/` directory alongside their source.

- **Two-tier knowledge architecture.** Cross-project knowledge (framework patterns, MUI behaviors) lives in `ops/sdlc/knowledge/testing/`. Project-specific knowledge (routes, data, business logic) lives in the project. This pattern generalizes beyond testing to all disciplines.

- **Token economics as an architectural constraint.** The 4x token reduction from PW CLI vs MCP, and the potential 10-15x from self-improving knowledge, are architectural decisions — they determine what's feasible in an AI-assisted workflow. Architecture should treat token budgets like performance budgets.
