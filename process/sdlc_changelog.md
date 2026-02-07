# SDLC Process Changelog

A living record of how the process evolves through real use. Each entry captures what changed, why, and where the change originated.

---

## 2026-02-07: Tester Knowledge Capture + Process Update Command

**Origin:** Harmoniq project — Agent Teams SDLC trial (users-flyout-test)

**What happened:** After a successful agent team test run (tester validated 8/8 test cases), we realized the tester didn't capture navigation paths or lessons learned. The knowledge from that test session was lost when the agent shut down.

**Changes made:**
1. **`process/overview.md`** — Added "Tester Knowledge Capture" section defining required Navigation & Learnings content in test results
2. **`process/overview.md`** — Added "Updating This Process" section with "Let's update the SDLC" trigger command
3. **`process/overview.md`** — Added principle #7: "Capture testing knowledge"
4. **`process/sdlc_changelog.md`** — Created this file as a living changelog
5. **Harmoniq `agent-teams-sdlc-evolution.md`** — Added tester knowledge capture convention and filled in Learnings section

**Rationale:** Testing the same UI areas repeatedly is expensive. Navigation paths and gotchas captured by the first tester compound into efficiency for all future runs. The "Update SDLC" command ensures process improvements discovered during real work have a path back into the canonical framework.
