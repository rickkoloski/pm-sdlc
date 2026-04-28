# Multi-Agent Handoff — Self-Verification Ritual + Validation Cycle

A pattern for **work split between two main agent sessions**: one session investigates and authors regression tests (the "testing/scaffold session"); the other session implements fixes (the "fix-impl session"). The pattern is two ritualized loops:

1. **Self-verification ritual** — what the fix-impl session does before handing back, to produce a clean, evidence-rich handoff.
2. **Validation cycle** — what the testing session does after handback, to independently confirm the fix is real.

Together they produce **three independent signals** that a fix actually closes the defect: a GREEN post-fix run (claimed by the fix-impl session), a RED-on-pre-fix run (the load-bearing artifact — proves the test pins the defect), and a GREEN-after-restore run (rules out test-state leakage).

## When this pattern applies

This pattern assumes:
- Two agent sessions running concurrently or in series
- A defect ticket the fix-impl session can read (with reproduction recipe + fix shape)
- A regression test that's RED on pre-fix code
- The testing session can pull the fix-impl session's branch independently

**Single-session teams don't need this pattern.** A single session that fixes its own bug should still capture RED→GREEN evidence in the commit message or chronicle — but the elaborate stash-revert-restore loop is overkill when there's no inter-session handoff to verify across.

**Three-or-more-session teams** can adapt the pattern (e.g., a separate code-reviewer session in addition to fix-impl + validation), but the load-bearing artifact (RED-on-pre-fix proof) doesn't fundamentally change.

---

## Part 1: Self-Verification Ritual (fix-impl session)

The fix-impl session walks through this ritual after applying the product fix and BEFORE writing the handback. The output is a chronicle the testing session can audit and replay.

### Stage 1 — Confirm the bug reproduces on pre-fix code

This proves you understand the repro before you change anything. Don't skip even if you trust the ticket.

1. Make sure you're on the base branch (or your feature branch's base, before any fix commits).
2. Reset any state the test depends on (database fixtures, cache state, in-memory counters).
3. Run the failing regression test. **Confirm RED.** Capture the full reporter output.

If the test passes here, STOP — the bug doesn't reproduce on your stack and something is different from what the ticket described. Don't proceed; flag back to the testing session.

### Stage 2 — Apply the fix and confirm it heals the symptom

4. Apply the product fix (single commit, focused scope per the ticket's fix-shape recommendation).
5. **Manually verify the user-visible symptom is gone.** Open the app, exercise the broken flow, observe the heal. Don't rely on the test alone for this — it's possible to write a test that passes for the wrong reason.

### Stage 3 — Confirm the regression test goes GREEN

6. Re-run the regression test. **Confirm GREEN.** Capture full reporter output.

### Stage 4 — Revert just the product fix; confirm tests go RED again

This is the **load-bearing step**. A test that doesn't fail on broken code isn't pinning the defect.

7. Stash only the product-fix files (not the test file, not test fixtures, not unrelated infrastructure):
   ```
   git stash push --keep-index -- <product-fix-files>
   ```
8. Re-run the test. **Confirm RED.** Capture output. The failure should match (in shape and assertion) the RED you captured in Stage 1.

If Stage 4 shows GREEN unexpectedly, the test isn't pinning THIS defect — fix the test before continuing. (Common pitfall: assertion is too lenient, e.g., it asserts "an element exists" when it should assert "a specific element with specific content exists." Tighten the assertion.)

9. `git stash pop` to restore the fix.
10. Re-run the test. **Confirm GREEN again.** Rules out state leakage between runs.

### Stage 5 — Cleanup state for the next session

11. Reset any test-fixture state your fix-impl run dirtied (database seeds, file system state, etc.). The testing session is going to repeat the cycle; they need a clean baseline.
12. Push your branch. Write the handback chronicle (see below).

### Optional Stage 5+ — Extended verification

Some defects benefit from additional pinning beyond the primary regression test. Examples from real pilots:

- **RSpec contract test** for backend defects whose primary regression test is end-to-end Playwright. The end-to-end test catches the user-visible symptom; the contract test pins the SQL or service-layer defect specifically and runs in milliseconds.
- **Adjacent-route smoke check.** When a fix touches infrastructure used across many routes (auth, navigation, common state), exercise an unrelated route after the fix to confirm no regression elsewhere.

When in doubt, ship what you have. Defer extended verification to the testing session's discretion.

### Handback chronicle

The chronicle is the artifact the testing session reads to start their validation cycle. It belongs as a comment on the parent ticket (or a Markdown file alongside the branch). It must contain:

- **Branch + final commit SHA** (the fix-impl session's pushed branch)
- **GREEN test output** from Stage 3
- **RED test output** from Stage 4 — *the load-bearing artifact*. Without this, the testing session has nothing to replay.
- **Files changed list**
- **Drive-by fixes** the agent shipped beyond the prescribed scope (with brief justification each)
- **Cleanup state** — what the next session inherits (database state, leftover test fixtures, etc.)

If the agent deviated from the ticket's fix-shape recommendation, the chronicle states the deviation up front and explains the reasoning. The testing session can then validate against the actual shape, not the prescribed one.

---

## Part 2: Validation Cycle (testing session)

When the fix-impl session hands back, the testing session runs a 4-step validation cycle. Same shape as Stages 3–4 of the self-verification ritual, but with an **independent observer** running it. That independence is what makes the validation meaningful — the fix-impl session may have unconsciously biased their own run.

### The four steps

1. **Pull the fix-impl branch.** Run the regression test. **Confirm GREEN.**
2. **Revert just the product fix** (keep the test, keep any new test infrastructure). Re-run the test. **Confirm RED matches the fix-impl session's reported RED** (same assertion, same failure shape).
3. **Restore the fix.** Re-run the test. **Confirm GREEN.**
4. **Approve the merge** to the integration branch.

If step 2's RED doesn't match the fix-impl session's reported RED — either it goes GREEN unexpectedly, or it goes RED for a different reason — the test isn't pinning the same defect the fix-impl session thinks it is. Flag back; don't merge.

### What the three independent signals prove

| Signal | What it proves |
|---|---|
| Step 1 GREEN | The fix works under fresh-pull conditions. Rules out fix-impl-session-only state. |
| Step 2 RED | The test pins THIS defect (not an adjacent one). Rules out tests passing for the wrong reason. |
| Step 3 GREEN | No state leaked between runs. Rules out stash-pop quirks. |

The cycle takes minutes. The protection it offers compounds — every fix shipped this way is independently audited.

### When step 2 surfaces something unexpected

Sometimes the validation cycle surfaces information the fix-impl session missed: a different defect class, a test selector that's anchored on something fragile, a state-derivation gap. When that happens:

- If it's the same defect class as the original ticket, fold the finding into a comment on that ticket and route back through fix-impl.
- If it's a different defect class entirely, file a sibling ticket. (See `~/src/ops/sdlc/disciplines/testing.md` § "Sibling fix-impl subtask pattern" for when sibling tickets share a parent vs stand alone.)
- If it's a test-design issue (selector miss, assertion too lenient), the testing session fixes it themselves — that's their domain, not the fix-impl session's.

---

## Cross-references

- `~/src/ops/sdlc/disciplines/testing.md` § Mutation Verification — The Persistence Rule (orthogonal but related; both are first-class verification rules)
- `~/src/ops/sdlc/disciplines/testing.md` § Selector Hygiene (paired anchoring rule)
- `~/src/ops/sdlc/work-patterns/parallel_sessions.md` *(queued)* — broader two-session protocol (this doc is the verification subset)
- `~/src/ops/sdlc/work-patterns/path-a-pilot.md` *(queued)* — coverage-first pilot template that uses this handoff pattern
- `~/src/ops/sdlc/work-patterns/collaboration_model.md` § Long-lived working-doc conventions — handback chronicles use these conventions

## Appendix: Example chronicle template

```markdown
# CC2 → CC1 handback — <ticket-id> <one-line-title>

**Branch:** `feature/<name>` @ `<final-sha>` (pushed)
**Files changed:** <list>

## Stage 3 — GREEN on post-fix

```
<full reporter output, GREEN>
```

## Stage 4 — RED on pre-fix (load-bearing)

```
<full reporter output, RED, with assertion message + test line number>
```

## Stage 4 final — Restore + GREEN

```
<full reporter output, GREEN — rules out state leakage>
```

## Drive-by fixes
<list, or "None — held scope tight">

## State left behind
<what the next session inherits>

## Open observations
<anything that surfaced but is out of scope; flagged for testing session's discretion>
```
