# Architecture Knowledge Base — Design Proposal

**Date**: 2026-02-24
**Context**: Pattern recognition across WebSocket/React bugs in Harmoniq Team Chat revealed the need for persistent architectural knowledge.

## Why

We keep encountering the same category of bugs at the seam between imperative singleton services and React's declarative lifecycle. Each fix teaches something generalizable, but the lessons live in git commits and session memories — not in a form that instructs future development.

## What It Would Contain

### Technology Integration Patterns
Rules of engagement when combining specific technologies. Not framework docs — our learned patterns for how they interact in our architecture.

Example entries:
- "ActionCable + React Hooks: Subscription Lifecycle" — how to safely wrap imperative subscriptions in useEffect, cleanup requirements, multi-consumer pitfalls
- "Singleton Services + React Component Tree" — when global state causes React-level bugs, scoping strategies
- "Circuit Breaker in SPA" — appropriate thresholds, per-endpoint vs global, UX when open

### Defensive Patterns (things to always do)
- Every real-time feature needs an HTTP fallback reseed path
- Every async chain in a useEffect needs AbortController + cleanup
- Every shared hook must assume multiple concurrent consumers
- StrictMode double-mount is a required test for any subscription hook
- Navigation state (`location`, `currentDomain`) belongs in dependency arrays when semantic meaning changes

### Anti-Patterns (things that have burned us)
- "Nuclear clear" on singleton state from a React hook (clearAllUnreadSubscriptions)
- Global circuit breakers in apps with multiple independent API consumers
- useEffect dependency arrays that omit navigation-derived state
- Async operations without cancellation paths
- Silent failures with no recovery mechanism

### Bug Families
Group related issues so future developers recognize the family when they see a new member:
- **The Singleton-React Seam**: Issues at the boundary between imperative services and declarative lifecycle
- **The Recovery Gap**: Features that work when everything is healthy but have no path back from failure
- **The Stale State Race**: Async operations that outlive the intent that triggered them

## Where It Would Live

Options (for discussion):
1. `~/src/ops/sdlc/knowledge/architecture/` — cross-project, alongside testing knowledge
2. Per-project `docs/ARCHITECTURE.md` — project-specific patterns
3. Both — general patterns in sdlc/knowledge, project-specific applications in project docs

Recommendation: Start with option 1 (cross-project). These patterns aren't Harmoniq-specific — they'll apply anywhere we use ActionCable + React, or any singleton-service + reactive-UI combination.

## How It Stays Current

Same staleness problem as test knowledge:
- Entries should reference the source commits/PRs that taught the lesson
- When those files change significantly, the pattern should be revalidated
- New bugs should be checked against existing patterns — if they match a known family, the fix should follow the established strategy
- If a new bug DOESN'T match a known family, that's a signal to add a new family

## Relationship to Agent Teams

This knowledge base directly reduces agent token costs:
- **Analyst agents** load architectural patterns instead of re-tracing code paths
- **Coder agents** follow defensive patterns instead of discovering them via review feedback
- **Tester agents** know which scenarios to prioritize based on bug families
- The knowledge base is the "experience" that makes each cycle cheaper than the last
