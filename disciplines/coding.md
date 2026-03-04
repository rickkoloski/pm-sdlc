# Coding Discipline

**Status**: Parking lot — capturing ideas as they emerge from other work

## Scope

Implementation patterns, conventions, tech debt management, code review, refactoring strategies.

## Parking Lot

### From Testing (2026-02-22)

- **Testability is a code quality concern.** When the testing agent has to use coordinate-based clicks or fragile DOM selectors, the code should change — not just the test. Testability debt items (missing data-testid, no aria-labels, timing-dependent state) are code quality items, triaged by the architect at planning boundaries.

- **Validation gaps found via test data design.** Designing boundary test cases for Brand create revealed: no numeric range validation on ABV/IBU/SRM (negative and 99.9 accepted), no string length limit on name. These are coding concerns surfaced by the testing discipline. The architect decides whether to fix or accept.

- **useRef gate pattern for StrictMode.** React 19 StrictMode double-mount interacts badly with useEffect + localStorage writes. The useRef gate pattern (D9) is a reusable coding pattern that should be in a shared patterns library, not just in datagrid-extended.

- **Slot replacement pattern for MUI.** MUI v8 component slots (columnMenuColumnsItem) can merge multiple contributors. Full slot replacement is sometimes the only clean approach. Document this as a coding pattern for MUI customization.
