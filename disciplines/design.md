# Design Discipline

**Status**: Parking lot — capturing ideas as they emerge from other work

## Scope

UI/UX design, visual design, interaction patterns, accessibility, component library design.

## Parking Lot

### From Testing (2026-02-22)

- **Accessibility and testability are the same problem.** Components that render meaningful accessibility trees are inherently more testable. When we invest in testability (data-testid, aria-label, semantic roles), we're simultaneously improving accessibility. Design should treat these as one concern, not two.

- **"AI-generated template" detection.** Advanced UI components (Gantt charts, flowcharts, custom widgets) are what make apps look purposeful. The shared component library at `~/src/shared/` is a design asset, not just a code asset. Test automation strategies for these components protect design investment.

- **StatusChip color-only information.** Found during testing that StatusChip conveys status via color alone (not in accessibility tree). This is both an accessibility gap and a testability gap. Design principle: never convey meaning through color alone.

- **Visual verification requires computed style checks.** The testing discipline discovered that DOM correctness does not equal visual correctness (D7 FxBadge). Design should ensure that visual states have programmatic equivalents (aria attributes, data attributes) that can be verified without pixel comparison.
