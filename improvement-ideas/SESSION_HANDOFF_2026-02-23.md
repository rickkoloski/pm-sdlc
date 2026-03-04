# Session Handoff — Hybrid Browser Testing Design + SDLC Disciplines

**Date**: 2026-02-22 through 2026-02-23
**Session purpose**: Design and prototype a hybrid browser testing process, establish SDLC discipline framework

---

## What Was Accomplished

### Step 1: Tooling Verification — DONE
- Playwright CLI v1.59.0-alpha installed globally (`npm install -g @playwright/cli@latest`)
- Verified against Rockcut: opened browser, navigated, filled login form, authenticated, captured snapshots, saved auth state
- Key findings:
  - `open` may land on about:blank — always follow with `goto`
  - Headless by default (no flag needed, passing `--headless` errors)
  - CLI auto-generates role-based Playwright code (`getByRole`, `getByTestId`)
  - Auth state saved to `.playwright-cli/storage-state-2026-02-22T14-37-19-634Z.json`
  - Snapshots are clean YAML (~100 lines per page), saved to `.playwright-cli/`
- Playwright Test Agents (`--loop=claude`) NOT YET TESTED — still a verification TODO

### Step 2: Spec Format Prototype — DONE
- File: `improvement-ideas/test-spec-format-draft.md`
- Prototyped against Brands list page with real data from CLI exploration
- Sections: Context, Prerequisites, Acceptance Criteria, Risk Areas, Test Data, Scenarios, Testability Debt, Knowledge References
- Test Data section added per user request — 5 sub-tables: Seed Data, Valid Inputs, Boundary & Edge Cases, Search/Filter Cases, Expected Outputs
- Boundary cases revealed missing backend validation (no range on ABV/IBU/SRM, no length on name)
- 5 testability debt items found on Brands page alone

### Step 3: Knowledge Store Bootstrap — DONE
**Cross-project** (`~/src/ops/sdlc/testing-knowledge/`):
- `tool-patterns.yaml` — PW CLI, CCP, PW MCP usage patterns
- `component-catalog.yaml` — MUI DataGrid, FormDialog, StatusChip, PageHeader test strategies
- `gotchas.yaml` — 7 failure patterns (DOM!=visual, server-down, StrictMode, virtual scroll, etc.)
- `timing-defaults.yaml` — Default waits by component type

**Rockcut project** (`rockcut/docs/testing/knowledge/`):
- `layer0-upstream.yaml` — 1 blocker (D10 auth), 4 fragile patterns (D9), 2 integration points, 3 data limitations, acceptance criteria for D8/D9/D10
- `app-map.yaml` — 12 routes with components, actions, testability status
- `element-catalog.yaml` — Stable selectors for login, app shell, brands list (from CLI snapshots)
- `timing-profile.yaml` — Measured values from CLI session

### SDLC Disciplines Framework — DONE
- Directory: `~/src/ops/sdlc/disciplines/`
- README with "Toolbox, Not Recipe" north star + disciplines-vs-phases mental model
- 8 discipline parking lots (testing, design, coding, architecture, BA, product research, deployment, process improvement)
- Each seeded with cross-discipline insights from testing work
- Process improvement includes foundational principles + CMMI maturity tracker

---

## Key Design Decisions (D1-D10)

Documented in `improvement-ideas/hybrid-browser-testing.md` § 11. Summary:

| # | Decision | Status |
|---|----------|--------|
| D1 | CLI first, MCP fallback | Locked — CLI verified |
| D2 | Stay in Claude Code | Locked — PW Agents not yet tested |
| D3 | Component-scoped test strategies | Locked |
| D4 | Custom markdown spec format (not Gherkin) | Locked — prototype done |
| D5 | Architect-proposed expected values | Locked |
| D6 | Defer CI/CD, optimize for learning | Locked |
| D7 | Human-assisted knowledge updates | Locked |
| D8 | Two-tier staleness detection (proactive + reactive) | Locked |
| D9 | Cross-project knowledge portability (two tiers) | Locked — structure built |
| D10 | Skill-readiness as design constraint | Locked |

---

## Key Principles Established

1. **Testability is a first-class code quality concern** (§ 1.1 of main doc)
2. **Tester captures facts, architect prescribes fixes** — remediation flow with role separation
3. **Self-improving knowledge loop** (Layers 0-6) — compound token savings 10-15x
4. **CCP as SDLC-aware validator** — primed with Layer 0 upstream context, not blind exploration
5. **Toolbox, not recipe** — process is pulled not pushed, ad hoc is legitimate, overhead earns its place
6. **Disciplines, not phases** — testing, design, coding etc. are persistent capabilities, not temporal stages

---

## What's Next: Step 4 — First Hybrid Test Cycle

This is where we are. The next concrete work is:

1. **Pick a feature to test** — Brands list page (spec already drafted)
2. **CCP exploration primed with Layer 0** — load layer0-upstream.yaml, target the D9 risks (StrictMode persistence, column menu slot)
3. **Capture findings** — update knowledge layers
4. **Generate Playwright tests** — from spec using CLI
5. **Execute and classify results** — PASS / FAIL / UNEXPECTED
6. **Update knowledge layers** — with human review
7. **Retrospective** — what worked, what to improve

### Prerequisite: Rockcut servers must be running
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:4002/api/health  # API
curl -s -o /dev/null -w "%{http_code}" http://localhost:5174/            # UI
curl -s -o /dev/null -w "%{http_code}" http://localhost:5174/api/health  # Proxy
```

### Known blocker
D10 user creation auth bug — created users can't log in. Don't test user management flows until this is resolved.

---

## File Inventory

| File | Purpose |
|------|---------|
| `ops/sdlc/improvement-ideas/hybrid-browser-testing.md` | Main design doc (~700+ lines) |
| `ops/sdlc/improvement-ideas/test-spec-format-draft.md` | Spec format prototype (Brands page) |
| `ops/sdlc/testing-knowledge/*.yaml` | Cross-project knowledge (4 files) |
| `ops/sdlc/disciplines/*.md` | Discipline parking lots (8 files + README) |
| `rockcut/docs/testing/knowledge/*.yaml` | Rockcut project knowledge (4 files + README) |
| `rockcut/docs/testing/browser_automation_playbook.md` | Existing playbook (reference, content decomposed into layers) |
| `rockcut-ui/.playwright-cli/` | CLI snapshots, screenshots, saved auth state from verification |
