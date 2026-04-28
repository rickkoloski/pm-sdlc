# Work Patterns

Patterns for **how teams operate** that are durable across SDLC choices. Distinct from `~/src/ops/sdlc/` (disciplines, lifecycles, operations), which describes **what the SDLC requires**.

## When to read here vs there

| Question | Look in |
|---|---|
| "What does the SDLC require for this deliverable?" | `~/src/ops/sdlc/` |
| "How do I structure work when I have one agent / two agents / three agents?" | `~/src/ops/sdlc/work-patterns/` |
| "How should agents and humans collaborate around documents?" | `~/src/ops/sdlc/work-patterns/collaboration_model.md` |
| "What's the convention for handing work between sessions?" | `~/src/ops/sdlc/work-patterns/multi-agent-handoff.md` (or `parallel_sessions.md`) |
| "What ticket structure should I use for a coverage-first defect-hunt pilot?" | `~/src/ops/sdlc/work-patterns/path-a-pilot.md` |

If a pattern only matters when you've adopted a specific SDLC, it belongs in `sdlc/`. If a pattern works alongside any SDLC (waterfall, scrum, kanban, native, prototyping), it belongs here.

## Catalog

- **`collaboration_model.md`** — Markdown review surface (recommended editor + conventions for marker annotations and decision logs in long-lived working docs).
- **`multi-agent-handoff.md`** — Two-agent split between a testing/analysis session and a fix-impl session: self-verification ritual, validation cycle, ticket-routing conventions.
- **`parallel_sessions.md`** *(queued)* — Operating two main agent sessions simultaneously: role-can-invoke table, runtime sharing, env-impact tagging, branch-collision avoidance, mid-flight handoff doc shape.
- **`path-a-pilot.md`** *(queued)* — Coverage-first defect-hunt pilot template: probe → tests RED → file → fix-impl → validate → close. Documented with one-session and multi-session variants.
- **`harmoniq/`** *(subfolder)* — Patterns specific to the PortableMind / Harmoniq tenancy structure. Carries its own README explaining why a tenancy-specific subfolder exists.

## Editorial principles

1. **Each doc is self-contained.** A reader can adopt the pattern without reading the rest of the catalog.
2. **Cross-link, don't duplicate.** When two patterns interact, point at each other.
3. **Cite real failures.** Patterns earn their place when they're tied to specific incidents that motivated them.
4. **Status notes are explicit.** When a pattern requires multiple agents (e.g., the verification ritual), say so up front. Single-session teams should be able to read the doc and decide it doesn't apply.
5. **No SDLC-specific language in the body.** If a pattern would read awkwardly to a team using a different SDLC, the body needs a generalize pass. Tenancy-specific examples belong in a clearly-labeled section or the `harmoniq/` subfolder.
