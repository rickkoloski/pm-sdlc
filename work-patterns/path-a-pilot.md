# Path A Pilot — Coverage-First Defect-Hunt Template

A template for **deliberately hunting defects on a specific surface by writing the regression tests FIRST against current code**, watching them go RED, filing the surfaced defects, fixing them, and closing with the tests as the durable regression anchors.

The pattern is "Path A" because the alternative ("Path B" — fix the user-reported bug, write a test alongside) trades coverage for speed. Path A trades speed for systematic coverage of a surface that's accumulating risk.

## When this pattern applies

Path A pilots are valuable when:

- A specific surface has accumulated multiple suspected defects, often architecturally related (a shared SQL pattern, a shared component lifecycle, a shared concern).
- Coverage of that surface is thin — most failures would slip through existing tests.
- The cost of investigating each defect individually is higher than the cost of writing a coordinated test cluster up front.

Path A pilots are NOT valuable when:

- A user has reported a single specific defect with a clear repro (just go fix it; Path B).
- The surface is too unstable for tests to be durable (architectural redesign in flight).
- The team can't afford the upfront probe + test-authoring time.

## The shape

```
   [scaffold]            [tests RED]            [defects filed]
        │                     │                       │
        ▼                     ▼                       ▼
   probe surface  →   write tests against   →   for each RED test,
   (architectural     current code, expect      file a defect ticket
   probe + scope)     RED on broken paths       (or sibling cluster)
        │                     │                       │
        ▼                     ▼                       ▼
                                                [fix-impl]
                                                      │
                                                      ▼
                                                fix defects, tests
                                                flip RED → GREEN
                                                      │
                                                      ▼
                                                [validation cycle]
                                                      │
                                                      ▼
                                                three-signal verify,
                                                merge, close pilot
```

Each phase has deliverables. The pilot closes when the regression tests are GREEN on `develop`-equivalent and the defect tickets are all marked complete.

---

## Phase 1: Scaffold

The scaffold phase establishes the pilot's premise and produces the artifacts the test-authoring phase needs.

### Architectural probe (~15-30 min)

Before authoring tests, write a short probe document covering the surface under test. The probe answers three questions:

1. **What does the system actually do on this surface?** (Not "what should it do." Not "what does the spec say." What does the running code do.) Grep every callsite of the relevant components; inspect the behavior end-to-end.
2. **Is there asymmetry across callsites?** (Different callsites of the same component sometimes hit different downstream paths. That asymmetry is often where defects hide.)
3. **Does the expected architectural pattern actually hold?** (Common case: a concern is enforced in some places and forgotten in others.)

The probe either:
- **Confirms a straightforward plan** — proceed with normal test-authoring scope.
- **Surfaces unknowns that change the pilot scope** — those findings MUST land in the pilot scaffold doc BEFORE test-authoring, because they determine what to test and where the live-defect hypothesis lives.

When the probe surfaces a likely defect without any tests running: still go through the full pilot. The probe is a *hypothesis generator*, not a diagnostic. Let the tests confirm and pin the defect precisely; don't let the probe's inference short-circuit the process.

### Scope + ticket structure

The pilot lives in two ledgers:
- **Bug ledger** (a tracker for defects observed in the project) — each defect that surfaces becomes a ticket here. Cross-project term; tenancy-specific overlays in `harmoniq/` describe how PortableMind teams structure this as Project #21.
- **Test-pipeline project** (a tracker for test-authoring work units) — the pilot itself becomes a parent task here, with subtasks for each phase. PortableMind teams use Project #52.

**The pilot parent task** carries the scaffold-level metadata: scope, conditional close-out branches (all-green vs some-red), retro trigger, hard rules of engagement.

**Subtasks under the pilot parent** mark handoff points:

| Subtask | Owner | Purpose |
|---|---|---|
| Scaffold | Testing session | Probe + plan + briefings + coverage map |
| Seed-impl (optional) | Testing session, optionally delegated | Extend test fixtures / seed data for the cluster |
| Spec-impl | Testing session, optionally delegated | Write the regression test cluster |
| Fix-impl | Fix-impl session (one OR multiple, see below) | Fix defects surfaced |
| Human review + close | Human reviewer | Approve merges, close tickets |

For multi-class defect pilots (defects span multiple architectural classes), multiple **sibling fix-impl subtasks** under the same pilot parent — one per class. See `~/src/ops/sdlc/disciplines/testing.md` § Sibling Fix-Impl Subtask Pattern.

---

## Phase 2: Tests RED on current code

The test-authoring phase writes the regression cluster against current code, watching tests go RED on the defective paths and GREEN on the working paths. RED tests are the surfacing mechanism.

### Test cluster shape

A pilot's test cluster typically covers a specific surface end-to-end:
- 5-15 tests is a typical cluster size. Larger clusters fragment focus.
- Each test pins a specific behavior. Names tag the cluster (e.g., `TST-XX-NN`) so they're greppable across docs and tickets.
- The cluster mixes RED-expected (the defect-hunt) and GREEN-expected (regression anchors for currently-working behavior).

### Authoring discipline

- **Test current code as-is.** Don't change implementation while authoring. Path A is about testing what's there.
- **No data-testid additions in this phase** — flag testability gaps as observations to be addressed before fix-impl. A separate testability-impl pass adds the testids the cluster needs. (See [Selector Hygiene](../sdlc/disciplines/testing.md) for why testid anchoring matters.)
- **Probe re-verification before authoring.** If the probe was written more than a day or two before test-authoring starts, re-verify the architectural claims still hold against current code. Codebases move; assumptions stale.

### When all tests pass (no defects surface)

Sometimes the probe's hypothesis is wrong — the surface is healthier than expected. **All-green is a valid pilot outcome.** The pilot still produces:
- Regression anchors for the surface (durable coverage)
- A coverage map refresh
- Possibly a testability-impl follow-up (add testids surfaced as gaps during authoring)

Don't treat a no-defect pilot as wasted work. It's the surface earning a clean bill.

---

## Phase 3: File defects

For each RED test, file a defect ticket in the bug ledger.

### Filing convention

- **Each defect gets its own ticket.** Don't bundle. Even when multiple defects share an architectural class, each gets a distinct repro recipe, a distinct affected-code citation, and a distinct fix shape.
- **Cross-link siblings.** Each ticket has a "Sibling defects" section listing related ones with one-line distinctions.
- **Tag the test that pins it.** The originating test's name (`regression-coverage: TST-XX-NN`) goes in the ticket; the ticket ID goes in the test's docstring.

### When multiple defects share an architectural class

A single fix-impl subtask under the pilot parent can cover multiple defects if they share an architectural class (same module surface, same fix shape). Each defect ticket stays distinct (so the bug ledger is honest), but the fix-impl subtask references all of them as the unified scope.

### When defects span multiple architectural classes

File **sibling fix-impl subtasks** under the same pilot parent — one per class. See `~/src/ops/sdlc/disciplines/testing.md` § Sibling Fix-Impl Subtask Pattern for the full rule.

---

## Phase 4: Fix-impl

The fix-impl phase applies the product fixes and produces handback evidence. The shape depends on whether the pilot is running with one session or multiple.

### Variant A: Single-session pilot

A solo agent (or solo human) handles probe, test-authoring, and fix-impl in one session.

**Advantages:**
- Lower coordination overhead. No handback chronicle to author.
- Faster iteration when defects are small.
- Single context across the pilot — no re-derivation when moving from test to fix.

**Limitations:**
- No independent observer. The same agent that wrote the test is the one verifying the fix passes it. Confirmation bias risk.
- The verification ritual (especially the stash-revert-restore loop) is harder to enforce on yourself.
- Ad-hoc test-design errors (a selector miss, an assertion too lenient) are harder to catch without a second set of eyes.

**Recommended discipline for single-session pilots:**
- **Capture RED-on-pre-fix evidence in the commit message or chronicle.** Even without a separate validation cycle, the GREEN→RED-via-revert→GREEN three-signal check still applies. Don't skip it just because there's no other session asking for it.
- **Take a hard break between writing the test and applying the fix.** Walk away from the test. Come back. Read the assertion fresh. This is a poor substitute for an independent reviewer, but it's better than nothing.
- **Surface the test-design for review** before fix-impl if possible — even an informal Slack or Teams scan from a colleague catches selector misses and lenient assertions cheaply.

### Variant B: Multi-session pilot (testing + fix-impl)

A testing session writes tests + files defects; a separate fix-impl session ships fixes.

**Advantages:**
- Independent verification. The testing session validates the fix-impl session's claims via the [validation cycle](multi-agent-handoff.md#part-2-validation-cycle-testing-session).
- Three independent signals (GREEN → RED-on-revert → GREEN-restored), captured by two different observers. Strong confidence in the result.
- Test-design errors surface during fix-impl (the fix-impl session sees the test fail for the wrong reason, flags it back).

**Limitations:**
- Higher coordination overhead. Handback chronicle, validation cycle, ticket routing, branch management.
- Slower for trivial defects. The handback overhead can exceed the fix itself.
- Requires both sessions to maintain context; mid-flight context loss in either session creates expensive recovery.

**The handback shape for multi-session pilots is documented in `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md`** (self-verification ritual + validation cycle). That doc IS the multi-session fix-impl protocol; this template just routes you to it.

### Choosing a variant

| Situation | Variant |
|---|---|
| Solo agent, no second session available | A |
| Trivial single-defect fix | A (overhead of B exceeds the fix) |
| Multi-defect pilot with architectural-class fan-out | B (coordination overhead amortized across multiple defects) |
| Defects in active-dev territory (someone else is mid-feature on this surface) | B, with B routing to whoever owns the active-dev surface |
| Time-critical fix where confirmation bias risk is acceptable | A |
| First time exercising a complex surface end-to-end | B (the validation cycle catches first-time-pattern errors) |

There's no universal answer. Pick by the specific situation; document the choice in the pilot parent's scope.

---

## Phase 5: Validation + close-out

### Conditional close-out branches

Path A pilots have two close-out paths depending on what the test cluster reveals:

- **All-green path** (no defects surfaced): the pilot's regression cluster + coverage-map refresh is the deliverable. Optionally surface testability-impl follow-ups as separate small tasks. Close cleanly.

- **Some-red path** (defects surfaced + fixed): the regression cluster + the defect tickets + the validation-cycle three-signal proof. The pilot closes when:
  - Every defect ticket is marked complete (status flipped after merge, not before)
  - All test cluster tests are GREEN on the integration branch
  - The pilot parent task is marked complete with a brief summary linking the fix branches and validation evidence

### Retro trigger

A retro is warranted when:
- The pilot surfaced ≥2 defects (the defect-density justifies the look-back)
- A defect class was novel (worth memorializing for future pilots on similar surfaces)
- A process variation was attempted (worth assessing if it should become canonical)

A retro is NOT needed for:
- Routine pilots that closed cleanly in expected shape
- Single-defect pilots where the fix and validation went without incident

When in doubt, skip. Retros earn their place when they produce learnings that compound across future pilots.

---

## Cross-references

- `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md` — the multi-session protocol used in Variant B
- `~/src/ops/sdlc/work-patterns/parallel_sessions.md` — coordination rules when both sessions run concurrently
- `~/src/ops/sdlc/disciplines/testing.md` — Persistence Rule, Selector Hygiene, Sibling Fix-Impl Subtask Pattern (all referenced in flight)
- `~/src/ops/sdlc/work-patterns/harmoniq/` — tenancy-specific bindings (when they land): bug-ledger structure (Project #21), test-pipeline-project structure (Project #52), party-id disambiguation
