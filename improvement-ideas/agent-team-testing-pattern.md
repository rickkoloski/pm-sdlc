# Agent Team Testing Pattern — Learnings from Chat Testing Sprint

**Date**: 2026-02-24
**Context**: First structured experiment using an agent team (tester + analyst + coder) to investigate and fix unread badge bugs in Harmoniq Team Chat.

## The Pattern: Parallel Analysis + Testing + Coding

### Team Structure
- **Lead** (human + coordinating agent): Directs work, synthesizes findings, makes scope decisions
- **Tester**: Runs Playwright multi-user scenarios against DEV, documents symptoms
- **Analyst**: Reads source code, traces execution paths, proposes root causes
- **Coder**: Implements targeted fixes on feature branches

### What Worked

**1. Separation of concerns prevents tunnel vision**
The analyst found root causes in code while the tester independently found symptoms in the running app. Neither polluted the other's perspective. The analyst identified `clearAllUnreadSubscriptions()` as the primary race condition; the tester found a completely separate "Failed to load messages" issue (circuit breaker) that the analyst then traced in minutes.

**2. Parallel execution saves wall-clock time**
Analyst and tester ran simultaneously. By the time the analyst delivered a full code path analysis (~15 min), the tester had desktop scenarios built and running. The coder then implemented fixes while the tester continued with mobile scenarios.

**3. The lead stays available for design decisions**
With agents handling the detailed work, the lead can discuss scope, prioritization, and architecture with the human. Key decision made mid-sprint: keep the fix branch focused on the primary bug, track circuit breaker/AbortController findings as follow-ups. This wouldn't happen naturally if one agent was doing everything.

**4. Tester findings feed analyst, and vice versa**
The tester found "badges work but messages fail" — the analyst traced this to the circuit breaker pattern in minutes because it already had the code context loaded. This feedback loop is faster than a single agent switching between browser and code.

**5. Before/after validation pattern**
Tester runs baseline (pre-fix) → coder implements fix → tester re-runs (post-fix). Same scenarios, same assertions, clear signal on whether the fix worked.

### What Needs Improvement

**1. Playwright test load causes false failures**
Running 4 heavy multi-user tests sequentially overwhelmed the DEV backend (rate limiting, circuit breaker tripping). Need to:
- Add delays between tests or reduce concurrent API calls
- Distinguish infrastructure failures from app bugs in test results
- Consider test-specific rate limit configuration on DEV

**2. Mobile results are the critical validation but run last**
The primary fix (dual-hook race) only manifests on mobile where both AppShell and MobileLayout mount the same hook. Desktop tests ran first and showed badges "mostly working" — which is expected since desktop doesn't trigger the race. Lesson: prioritize test scenarios by where the bug is most likely to manifest.

**3. Agent coordination overhead**
Sending status checks, relaying findings between agents, and translating results for the human takes lead attention. For a 3-agent team this is manageable. At larger scale, agents would need more structured hand-off protocols (e.g., shared findings document rather than messages).

**4. Deploy gating needs a clearer workflow**
The "baseline → merge → deploy → re-test" sequence works but requires manual coordination. Could be formalized: tester signals "baseline complete" → triggers merge/deploy → triggers re-test automatically.

## Findings That Informed SDLC Process

### Testing Discipline
- Multi-user Playwright scenarios with two browser contexts are the right tool for real-time features (WebSocket, presence, notifications)
- Smoke tests should establish a healthy baseline before feature-specific scenarios
- Test scenarios should be ordered by bug-likelihood (mobile before desktop if the bug is mobile-specific)

### Code Analysis Discipline
- Tracing full execution paths (WebSocket → service → hook → component → render) catches issues that point-debugging misses
- The "what does the manual workaround do that the automatic path doesn't" question is a powerful diagnostic heuristic
- Singletons and shared hooks that assume single-consumer usage are a recurring source of bugs

### Fix/Deploy Discipline
- One concern per branch — don't mix primary fix with secondary findings
- Validate → deploy → re-validate is the minimum bar
- Track secondary findings as tasks, not branch scope creep

## Token Efficiency Through Persistent Knowledge

The biggest cost multiplier in agent teams is **redundant context building**. Each agent starts cold and re-reads unchanged files to understand structure it could have inherited.

### Knowledge Layers That Save Tokens
- **App map** (routes, layouts, component hierarchy): Analyst doesn't need to discover that AppShell wraps all routes
- **Element catalog** (locators, selectors, interaction patterns): Tester doesn't need to explore the DOM to find the unread badge
- **Execution flow maps** (WebSocket → service → hook → component): Once traced, this is stable across sessions — the analyst today spent most of its tokens building a map that already existed in our memories
- **Gotchas** (rate limiting, auth quirks, race conditions): Known pitfalls that every agent would otherwise rediscover

### Where This Leads
- Agents should **read** knowledge layers before starting, not discover from scratch
- Agents should **write back** new findings (the analyst's flow trace, the tester's new locators) so the next run is cheaper
- The knowledge layers become the **shared context** that replaces re-reading source files
- Over cycles, token cost per investigation drops as the knowledge base grows
- Goal: agent teams become cheaper and faster with each run, not just at parity

### Staleness Detection (Open Design Question)
Stored knowledge goes stale when source code changes. Agents need a way to know which parts of their knowledge to trust and which to revalidate.

**Possible approaches:**
- **Git-anchored knowledge**: Each knowledge layer records the commit hash it was derived from + the source files it covers. Before loading, diff those files against current HEAD. Changed files → flag those knowledge sections as "needs revalidation."
- **File-level checksums**: Lighter than full git history — store SHA of each source file alongside the knowledge it informed. Quick comparison on load.
- **Change surface mapping**: Not just "did the file change" but "did the relevant parts change." A knowledge layer about unread state only cares about changes to notificationService.ts, useUnreadSubscription.ts, AppShell.tsx — not every file in the repo. Could use function/export-level granularity.
- **Confidence decay**: Knowledge gets a freshness score that decays over time/commits. Agents prioritize revalidation of the stalest layers first, spending tokens where uncertainty is highest.

**The trade-off**: More granular staleness detection saves more tokens (fewer false revalidations) but costs more to maintain. Start simple (git-anchored with file-level tracking), refine as the pattern matures.

## Applicable Beyond Testing

This team pattern (analyst + executor + coder, coordinated by lead) could apply to:
- **Performance investigation**: Profiler agent + load test agent + optimization coder
- **Security audit**: Static analysis agent + penetration test agent + remediation coder
- **Migration planning**: Impact analysis agent + compatibility test agent + migration coder

The key principle: **separate the "what's happening" (tester) from the "why it's happening" (analyst) from the "how to fix it" (coder)**. Each role has different tools, different context needs, and different definitions of done.

## Refined Learnings from Full Session (4 Deploy Cycles)

### The Cycle Solidified
We ran the analyze → code → test → deploy cycle four times in one session. By the third cycle, it was mechanical:
1. Analyst traces code paths, delivers findings with file:line references
2. Lead + human decide scope (fix now vs defer)
3. Coder implements on feature branch, commits
4. Coder merges to develop, deploys to DEV
5. Tester runs targeted validation + full regression suite
6. If clean → coder promotes to main, deploys to PROD
7. Tester's suite becomes the regression baseline for the next cycle

### What Improved Over the Session

**1. Analyst-to-coder handoff became a design spec**
By the circuit breaker cycle (#3), the analyst delivered a complete implementation design (data structures, method signatures, line-by-line changes, edge cases). The coder followed it like a spec. No back-and-forth, no ambiguity. This is the right level of detail for analyst → coder handoffs.

**2. Tester writing tests in parallel with coder**
By the scroll fix cycle (#4), the tester started writing test scenarios while the coder was still implementing. Both finished around the same time — tests were ready to run the moment DEV deployed. This parallel track should be the default.

**3. One finding triggers the next investigation naturally**
Subscription race → "Failed to load messages" → circuit breaker → scroll position. Each fix surfaced the next concern. The team structure made it easy to follow the thread without losing momentum — just redirect an idle agent.

**4. Feature branch per concern, merge independently**
Four separate feature branches, each merged and deployed independently. Test infrastructure got its own branch. No tangled merges, easy to revert any single change if needed.

### The Deploy Cadence Pattern
For future projects, this cadence worked:
- **Batch 1**: Primary fix (targeted, high-confidence) → DEV → test → PROD
- **Batch 2**: Secondary fixes (reliability improvements) → DEV → test → PROD
- **Batch 3**: Architectural improvement (circuit breaker redesign) → DEV → test → PROD
- **Batch 4**: Related fixes surfaced during analysis (scroll position) → DEV → test → PROD

Each batch is independently deployable and independently revertable. Don't batch unrelated changes.

### Reusable Test Infrastructure Carries Forward
The tester produced 14 scenarios across 3 test files with shared fixtures, auth helpers, and scroll utilities. This infrastructure is reusable:
- `tests/fixtures/chat-fixtures.ts` — multi-user auth with API login + localStorage injection
- `tests/helpers/` — shared utilities (scroll detection, message waiting)
- `tests/chat/away-and-back-unread.spec.ts` — 4 desktop unread scenarios
- `tests/chat/scroll-position.spec.ts` — 5 scroll behavior scenarios
- `tests/mobile/mobile-away-and-back.spec.ts` — 2 mobile scenarios

Next project should start by reading these patterns, not reinventing them.

### What the Analyst Traced That Should Persist
These code path traces are stable architectural knowledge:
- **Unread state flow**: WebSocket → notificationService → useUnreadSubscription → AppShell state → badge render
- **Scroll position flow**: loadMessages → setMessages → useLayoutEffect → scrollToBottom (gated by isAtBottom)
- **Circuit breaker flow**: callApi → getCircuitFamily → isCircuitOpen → recordSuccess/recordFailure per-family
- **Component lifecycle**: AppShell never unmounts on navigation; TeamChatPage unmounts/remounts; MobileLayout wraps all mobile routes

These should be captured as execution flow maps in the architecture knowledge base.

### Cross-Project Portability
This entire session pattern ports to any project with:
- A deployed DEV and PROD environment
- A test framework (Playwright, Cypress, etc.)
- A feature branch git workflow
- Real-time features (WebSocket, SSE, polling) that need multi-user testing

The agent roles, cycle cadence, and branch strategy are project-agnostic. The test fixtures and knowledge layers are project-specific but follow a reusable template.
