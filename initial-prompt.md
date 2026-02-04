# Getting Started with SDLC

## For New Projects

Copy and paste this prompt to Claude Code:

```
I'd like to implement the SDLC process from ~/src/SDLC in this project.

Please:
1. Read ~/src/SDLC/BOOTSTRAP.md for instructions
2. Analyze my existing documentation
3. Propose a structure and categorization
4. Wait for my approval before creating anything
```

---

## For Existing Projects (with docs already organized)

```
This project uses the SDLC process from ~/src/SDLC.

Please read:
- ~/src/SDLC/process/overview.md for workflow
- ./CLAUDE.md for project-specific context
- ./docs/current_work/ for active deliverables
```

---

## Quick Reference

| Task | What to Say |
|------|-------------|
| Start new deliverable | "Let's create a spec for D[next number]: [feature name]" |
| Implement a spec | "Please implement D42 following the planning doc" |
| Archive completed work | "Let's organize the chronicles - see ~/src/SDLC/process/chronicle_organization.md" |
| Check process | "What's our SDLC workflow?" → CC reads ~/src/SDLC/process/overview.md |

---

## Files CC Should Know About

| File | Purpose |
|------|---------|
| `~/src/SDLC/BOOTSTRAP.md` | How to initialize a project |
| `~/src/SDLC/process/overview.md` | The workflow |
| `~/src/SDLC/templates/` | Document templates |
| `./CLAUDE.md` | Project-specific context |
| `./docs/current_work/` | Active deliverables |
