# Hybrid Browser Testing: Claude Chrome Plugin + Playwright

**Status**: WIP — Research & Design Phase
**Date Started**: 2026-02-22
**Testing Ground**: ~/src/apps/rockcut (Rockcut Brewing App)

---

## 1. The Problem

Our current browser automation testing uses Claude's Chrome plugin exclusively. This works for exploratory validation but has three structural limitations:

1. **Session management** — Single WebSocket connection goes stale; silent failures; "Browser extension is not connected" errors; no headless/CI mode
2. **Speed** — Extension bridge adds latency per action; AI reasoning adds seconds per step; multi-step flows are slow to execute repeatedly
3. **Advanced interactions** — Drag-and-drop via coordinate simulation is unreliable; no cross-browser support; limited to Chrome/Edge

These limitations make the plugin unsuitable as our *sole* testing tool, especially for regression suites and CI/CD.

But there's a fourth problem that's upstream of tooling:

4. **Testability** — Code that's hard to test stays hard to test regardless of which tool you throw at it. When the testing agent resorts to screen coordinates, fragile timing hacks, or DOM structure guessing, that's a signal the code should change, not just the test.

## 1.1 Core Principle: Testability Is a First-Class Concern

**If you have to fight the UI to test it, the UI should change.**

This principle applies to every participant in the process:

| Who | When they encounter poor testability | What they do |
|-----|--------------------------------------|--------------|
| **Architect** (code review) | Sees a component with no stable selectors, no accessible labels, or interaction only possible via coordinates | Flags it. Proposes `data-testid`, `aria-label`, semantic HTML, or structural refactoring. |
| **CCP** (exploration) | Has to click by coordinates because there's no accessible target. Has to guess at DOM structure because elements lack identity. | Reports it as a **testability debt item**, not just a test workaround. |
| **Test generator** (spec → test) | Writes a test that depends on pixel position, DOM child index, or MUI-generated class names | Emits a warning: "This assertion is fragile — consider adding `data-testid` to [component]." |
| **Human** (review) | Sees coordinate-based or structure-dependent tests in a PR | Asks: "Can we make this testable instead of testing around it?" |

### What "testable" means concretely

- **Stable identity**: Interactive elements have `data-testid`, `aria-label`, or semantic role attributes that survive refactoring and framework upgrades
- **Accessible tree coverage**: Components render meaningful accessibility trees — not just for testing, but because accessibility and testability are the same problem
- **Deterministic state**: Components reach a testable state via data/props, not timing. Loading states are explicit (skeleton, spinner), not implicit (maybe the data is there after 3 seconds)
- **Interaction contracts**: Drag targets accept standard DnD events. Custom interactions expose keyboard alternatives. Complex widgets provide programmatic APIs alongside visual interaction.

### The remediation flow: tester captures, architect prescribes

Testability debt doesn't just get logged — it gets healed. But the **information flows in two directions at different cadences**, with clear role separation:

**Direction 1: Tester → Knowledge Store (during test runs)**

The tester captures factual observations — what was hard, what was fragile, what required workarounds. It does NOT prescribe fixes. This keeps the tester fast and focused on testing.

```
Test run encounters fragile interaction
        │
        ▼
Agent classifies the debt item factually:
  WHAT:  "Coordinate-based click required on BrandRow action button"
  WHERE: "BrandsList.tsx, row action area"
  WHY:   "No data-testid, no aria-label, no distinguishing role"
  WORKAROUND: "Clicked at coordinates (450, 312)"
        │
        ▼
Written to knowledge store:
  testability-debt section of project knowledge
  (factual observation, not a recommendation)
```

**Direction 2: Knowledge Store → Architect (at planning/coding time)**

The architect polls the knowledge store before coding sessions — not after every test run, but at natural planning boundaries (deliverable kickoff, sprint planning, pre-coding review). The architect understands context the tester doesn't:

```
Architect reads testability debt before coding session
        │
        ▼
Architect triages each item with domain knowledge:
  - "BrandRow needs data-testid → FIX NOW (quick, high value)"
  - "StatusChip color not in a11y tree → FIX NOW (accessibility + testability)"
  - "FormDialog anchor selector fragile → DEFER (component refactor coming in D12)"
  - "ABV has no range validation → ACCEPT (domain decision: we allow any value)"
        │
        ▼
Fix items enter the coding session backlog
  (alongside feature work for the current deliverable)
        │
        ▼
Fixes implemented during normal coding
        │
        ▼
Knowledge store updated:
  - Element catalog gains stable refs
  - Testability debt items marked resolved
  - Next test run uses stable selectors instead of workarounds
```

**Why this separation matters:**

| Concern | Tester role | Architect role |
|---------|-------------|----------------|
| **What's hard to test** | Knows this directly (experienced it) | Learns from knowledge store |
| **What's worth fixing** | Doesn't know (no code context) | Knows this (understands roadmap, refactors, tradeoffs) |
| **How to fix it** | Doesn't need to know | Prescribes the right approach |
| **When to fix it** | Irrelevant to tester | Triages: now, defer, accept |
| **Token cost** | Stays low (captures facts, doesn't reason about fixes) | Normal (reads curated debt, makes decisions) |

**Cadence:**

| Event | What happens |
|-------|-------------|
| After each test run | Tester writes debt items to knowledge store (automatic, factual) |
| At deliverable planning | Architect polls knowledge store, triages debt into backlog |
| During coding session | Architect fixes triaged items alongside feature work |
| After fixes deployed | Next test run verifies fixes, resolves debt items in knowledge store |

This avoids two failure modes:
1. **Tester-push noise**: If the tester filed fix recommendations after every fragile interaction, the architect would be buried in low-context suggestions
2. **Stale debt**: If nobody polls the knowledge store, debt accumulates silently and tests stay fragile

The architect polling at planning boundaries is the natural integration point — it's when you're already thinking about what to build and how.

This is the same self-improving loop from Section 10, but applied to the **code under test**, not just the test infrastructure. The test process improves the product, not just itself.

## 2. The Hybrid Idea

Use each tool where it's strongest:

| Phase | Tool | Why |
|-------|------|-----|
| **Explore & validate** | Claude Chrome Plugin (CCP) | Uses your logged-in session, visual feedback, natural language — AND can be primed with upstream SDLC context |
| **Map navigation & generate specs** | CCP + Playwright CLI | CCP explores with intent; we capture patterns as structured specs |
| **Execute regression tests** | Playwright (CLI or direct) | Fast (milliseconds per action), deterministic, headless, CI-ready, cross-browser |
| **Heal broken tests** | Playwright Test Agents (Healer) | AI-powered test repair when UI changes break selectors |

### 2.1 The CCP as SDLC-Aware Validator (Not Just an Explorer)

The Chrome plugin's role is deeper than "manual exploratory tool." It operates inside a Claude Code session, which means it can be **primed with upstream context from every prior SDLC phase** before it touches the browser. This makes it a bridge between design intent and runtime verification.

**What flows into the CCP from upstream phases:**

```
┌──────────────────────────────────────────────────────────────┐
│                    UPSTREAM SDLC PHASES                       │
│                                                              │
│  Specs ──────────► Acceptance criteria                       │
│                    "Formula columns display italic + fx badge"│
│                                                              │
│  Planning ───────► Known risk areas                          │
│                    "DataGrid virtualizes rows — persistence  │
│                     may break under StrictMode"              │
│                                                              │
│  Architecture ───► Integration points to verify              │
│                    "INVENTORY_ON_HAND calls POST             │
│                     /api/formulas/execute — verify proxy"    │
│                                                              │
│  Code review ────► Fragile patterns spotted during impl      │
│  (Architect)       "This useEffect has no dep guard —        │
│                     double-mount will clobber localStorage"  │
│                                                              │
│  Prior test runs ► Known gotchas, timing profiles,           │
│  (Knowledge store)  element catalog, failure patterns        │
│                                                              │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                    CCP EXPLORATION SESSION                    │
│                                                              │
│  The plugin doesn't wander blindly. It arrives with:         │
│                                                              │
│  ✓ Specific acceptance criteria to verify                    │
│  ✓ Known risk areas to probe first                          │
│  ✓ Integration points that need end-to-end confirmation      │
│  ✓ Fragile code patterns to stress-test                     │
│  ✓ Prior knowledge (what worked, what broke, what to watch)  │
│                                                              │
│  Result: targeted exploration, not aimless clicking          │
└──────────────────────────────────────────────────────────────┘
```

**Why this matters for token efficiency:**

Undirected exploration is expensive — the AI visits pages, reads full snapshots, reasons about what matters, often backtracks. Directed exploration with upstream context skips the "figure out what to look at" phase entirely. The CCP knows *what* to verify and *where* to look before it opens a single tab.

**Why this matters for test quality:**

An architect reviewing code sees things the test runner can't infer from the UI alone:
- Race conditions that only manifest under specific timing
- State management bugs that require a specific sequence to trigger
- Edge cases in business logic that the happy-path UI hides
- Infrastructure assumptions (proxy routing, CORS, auth token expiry) invisible to UI-only testing

These become **targeted probes** during CCP exploration, not afterthoughts.

**Practical example (D9 Column Visibility):**

Without upstream context:
> CCP: "Navigate to /brands. Grid renders. Columns visible. PASS."

With upstream context (architect flagged React 19 StrictMode double-mount concern):
> CCP: "Navigate to /brands. Toggle column off. Refresh page. Verify column stays hidden. Toggle column on. Navigate away. Navigate back. Verify column visible. Refresh. Verify still visible. Now hard-refresh with DevTools cache disabled. Verify persistence survives. **This is specifically testing the useRef gate on the useEffect write that was added to prevent StrictMode double-mount from clobbering localStorage.**"

The second version catches real bugs. The first catches nothing.

### 2.2 Naming Convention

For brevity in this doc and future SDLC artifacts:
- **CCP** = Claude Chrome Plugin (the browser MCP tools)
- **PW CLI** = Playwright CLI (`@playwright/cli`)
- **PW MCP** = Playwright MCP server (`@playwright/mcp`)
- **PW Agents** = Playwright Test Agents (Planner, Generator, Healer)

## 3. Tool Landscape (Feb 2026)

### 3.1 Playwright CLI (`@playwright/cli`) — NEW, KEY FINDING

Microsoft released this in early 2026 specifically for coding agents like Claude Code.

**Why it matters**: ~27,000 tokens per automation task vs ~114,000 via MCP (4x reduction). Snapshots and screenshots are saved as files on disk rather than returned inline in tool responses.

**Command set** (50+ commands):

| Category | Commands |
|----------|----------|
| Core | `open`, `close`, `goto`, `click`, `dblclick`, `fill`, `type`, `drag`, `hover`, `select`, `upload`, `check`, `uncheck`, `snapshot`, `eval` |
| Navigation | `go-back`, `go-forward`, `reload` |
| Keyboard | `press`, `keydown`, `keyup` |
| Mouse | `mousemove`, `mousedown`, `mouseup`, `mousewheel` |
| Save as | `screenshot`, `pdf` |
| Tabs | `tab-list`, `tab-new`, `tab-close`, `tab-select` |
| Storage | `state-save`, `state-load`, `cookie-*`, `localstorage-*`, `sessionstorage-*` |
| Network | `route`, `route-list`, `unroute` |
| DevTools | `console`, `network`, `tracing-start`, `tracing-stop`, `video-start`, `video-stop` |
| Dialogs | `dialog-accept`, `dialog-dismiss` |

**Skills system**: Records YAML workflow files that agents can replay — natural bridge from exploration to automation.

**Setup**:
```bash
npm install -g @playwright/cli@latest
playwright-cli install

# Interactive usage
playwright-cli open https://localhost:5174 --headed
playwright-cli snapshot    # → saves YAML to .playwright-cli/
playwright-cli fill e8 "Pale Ale"
playwright-cli click e12
playwright-cli screenshot  # → saves PNG to .playwright-cli/
```

**When to use CLI vs MCP**:
- CLI: Claude Code sessions (filesystem access), test generation workflows, CI/CD — **our primary choice**
- MCP: Sandboxed environments without filesystem access

### 3.2 Playwright MCP (`@playwright/mcp`)

Official MCP server from Microsoft. Operates on accessibility tree snapshots (not screenshots). ~21-26 tools including `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_type`, `browser_drag`, etc.

**Operating modes**:
- Persistent profile (stores login state)
- Isolated (clean-state per session)
- Extension mode (connects to existing browser tabs)

**Setup with Claude Code**:
```bash
claude mcp add playwright npx '@playwright/mcp@latest'
```

**Limitation**: High token consumption (~114k per task). Tool schemas alone consume ~13.7k tokens.

### 3.3 Playwright Test Agents (v1.56+, Oct 2025)

Three built-in AI agents as a first-class Playwright feature:

- **Planner Agent** — Explores app, produces structured Markdown test plans in `specs/`
- **Generator Agent** — Reads plans from `specs/`, generates executable test files in `tests/`, validates live
- **Healer Agent** — Repairs failing tests by replaying, inspecting UI state, patching locators

**Init**: `npx playwright init-agents --loop=claude`

These agents could replace our manual test-spec-writing step.

### 3.4 Claude Chrome Plugin (Current Tool)

**Strengths**: Uses your actual browser session (already logged in), visual verification, natural language interaction, GIF recording of test runs.

**Documented limitations**:
- Silent WebSocket failures (GitHub Issue #26217)
- Permission/plan approval failures (GitHub Issue #27073)
- CVE-2025-52882 (WebSocket auth bypass, fixed)
- Beta status; no headless mode; Chrome/Edge only; no CI/CD
- Tool schemas consume ~15.4k tokens

**Best role**: Exploratory testing, initial validation, navigation discovery.

### 3.5 Other Notable Tools

| Tool | Description | Relevance |
|------|-------------|-----------|
| **Stagehand** (browserbase) | Hybrid AI+Playwright framework: `act()`, `extract()`, `observe()` primitives mixed with Playwright | Alternative to our hybrid approach; v3 moved to raw CDP |
| **Browser Use** | Fully autonomous AI browser agent (50k+ GitHub stars) | Too autonomous / unreliable for testing (~72% completion) |
| **playwright-bdd** | Gherkin → Playwright test converter | Could be our spec format |
| **executeautomation/mcp-playwright** | Community MCP server, single-tool design (~80% less schema overhead) | Alternative if token cost matters |

## 4. Proposed Hybrid Workflow

### Phase 1: Explore (Claude Chrome Plugin)

Human + Claude Chrome plugin explore the app together:
- Navigate routes, interact with UI, verify behavior
- Plugin's `read_page` gives accessibility tree (reliable for DataGrid content)
- Plugin's `screenshot` + `computer` give visual verification
- Record navigation patterns and element references
- Document what works, what's broken, what needs regression coverage

**Output**: Observations, navigation map, bug reports

### Phase 2: Specify (Claude Code + Playwright CLI)

Convert exploration findings into structured test specs:
- Use Playwright CLI's `snapshot` to capture accessibility trees as YAML files
- Write test specifications in one of these formats:
  - **Structured Markdown** (native to Playwright Test Agents)
  - **Gherkin/BDD** (via playwright-bdd — human-readable, AI-friendly)
  - **YAML skill files** (Playwright CLI native recording format)
- Organize specs by feature area (Brands, Ingredients, Batches, Settings, Auth)

**Output**: `specs/` directory with test plans

### Phase 3: Generate (Playwright Test Agents or Claude Code)

Convert specs into executable Playwright tests:
- Option A: Playwright's Generator Agent reads `specs/`, outputs `tests/`
- Option B: Claude Code generates test files using Playwright CLI for live validation
- Either way, tests use Page Object Model for maintainability

**Output**: `tests/` directory with executable test files

### Phase 4: Execute (Playwright Direct)

Run tests deterministically:
```bash
npx playwright test                    # full suite
npx playwright test --grep @smoke      # smoke tests
npx playwright test --grep @regression # regression suite
```

- Millisecond-speed execution
- Headless mode for CI/CD
- Cross-browser (Chromium, Firefox, WebKit)
- HTML report generation

### Phase 5: Heal (Playwright Healer Agent)

When UI changes break tests:
- Healer Agent replays failing steps
- Inspects current UI state
- Patches locators and waits
- Re-runs until passing

This closes the loop — tests stay current without manual maintenance.

## 5. Architecture Decisions to Make

### 5.1 Test Spec Format

| Format | Pros | Cons |
|--------|------|------|
| Structured Markdown | Native to Playwright Agents; simple | Less formal than BDD |
| Gherkin/BDD | Human-readable; AI-friendly; executable via playwright-bdd | Extra dependency; more ceremony |
| YAML skill files | Native to Playwright CLI; recordable | New format; less tooling |

**Leaning**: Start with Structured Markdown (path of least resistance with Playwright Test Agents), evaluate Gherkin if we need more formality.

### 5.2 When to Use Which Tool

```
                    ┌─────────────────────────────────────────┐
                    │          Testing Workflow                │
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────▼────────────────────────┐
                    │   Is this exploratory / first time?      │
                    └────┬───────────────────────────┬────────┘
                         │ YES                       │ NO
                    ┌────▼────────────┐    ┌────────▼─────────┐
                    │ Claude Chrome   │    │ Do tests exist?   │
                    │ Plugin          │    └──┬─────────┬──────┘
                    │ (explore, map,  │       │ YES     │ NO
                    │  validate)      │  ┌────▼───┐ ┌───▼──────┐
                    └────────┬────────┘  │ Run PW │ │ Generate  │
                             │           │ tests  │ │ from spec │
                    ┌────────▼────────┐  └────────┘ └──────────┘
                    │ Capture specs   │
                    │ (PW CLI + CC)   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ Generate tests  │
                    │ (PW Agents/CC)  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ Execute (PW)    │
                    └─────────────────┘
```

### 5.3 Session & Auth Strategy

Playwright's persistent profile mode stores login state at `~/.cache/ms-playwright/mcp-{channel}-profile`. For Rockcut:
- Save authenticated state with `state-save` after login
- Load it with `state-load` or `--storage-state` on subsequent runs
- Reduces automation runtime by up to 70% (no re-auth per test)

### 5.4 Drag and Drop (the Hard Problem)

Playwright's `locator.dragTo()` works for native HTML5 DnD but fails with custom JS drag libraries. Workarounds:
- Use lower-level `page.mouse.move/down/up` with `{ steps: 10 }` for intermediate events
- For `dragover`-dependent UIs: double-hover pattern (hover target twice before mouse up)
- CDP `Input.dispatchDragEvent` as last resort
- Our MUI DataGrid column reordering will likely need the lower-level approach

## 6. Rockcut as Testing Ground

### Current State
- **Backend**: 47 ExUnit tests (formula runtime, catalog, controller)
- **Frontend**: Zero automated tests
- **Browser testing**: Manual via Claude Chrome plugin, documented in `docs/testing/browser_automation_playbook.md`
- **Known bug**: D10 user creation auth issue (created users can't log in)

### What to Test First (by priority)

1. **Smoke tests**: Login, navigate to each page, verify data loads
2. **CRUD flows**: Brands → Brand → Recipe drill-down; Ingredients with search; Batches with status filter
3. **Formula engine**: Edit formula, verify computed values (EST_IBU, EST_OG)
4. **Column visibility**: Toggle columns, verify localStorage persistence
5. **User management**: Create user, assign role, login as new user (blocked by D10 bug)
6. **Settings**: Category CRUD

### App Routes to Cover

```
/login                            — Auth gate
/                                 — Home (active batches, recent recipes)
/brands                           — Brands list
/brands/:id                       — Brand detail → recipes sub-section
/brands/:brandId/recipes/:id      — Recipe detail (formula columns)
/ingredients                      — Ingredients list (search, category filter)
/ingredients/:id                  — Ingredient detail → lots
/batches                          — Batches list (status filter)
/batches/:id                      — Batch detail (log timeline)
/settings                         — Settings (categories list)
/settings/categories/:id          — Category detail
/settings/users                   — User management
```

## 7. Implementation Roadmap (Proposed)

### Step 1: Install & Configure Playwright
- Add Playwright CLI and test runner to rockcut-ui
- Configure `playwright.config.ts` (base URL, browsers, auth state)
- Verify basic navigation works headless

### Step 2: Capture Auth State
- Script login flow, save state with `state-save`
- Configure tests to load saved state

### Step 3: Exploratory Session (Chrome Plugin)
- Walk through each route with Claude Chrome plugin
- Capture accessibility tree patterns for key pages
- Document element references and interaction patterns

### Step 4: Write Initial Specs
- Convert exploration into structured Markdown specs in `specs/`
- Start with smoke tests (every route loads, renders data)

### Step 5: Generate & Run Tests
- Use Playwright Test Agents or Claude Code to generate test files
- Run headless, verify all pass
- Set up npm script: `pnpm test:e2e`

### Step 6: Expand Coverage
- Add CRUD flow tests, formula validation, column persistence
- Tag tests: `@smoke`, `@regression`, `@feature-{name}`

### Step 7: CI Integration (Future)
- GitHub Actions workflow running on push/PR
- HTML report artifacts

## 8. Speed & Cost Comparison

| Metric | Claude Chrome Plugin | Playwright MCP | Playwright CLI | Playwright Direct |
|--------|---------------------|----------------|----------------|-------------------|
| Tokens per task | ~15.4k schema + variable | ~114k | ~27k | 0 (no LLM) |
| Action speed | Seconds (extension + AI) | Milliseconds (browser) + AI reasoning | Milliseconds + file I/O | Milliseconds |
| Headless | No | Yes | Yes | Yes |
| CI/CD | No | Yes | Yes | Yes |
| Cross-browser | No (Chrome/Edge) | Yes | Yes | Yes |
| Drag & drop | Coordinate simulation | `browser_drag` tool | `drag` command | `locator.dragTo()` + mouse API fallback |
| Session stability | WebSocket issues | Stable (direct control) | Stable | Stable |
| Auth handling | Uses your session | Persistent profile or state files | `state-save`/`state-load` | Storage state files |
| Best for | Exploration, validation | Sandboxed agents | Claude Code integration | Regression suites, CI |

## 9. Key Sources

- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [Playwright CLI (TestCollab)](https://testcollab.com/blog/playwright-cli) / [TestDino deep dive](https://testdino.com/blog/playwright-cli/)
- [Playwright Test Agents docs](https://playwright.dev/docs/test-agents)
- [Simon Willison: Playwright MCP + Claude Code](https://til.simonwillison.net/claude-code/playwright-mcp-claude-code)
- [Checkly: Generating E2E Tests with AI + Playwright MCP](https://www.checklyhq.com/blog/generate-end-to-end-tests-with-ai-and-playwright/)
- [Stagehand v3 (browserbase)](https://www.browserbase.com/blog/stagehand-v3)
- [NxCode: Stagehand vs Browser Use vs Playwright](https://www.nxcode.io/resources/news/stagehand-vs-browser-use-vs-playwright-ai-browser-automation-2026)
- [playwright-bdd](https://github.com/vitalets/playwright-bdd)
- [Claude in Chrome vs DevTools MCP comparison](https://github.com/shanraisshan/claude-code-best-practice/blob/main/reports/claude-in-chrome-v-chrome-devtools-mcp.md)
- [Microsoft ISE: LLM-Driven UI Tests](https://devblogs.microsoft.com/ise/app-modernization-llm-driven-ui-tests-hve/)
- [executeautomation/mcp-playwright](https://github.com/executeautomation/mcp-playwright)

## 10. Self-Improving Test Loop — The Real Multiplier

### 10.1 The Insight

The 4x token reduction from Playwright CLI is a mechanical baseline. The real gains come from the process **not starting from zero every time**.

Today, every exploratory session re-discovers the same app structure, re-learns the same conventions, and re-encounters the same gotchas. That's all repeat token spend. If each cycle captures what it learns and feeds it forward, the compound savings dwarf the mechanical ones.

### 10.2 What Gets Re-Spent Today

| Repeated Work | Tokens per occurrence | Frequency |
|---------------|----------------------|-----------|
| Full page snapshot (accessibility tree) | ~5-10k | Every page visit |
| Re-discovering element patterns (DataGrid cells, MUI components) | ~2-3k | Every test session |
| Re-learning timing (wait 3s for async, 2s for navigation) | ~500 | Every test session |
| Re-encountering known gotchas (DOM != visual, virtualized rows) | ~1-2k | Every failure |
| Re-writing login flow, CRUD patterns, assertion boilerplate | ~1-2k | Every test file |

A typical test session that touches 5 pages might burn ~50-60k tokens on things it already "knew" last time.

### 10.3 Knowledge Layers That Accumulate

The self-improving loop builds seven knowledge layers, each reducing tokens for subsequent runs:

**Layer 0: Upstream SDLC Context** (design intent, risk signals, architect observations)
```
Flows in from specs, planning, code review, and architect sessions.
Exists BEFORE any browser interaction.
Eliminates: undirected exploration, missed edge cases, wasted coverage
Tokens saved: Hard to quantify directly — but transforms exploration quality
```

What it stores:
- Acceptance criteria extracted from specs (what "done" looks like)
- Risk areas flagged during planning (what might break)
- Integration points identified in architecture (what needs end-to-end verification)
- Fragile patterns spotted during code review (what to stress-test)
- Architect observations ("this useEffect has no dep guard — probe double-mount behavior")

Why it's Layer 0:
This is the only layer that doesn't come from testing — it comes from *building*. The architect (Claude Code in design/review mode) sees the code being written and can flag concerns that are invisible from the UI. These become targeted probes during CCP exploration rather than afterthoughts. Without this layer, the CCP explores blind. With it, the CCP arrives knowing what to verify and where the bodies might be buried.

**Layer 1: App Map** (routes, page structure, data relationships)
```
Captured once. Updated only when UI changes.
Eliminates: route discovery, navigation trial-and-error
Tokens saved: ~5-8k per session
```

What it stores:
- Every route, what renders there, what data it fetches
- Navigation relationships (Brands → Brand → Recipe drill-down)
- Auth requirements per route
- Which routes have DataGrids, forms, dialogs

**Layer 2: Element Catalog** (stable selectors, accessibility patterns per component type)
```
Built incrementally from snapshots. Distilled into reusable references.
Eliminates: full-page snapshots for known pages
Tokens saved: ~8-15k per session (biggest single savings)
```

What it stores:
- Per-page element reference maps (DataGrid columns, form fields, buttons)
- Component-type patterns (all DataGrids share these selector patterns, all FormDialogs share these)
- MUI-specific discovery patterns (columnheader role, gridcell role, MuiBox-root wrappers)

**Layer 3: Interaction Recipes** (verified step sequences for common actions)
```
Captured as Playwright CLI skills (YAML) or Page Object methods.
Eliminates: AI reasoning about "how do I log in" or "how do I open an edit dialog"
Tokens saved: ~3-5k per session
```

What it stores:
- Login flow (exact steps, element refs, timing)
- CRUD patterns per entity type (create, read, update, delete)
- DataGrid interactions (search, filter, column visibility toggle, formula edit)
- Navigation sequences (sidebar click → page load → verify)

**Layer 4: Timing Profile** (empirically measured waits, not guesses)
```
Measured from actual runs. Updated when infrastructure changes.
Eliminates: conservative over-waiting or flaky under-waiting
Tokens saved: ~500-1k (small but improves reliability → fewer retries)
```

What it stores:
- Page load times per route (measured P50, P95)
- Async resolution times (API calls, formula computation, remote functions)
- Infrastructure startup times (Phoenix boot, Vite HMR, etc.)

**Layer 5: Gotcha Database** (failures encountered and their resolutions)
```
Accumulated from every failure. Read before every run.
Eliminates: repeat failures → retry loops → wasted tokens
Tokens saved: ~2-5k per session (prevents cascading retry waste)
```

What it stores:
- "MUI DataGrid virtualizes rows — don't assert on rows outside viewport"
- "DOM correctness ≠ visual correctness — always verify computed styles"
- "Server down → formula cells show Error — run health check first"
- "React 19 StrictMode double-mount — useRef guard on localStorage writes"
- Each entry: symptom, root cause, resolution, prevention

**Layer 6: Test Templates** (parameterized structures for common page types)
```
Generalized from successful tests. New pages get tests by filling parameters.
Eliminates: writing boilerplate from scratch for each new page
Tokens saved: ~2-3k per new test file
```

What it stores:
- List page template: navigate, verify grid renders, check column headers, test search, test filter
- Detail page template: navigate from list, verify form fields, test edit, verify save
- CRUD template: create dialog, fill fields, submit, verify in list, edit, delete
- Formula column template: verify fx badge, check computed values, verify italic styling

### 10.4 The Token Math

**Baseline comparison (single test session touching 5 pages):**

| Approach | Tokens | Multiplier |
|----------|--------|------------|
| Claude Chrome plugin (current) | ~75-100k | 1x |
| Playwright MCP (mechanical) | ~114k | Worse (more verbose tool responses) |
| Playwright CLI (mechanical) | ~25-30k | ~3-4x better |
| **CLI + self-improving knowledge** | **~5-10k** | **~10-15x better** |

The compound improvement: CLI saves ~4x mechanically. Knowledge layers save another ~3-5x on top of that. Combined: **10-20x reduction** from the Chrome plugin baseline.

And it gets better over time. First run is close to the CLI baseline (~25k). By the 5th run of the same feature area, it's down to ~5k because the system knows the app.

### 10.5 The Feedback Loop (After Each Cycle)

```
┌─────────────────────────────────────────────────┐
│                 TEST CYCLE N                     │
│                                                  │
│  1. Load knowledge layers (app map, catalog,     │
│     recipes, timing, gotchas, templates)         │
│                                                  │
│  2. Execute tests (Playwright CLI/direct)        │
│                                                  │
│  3. On completion OR failure:                    │
│     ┌──────────────────────────────────────┐     │
│     │  DIFF & UPDATE                       │     │
│     │                                      │     │
│     │  • App map: new/changed routes?      │     │
│     │  • Element catalog: new components?  │     │
│     │    Changed selectors?                │     │
│     │  • Timing: actual vs expected waits? │     │
│     │  • Gotchas: new failure patterns?    │     │
│     │  • Templates: new generalizable      │     │
│     │    patterns?                         │     │
│     │  • Recipes: new interaction flows?   │     │
│     └──────────────────┬───────────────────┘     │
│                        │                         │
│  4. Write updates to knowledge store             │
│                                                  │
└────────────────────────┬────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│                 TEST CYCLE N+1                   │
│                                                  │
│  Starts with everything N learned.               │
│  Skips what's already known.                     │
│  Focuses only on what's new or changed.          │
└─────────────────────────────────────────────────┘
```

### 10.6 Knowledge Store Format

Where does this knowledge live? Options:

**Option A: Structured files in the repo** (simplest, version-controlled)
```
docs/testing/
├── browser_automation_playbook.md  ← EXISTING (manual, unstructured)
├── knowledge/
│   ├── app-map.yaml                ← Routes, pages, data
│   ├── element-catalog.yaml        ← Selectors, patterns per page
│   ├── interaction-recipes.yaml    ← Step sequences (or PW skills)
│   ├── timing-profile.yaml         ← Measured waits
│   ├── gotchas.yaml                ← Failure patterns + resolutions
│   └── templates/                  ← Parameterized test templates
│       ├── list-page.template.ts
│       ├── detail-page.template.ts
│       └── crud-flow.template.ts
```

**Option B: Playwright CLI skills directory** (native integration)
```
.playwright-cli/
├── skills/
│   ├── login.yaml
│   ├── navigate-brands.yaml
│   └── verify-datagrid.yaml
├── snapshots/                      ← Cached page snapshots
└── screenshots/                    ← Reference screenshots
```

**Option C: Hybrid** — YAML knowledge store (A) + Playwright skills (B)
Knowledge layers 1-5 in structured YAML (human-readable, AI-parseable). Layer 6 templates as actual Playwright test files. Skills as the bridge between exploration and automation.

**Leaning**: Option C. The knowledge store is conceptually separate from the test framework. YAML is easy for both humans and AI to read/write. Playwright skills handle the executable side.

### 10.7 What Makes This Different From a Traditional Page Object Model

Traditional POM captures element selectors and interaction methods. This captures *everything the AI needs to avoid re-learning*:

| POM captures | Self-improving loop also captures |
|-------------|-----------------------------------|
| Element selectors | **Design intent & risk signals (Layer 0)** |
| Page navigation | Timing profiles |
| Action methods | Failure patterns + resolutions |
| | Component-type heuristics |
| | Empirical wait measurements |
| | Visual verification requirements |
| | Infrastructure dependencies |
| | Inter-page data relationships |

POM is Layer 3 (interaction recipes). The self-improving loop is Layers 0-6. Layer 0 is the critical differentiator — no traditional testing framework captures upstream design context and feeds it into test execution. This is unique to the AI-in-the-loop approach where the same agent (or agents sharing context) participates in both building and testing.

### 10.8 Bootstrapping: How to Seed the Knowledge Store

The existing `browser_automation_playbook.md` is already a manually-built version of Layers 1, 2, 3, and 5. The bootstrap plan:

0. **Extract Layer 0 from existing SDLC artifacts** — pull acceptance criteria from specs, risk areas from planning docs, architect notes from code review sessions (Rockcut has all of these in `docs/current_work/` and `docs/chronicle_by_concept/`)
1. **Parse the existing playbook** into structured YAML (app map, element catalog, recipes, gotchas)
2. **Run an initial Playwright CLI exploration** of each route — capture snapshots, build element catalog
3. **Measure timing** — run each page 3x, record actual load/render/async times
4. **Convert existing test recipes** into Playwright CLI skills or Page Object methods
5. **After first full test cycle**, update all layers from actual results

This gives us a warm start rather than cold start for the first "real" cycle. Rockcut is particularly well-suited because we have rich upstream artifacts — 10 deliverables worth of specs, planning docs, code review observations, and known issues.

## 11. Decisions (from Open Questions Discussion, 2026-02-22)

### D1: CLI first, MCP fallback
**Decision**: Start with Playwright CLI. Reserve MCP as temporary fallback if CLI has maturity issues.
**Rationale**: CLI's 4x token savings and file-based snapshots align with our self-improving loop design. MCP returns data inline which fights against knowledge caching.
**Verification needed**: Install CLI, run basic automation against Rockcut, assess stability.

### D2: Stay in Claude Code
**Decision**: All tooling must work from Claude Code. No VS Code dependency.
**Rationale**: Claude Code is our primary development environment. Adding VS Code as a requirement fragments the workflow.
**Verification needed**: Confirm Playwright Test Agents work via `npx playwright init-agents --loop=claude`. This is documented as a supported target but needs hands-on validation. Go/no-go for the Healer workflow.

### D3: Component-scoped test strategies for advanced widgets
**Decision**: Test automation strategies must cover advanced UI components (Gantt charts, flowcharts, and the growing shared library at `~/src/shared/`), not just forms and lists. Knowledge layers (especially Element Catalog and Interaction Recipes) should be component-scoped and shareable across projects.
**Rationale**: These widgets are what make apps look purposeful rather than "AI-generated template." Testing them well is what keeps them working as they evolve. Components are treated as projects for knowledge scoping purposes.
**Implication**: Layer 2 (Element Catalog) and Layer 3 (Interaction Recipes) need a `component://` scope in addition to `project://` scope.

### D4: Custom markdown spec format (not Gherkin)
**Decision**: Design our own test spec format in pure markdown, optimized for both human readability and LLM context precision. No Gherkin.
**Rationale**: Gherkin tried to serve two audiences and ended up cumbersome for both. We have an opportunity to design something that natively serves human reviewers (skimmable, clear) and AI test generators (unambiguous, structured, context-efficient) without the ceremony overhead.
**Design challenge**: The spec format needs to carry:
- What to verify (acceptance criteria)
- How to get there (navigation/interaction sequences)
- What "right" looks like (expected values, visual states)
- What might go wrong (risk areas from Layer 0)
- What the agent already knows (references to knowledge layers)
**Next step**: Prototype with a real Rockcut feature (e.g., Brands list page smoke + CRUD test).

### D5: Architect-proposed expected values
**Decision**: For computed/formula columns and business logic, the architect proposes expected values, human validates, and they're stored in a dedicated section of the test spec (format per D4).
**Rationale**: The architect understands the calculation logic from code review. The human confirms against domain knowledge. The spec becomes the source of truth for assertions.

### D6: Defer CI/CD — optimize for learning velocity
**Decision**: No CI/CD integration for now. Keep everything local, manual, fast iteration. Design the CI/CD integration point once the process has proven itself.
**Rationale**: We're in learning mode. Premature CI/CD adds friction without adding value until we know what a good test run looks like.

### D7: Human-assisted knowledge updates, evolving toward automation
**Decision**: Start with human review of all knowledge layer updates. Progress toward automation as patterns prove reliable and repeatable.
**Progression**: Manual → Human-approved automation → Full automation (earned through demonstrated reliability at each stage).

### D8: Two-tier staleness detection
**Decision**: Staleness is detected through two complementary mechanisms:

**Tier 1 — Proactive (architect pre-review):**
The architect knows when code changes invalidate existing tests. Before a test run, the architect reviews the test harness against current code, identifies deprecated paths, changed functionality, and stale assertions. This prevents sending the testing agent on a wild goose chase.

**Tier 2 — Reactive (agent-detected unexpected results):**
During execution, the testing agent classifies results into three categories:
- **PASS** — Result matches expectation
- **FAIL** — Result clearly contradicts expectation (e.g., element missing, wrong value)
- **UNEXPECTED** — Result doesn't match expectation but might reflect an intentional change

"Unexpected" is not "broken." The loop: unexpected result → agent flags it → human decides (update test, update code, or investigate further). This prevents false negatives from intentional UI changes being classified as failures.

### D9: Cross-project knowledge portability
**Decision**: Two-tier knowledge architecture:
- **Cross-project knowledge**: Framework patterns, common widget strategies, testing conventions, MUI component behaviors — lives in `~/src/ops/sdlc/` or `~/src/shared/`
- **Project-specific knowledge**: Routes, data, business logic, acceptance criteria — lives in the project's `docs/testing/knowledge/`

Components are treated as projects, so component-specific knowledge (e.g., "how to test datagrid-extended column visibility") lives with the component and is inherited by any project that uses it.

### D10: Skill-readiness as a design constraint
**Decision**: Everything we build — spec formats, knowledge layers, decision rules, feedback loops — should be structured so it can eventually become a Claude Code skill (or a set of skills composing an SDLC).
**Rationale**: The natural progression is: manual process → documented SDLC steps → automated skills. If we design with that trajectory in mind, we avoid a costly rewrite later. This doesn't add process weight now — it just means we keep formats clean, conventions explicit, and decision logic written as consumable rules rather than narrative prose.
**What this means in practice**:
- Spec formats should have consistent section names and structure (a skill's input schema)
- Knowledge layers should be machine-parseable (YAML, not freeform markdown)
- Decision points should be expressed as rules: "if X → do Y" (a skill's branching logic)
- Feedback loops should have named inputs and outputs (a skill's tool calls)
- The test spec format (D4) is a skill prototype that doesn't know it's a skill prototype yet

**What this does NOT mean**:
- We're not building skills now — we're learning what works through manual iteration
- We're not adding abstraction overhead — "skill-ready" just means "well-structured"
- We're not constraining ourselves to current skill capabilities — the process may push skill design forward

**Future trajectory**:
```
NOW:     Manual process, learning on Rockcut
         ↓
SOON:    Process stabilizes → documented as repeatable SDLC steps
         ↓
THEN:    Steps become skill definitions
         - Spec format → skill input schema
         - Knowledge layers → skill context loading
         - Scenarios → skill execution instructions
         - Feedback loops → skill output handlers
         ↓
LATER:   Individual skills compose into an SDLC skill suite
         - /test-explore → CCP-driven exploration with Layer 0 priming
         - /test-spec → generate spec from exploration findings
         - /test-generate → convert spec to Playwright tests
         - /test-run → execute and classify results
         - /test-learn → update knowledge layers from results
```

## 12. What to Work on Next

Based on the decisions above, the concrete next steps in priority order:

### Step 1: Verify tooling (D1, D2)
- Install Playwright CLI, run against Rockcut, assess stability
- Test `npx playwright init-agents --loop=claude` — does it work from Claude Code?
- Document any gaps or fallback needs

### Step 2: Design the spec format (D4)
- Pick a real Rockcut feature (suggest: Brands list page — smoke + CRUD)
- Draft a test spec in our custom markdown format
- Iterate until it reads well for both human and LLM
- This becomes the template for all future specs

### Step 3: Bootstrap the knowledge store (D3, D9)
- Parse existing `browser_automation_playbook.md` into structured layers
- Create the directory structure for cross-project and project-specific knowledge
- Seed Layer 0 from Rockcut's existing SDLC artifacts

### Step 4: First hybrid test cycle on Rockcut
- CCP exploration primed with Layer 0 context
- Capture findings → specs (Step 2 format)
- Generate Playwright tests (CLI or Agents)
- Execute, review results, update knowledge layers (with human review per D7)
- Retrospective: what worked, what to improve for next cycle