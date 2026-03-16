# Testing Discipline

**Status**: Adopted — codified in `lifecycles/native.md` (Validation phase) and `operations/deliverable_lifecycle.md` (Validated/Deployed states)
**Architecture reference**: `improvement-ideas/hybrid-browser-testing.md`
**Knowledge store**: `knowledge/testing/` (cross-project), project `docs/testing/knowledge/` (project-specific)
**Spec format prototype**: `improvement-ideas/test-spec-format-draft.md`

## Summary

Hybrid browser testing approach combining Claude Chrome Plugin (CCP) for SDLC-aware exploration with Playwright CLI for fast, deterministic execution. Self-improving knowledge loop (Layers 0-6) reduces token cost and improves test quality over time. Testability as a first-class code quality concern.

## Key Decisions (D1-D10)

Documented in `improvement-ideas/hybrid-browser-testing.md` § 11.

## Mutation Verification — The Persistence Rule

Any test that validates a write operation (create, update, delete) MUST verify persistence, not just UI response. Optimistic UI updates and HTTP 200s are not proof that data was saved.

**Required sequence for mutation verification:**
1. **Perform** the write action (create, update, delete, link, unlink)
2. **Force state reset** — close the modal, navigate away, or reload the page. This discards all client-side state including optimistic updates, local cache, and component state.
3. **Return** to the same view via fresh navigation (reopen modal, revisit page)
4. **Confirm** the change persisted — the data must reflect the mutation from a fresh server round-trip

**What does NOT count as verification:**
- Seeing the UI update immediately after the action (optimistic update)
- A 200 status code in the network tab (the request succeeded but the backend may not have persisted)
- Console logs showing "success" (frontend interpretation, not server truth)
- The absence of an error (silent failures exist)

**What counts:**
- Server-round-tripped state: data loaded fresh from the API after a full state reset
- A subsequent GET request returning the mutated data
- Reopening the entity and seeing the change reflected

**Why this matters:**
This rule exists because of a real failure: a test agent declared a file-delete feature "fixed" because the file disappeared from the UI after clicking delete. The delete button triggered an optimistic removal from the local list, but the backend returned 401 (unauthorized) — the file was never actually deleted. The next time the user opened the task, all "deleted" files reappeared. The test passed, but the feature was broken.

**Shorthand for test specs:** "Persist-verify" or "round-trip verify" — meaning the test must prove the mutation survived a full client reset.

## Agent Capability Requirements for Testing Roles

When spawning agent teams with a testing role, the agent type must match the SDLC testing workflow. The hybrid approach (CiC exploration + Playwright execution) requires specific tool access.

**Lesson learned (2026-03-07, Burndown feature):** A `test-agent` type was spawned for convergence testing. It could only use CiC browser automation (screenshots, clicks, find, read_page) — no file write, no bash. It could not write Playwright test scripts or run `npx playwright test`, making it unable to execute the SDLC testing workflow. It faithfully attempted workarounds within its constraints but could not follow the process as designed.

**Rule:** Testing agents that follow this SDLC must be `general-purpose` type, not `test-agent`. They need:

| Capability | Why | Tool |
|-----------|-----|------|
| File write | Create Playwright test specs | Write, Edit |
| Bash execution | Run `npx playwright test`, check results | Bash |
| File read | Read existing test patterns, fixtures, config | Read, Glob, Grep |
| CiC (optional) | Visual recon, UI maps, element catalog | mcp__claude-in-chrome__* |

**Future enhancement:** Agent role definitions in planning docs should include a "required capabilities" checklist. The team creation system could eventually validate that the chosen agent type satisfies the role's requirements. See Harmoniq memory #1076 for the full design note.

**CiC vs Playwright division of labor:**
- **CiC:** Quick visual validation, element discovery, UI mapping, exploratory checks
- **Playwright:** Actual test execution, persistence rule verification, regression suites
- **Never:** Use `javascript_tool` as a substitute for real UI interactions in either tool

## Implementation-Derived Test Specifications

**Status:** Adopted (2026-03-08) — learned from Burndown feature Phases 1-3
**Harmoniq memory:** #1076 (agent capabilities), #1077 (auth fixture refactor)

### The Problem

During the Burndown feature build, the tester was given high-level scenario descriptions ("verify sprint CRUD persists", "verify burndown chart renders") but NOT the implementation-level detail needed to write precise tests. This caused three classes of misses:

1. **Missing UI elements** — Sprint assignment dropdown was in the plan (Step 1.17) but never implemented. The tester couldn't test for something it didn't know should exist.
2. **Wrong values accepted as correct** — Burndown chart showed "5" (task count) instead of "96" (sum of etc_hours). The tester verified "chart renders with data" without knowing the expected value.
3. **Cross-cutting pattern gaps** — `task.project_id` vs `task.projectId` camelCase inconsistency was a known pattern in other code but wasn't flagged as a risk for new components.

### The Solution: Plan-Derived Test Spec

After implementation and before testing, the **Lead/Architect generates a test specification derived from the implementation plan and actual code.** This is the bridge artifact between "what we built" and "what to test."

The test spec is NOT written by the tester (who discovers elements at runtime) or the coder (who knows the code but not the test perspective). It's written by the Lead who has both the plan context and access to the implemented code.

### Test Spec Format

Each entry in a plan-derived test spec follows this structure:

```yaml
- id: TST-001
  plan_step: "1.17"  # Reference to implementation plan step
  feature: "Sprint assignment on task Estimates tab"

  element:
    component: TaskViewEditModal
    location: "Estimates tab (activeTab === 5)"
    type: FormControl + Select
    label: "Sprint"
    condition: "Only visible when task.project_id is set"

  action:
    type: select
    value: "Sprint 1 - Auth Module (2026-03-09 — 2026-03-20)"

  api:
    method: POST
    endpoint: "/api/v1/sprint_items"
    payload: "{ sprint_id, sprintable_type: 'Task', sprintable_id }"
    response: "SprintItem with added_at timestamp"

  verify:
    immediate: "Dropdown shows selected sprint name"
    persistence: "Close modal → reopen → Estimates tab → dropdown still shows sprint"
    value: "Sprint name with date range in parentheses"

  risks:
    - "task.project_id may be camelCase (projectId) depending on source page"
    - "Sprint list empty if no sprints created for this project"
```

### Key Principles

1. **Every plan step becomes a test entry.** If the plan says "Step 1.17: Sprint assignment dropdown", the test spec has a TST entry for it. Missing test entries = missing implementation.

2. **Expected values, not just presence.** "Chart shows remaining = 96" not "chart renders." "Scope = sum(etc_hours) for sprint tasks" not "scope > 0."

3. **Conditions documented.** "Only visible when project uses 'both' estimation unit" — the tester knows to set up the precondition and also test the negative case.

4. **API contracts included.** The tester knows what endpoint should fire and what the response should contain. This catches "API returns 401" issues early.

5. **Risks carry forward.** The Analyst's review findings become test risks. "camelCase fallback pattern" becomes a test that exercises both code paths.

### Workflow Integration

```
Implementation Plan (Lead)
    ↓
Coder implements + Analyst reviews
    ↓
Lead generates Plan-Derived Test Spec (reads plan + actual code)
    ↓
Tester writes Playwright scripts from test spec
    ↓
Convergence testing with value-level assertions
```

The test spec is stored alongside other SDLC artifacts:
- Location: `docs/00_CURRENT_WORK/testing/{feature}_test_spec.yaml` (or .md)
- Archived after feature completion to `docs/00_CHRONICLE/`

### Relationship to Existing Test Spec Format

The existing test spec format (`improvement-ideas/test-spec-format-draft.md`) is designed for **standalone feature pages** — it describes what exists and how to test it. The plan-derived test spec is its upstream input — it describes **what SHOULD exist** based on the plan, so the tester can verify completeness before exploring behavior.

They work together:
- **Plan-derived spec** → "These 12 UI elements should exist with these behaviors" (completeness)
- **Feature test spec** → "Here are 7 scenarios exercising those elements including edge cases" (depth)

The plan-derived spec catches "sprint dropdown is missing." The feature test spec catches "sprint dropdown doesn't handle the no-sprints-available edge case."

### Generating the Test Spec

The Lead generates the test spec by:
1. Reading each step in the implementation plan
2. Reading the actual implemented code (component files, service files)
3. For each plan step, extracting: element location, label, condition, action, expected API call, expected value, persistence check
4. Adding risks from the Analyst's review findings
5. Cross-referencing with existing patterns (e.g., "other tabs handle camelCase — does this one?")

This is the same analysis used to create the Burndown testing walkthrough guide — formalized as a repeatable artifact.

## Active Questions

- Playwright Test Agents (`--loop=claude`) — verified CLI works, agents not yet tested
- Custom spec format — prototype done for Brands list, needs iteration
- Mobile test suite expansion — 5/6 smoke tests pass, need to build out feature-specific mobile tests
- Agent capability validation — should team creation warn when role needs tools the agent type doesn't have?
- Plan-derived test spec tooling — should this be auto-generated by the Lead agent, or should there be a skill/command for it?

## Cycle 1 Results (2026-02-23)

First hybrid test cycle completed against Brands list page. 5/5 scenarios PASS. Key validation: D9 StrictMode persistence fix verified, element catalog populated with Playwright locators. Full results in `improvement-ideas/test-cycle-1-results.md`.

## Parking Lot

*(Ideas that emerged during testing work but aren't yet addressed)*

- **Cross-discipline**: Testability debt remediation flow (tester captures → architect polls → backlog) could generalize to other debt types (design debt, architecture debt, documentation debt)
- **Skill readiness**: Test spec format sections map to skill input schema; knowledge layers map to skill context loading; scenarios map to skill execution instructions (D10)

### From Cycle 1 (2026-02-23)

- **Element catalog needs an operations layer.** Current catalog captures elements (nouns — how to find them) and locators (automation access paths). Future advanced components (Gantt charts, Kanban boards, flowcharts) need an operations layer (verbs — how to interact). Operations are multi-step interaction sequences that reference catalog elements but add choreography: action sequence, coordinate math, settlement timing, success verification. Each operation also carries a tool selection dimension (playwright-only for drag, CCP-preferred for visual exploration, either for clicks). Build this when advanced components arrive in projects; the element catalog structure is ready to host it.

- **CLI auto-generates idiomatic locator code.** This is a major efficiency finding. Every CLI command outputs the equivalent Playwright JS — `fill e225` becomes `page.getByRole('textbox', { name: 'Name' }).fill(...)`. The catalog should capture these directly as the canonical access path. Element catalog updated to include `locator` field on all entries.

## Harmoniq Bootstrapping (2026-02-23)

Transitioned testing approach from Rockcut to Harmoniq ("the big kahuna"). Planning & Tracking chosen as first domain (avoids multi-user chat simulation). Exploration phase completed:

### Health checks
- Frontend: localhost:3006 — UP
- Remote API: dev.dsiloed.com — UP
- Auth: CLI login via landing page modal → /app/ redirect — WORKING
- Auth state saved for reuse across sessions

### Knowledge files created
- `harmoniq-frontend/docs/10_TESTING/knowledge/app-map.yaml` — 6 routes, sidebar nav, key architectural notes
- `harmoniq-frontend/docs/10_TESTING/knowledge/element-catalog.yaml` — Full catalog: auth, app shell, sidebar, projects list, kanban/gantt/time-reporting (from source)
- `harmoniq-frontend/docs/10_TESTING/knowledge/layer0-upstream.yaml` — Remote API dependency, HTML table (not DataGrid), feature gating, loading delay
- `harmoniq-frontend/docs/10_TESTING/knowledge/test-spec-projects-list.yaml` — 7 scenarios for first cycle (PT-01 through PT-07)

### Key differences from Rockcut
- Remote API (not local) — introduces latency, need wait-for-content patterns
- HTML table (not MUI DataGrid) — different roles (table/row/cell vs grid/row/gridcell)
- Project detail is modal overlay (not route) — can't goto directly
- Kanban/Gantt have excellent data-testid coverage; Projects/Tasks have none
- Existing Playwright test suite (7 specs, 8 time-reporting tests) — review before writing new

### New gotcha documented
- `remote-api-initial-load-delay`: First snapshot captures "Loading..." — must wait for content

### Next: Execute cycle 1 (PT-01 through PT-07)

## Mobile Playwright Emulation (2026-02-23)

Mobile testing pipeline proven end-to-end against deployed DEV environment.

### Setup
- Playwright `mobile` project using `devices['Pixel 7']` (412x839, touch, mobile UA)
- Auth fixtures forward `contextOptions` through `getAuthenticatedContext()` so device emulation (viewport, userAgent, hasTouch, isMobile) carries through to `browser.newContext()`
- Viewport-aware storageState cache key (`_mobile` suffix for viewports < 768px) prevents desktop/mobile auth state cross-contamination
- Tests target deployed DEV (`app-dev.portablemind.ai`) by default, override with `TEST_BASE_URL`

### Results (5/6 PASS)
- Login on mobile viewport — PASS
- App shell renders — PASS
- Viewport dimensions match Pixel 7 — PASS
- Bottom navigation visible — PASS
- Desktop sidebar hidden on mobile — PASS
- Message input accessible — FAIL (locator mismatch: mobile uses "Tap for keyboard" not "Message #channel")

### Key files
- `playwright.config.ts` — mobile project definition
- `tests/fixtures/chat-fixtures.ts` — contextOptions forwarding, viewport-aware cache
- `tests/mobile/mobile-smoke.spec.ts` — 6 smoke tests

### Gotchas discovered
- `gcp-vm-disk-full-truncated-bundles` — Docker images fill VM disk, nginx serves corrupted JS (see `knowledge/testing/gotchas.yaml`)
- `mobile-locators-differ-from-desktop` — responsive layouts change placeholders and DOM structure (see `knowledge/testing/gotchas.yaml`)

### Deployment prerequisite
DEV environment must be healthy before mobile tests run. Deploy: `./scripts/deploy_harmoniq.sh deploy DEV`. If deploy fails with disk space error, prune Docker on VM first.
