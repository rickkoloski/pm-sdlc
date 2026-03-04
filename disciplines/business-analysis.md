# Business Analysis Discipline

**Status**: Parking lot — capturing ideas as they emerge from other work

## Scope

Requirements elicitation, domain modeling, stakeholder needs, acceptance criteria, user stories.

## Parking Lot

### From Testing (2026-02-22)

- **Test data design is lightweight domain modeling.** When we built the Test Data section for Brands, we had to reason about: What's a reasonable ABV range? What beer styles exist? What status transitions make sense? This is domain analysis happening inside the testing discipline. The BA discipline should feed valid domain ranges and business rules into test specs — and test specs should surface domain questions back to BA.

- **Acceptance criteria flow both directions.** BA writes acceptance criteria in specs. Testing verifies them. But testing also *discovers* missing acceptance criteria ("the spec didn't say what happens when you search for a numeric value in the ABV field"). These discoveries should flow back to BA for future specs.

- **Architect-proposed expected values need domain validation.** For formula columns (EST_IBU, EST_OG), the architect proposes expected values based on code understanding, but the human (domain expert) validates against brewing knowledge. This is a BA function — ensuring computed values make domain sense.
