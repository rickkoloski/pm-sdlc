# Collaboration Model — Markdown Review Surface + Long-lived Working-Doc Conventions

A pattern for **how humans and agents collaborate around structured working documents**: probes, plans, briefings, handback chronicles, ticket-update drafts, retros. Two parts:

1. **Markdown review surface** — where the docs live and how a human reviews agent-authored content.
2. **Long-lived working-doc conventions** — marker convention for needs-attention spots inside docs + the Open questions / Decision log accumulator pattern.

Both parts work with any editor. The review surface section recommends a specific dogfooded editor for PortableMind users; the conventions are editor-independent.

---

## Markdown Review Surface

### The default

All structured working documents live as plain markdown on disk in the project's working-docs tree (typically `docs/00_CURRENT_WORK/` or equivalent). Any markdown editor works — VS Code, IntelliJ, Obsidian, Typora, plain `cat | less`. A team can adopt this entire collaboration model with whatever editor they already use.

The agent writes and edits files via the file system. The human reviews via their editor. Standard. The conventions below (markers + decision log) are what make this collaboration scale across long-running work.

### Recommended for PortableMind users — `md-editor-mac`

PortableMind has built and dogfoods a markdown editor specifically tuned for agent-mediated collaboration:

- **Repo:** https://github.com/rickkoloski/portablemind-md-editor-mac
- **Local install path:** `~/src/apps/md-editor-mac`
- **Live external-edit watcher:** when an agent writes or edits a file, the open tab refreshes immediately. No "reload" round-trip.
- **CLI surface:** `~/src/apps/md-editor-mac/scripts/md-editor /absolute/path/to/file.md[:line[:col]]` — idempotent, refocuses an existing tab if the file is already open. Agents can call this to "surface a doc for review" without launching new windows.
- **Marker rendering** — the convention markers below render with consistent visual weight in the editor.

When CC writes a doc that needs human review, the typical flow is:

1. CC writes/edits the file.
2. CC calls the CLI to surface it (often at the changed section's line).
3. Human reviews in the editor; live-watcher reflects subsequent edits.
4. Human annotates inline or in chat; resolutions move into the Decision log (see below).

### Alternatives are fine

Teams not on PortableMind tooling — or PortableMind teams who prefer their existing editor — can adopt every convention below with any markdown editor. Things they'll lose:

- **No agent-driven live refresh.** The reviewer hits "reload" or relies on the editor's file-watch (most editors have this; the MD-editor-mac watcher is just tighter).
- **No CLI surface for line-targeted opens.** Reviewer navigates manually. Cost: small.
- **No marker-styled rendering.** The markers still work — they're plain markdown bold + colon — but the editor doesn't visually distinguish them. Greppability (the load-bearing property) is unaffected.

These are minor frictions. The conventions deliver most of the value; the dogfooded editor is a nice-to-have multiplier.

---

## Long-lived working-doc conventions

These conventions apply to any markdown document that:
- Will be edited multiple times across multiple sessions
- Carries questions or decisions that need an audit trail
- Is referenced by other artifacts (tickets, commit messages, briefings)

Examples: probe docs, handoff chronicles, retros, briefings, decision queues, coverage maps.

### Inline marker convention

When something inside the doc needs attention, mark it inline with one of these patterns. Each marker on its own line, bold + colon literal so it renders bold AND is greppable from the CLI:

| Marker | When to use |
|---|---|
| `**Question:**` | An open question for the human reviewer (default — use this when in doubt). |
| `**Bug:**` | A bug encountered IN the editing/review tooling (not the project under review). |
| `**Decision:**` | A fork in the road where the agent picked one path and wants the human to confirm or override. |
| `**Assumption:**` | A load-bearing assumption the agent wants flagged for visibility. |

**Rules:**
- Marker on its own line. Don't bury it inside a paragraph.
- Greppable: the literal `**Marker:**` is what tools search for (`grep -rn '\*\*Question:' docs/`).
- Self-contained text after the marker. Don't assume the reader has the surrounding context — the marker statement should make sense on its own.
- One concern per marker. Don't pack three questions into one block.

### Open questions section + Decision log

For docs that accumulate Q/A and decisions over their lifetime, end the doc with two sections:

```markdown
## Open questions

_(none)_   ← when empty; mark explicitly so the section's existence stays auditable

## Decision log

| Date | Decision | Decided by |
|---|---|---|
| 2026-04-27 | Fix avatar bug via Option B (refetch trigger) instead of Options A+B. | RAK |
| 2026-04-28 | New peer dir `work-patterns/` for SDLC-agnostic patterns. | RAK |
```

**Resolution pattern (when a `**Question:**` gets answered):**

1. **Don't delete the section header.** Keep `## Open questions` as a permanent greppable anchor. Mark `_(none)_` when empty so the section's existence stays auditable.
2. **Move the resolved item into the Decision log table** at the bottom of the doc.
3. **Each Decision log row is self-contained** — describes what was decided, not the deliberation. The deliberation lives in chat or in the proposal text; the row is the audit trail.
4. **The decision-log table accumulates over the doc's life.** Old decisions don't get deleted; they get added to.

**Why this matters:**

Without the convention, long-lived docs slowly drift. Questions get answered in chat and never close out the marker. Decisions get implicit. Six months later someone reads the doc and can't tell what was settled vs what was still open. The convention makes the answer visible-in-doc.

The distinction between "questions are transient, decisions are durable" matters. Collapsing both into a single "Decisions" section loses information — readers can't tell what was open and got answered vs what was planned from the start. Keeping `## Open questions` as a greppable anchor (even when empty) preserves the distinction.

### When to apply

Not every doc needs this structure. Apply it when:
- The doc will be reviewed in multiple cycles
- Decisions in it will be cited from elsewhere (commit messages, tickets, briefings)
- More than one person will edit or review

Skip it for:
- Single-pass throwaway docs (a quick scratchpad, a one-off email draft)
- Generated artifacts (test results, build logs)
- Docs whose history is captured by an external system (e.g., GitHub PR descriptions — the PR's review log already handles this)

---

## Cross-references

- `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md` — uses these conventions for handback chronicles and verification ritual artifacts
- `~/src/ops/sdlc/operations/conventions.md` — broader SDLC convention layer; this doc is the work-pattern overlay for human↔agent doc collaboration specifically
