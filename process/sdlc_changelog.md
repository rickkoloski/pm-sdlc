# SDLC Process Changelog

A living record of how the process evolves through real use. Each entry captures what changed, why, and where the change originated.

---

## 2026-02-26: Validation & Deploy Stage Gates

**Origin:** Rockcut D12-D14 session — first full agent team (analyst, coder, tester, deployer) with hybrid testing pipeline

**What happened:** Implemented Matt's process & standards feedback (Brewhouse, Process Profiles, Recipe Operations, UI fixes) using an agent team. The session naturally produced a 5-step validation pipeline:
1. CCP exploratory testing (18/18 pass)
2. Test spec authoring (5 spec docs)
3. Playwright test generation (5 `.spec.ts` files, 13 tests)
4. Playwright execution (13/13 pass)
5. Deployment with smoke tests

This pipeline was implicit — agents followed it because the SDLC testing docs described it, but it wasn't codified as a required process step. The deploy was also ad hoc (CD asked for it mid-session to parallelize human review). Both should be explicit.

**Changes made:**
1. **`process/overview.md`** — Updated flow from `Implementation → Result` to `Implementation → Validation → Deploy → Result`. Added full Validation section (5-phase hybrid testing with minimum bar and skip criteria). Added Deploy section (gated by validation, parallel option documented). Added principles #8 and #9.
2. **`process/deliverable_lifecycle.md`** — Added `Validated` and `Deployed` states between In Progress and Complete. Defined transition criteria for each. Documented the parallel deploy pattern (CD can authorize deploy before validation completes, but Complete requires both).
3. **`process/sdlc_changelog.md`** — This entry.

**Rationale:** The Rockcut D12-D14 session proved the pipeline works end-to-end with an agent team. Codifying it ensures future deliverables follow the same gates rather than relying on ad hoc discipline. The parallel deploy pattern (human review while tests run) emerged as a practical need and is now explicitly supported.

**Session artifacts that informed this:**
- `rockcut/docs/testing/d12-d14-TEST_RESULTS.md` (CCP Phase 1 results)
- `rockcut/docs/testing/specs/` (Phase 2 test specs)
- `rockcut/rockcut-ui/e2e/` (Phase 3-4 Playwright tests)
- `~/src/ops/sdlc/improvement-ideas/hybrid-browser-testing.md` (source architecture)

---

## 2026-02-07: Tester Knowledge Capture + Process Update Command

**Origin:** Harmoniq project — Agent Teams SDLC trial (users-flyout-test)

**What happened:** After a successful agent team test run (tester validated 8/8 test cases), we realized the tester didn't capture navigation paths or lessons learned. The knowledge from that test session was lost when the agent shut down.

**Changes made:**
1. **`process/overview.md`** — Added "Tester Knowledge Capture" section defining required Navigation & Learnings content in test results
2. **`process/overview.md`** — Added "Updating This Process" section with "Let's update the SDLC" trigger command
3. **`process/overview.md`** — Added principle #7: "Capture testing knowledge"
4. **`process/sdlc_changelog.md`** — Created this file as a living changelog
5. **Harmoniq `agent-teams-sdlc-evolution.md`** — Added tester knowledge capture convention and filled in Learnings section

**Rationale:** Testing the same UI areas repeatedly is expensive. Navigation paths and gotchas captured by the first tester compound into efficiency for all future runs. The "Update SDLC" command ensures process improvements discovered during real work have a path back into the canonical framework.
