# Testing Knowledge Store — Cross-Project

Knowledge that applies across all projects using our hybrid testing approach.
Project-specific knowledge lives in each project's `docs/testing/knowledge/`.

## Structure

```
testing-knowledge/
├── README.md                          ← This file
├── tool-patterns.yaml                 ← How to use PW CLI, CCP, etc.
├── component-catalog.yaml             ← Test strategies for shared UI components
├── gotchas.yaml                       ← Cross-project failure patterns
└── timing-defaults.yaml               ← Default wait profiles by component type
```

## How This Gets Used

1. Before a test session, the agent loads relevant cross-project knowledge
2. Project-specific knowledge (app map, element catalog) is loaded next
3. Together they form the context that prevents re-learning

## Maintenance

- Updated after test cycles when a pattern proves generalizable
- Human-reviewed per D7 (manual updates, evolving toward automation)
- Component knowledge travels with the component (per D3)
