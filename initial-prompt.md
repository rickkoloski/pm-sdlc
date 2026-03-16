# Getting Started with SDLC

## For New Projects

Copy and paste this prompt to Claude Code:

```
I'd like to implement the SDLC process from ~/src/ops/sdlc in this project.

Please:
1. Read ~/src/ops/sdlc/BOOTSTRAP.md for instructions
2. Analyze my existing documentation
3. Propose a structure and categorization
4. Wait for my approval before creating anything
```

---

## For Existing Projects (with docs already organized)

```
This project uses the SDLC process from ~/src/ops/sdlc.

Please read:
- ~/src/ops/sdlc/lifecycles/native.md for our default workflow
- ./CLAUDE.md for project-specific context
- ./docs/current_work/ for active deliverables
```

---

## Quick Reference

| Task | What to Say |
|------|-------------|
| Start new deliverable | "Let's create a spec for D[next number]: [feature name]" |
| Implement a spec | "Please implement D42 following the planning doc" |
| Archive completed work | "Let's organize the chronicles" |
| **Reconcile prototyping work** | **"Let's catalog our ad hoc work"** |
| **Audit compliance** | **"Let's run an SDLC compliance audit"** |
| Check process | "What's our SDLC workflow?" |

---

## After Prototyping Work

Been doing quick fixes or exploratory work without formal specs? That's fine. When ready to reconcile:

```
Let's catalog our ad hoc work.

Please:
1. Read ~/src/ops/sdlc/lifecycles/prototyping.md (see Reentry section)
2. Review commits since last formal deliverable
3. Propose how to document what we've done
4. Help me update any specs that need it
```

---

## Files CC Should Know About

| File | Purpose |
|------|---------|
| `~/src/ops/sdlc/BOOTSTRAP.md` | How to initialize a project |
| `~/src/ops/sdlc/lifecycles/native.md` | Our default workflow |
| `~/src/ops/sdlc/lifecycles/prototyping.md` | Prototyping lifecycle + reconciliation process |
| `~/src/ops/sdlc/operations/conventions.md` | Deliverable IDs, file suffixes, principles |
| `~/src/ops/sdlc/operations/compliance_audit.md` | Auditing project compliance |
| `~/src/ops/sdlc/templates/` | Document templates |
| `./CLAUDE.md` | Project-specific context |
| `./docs/current_work/` | Active deliverables |
