# AI Challenge #15 – Multi-Tenant Configuration Platform

> Status: in progress. Sections marked TODO are filled in as the work moves forward.

## 1. Overview

TODO: One paragraph. Who is the user, what problem does this solve, what does the platform let an admin do?

## 2. Scope

| In scope | Out of scope (and why) |
|----------|------------------------|
| TODO | TODO |

## 3. Assumptions

The brief leaves some things open. My assumptions are listed in [docs/assumptions.md](./docs/assumptions.md).

## 4. Approach and architecture

TODO:
- Tenant model and isolation strategy
- How configuration is layered and resolved (for example global → country → tenant → product)
- Versioning, publishing, rollback, and audit trail
- Admin roles and permissions
- A simple diagram

## 5. Key decisions and trade-offs

Each decision is recorded in [docs/decisions.md](./docs/decisions.md). The most important ones:

1. TODO
2. TODO

## 6. Tech stack

TODO: TypeScript for frontend and backend. Fill in the rest once the stack is chosen.

## 7. How to run

```bash
# TODO: install, configure, run, seed demo data
```

## 8. Tests

TODO: what is covered, and how to run them. Priority tests:
- A tenant can never read or change another tenant's data
- Configuration resolution order and overrides
- Validation of invalid configuration
- Version history and rollback

## 9. Project layout

```
ai-challenge-15-multi-tenant-config/
├── README.md
├── docs/
│   ├── assumptions.md
│   └── decisions.md
├── ai-conversation/    # full AI conversation log
├── src/                # application code
└── tests/              # automated tests
```

## 10. What I would do next

TODO: known gaps and what I would build with more time.

## 11. How I used AI

The full conversation with my coding agent is in [ai-conversation/](./ai-conversation). Notes on where the AI made mistakes and how I caught them are in [ai-conversation/notes.md](./ai-conversation/notes.md).
