# SDLC Bootstrap Instructions

**For Claude Code:** Follow these instructions when a user asks to implement this SDLC process in their project.

---

## Phase 1: Discovery

### 1.1 Identify Existing Documentation

Scan the project for documentation:

```bash
# Check for existing docs directory
ls -la docs/ 2>/dev/null || echo "No docs/ directory"

# Find markdown files
find . -name "*.md" -type f | head -50

# Check for common doc patterns
ls -la {docs,documentation,doc,specs,design}/ 2>/dev/null
```

### 1.2 Categorize Existing Artifacts

For each document found, classify it:

| Category | Indicators |
|----------|------------|
| **Spec** | "Specification", "Design", requirements, API definitions |
| **Planning** | "Instructions", "How to", implementation steps |
| **Result** | "COMPLETE", "DONE", completion records |
| **Roadmap** | Future plans, phases, milestones |
| **Reference** | API docs, architecture overview, README |
| **Issue** | "BLOCKED", problems, open questions |

### 1.3 Identify Concepts

Group related documents into logical concepts:
- What domains/features does this project cover?
- Are there natural groupings (e.g., "authentication", "data model", "UI")?
- What would someone ask about? ("How does X work?")

---

## Phase 2: Proposal

### 2.1 Create Categorization Table

Present findings to the user:

```markdown
## Existing Documentation Analysis

### Documents Found: N files

### Proposed Categorization

| File | Current Location | Type | Proposed Concept |
|------|------------------|------|------------------|
| auth_spec.md | docs/ | Spec | 01_authentication |
| api_design.md | design/ | Spec | 02_api |
| ... | ... | ... | ... |

### Proposed Concepts

| # | Concept | Description | Files |
|---|---------|-------------|-------|
| 01 | authentication | User auth, sessions, tokens | 3 |
| 02 | api | REST endpoints, schemas | 5 |
| ... | ... | ... | ... |

### Recommended Actions

1. Create SDLC directory structure
2. Move/copy existing docs to appropriate locations
3. Create _index.md for each concept
4. Add templates for future work
```

### 2.2 Get User Approval

Ask: "Does this categorization look correct? Any adjustments before I create the structure?"

---

## Phase 3: Implementation

### 3.1 Create Directory Structure

```bash
# Create from skeleton
cp -r ~/src/SDLC/skeleton/docs/* PROJECT/docs/

# Remove template placeholders
rm -rf PROJECT/docs/chronicle_by_concept/_template_concept
rm -rf PROJECT/docs/chronicle_by_step/_template_step
```

### 3.2 Create Project-Specific Concepts

For each identified concept:

```bash
mkdir -p docs/chronicle_by_concept/NN_concept_name/{specs,planning,results}
```

### 3.3 Create Index Files

For each concept, create `_index.md`:

```markdown
# NN_concept_name

## Overview
[One paragraph describing this concept]

## Deliverables
[Table of specs, planning docs, results]

## Common Tasks
[How to do common things in this area]

## Key Decisions
[Important architectural choices]
```

### 3.4 Copy Templates

```bash
cp ~/src/SDLC/templates/* PROJECT/docs/templates/
```

### 3.5 Move Existing Documents

For each categorized document:
- Copy to appropriate chronicle location
- Or move to `current_work/` if still active

### 3.6 Create/Update CLAUDE.md

Ensure project has a CLAUDE.md with:
- Project purpose
- Technology stack
- Current work locations
- Key APIs/patterns
- **SDLC compliance directive** (see below)

Use `~/src/SDLC/skeleton/CLAUDE.md` as a starting template.

**CRITICAL:** Add this directive to enforce process compliance:

```markdown
## SDLC Process Compliance

This project follows the SDLC framework from `~/src/SDLC/`.

**CC must:**
- Follow the deliverable workflow (Spec → Planning → Implementation → Result)
- Use deliverable IDs (D1, D2, ...) for all new work
- Create specs before implementing non-trivial features
- Document completions in stepwise_results/
- Ask before deviating from established process

**Do not:**
- Skip the spec phase for significant work
- Implement features without deliverable IDs
- Deviate from the process without explicit approval

If unsure about process, reference `~/src/SDLC/process/overview.md`.
```

---

## Phase 4: Verification

### 4.1 Structure Check

```bash
# Verify directory structure
find docs/ -type d | sort

# Count files by location
echo "Specs: $(ls docs/current_work/specs/*.md 2>/dev/null | wc -l)"
echo "Concepts: $(ls -d docs/chronicle_by_concept/*/ 2>/dev/null | wc -l)"
```

### 4.2 Index Completeness

Verify each concept has `_index.md`:

```bash
for dir in docs/chronicle_by_concept/*/; do
  [ -f "$dir/_index.md" ] || echo "Missing index: $dir"
done
```

### 4.3 Report to User

```markdown
## SDLC Implementation Complete

### Structure Created
- current_work/: specs, planning, prompts, stepwise_results, issues
- chronicle_by_concept/: N concepts with indexes
- templates/: M templates copied

### Next Steps
1. Review the concept indexes for accuracy
2. Start using deliverable IDs (D1, D2, ...) for new work
3. See docs/process/overview.md for workflow details
```

---

## Artifact Type Reference

### Spec (`specs/`)
**Purpose:** Define WHAT to build
**Naming:** `dNN_feature_name_spec.md` or `feature_name_spec.md`
**Contains:**
- Problem statement
- Requirements
- Design/approach
- Success criteria

### Planning (`planning/`)
**Purpose:** Define HOW to build it
**Naming:** `dNN_feature_name_instructions.md`
**Contains:**
- Step-by-step implementation guide
- Code locations to modify
- API signatures to use
- Test approach

### Prompt (`prompts/`)
**Purpose:** Instructions for CC to execute
**Naming:** `dNN_feature_name_prompt.md`
**Contains:**
- Context summary
- Explicit task list
- Success criteria
- Files to read/modify

### Result (`stepwise_results/`)
**Purpose:** Record completion
**Naming:** `dNN_feature_name_COMPLETE.md`
**Contains:**
- What was implemented
- Files created/modified
- Test results
- Any deviations from spec

### Issue (`issues/`)
**Purpose:** Track blockers
**Naming:** `dNN_feature_name_BLOCKED.md`
**Contains:**
- What's blocked
- Why it's blocked
- What's needed to unblock
- Workarounds if any

---

## Concept Index Template

```markdown
# NN_concept_name

## Overview
[Brief description of what this concept covers]

## Deliverables

### Foundation
| File | Purpose | Dependencies |
|------|---------|--------------|
| ... | ... | ... |

### Extensions
| File | Purpose | Dependencies |
|------|---------|--------------|
| ... | ... | ... |

## Common Tasks
- "Do X" → See file_a.md
- "Do Y" → See file_b.md, file_c.md

## Key Decisions
- Decision 1: [rationale]
- Decision 2: [rationale]
```

---

## Notes for CC

1. **Always propose before acting** — Show the user the categorization and get approval
2. **Preserve existing structure** — Don't delete or move without confirmation
3. **Create indexes thoughtfully** — They're the primary navigation tool
4. **Adapt to project size** — Small projects may not need all concepts
5. **Document decisions** — Record why things are organized the way they are
