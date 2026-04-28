# SDLC Process Changelog

A living record of how the process evolves through real use. Each entry captures what changed, why, and where the change originated.

---

## 2026-04-28 (evening): Work Patterns batch 2 — parallel_sessions + path-a-pilot + memory pair

**Origin:** Continuation of the same Pilot #1434 close-out. After the work-patterns directory was scaffolded with `README.md`, `collaboration_model.md`, and `multi-agent-handoff.md` (batch 1), the queue still held two larger work-pattern docs and two memory writes. Rick approved continuing.

**Changes made:**

1. **`~/src/ops/sdlc/work-patterns/parallel_sessions.md`** — NEW. Operating two main agent sessions concurrently. Sections: when this pattern applies, role-can-invoke table (with cross-project hard rule that the testing session never invokes `fix-impl`), runtime sharing (share by default, escalate narrowly), env-impact tagging (`none` / `restart-required` / `migration` / `seed-reset`), shared-source-tree gotcha (git pull on a bind-mounted backend tree IS an env-impact event), branch-collision avoidance via worktrees, mid-flight handoff doc, diagnostic instrumentation lifecycle (Proposal I folded in here).

2. **`~/src/ops/sdlc/work-patterns/path-a-pilot.md`** — NEW. Coverage-first defect-hunt pilot template. Documents the shape (scaffold → tests RED → file defects → fix-impl → validate + close-out) with explicit one-session vs multi-session variants for the fix-impl phase. Variant comparison table at the end of Phase 4 helps teams choose. References `multi-agent-handoff.md` for the multi-session protocol details rather than duplicating.

3. **`~/.claude/projects/-Users-richardkoloski-src/memory/harmoniq_mcp_categories_and_messaging.md`** — NEW memory file. URL format with `?categories=...` filter, full category-to-tool mapping from `lib/service/llm_tools.rb`, restart-vs-reconnect lesson (`/mcp reconnect` keeps the existing process; full restart needed for URL changes to take effect), `general_crud_tool` LlmMessage.create fallback recipe with field gotchas (cross-tenant `tenant_id` trap, polymorphic sender, sequence_position).

4. **`~/.claude/projects/-Users-richardkoloski-src/memory/feedback_briefing_override_pattern.md`** — NEW memory file. When a delegated/handed-off agent finds a clear technical reason to deviate from the briefing's instructions, override with documented justification. Don't silently follow OR escalate without reasoning. Examples from Pilot #1434 D5 fix-impl (wrong-branch override + nuanced "selector miss vs D6" partial override).

5. **`MEMORY.md`** — index updated with two new memory entries (briefing-override + MCP categories).

6. **`operations/sdlc_changelog.md`** — this entry.

**Still queued for follow-on batches** (not addressed today):
- Update `parallel_session_protocol.md` memory with the diagnostic instrumentation lifecycle section (the work-pattern doc has the cross-project version; the memory will get a Harmoniq-flavored echo).
- Tenancy-specific overlay docs in `work-patterns/harmoniq/` (bug-ticket-conventions, test-pipeline-project, party-id-disambiguation) — wait until the parent docs stabilize before adding.

**Rationale:** Batch 2 finishes the work-patterns directory's foundational coverage. With these four files (README + collaboration_model + multi-agent-handoff + parallel_sessions + path-a-pilot), a team adopting the patterns has end-to-end documentation: how to operate two sessions, how to hand off between them, how to run a Path A pilot using either one or two sessions, how humans review the docs that flow through it all. The two memory files capture session-specific lessons that didn't earn first-class SDLC placement.

**Session artifacts that informed this:**
- `~/src/ops/sdlc/improvement-ideas/2026-04-28_proposals_recent-experience.md` (the reviewed proposals doc)
- Personal memory pulled in: `parallel_session_protocol.md`, `harmoniq_test_automation_project.md`, `md_editor_dogfood_workflow.md`
- Pilot #1434's three CC1↔CC2 cycles (handback chronicles, diagnostic traces, override rationale)

---

## 2026-04-28 (afternoon): Work Patterns directory + first batch + Sibling Fix-Impl rule

**Origin:** Same session as the morning's Selector Hygiene write. After Pilot #1434 closed (5 defects, 3 CC1↔CC2 cycles), Rick reviewed a proposals doc surfacing a backlog of patterns observed across recent pilots and made a structural call: **introduce a peer directory `~/src/ops/sdlc/work-patterns/`** for SDLC-agnostic patterns. The patterns describe HOW we operate (multi-agent handoff, document-collaboration conventions, parallel sessions, coverage-first pilots) — distinct from `sdlc/` which describes WHAT the SDLC requires.

**What happened:** Pilot #1434 produced concrete, repeating evidence for a set of patterns that had been load-bearing across multiple pilots (#1413, 4a-4d) but were stuck in personal memory or implicit in `cc-role:` vocabulary. Codifying them at the work-pattern tier (rather than as SDLC discipline rules) preserves their cross-SDLC applicability while giving teams a canonical home to reference.

**Changes made:**

1. **`~/src/ops/sdlc/work-patterns/`** — NEW peer directory. Contains:
   - `README.md` — framing (patterns that work alongside any SDLC), reading guide for when to look here vs. `sdlc/`, catalog, editorial principles.
   - `collaboration_model.md` — Markdown review surface (recommended editor + alternatives; PortableMind users get the dogfooded `md-editor-mac` at https://github.com/rickkoloski/portablemind-md-editor-mac) AND the long-lived working-doc conventions (inline marker convention `**Question:**`/`**Bug:**`/`**Decision:**`/`**Assumption:**` + Open questions / Decision log accumulator pattern).
   - `multi-agent-handoff.md` — Self-verification ritual (5 stages, fix-impl session) + Validation cycle (4 steps, testing session). Three independent signals framing. Includes a chronicle template.
   - `harmoniq/README.md` — tenancy-specific subfolder, explains why it exists. Catalog placeholders for future bug-ticket-conventions, test-pipeline-project, party-id-disambiguation overlays.
2. **`~/src/ops/sdlc/disciplines/testing.md`** — added `## Sibling Fix-Impl Subtask Pattern (Multi-Class Defect Pilots)` rule. Crisp guidance: when a pilot surfaces defects across multiple architectural classes, file sibling fix-impl subtasks (one per class) under the same parent rather than expanding a closed subtask's scope. Cross-referenced from `work-patterns/multi-agent-handoff.md`.
3. **`operations/sdlc_changelog.md`** — this entry.

**Queued for follow-up batches** (not in this commit):
- `work-patterns/path-a-pilot.md` — coverage-first defect-hunt pilot template (with one-session AND multi-session variants).
- `work-patterns/parallel_sessions.md` — broader two-main-session protocol (role-can-invoke table, runtime sharing, env-impact tagging, branch-collision avoidance, mid-flight handoff doc shape, diagnostic instrumentation lifecycle).
- Memory writes (`harmoniq_mcp_categories_and_messaging.md`, `feedback_briefing_override_pattern.md`).

**Rationale:** Patterns that operate alongside (not within) the SDLC needed a home. Bundling them into `disciplines/` would have implied they're universal SDLC requirements — they're not. Pulling them into a peer directory makes the layering explicit: an SDLC adopter who doesn't run multi-agent sessions doesn't need to read `multi-agent-handoff.md`; they can skip cleanly to the SDLC requirements proper. Teams that DO run multi-agent setups get a canonical reference that doesn't presume tenancy-specific structure.

**Session artifacts that informed this:**
- `~/src/ops/sdlc/improvement-ideas/2026-04-28_proposals_recent-experience.md` (the reviewed-and-approved proposals doc with full Decision log)
- Pilot #1434's three CC1↔CC2 cycles + handback chronicles (closed 2026-04-28)
- Personal memory: `parallel_session_protocol.md`, `harmoniq_test_automation_project.md`, `md_editor_dogfood_workflow.md` (sources for the patterns; will be cross-referenced from the new docs as the queued batches land)

---

## 2026-04-28: Selector Hygiene rule

**Origin:** Harmoniq Pilot #1434 — cross-tenant unread badges (closed 2026-04-28). Two distinct test-selector failures surfaced (TST-CTU-01 + TST-CTU-02), both of the same architectural class: the test anchored on framework-generated CSS classes / assumed-but-not-rendered HTML semantic elements rather than on a stable `data-testid` contract.

**What happened:** The pilot fixed five backend + frontend defects in three CC1↔CC2 rounds. In the second and third rounds, both regression tests were RED for selector reasons — not because the bug wasn't fixed. CC2's discipline (verifying the data flow end-to-end via fiber walks + DOM inspection) caught it instead of papering over the test by editing it. Both fixes followed the same shape: add a `data-testid` on the component, key the test off it. The pattern was implicit in the existing `cc-role: testability-impl` vocabulary but not crisply stated as a discipline rule.

**Changes made:**
1. **`disciplines/testing.md`** — Added `## Selector Hygiene — Anchor on data-testid` section after the Persistence Rule. Same first-class weight + framing: rule, what counts / what doesn't, required sequence when adding a regression test, two real failures, shorthand, tie to `cc-role: testability-impl`.
2. **`operations/sdlc_changelog.md`** — This entry.

**Rationale:** Filing the selector misses as Project #21 tickets would have mis-categorized them — they're test-design defects, not product defects. But the lesson DOES belong somewhere durable. Codifying as a first-class testing discipline rule (alongside the Persistence Rule) ensures future regression tests start with a testid anchor by default rather than re-discovering the failure mode.

**Session artifacts that informed this:**
- `harmoniq-frontend/docs/00_CURRENT_WORK/stepwise_results/2026-04-28_cc2-cc1-instructions_pilot-1434-test-selector-fix.md` (CC2's first handback — diagnosed the channel-row Box-vs-Badge mismatch)
- `harmoniq-frontend/docs/00_CURRENT_WORK/stepwise_results/2026-04-28_cc2-cc1-instructions_pilot-1434-d5-and-tst-ctu-02-selector.md` (CC2's second handback — diagnosed the `<header>`-vs-`<div class="border-layout-north">` mismatch)
- Project #52 #1434 (pilot parent) + #1431, #1432, #1433, #1436, #1437 (defects, all closed)

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
