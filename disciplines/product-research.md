# Product Research Discipline

**Status**: Active — competitive analysis capability
**Knowledge store**: `knowledge/product-research/` (cross-project)

## Scope

Market research, user research, competitive landscape, feature ideation, product-market fit.

## Competitive Feature Analysis

The primary active capability in this discipline. When the SDLC identifies a feature to build — whether a UI component (Gantt, datagrid, kanban) or an app feature (chat, ticketing, reporting) — competitive analysis answers: "What do others do, and what should we do?"

### When to Invoke

- **New feature scoping** — before writing a spec, understand the landscape
- **Build vs buy** — evaluating whether to use a library or build custom
- **Feature enhancement** — deciding how far to take an existing feature
- **Architecture decisions** — when competitors use fundamentally different approaches

### What It Produces

1. **Feature comparison matrix** — competitors as columns, dimensions as rows, grouped by category
2. **Dimension detail cards** — for non-binary dimensions, what each level means
3. **Competitor spotlight notes** — notable approaches worth knowing about
4. **Scoping questions** — 5-10 targeted questions for the product owner to answer before writing the spec

### How to Use

An agent follows the methodology in `knowledge/product-research/competitive-analysis-methodology.yaml`:
1. **Discovery** — identify 3-5 competitors via web search
2. **Dimensions** — load starter dimensions from `knowledge/product-research/dimension-catalog.yaml`, refine based on what competitors actually offer
3. **Research** — gather data per competitor per dimension from official docs
4. **Synthesis** — build the comparison matrix and detail cards
5. **Scoping** — generate questions that help the product owner define scope

### Skill Trajectory

```
NOW:     Knowledge store with methodology + dimension catalog
NEXT:    Prove it works on 2-3 real analyses (refine methodology)
THEN:    Package as /feature-compare skill
```

## Parking Lot

### From Testing (2026-02-22)

- **Advanced UI components as product differentiator.** Gantt charts, flowcharts, and the growing shared component library are what make apps look purposeful rather than "AI-generated template" (D3 decision rationale). Product research should track: which advanced components deliver the most user value? Which are table stakes vs differentiators?

- **Testability as product quality signal.** Apps that are easy to test tend to be more reliable, more accessible, and more polished. The testing discipline's investment in testability directly improves the product experience. Product research should consider: how do users perceive quality differences between well-tested and poorly-tested features?

### Future Questions

- How do brewing professionals actually use management software? (Informs what to test, what to prioritize)
- What are the competitive alternatives to Rockcut? (Informs feature testing priorities)
- Which UI patterns do brewers find intuitive vs confusing? (Informs design + testing)
