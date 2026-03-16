---
name: RUP Scaffolding Descriptions
description: Teaching-quality task descriptions for RUP lifecycle scaffolding — explains WHY each discipline appears in each phase
type: knowledge
lifecycle: rup
verified: 2026-03-16
---

# RUP Task Descriptions

Descriptions for each discipline-in-phase task, keyed by `{phase}.{discipline}`. Used by the scaffold-project algorithm when creating tasks. Each description explains why that discipline appears in that phase and what work happens there.

## Inception

### inception.phase
Establish business case, define scope, identify key risks, stakeholder alignment. LCO milestone.

### inception.business_modeling
Define the business context: who are the stakeholders, what problem are we solving, what does success look like? This is where business modeling peaks — the domain model and business case originate here.

### inception.requirements
Capture the architecturally significant use cases (10-20% of the full model). These aren't all the requirements — just enough to validate scope and identify the hardest problems early.

### inception.analysis_design
Initial architecture sketch and candidate technology evaluation. Not detailed design — just enough to assess feasibility and identify major technical risks.

### inception.implementation
Proof-of-concept or throwaway prototype only — used to mitigate a specific critical risk, not to build production code. Most Inception phases need zero implementation.

### inception.test
Not testing code — testing assumptions. Can this requirement be validated? Is this risk measurable? Test thinking starts here even though there's nothing to execute yet.

### inception.deployment
Deployment planning only — identify target environments, constraints, compliance requirements. No actual deployment happens in Inception.

### inception.project_management
Establish the project plan: scope, schedule, cost estimate, risk register. The business case lives here. This is where the go/no-go decision gets its data.

### inception.environment
Set up the development environment, tools, build system, source control. Environment peaks here — 40% of total environment effort happens in Inception. Getting tooling right early pays compound dividends.

### inception.milestone
Lifecycle Objectives milestone gate

## Elaboration

### elaboration.phase
Validate architecture through working code, mitigate high risks, build executable architecture. LCA milestone.

### elaboration.business_modeling
Refine the domain model as architecture reveals gaps. The business model from Inception gets stress-tested against technical reality — 'can we actually model this workflow?' questions surface here.

### elaboration.requirements
Detail the architecturally significant use cases end-to-end. By the LCA milestone, 80% of use cases should be identified and ~10% fully specified — enough to prove the architecture can carry them.

### elaboration.analysis_design
This is the heart of Elaboration. Design and validate the executable architecture — not through documents, but through working code that proves the major technical decisions. Architecture peaks here.

### elaboration.implementation
Build the architectural skeleton — the key use cases implemented end-to-end. This is production-quality code (unlike Inception prototypes), but focused on architecture validation, not feature completeness.

### elaboration.test
Test the architecture, not just the features. Can data flow end-to-end? Do integration points work under load? Test infrastructure (CI, test frameworks) is established here to carry through Construction.

### elaboration.deployment
Establish the deployment pipeline and staging environments. The executable architecture needs somewhere to run. CI/CD setup happens here so Construction iterations can deploy continuously.

### elaboration.project_management
Refine the plan based on Elaboration actuals. The Construction plan must be credible — based on measured velocity from Elaboration iterations, not Inception estimates. This is where PM effort peaks at 28%.

### elaboration.environment
Finalize the development environment. Any tooling gaps discovered during Inception get resolved. Environment work drops off after Elaboration — the team should be fully productive from Construction onward.

### elaboration.milestone
Lifecycle Architecture milestone gate

## Construction

### construction.phase
Implement remaining features against validated architecture. Iterative, feature-by-feature. IOC milestone.

### construction.business_modeling
Still present but winding down. Handle domain model changes discovered during feature implementation. New business rules surface when real users interact with real features.

### construction.requirements
Detail remaining use cases as they come up for implementation. Requirements work continues because RUP is iterative — you don't freeze requirements after Elaboration, you refine them as you build.

### construction.analysis_design
Detailed design for each feature being built. The architecture is stable from Elaboration; this is component-level design within that architecture. Still 36% of total A&D effort.

### construction.implementation
This is where most code gets written — 56% of total implementation effort. Feature-by-feature, iteration-by-iteration, against the validated architecture. The team is at full velocity.

### construction.test
Test peaks here at 52% of total effort. Unit testing alongside implementation, integration testing at iteration boundaries, and growing regression suites. Continuous integration keeps the build green.

### construction.deployment
Regular deployments to staging. Deployment grows through Construction (24% of total) as the release pipeline matures and release candidates become more frequent.

### construction.project_management
Track progress against the Elaboration-calibrated plan. Iteration planning, risk management, stakeholder communication. PM effort is steady at 32% — the largest phase but routine work.

### construction.environment
Maintain and evolve the build/test/deploy toolchain. Most environment work is done — this is maintenance and minor enhancements. Only 20% of total environment effort.

### construction.milestone
Initial Operational Capability milestone gate

## Transition

### transition.phase
Deliver to users, acceptance testing, defect resolution, training, production deployment. GA milestone.

### transition.business_modeling
Final validation that what was built matches the business need. User acceptance often reveals business model gaps invisible during development — 'this isn't how we actually work' moments happen here.

### transition.requirements
Capture requirement changes discovered during user acceptance. These feed into the next release cycle or become known gaps documented in the release notes.

### transition.analysis_design
Performance tuning and production architecture adjustments. The architecture was validated in Elaboration, but production load patterns may require design refinements. 20% of total A&D effort.

### transition.implementation
Defect fixes, performance patches, and last-mile configuration. No new features — only fixes for issues discovered during acceptance testing and pilot deployment.

### transition.test
Acceptance testing with real users, regression testing on fixes, and production smoke tests. Test effort is 32% of total here — second only to Construction. The last gate before GA.

### transition.deployment
This is where Deployment peaks at 68% of total effort. Production deployment, data migration, user training, documentation delivery, operations handoff, and support readiness.

### transition.project_management
Project closeout: lessons learned, final metrics, handoff to operations, release notes. Capture what worked and what didn't for the next project or release cycle.

### transition.environment
Production environment hardening, monitoring setup, runbook creation. The last 20% of environment work ensures operations can support the system after the project team disbands.

### transition.milestone
General Availability milestone gate
