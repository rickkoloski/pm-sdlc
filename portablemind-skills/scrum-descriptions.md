---
name: Scrum Scaffolding Descriptions
description: Teaching-quality task descriptions for Scrum lifecycle scaffolding — explains each sprint event's purpose and what good looks like
type: knowledge
lifecycle: scrum
verified: 2026-03-16
---

# Scrum Task Descriptions

Descriptions for each sprint event, keyed by `sprint.{event}`. Used by the scaffold-project algorithm when creating tasks. Each sprint reuses the same descriptions since the pattern repeats.

### sprint.phase
A time-boxed iteration where the team commits to delivering a potentially shippable increment. The Sprint Goal gives the sprint a coherent purpose beyond a collection of tasks. All disciplines are active within the sprint, with intensity varying by event.

### sprint.sprint_planning
Select backlog items for the sprint, define the Sprint Goal, and create the Sprint Backlog. The team commits to what they believe they can deliver — not what they hope to. Key question: 'What can we deliver this sprint, and how will we do it?' Typical duration: 4 hours max for a 2-week sprint. Disciplines peak here: analysis, architecture, and design shape the approach before coding begins.

### sprint.daily_work
The core development timebox where all disciplines are active: design, coding, testing, integration. Daily standups (15 min) sync the team on progress and surface impediments. Each backlog item moves through 'in progress' to 'done' against the Definition of Done. This is where the team's cross-functional nature matters most — no handoffs between phases, just continuous collaboration toward the Sprint Goal.

### sprint.backlog_refinement
Ongoing preparation of upcoming backlog items — clarifying acceptance criteria, estimating, decomposing large items, identifying dependencies. Consumes ~10% of sprint capacity. This is where disciplines like research, analysis, and architecture do quiet preparatory work for future sprints. Well-refined backlogs prevent sprint planning from becoming a design session.

### sprint.sprint_review
Demonstrate the working increment to stakeholders. Gather feedback on what was built. Inspect the Product Backlog and adapt upcoming priorities based on what was learned. This is not a status meeting — it's a working session focused on the product. The increment must be potentially shippable, meaning it meets the Definition of Done regardless of whether it's actually released.

### sprint.sprint_retrospective
Inspect the team's process and identify at least one actionable improvement for the next sprint. What went well? What didn't? What will we change? The retrospective is where the adaptation pillar of Scrum lives — skip it and the same problems repeat sprint after sprint. Good retros produce concrete, assigned actions, not vague complaints.
