# Parallel Sessions — Operating Two Main Agent Sessions Concurrently

A pattern for **running two main agent sessions on the same project simultaneously**. Distinct from `multi-agent-handoff.md`, which describes the handback shape between sessions; this doc describes the shared-state and coordination rules that make running them at the same time tractable.

## When this pattern applies

This pattern assumes:
- Two main agent sessions (call them A and B), often a "testing/scaffold" session and a "fix-impl" session
- A shared development runtime (services, database, file system, git working trees)
- Concurrent work that may overlap in unexpected ways (auth state, cache, schema)

**Single-session teams don't need most of this**; some elements (env-impact tagging, mid-flight handoff doc) are still useful for restart-recovery or interrupted work.

**Three-or-more-session teams** can extend the role table and shared-runtime rules; the principles don't change but the coordination overhead grows non-linearly. Pause to reconsider whether the work needs to be that parallel.

---

## Role-can-invoke table

When two sessions are active, who can spawn which sub-agent role? The default is "share the workload to keep cognitive load down per session." But role drift creates problems:

- A session that invokes a role outside its scope risks duplicating or conflicting with the other session's work.
- A session that invokes a role it lacks the context for often does worse work than the session that should own it.

**Cross-project default table:**

| Role | Session A (testing/scaffold) | Session B (fix-impl) |
|---|---|---|
| `plan-owner` (scope, briefings) | ✓ | — |
| `test-analyst` (test spec authoring) | ✓ | — |
| `backend-verifier` (read-only audit) | ✓ | — |
| `ui-auditor` (read-only UI map / testid coverage audit) | ✓ | — |
| `spec-impl` (write + execute regression tests) | ✓ | — |
| `seed-impl` (extend test fixtures / seed data) | ✓ | — |
| `testability-impl` — *trivial* (data-testids, small pass-through props) | ✓ (solo) or default to B if both active | ✓ |
| `testability-impl` — *non-trivial* (new components, logic changes) | — | ✓ |
| `fix-impl` (any product bug fix, backend or frontend feature code) | — | ✓ |
| `human-reviewer` | (the human, not an agent) | (the human) |

**The hard rule:** Session A must NEVER invoke `fix-impl`. Even when it would be faster to "just make the small change here," routing to Session B preserves audit boundaries and prevents accidental scope conflicts.

The "default to B if both active" pattern for trivial testability work is a *consistency rule, not a capability rule*. Either session is capable of adding a `data-testid` attribute. Routing trivial product edits consistently to Session B avoids "who does this one?" decision churn.

### Self-check before either session spawns an agent

Ask: does this agent's output land in **product code** (backend feature code, non-trivial component changes, bug fixes) or **test infrastructure** (specs, plans, seeds, fixtures, docs, trivial testids)?
- Product code → Session B's domain. Session A files/updates a ticket and routes.
- Test infrastructure → either session, with the consistency-default applied.

---

## Runtime sharing — share by default, escalate narrowly

**Default: both sessions share the running development runtime.** Whatever services / containers / dev-server processes are up — both sessions use them directly. No need to spin up duplicates.

### When sharing works well (the typical case)

- **Frontend code edits.** Hot-module-reload handles refresh; both sessions see updates as they save.
- **Backend fixes verified via unit tests.** Tests run against a separate test environment (in-process or test database), isolated from whatever dev data the other session is touching.
- **Read-only data inspection.** Two sessions reading the same dev database is fine.

### When to stand up a separate runtime

A separate runtime (different compose project name, different ports, different volumes) is warranted for:
- **Schema migrations** that would change the dev database the other session is using.
- **Work that will likely crash the runtime repeatedly.** Don't make the other session pay for your debug cycles.
- **Seed/data resets** that would destroy fixtures the other session depends on.
- **Long-running debugger sessions** that attach to the main runtime PID.

The pattern is "share until you're going to disrupt; then split." Splitting too eagerly creates state-divergence headaches; splitting too late creates step-on-toes incidents.

---

## Env-impact tagging

When one session is about to do something that disrupts the shared runtime (migration, seed reset, service restart, dependency upgrade), tag the relevant ticket or comment with an explicit env-impact label so the other session can pause and react:

| Tag | Meaning |
|---|---|
| `env-impact: none` | Safe to apply without coordination. Most code edits. |
| `env-impact: restart-required` | The runtime needs a restart; brief pause for in-flight test runs. |
| `env-impact: migration` | Schema is changing; other session should pull + migrate before next test run. |
| `env-impact: seed-reset` | Dev fixture data is being rebuilt; active test fixtures will vanish. |

The other session watches for these tags and sequences accordingly.

### A specific kind of env-impact: shared source trees

When both sessions share a working tree for a backend service that's bind-mounted into the running runtime, **a `git pull` on that tree is itself an env-impact event**. The runtime's autoreload picks up code changes immediately, but does NOT run schema migrations or restart long-lived services. Pulling a branch that includes migrations + then running the runtime against the old schema produces runtime crashes (`UnknownAttributeError`, missing columns, etc.).

**Protocol before pulling on a shared backend tree:**
1. Check the gap (`git log local..origin -- 'db/migrate/**'`) for migrations.
2. Announce env-impact on the relevant ticket.
3. Stop in-flight test runs.
4. Pull. Run migrations immediately. Restart the runtime (or relevant services).
5. Verify schema matches expectations before unblocking the other session.

**A real failure** that motivated this rule: a session pulled a backend branch that included a migration. The runtime's autoreload picked up the new model code referencing the new column. The other session's next test attempted to authenticate; auth went through the model layer; the model layer expected the new column to exist; the column didn't (migration hadn't run). Auth API returned 500. Both sessions wasted ~30 minutes diagnosing what looked like an auth bug before the env-impact root cause was found.

---

## Branch-collision avoidance — use worktrees

Two sessions sharing the same git working tree share a single checked-out branch at any moment. If session A is mid-edit on `feature/A` and session B does `git checkout feature/B` in the same tree, session A's working tree silently flips. Uncommitted edits get carried over, conflict, or get lost.

**The rule:** when a session needs to be on a different branch from the other session, **use `git worktree add`**, not `git checkout`. A worktree gives that session an independent working directory for the branch; both sessions edit freely without stepping on each other.

```
# Wrong:
git checkout feature/B   # disrupts the other session's working tree

# Right:
git worktree add ../<repo>-<branch> feature/B
# session B now operates from ../<repo>-<branch>
```

When the worktree's work is done and merged, `git worktree remove` cleans it up.

**Spawned-agent caveat:** when one session spawns a delegated sub-agent without isolation, the sub-agent shares the spawning session's working tree. If the sub-agent creates a feature branch and `git checkout`s it, the spawning session's CWD silently follows the sub-agent's branch. To avoid this, pass an isolation flag to the sub-agent (or have the sub-agent operate in a worktree it created itself).

---

## Mid-flight handoff doc

When a session expects to be interrupted (timeout, restart, context limit, end-of-day) while another session has work in flight, write a **mid-flight handoff doc** before the interruption. The doc captures enough state that a fresh session can resume without re-deriving context.

**When to use proactively:** at ~25 minutes into any agent task expected to run >30 minutes, write the doc even if everything is going fine. It's cheaper-than-insurance: if the agent finishes cleanly, throw the doc away.

**Required sections:**
- Branch state on every active repo (with current HEAD SHAs)
- Uncommitted files list (with critical "do not reset these" notes)
- What's NOT done yet — specific deliverables / sections still pending
- Recovery options laid out as A/B/C with cost analysis (resume existing, fall back to inline, fresh restart with re-brief)
- Key files to read on resume — pointers to briefings, prior results, active specs
- Things-likely-to-have-drifted — what the other session might have changed in the interim

**Ownership:** the session expecting interruption writes the doc. The post-restart fresh session is the consumer.

This pattern is distinct from the [self-verification ritual](multi-agent-handoff.md) — that's between two roles (testing/fix-impl) doing different work; this is between the same role re-entering after a pause.

---

## Diagnostic instrumentation lifecycle

When a fix doesn't appear to work, or a test fails for unclear reasons, agents often need to add temporary diagnostic logging to surface the trace. The instrumentation lifecycle protects against shipping debug-only code:

1. **Add the diagnostic.** Console logs, fiber-walk probes, request/response capture, whatever is needed to get the trace. Make it noisy and explicit (tag with a clearly-greppable prefix like `[DEBUG-XYZ]`).
2. **Reproduce the failure.** Capture the trace.
3. **Paste the trace into the relevant ticket** as repro evidence. The ticket — not the source code — is where the trace lives going forward.
4. **Revert the diagnostic before commit.** The fix branch must be clean of debug-only instrumentation.
5. **The diagnostic is gone, but the trace is durable.** Future readers of the ticket can see exactly what the agent observed.

The discipline is critical because debug logging that ships accidentally:
- Pollutes production logs (cost + signal-to-noise)
- Creates expectations about log shapes that other code starts depending on
- Implies to the next reader that the diagnostic was load-bearing when it wasn't

**Real-world example (Pilot #1434, 2026-04-28):** during D5 investigation, an agent added a `[UNREAD-HOOK] 🪝 subscribed at` log to a React hook to determine whether subscription was firing. The trace showed the log NEVER appeared post-mount, which proved the hook was bailing before reaching `subscribeToUnreadUpdates`. The trace went into the D5 bug ticket; the diagnostic was reverted before the testid + selector fix shipped. Three minutes of cleanup saved future readers from wondering why a hook had a spurious console log.

---

## Cross-references

- `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md` — the verification ritual + validation cycle that govern hand-offs between sessions
- `~/src/ops/sdlc/work-patterns/path-a-pilot.md` *(queued)* — a coverage-first pilot template that uses these parallel-session conventions
- `~/src/ops/sdlc/work-patterns/collaboration_model.md` — markdown review surface + working-doc conventions used for handoff docs and tickets
