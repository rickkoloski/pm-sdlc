# SDLC update proposals — surfaced from recent experience (2026-04-28)

**Author context:** drafted from CC1's perspective at the close of Harmoniq Pilot #1434 (cross-tenant unread badges, 5 defects, 3 CC1↔CC2 cycles). Pulls together patterns observed across Pilots 4a-4d, #1413 (avatar), and #1434 (unread).

**Reading guide:** each proposal carries motivation, evidence, recommended shape, and a **status flag**:

- **READY-TO-WRITE** — concrete, evidenced enough to land as-is in the canonical files
- **DEEPER-DESIGN** — direction is clear; needs a design pass before codification
- **MEMORY-TIER** — smaller scope, cross-project useful, better as a memory file than an SDLC discipline statement

Pick what to ship now vs queue. Several of these have been load-bearing across multiple pilots; their non-codified status is what made me re-derive them this morning instead of pointing teams at a canonical reference.

---

## Already shipped this session

For context, two writes have already landed before this proposal doc:

1. **`disciplines/testing.md`** — added `## Selector Hygiene — Anchor on data-testid` rule (first-class alongside the Persistence Rule). See `operations/sdlc_changelog.md` 2026-04-28 entry.
2. **`operations/sdlc_changelog.md`** — recorded the selector hygiene addition.

Everything below is a proposal, not yet written.

---

## Proposal A — Codify the CC2 self-verification ritual

**Status:** READY-TO-WRITE. Has been used identically across three handoffs (#1413, #1435, #1438), produces high-quality handbacks every time.

**Motivation:** CC2 fix-impl handoffs follow a 5-or-6-stage ritual: confirm RED on pre-fix, apply fix, confirm GREEN, stash JUST the product fix and confirm RED again (the load-bearing artifact), restore + GREEN, cleanup. The ritual has produced clean handback chronicles three pilots running. Today's work showed exactly why it matters: in #1438, stashing JUST `notificationService.ts` proved the test pinned D5 specifically and not adjacent surfaces.

**Evidence:**
- `~/src/apps/harmoniq/harmoniq-frontend/docs/00_CURRENT_WORK/stepwise_results/2026-04-27_cc2-handback_pilot-4c-fix.md` (Pilot 4c, multi-defect)
- Project #21 #1413 description + #1395/#1438 descriptions (avatar + unread, single-defect through multi-defect)
- Each pilot's handback chronicle includes RED/GREEN/RED/GREEN evidence in the same shape

**Recommended shape (post-decision):** new file at `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md` (NOT in `disciplines/testing.md` — this is a work pattern, not a universal SDLC rule). Note explicitly that it applies when work is split across two main agent sessions; teams running a single session don't need it. Codify as 5 stages with the explicit stash-revert step as the load-bearing artifact. Reference the handback-chronicle convention.

**Why now:** the ritual is what makes the validation cycle tractable for CC1. Without it, we'd have no way to independently confirm CC2's fixes pin the right defect.

**Decision (RAK 2026-04-28):** ship to a new `work-patterns/` directory; note the two-agent-split context.

---

## Proposal B — Codify the CC1 validation cycle

**Status:** READY-TO-WRITE. Pairs with Proposal A.

**Motivation:** When CC2 hands back, CC1 runs a four-step validation cycle: pull branch, run tests (GREEN), revert just the product fix (RED matches CC2's reported RED), restore (GREEN). This three-signal proof confirms (a) the fix works, (b) the test pins the bug, (c) reversion of the fix re-surfaces the bug. Three independent signals.

**Evidence:**
- #1413 description "CC1 validation cycle (post-handback)" section
- #1438 description "CC1 validation cycle (after handback)" section
- Today's session: the three-signal validation ran successfully on #1434, surfacing TST-CTU-02's selector miss in the process

**Recommended shape (post-decision):** companion section to Proposal A inside `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md`. Format: 4 numbered steps + the "three independent signals" framing.

**Why now:** without canonical statement, every pilot re-derives the cycle. With it, CC1 + CC2 share a contract that defines "done" objectively.

**Decision (RAK 2026-04-28):** ship to `work-patterns/` alongside Proposal A; same two-agent-split framing.

---

## Proposal C — Path A coverage-first pilot as a process template

**Status:** DEEPER-DESIGN. Direction is clear; placement + scope of statement needs design pass.

**Motivation:** Six pilots (4a, 4b, 4c, 4d, 1413, 1434) followed an identical shape: probe → tests RED on current code → file defect tickets in #21 → fix-impl in #52 → handoff/validation/merge. The shape is partially captured in `harmoniq_test_automation_project.md` memory, but isn't an SDLC process template. Teams that want to adopt the pattern have to read scattered Harmoniq-specific memory files instead of pulling a template.

**Evidence:**
- `harmoniq_test_automation_project.md` — captures the conventions but is keyed to Harmoniq tenancy
- The 6 pilots above all matched the shape with minor variations (single-defect vs multi-defect, sibling architectural classes)

**Recommended shape (post-decision):** new file at `~/src/ops/sdlc/work-patterns/path-a-pilot.md`. Sections: probe deliverables, ticket structure (Project #21 for defects + Project #52 for pilot/fix-impl tasks — generalize from Harmoniq to "bug ledger + test automation" projects), CC1↔CC2 handoff points (or single-session equivalent), conditional close-out branches (all-green vs some-red), retro trigger. Document explicitly how this works with one session OR with multiple, with limitations / advantages of each noted.

**Why this is DEEPER-DESIGN:** it's a multi-page template, requires design choices about how to phrase Harmoniq-specific structures (Project #21 / #52) in a project-agnostic way. Not a one-line addition.

**Decision (RAK 2026-04-28):** important — ship to `work-patterns/`. Document one-session vs multi-session variants side-by-side. Status promoted from DEEPER-DESIGN to QUEUED-PRIORITY.

---

## Proposal D — Multi-CC handoff protocol as an SDLC operation

**Status:** DEEPER-DESIGN. Currently scattered across `parallel_session_protocol.md` memory (richly captured but Harmoniq-specific in language).

**Motivation:** Two-CC parallel-session work is now the dominant mode for Harmoniq pilots. The protocol — role boundaries, runtime sharing, env-impact tagging, branch-collision gotchas, mid-flight handoff doc — is durable and cross-project useful. Currently lives in personal memory.

**Evidence:** `parallel_session_protocol.md` memory has ~250 lines of patterns. Today alone: branch-collision gotcha avoided via worktree, env-impact migration recovered correctly, role table prevented CC1 from accidentally invoking `fix-impl`.

**Recommended shape (post-decision):** new file at `~/src/ops/sdlc/work-patterns/parallel_sessions.md`. NOT under `sdlc/operations/` — these patterns work alongside any SDLC, not within ours specifically. Covers: role-can-invoke table (cross-project version), runtime sharing rules, env-impact tagging convention, mid-flight handoff doc shape, branch-collision avoidance.

**Why this is DEEPER-DESIGN:** the memory file mentions Harmoniq-specific tmux session names + Docker-compose paths. Generalizing requires a careful lift.

**Decision (RAK 2026-04-28):** ship to `work-patterns/`, NOT inside the SDLC framework. These are SDLC-agnostic.

---

## Proposal E — PM MD editor as optional process extension and review surface

**Status:** READY-TO-WRITE.

**Motivation:** During Rick + CC1 sessions, structured markdown documents (probes, handback chronicles, decision logs, briefings, coverage maps, ticket-update drafts) are the primary medium of structured review. Rick has been dogfooding `md-editor-mac` (`~/src/apps/md-editor-mac`) as the review surface for these documents. The pattern works well — CC writes, Rick reviews in md-editor, CC's edits show up live via the external-edit watcher. There's also a documented marker convention (`**Question:**` / `**Bug:**` / `**Decision:**` / `**Assumption:**`) for inline needs-attention spots, plus a `## Decision log` accumulator at end-of-doc.

This entire workflow is currently personal-memory-tier (`md_editor_dogfood_workflow.md`). It's a clean cross-project pattern that any agent-mediated SDLC team would benefit from.

**Evidence:**
- 6+ pilots reviewed via this surface
- The marker convention surfaces unresolved questions efficiently (greppable: `grep -rn '\*\*Question:' docs/`)
- Decision log accumulation prevents context loss across sessions
- Today's session opened ~6 docs via the CLI (probes, handbacks, prompts, results)

**Recommended shape (post-decision):** new section in `~/src/ops/sdlc/work-patterns/collaboration_model.md` (sub-category of work-patterns). Cover:
- Default surface for review docs is markdown on disk; any editor works
- **Recommended for PortableMind users:** dogfooded MD editor with external-edit watcher. Repo: https://github.com/rickkoloski/portablemind-md-editor-mac. Local: `~/src/apps/md-editor-mac`.
- **Alternative editors are fine** with potential limitations (no external-edit watcher live-refresh on agent writes; no integrated `:line` jump from CLI; etc.). The marker + decision-log conventions are the durable, editor-agnostic parts.
- Marker convention (`**Question:**` / `**Bug:**` / `**Decision:**` / `**Assumption:**`) — greppable, on its own line, self-contained
- Resolution pattern: keep `## Open questions` greppable anchor (mark `_(none)_` when empty); accumulate resolved items into `## Decision log` table at end of doc with columns `Date | Decision | Decided by`
- Surface CLI: `~/src/apps/md-editor-mac/scripts/md-editor /abs/path.md[:line[:col]]` — idempotent, refocuses existing tab

**Why now:** Rick explicitly asked for this. Teams reviewing our work want to see the surface. The conventions (markers + decision log) are project-agnostic and worth canonical placement even for teams that never touch our specific MD editor.

**Decision (RAK 2026-04-28):** ship to `work-patterns/collaboration_model.md` as a sub-category. Recommend the PortableMind editor for our users; note alternatives possible. Include the GitHub link.

---

## Proposal F — Decision log + Open questions convention

**Status:** READY-TO-WRITE. Subset of Proposal E but worth an independent statement since teams may adopt the convention without the editor.

**Motivation:** The `## Open questions` (greppable, mark `_(none)_` when empty) + `## Decision log` (accumulating table at end of doc) pattern keeps long-lived working docs honest about open vs settled state. Used in this pilot's probe doc, the avatar handoff doc, the cross-tenant unread probe.

**Recommended shape (post-decision):** `~/src/ops/sdlc/work-patterns/collaboration_model.md` addition (alongside Proposal E since it's the editor-independent half) titled `## Long-lived working docs — Open questions + Decision log`. Spec the structure + the resolution pattern (keep header even when empty; never delete questions, move them).

**Why pull out from Proposal E:** the convention is durable independent of the editor. A team writing in plain VS Code benefits from this just as much as a team using md-editor.

**Decision (RAK 2026-04-28):** ship to `work-patterns/collaboration_model.md` alongside Proposal E.

---

## Proposal G — Sibling fix-impl subtask pattern (multi-architectural-class pilots)

**Status:** READY-TO-WRITE. Refines an existing memory pattern.

**Motivation:** `harmoniq_test_automation_project.md` codifies the "one fix-impl subtask covers multiple bugs in the same architectural class" pattern (Pilot 4c's #1395 covered #1394 + #1396, both `IsPrivatable` defects). Today's pilot extended that: #1434 had FIVE defects across TWO architectural classes (backend tenant-scoping + frontend hook lifecycle race), so it got TWO sibling fix-impl subtasks (#1435 for D1-D4, #1438 for D5). Don't expand a closed subtask's scope; create a sibling.

**Evidence:**
- Pilot 4c's #1395 — single-class pattern (memory-codified)
- Today's #1435 + #1438 — multi-class extension (not yet codified)
- Both shapes have produced clean handback chronicles

**Recommended shape (post-decision):** add to `~/src/ops/sdlc/disciplines/testing.md` as a small section. Crisp rule: "When a pilot surfaces defects across two or more architectural classes, file sibling fix-impl subtasks under the same pilot parent — one per class. Don't expand a completed subtask's scope; that muddies its handback chronicle."

**Decision (RAK 2026-04-28):** ship to `disciplines/testing.md` (this one IS a testing discipline rule).

---

## Proposal H — Branch deviation override pattern

**Status:** MEMORY-TIER. Small but durable; better as a memory file than a discipline statement.

**Motivation:** When CC2 encounters a clear technical reason to deviate from the briefing's branching instructions (e.g., the briefing says "commit to backend branch" but the actual fix turns out to be frontend), they should override with documented justification — not silently follow, not escalate. Today this happened twice on #1438.

**Recommended shape:** small memory file at `~/.claude/projects/-Users-richardkoloski-src/memory/feedback_briefing_override_pattern.md` (or fold into `parallel_session_protocol.md`).

**Decision (RAK 2026-04-28):** approved as memory-tier write.

---

## Proposal I — Diagnostic instrumentation lifecycle

**Status:** MEMORY-TIER.

**Motivation:** Pattern: when a fix doesn't appear to work or a test fails for unclear reasons, add temporary diagnostic logging (NOT committed), capture the trace, paste the trace into the relevant bug ticket as repro evidence, revert the diagnostic before commit. Used in #1413 and #1437 (D5).

**Recommended shape (post-decision):** ship BOTH — memory file (or fold into `parallel_session_protocol.md` memory) AND a section in `~/src/ops/sdlc/work-patterns/parallel_sessions.md` (Proposal D's home).

**Decision (RAK 2026-04-28):** ship both — memory tier AND work-patterns/parallel_sessions.md.

---

## Proposal J — MCP categories awareness + restart-vs-reconnect

**Status:** MEMORY-TIER. Specific to current toolchain, may shift.

**Motivation:** Today we lost an hour because the harmoniq MCP URL had `?categories=data,memory,admin,communication` but no `conversation` — so `conversation_chat_tool` was never exposed, and we couldn't send Russell a DM via the proper API. Compounding factor: `/mcp reconnect` keeps the existing `mcp-remote` process running with the original URL args, so a full Claude Code restart is needed for category changes to surface.

**Recommended shape:** memory file at `~/.claude/projects/-Users-richardkoloski-src/memory/harmoniq_mcp_categories_and_messaging.md`. (Already on TODO from earlier in this session.) Capture the URL format, the category-to-tool mapping, the restart-vs-reconnect lesson, and the `general_crud_tool` LlmMessage.create fallback recipe.

**Decision (RAK 2026-04-28):** approved as memory-tier write.

---

## Post-decision execution shape

The dominant architectural shift Rick approved: introduce a new **`~/src/ops/sdlc/work-patterns/`** directory as a peer to `sdlc/`. Work patterns are SDLC-agnostic; they describe HOW we do work, not WHAT the SDLC requires.

**New structure:**

```
~/src/ops/
├── sdlc/                            # existing — what the SDLC requires
│   ├── disciplines/
│   │   └── testing.md               # + Selector Hygiene (shipped) + Sibling fix-impl rule (G)
│   ├── lifecycles/
│   ├── operations/
│   └── ...
└── work-patterns/                   # NEW — patterns that work alongside any SDLC
    ├── README.md                    # framing + scope
    ├── collaboration_model.md       # E + F (MD editor + Decision log convention)
    ├── multi-agent-handoff.md       # A + B (CC2 ritual + CC1 validation)
    ├── parallel_sessions.md         # D + I (parallel session protocol + diagnostic instrumentation)
    ├── path-a-pilot.md              # C (one-session and multi-session variants)
    └── harmoniq/                    # tenancy-specific patterns
        └── README.md                # explains why this subfolder exists
```

**Memory writes (separate from work-patterns):**
- `harmoniq_mcp_categories_and_messaging.md` (J)
- `feedback_briefing_override_pattern.md` (H)
- update `parallel_session_protocol.md` with diagnostic instrumentation lifecycle (I)

## Recommended ship order

**Tight cut (recommended for first batch):**
1. Create `work-patterns/` peer + `README.md` explaining the framing
2. `work-patterns/collaboration_model.md` (E + F) — durable conventions, immediate teach-ability
3. `work-patterns/multi-agent-handoff.md` (A + B) — most directly tied to today's work
4. Sibling fix-impl rule (G) added to `disciplines/testing.md`

That's roughly 4 file writes. Tightly scoped, all READY-TO-WRITE. Lays the foundation; subsequent batches expand.

**Follow-up batches (queued):**
- `work-patterns/path-a-pilot.md` (C) — pulls from harmoniq_test_automation_project memory; needs careful generalize pass for one-session/multi-session variants
- `work-patterns/parallel_sessions.md` (D) — pulls from parallel_session_protocol memory + folds in diagnostic instrumentation lifecycle (I)
- Memory writes (H, J, I-memory) — small, fold in opportunistically

**Minimum (if budget tight):** just step 1 (work-patterns scaffold + README). Establishes the home; everything else can land incrementally over coming sessions.

---

## Open questions

_(none — all resolved per RAK annotations 2026-04-28; decisions captured per-proposal above + in Decision log below)_

---

## Decision log

| Date | Decision | Decided by |
|---|---|---|
| 2026-04-28 | Selector Hygiene shipped to `disciplines/testing.md` (already landed pre-review). | RAK |
| 2026-04-28 | New peer directory `~/src/ops/sdlc/work-patterns/` introduced for SDLC-agnostic patterns. Distinct from `~/src/ops/sdlc/`. | RAK |
| 2026-04-28 | Proposal A (CC2 self-verification ritual) ships to `work-patterns/multi-agent-handoff.md`, NOT `disciplines/testing.md`. Note the two-agent-split context. | RAK |
| 2026-04-28 | Proposal B (CC1 validation cycle) ships alongside A in `work-patterns/multi-agent-handoff.md`. | RAK |
| 2026-04-28 | Proposal C (Path A pilot template) promoted from DEEPER-DESIGN to QUEUED-PRIORITY. Ships to `work-patterns/path-a-pilot.md`. Document one-session AND multi-session variants side-by-side with limitations/advantages. | RAK |
| 2026-04-28 | Proposal D (Multi-CC handoff protocol) ships to `work-patterns/parallel_sessions.md`, NOT inside SDLC. These are SDLC-agnostic. | RAK |
| 2026-04-28 | Proposal E (MD editor + conventions) ships to `work-patterns/collaboration_model.md`. Recommend the PortableMind editor for our users (link: https://github.com/rickkoloski/portablemind-md-editor-mac); note alternative editors usable with potential limitations. | RAK |
| 2026-04-28 | Proposal F (Decision log + Open questions convention) ships alongside E in `work-patterns/collaboration_model.md`. Editor-independent. | RAK |
| 2026-04-28 | Proposal G (Sibling fix-impl subtask pattern) ships to `disciplines/testing.md`. This one IS a testing discipline rule. | RAK |
| 2026-04-28 | Proposal H (Branch deviation override) approved as memory-tier write. | RAK |
| 2026-04-28 | Proposal I (Diagnostic instrumentation lifecycle) ships BOTH to memory (`parallel_session_protocol.md`) AND to `work-patterns/parallel_sessions.md`. | RAK |
| 2026-04-28 | Proposal J (MCP categories + restart-vs-reconnect) approved as memory-tier write. | RAK |
| 2026-04-28 | `work-patterns/harmoniq/` subfolder pattern introduced — Harmoniq-specific patterns nested with their own README explaining why the subfolder exists. | RAK |
