# code-standards

A SKILL.md file for Claude Code that enforces consistent engineering standards across JavaScript/TypeScript and Python projects. Loaded automatically at the start of every Claude Code session.

## What it covers

- **Naming**: variables, booleans, constants, functions
- **Functions**: single responsibility, 40-line limit, no magic numbers
- **Error handling**: no silent failures, typed errors, global handlers
- **Project structure**: feature-based layout for both JS/TS and Python
- **Testing**: Arrange/Act/Assert, behaviour-focused, branch coverage
- **Git discipline**: Conventional Commits, branch strategy, what never to commit
- **Security**: secrets in env vars, parameterised queries, input validation
- **Documentation**: JSDoc/Google-style docstrings, ADRs for architectural decisions
- **Logging**: structured logging via project logger, never bare `print`/`console.log`
- **CI enforcement**: lint, type check, tests, dependency audit, secret scan

## Setup

Copy `SKILL.md` into your repository root (or your Claude Code project directory). Claude Code reads it automatically at session start.

```
cp SKILL.md /your/project/SKILL.md
```

No other configuration required.

## CI checks

The following run on every pull request and must pass before merge:

| Check | Tool |
|---|---|
| Lint | `ruff` (Python) / `eslint` (JS/TS) |
| Type check | `mypy` (Python) / `tsc --noEmit` (TS) |
| Tests | `pytest` / `jest` |
| Dependency audit | `pip-audit` / `npm audit` |
| Secret scan | `gitleaks` |

Everything else in the standards is enforced at code review.

## Contributing

Changes to the standards should reflect real team consensus. If you update a rule, update any related CI config or tooling to match. Open a PR with a brief rationale especially for anything that contradicts common defaults.
