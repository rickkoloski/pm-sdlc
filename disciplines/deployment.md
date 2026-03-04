# Deployment Discipline

**Status**: Parking lot — capturing ideas as they emerge from other work

## Scope

CI/CD, infrastructure, release management, monitoring, rollback strategies.

## Parking Lot

### From Testing (2026-02-22)

- **E2E tests deferred from CI/CD (D6 decision).** Optimizing for learning velocity now. Will design the CI/CD integration point once the testing process has proven itself. When ready, consider: pre-deploy smoke suite, post-deploy verification, Playwright in GitHub Actions.

- **Token format migration requires deployment coordination (D10).** Auth tokens changed from email-encoded to user_id-encoded. All existing tokens are invalid after D10 deployment. Users must re-login. Frontend 401 interceptor handles it, but deployment should include a comms step.

- **Playwright CLI runs headless by default.** Good for CI. The CLI can be invoked from GitHub Actions without browser display. `state-save` / `state-load` handles auth persistence across CI runs.

- **Health check prerequisite pattern.** The testing discipline's health check recipe (curl API, UI, proxy) should become a deployment readiness check. Same checks, different context.
