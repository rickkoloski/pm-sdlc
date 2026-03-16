---
name: SAFe Scaffolding Descriptions
description: Teaching-quality task descriptions for Essential SAFe lifecycle scaffolding — explains PI structure and each event's purpose
type: knowledge
lifecycle: safe
verified: 2026-03-16
---

# SAFe Task Descriptions

Descriptions for each PI element, keyed by `pi.{element}`. Used by the scaffold-project algorithm when creating tasks. Each PI reuses the same descriptions since the pattern repeats.

### pi.phase
A Program Increment (PI) is the heartbeat of SAFe — a fixed timebox where an Agile Release Train (ART) of aligned teams delivers an integrated, tested increment. The PI provides a regular cadence for planning, execution, and adaptation at scale. Everything in SAFe revolves around this cycle.

### pi.pi_planning
The most important event in SAFe. All teams align on PI Objectives, identify cross-team dependencies, and commit to deliverables. This is not a status meeting — it's a working session where teams self-organize around the work. The confidence vote (target: 3+ out of 5) is an honest pulse check, not a rubber stamp. Typical duration: 2 days, all teams co-located or virtual.

### pi.iteration
A 2-week development iteration within the PI context. Teams run standard Scrum or Kanban at this level. Each iteration produces a working, tested increment demonstrated at the System Demo. The iteration boundary is the team-level inspect-and-adapt point — what Scrum calls the sprint. PI Objectives guide the work; iteration planning selects the specific stories.

### pi.ip_iteration
The Innovation and Planning (IP) iteration is protected time for work that would otherwise be squeezed out: hackathons, technical debt reduction, education, and preparation for the next PI Planning. This is NOT overflow capacity for unfinished features — raiding the IP iteration is one of the most common SAFe anti-patterns. It's also where the next PI's planning preparation happens, making it the bridge between PIs.

### pi.inspect_and_adapt
Quantitative review of PI Objectives (did we deliver what we committed to?) followed by a problem-solving workshop. Root cause analysis on missed objectives, and improvement backlog items are created with owners. This is SAFe's adaptation mechanism at the program level — the equivalent of Scrum's retrospective, but data-driven and focused on systemic issues rather than team-level process tweaks.
