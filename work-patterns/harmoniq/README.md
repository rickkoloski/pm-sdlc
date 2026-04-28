# Harmoniq-specific work patterns

This subfolder holds work-pattern conventions that depend on the PortableMind / Harmoniq tenancy structure — Project #21 (bug ledger), Project #52 (test automation work units), the cross-tenant sharing model, the canonical party_ids of active collaborators.

## Why a tenancy-specific subfolder exists

The patterns in `~/src/ops/sdlc/work-patterns/` (parent directory) are written to be SDLC-agnostic and tenancy-agnostic. They reference "a bug ticket" or "the test-pipeline project" abstractly. Real adoption of those patterns inside the PortableMind tenancy uses specific project IDs, specific status conventions, specific party-id disambiguation rules.

Rather than:
- Pollute the parent docs with tenancy-specific specifics, or
- Force PortableMind operators to mentally translate between abstract patterns and their concrete tooling,

we keep tenancy-specific elaborations here. Each file in this subfolder is a thin overlay on a parent pattern, citing it by name and adding the PortableMind-specific bindings.

## Catalog

*(Empty for the moment. Files will land as the parent work-patterns docs stabilize and tenancy-specific overlays become useful.)*

Anticipated content:
- `bug-ticket-conventions.md` — Project #21 layout, `task_type_id: 4`, owner_party_id defaults, sibling-ticket cross-ref shape.
- `test-pipeline-project.md` — Project #52 layout, pilot parent + fix-impl subtask hierarchy, status flow.
- `party-id-disambiguation.md` — The "DM-membership disambiguator" rule for resolving canonical party_ids when test/demo Individuals share names with real users.

## How to use

When reading the parent work-patterns docs, look here for the concrete bindings for any abstract reference. When the parent doc says "file the bug in your bug ledger," `bug-ticket-conventions.md` (when it lands) will tell you exactly how PortableMind teams do that.
