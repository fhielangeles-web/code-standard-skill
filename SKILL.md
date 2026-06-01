---
name: code-standards
description: "This file is read by Claude Code at the start of every session. Follow every rule here without being asked."
---

This file is read by Claude Code at the start of every session. Follow every rule here without being asked.


## Code quality

**Naming**
- Variables and functions are named for what they hold or do, not what type they are. `userEmail` not `str1`. `fetchOrderById` not `getData`.
- Booleans are named as questions: `isLoading`, `hasPermission`, `canSubmit`.
- Constants are SCREAMING_SNAKE_CASE: `MAX_RETRY_ATTEMPTS`, `DEFAULT_TIMEOUT_MS`.
- No abbreviations unless they are universally understood (`id`, `url`, `db`). Never `usr`, `cfg`, `tmp`.

**Functions**
- One function does one thing. If you need "and" to describe what it does, split it.
- Maximum 40 lines per function. If it is longer, extract helper functions.
- Functions are named with a verb: `getUser`, `validateEmail`, `formatCurrency`.
- No magic numbers. Extract to named constants with units in the name where relevant: `TIMEOUT_MS = 5000`, not `5000`.

**Variables (JavaScript/TypeScript)**
- Declare variables as close to their use as possible.
- Prefer `const` over `let`. Never use `var`.
- Destructure objects and arrays when reading more than one property.

**Variables (Python)**
- Declare variables as close to their use as possible.
- Use `snake_case` for variables, functions, and modules. Use `PascalCase` for classes. Use `SCREAMING_SNAKE_CASE` for module-level constants.
- Unpack tuples and dicts explicitly rather than indexing by magic position: `user_id, email = row` not `row[0], row[1]`.
- Annotate all function signatures with type hints. Use `from __future__ import annotations` for forward references.

---

## Error handling

**Core rules — non-negotiable**
- Never leave a catch block empty. Every caught error is either handled, logged, or rethrown.
- Never swallow errors silently. If you catch it and cannot recover, rethrow it.
- Always log errors with context: the function name, the relevant IDs, and `error.message`. Never log just the string "error occurred".
- Show users actionable messages, not raw error objects or stack traces.

**Be specific**
- Do not catch all errors with a single generic handler. Distinguish between validation errors, auth errors, network errors, and unexpected errors.
- Handle HTTP errors by status code. 401 redirects to login. 422 highlights the invalid field. 500 shows a generic message and logs the full detail.
- Use `instanceof` checks when catching custom error classes.

**Async (JavaScript/TypeScript)**
- Every `async` function that can fail must be wrapped in try/catch or have `.catch()` chained.
- Never fire-and-forget a Promise without handling its rejection.
- Set a global unhandled rejection listener in every application entry point.

**Async (Python)**
- Every `async` function that can fail must be wrapped in `try/except`.
- Never schedule a coroutine with `asyncio.create_task` without attaching a done-callback or awaiting it.
- Register a global `loop.set_exception_handler` in every application entry point.

**Custom error classes**
- Create custom error classes for distinct failure modes in the domain: `ValidationError`, `AuthError`, `NotFoundError`.
- Custom errors extend `Error`, set `this.name`, and carry relevant context as properties.
- Use these classes so callers can catch by type, not by parsing strings.

**Error boundaries**
- Every server has a single top-level error handler that catches anything that slips through.
- The top-level handler logs the full stack trace and returns a sanitised response to the client — never expose internal error details to end users in production.

---

## Project structure

**JavaScript/TypeScript**
- Source code lives in `src/`. Tests live in `src/__tests__/` or alongside the file as `*.test.ts`.
- Group files by feature, not by type. `src/orders/` contains `orders.service.ts`, `orders.controller.ts`, `orders.test.ts`.
- Index files (`index.ts`) only re-export — no logic inside them.

**Python**
- Source code lives in a package directory named after the project (e.g. `my_project/`). Tests live in `tests/` at the repo root.
- Group modules by feature: `orders/service.py`, `orders/routes.py`, `orders/test_orders.py`.
- `__init__.py` files only re-export public API — no business logic inside them.

**Both**
- One module does one job. No file that is both a data-fetcher and a UI renderer.
- No circular dependencies. If two modules need each other, extract the shared logic into a third.

---

## Testing

- Every function that contains logic ships with a test. No exceptions.
- Tests are written alongside the code, not added later.
- Test structure: Arrange → Act → Assert. One assertion per test where possible.
- Test names describe what the code does, not what the test does: `returns null when user not found`, not `test getUser`.
- Test the behaviour, not the implementation. Tests should not break when you refactor internals without changing behaviour.
- Aim for coverage on branches, not just lines. Every `if` has a test for both paths.
- Do not mock things you do not own (e.g. third-party libraries). Wrap them first and mock the wrapper.

---

## Git discipline

- Commits are small and focused. One logical change per commit.
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`.
- The commit subject line is a sentence in the imperative mood: `fix: handle null user on login` not `fixed bug`.
- Never commit: `.env` files, API keys, passwords, `node_modules/`, build artefacts, or `console.log` left for debugging.
- Feature work happens on a branch. `main` is always deployable.

---

## Security

- No secrets in code. All credentials, API keys, and tokens live in environment variables.
- `.env` is in `.gitignore`. A `.env.example` with placeholder values is committed instead.
- Validate and sanitise all input that arrives from outside the application — API requests, query params, form data, file uploads.
- Never trust user-supplied data in database queries. Use parameterised queries or an ORM. No string concatenation in SQL.
- Never log sensitive data: passwords, tokens, card numbers, full names combined with identifiers.
- **JavaScript/TypeScript:** Dependencies are pinned to exact versions in production. Run `npm audit` regularly.
- **Python:** Pin dependencies in `requirements.txt` or `pyproject.toml` with exact versions. Run `pip-audit` regularly.

---

## Documentation

- Comments explain *why*, not *what*. The code itself explains what. A comment on the same line as obvious code is noise.
- **JavaScript/TypeScript:** Every exported function has a JSDoc comment with `@param`, `@returns`, and `@throws` where relevant.
- **Python:** Every public function has a docstring in Google style covering `Args`, `Returns`, and `Raises` where relevant.
- Every repository has a `README.md` that covers: what the project does, how to set it up locally, how to run tests, and how to deploy.
- When adding a non-obvious architectural decision, add an ADR (Architecture Decision Record) in `docs/decisions/`. Write an ADR when: you are choosing between two or more reasonable approaches, the decision has consequences that will be felt for more than a sprint, or a future reader would otherwise wonder "why on earth did they do it this way". A code comment is enough for local implementation choices; an ADR is for decisions that shape the system.

---

## Performance

- Do not optimise prematurely. Write clear code first. Optimise only when you have measured a problem.
- Avoid unnecessary work in loops. Move invariants outside the loop.
- Paginate or stream large data sets. Never load unbounded collections into memory.
- Cache expensive operations at the right level — but document that something is cached and when it invalidates.

---

## Code review checklist

Before marking any task complete, verify:

- [ ] All new functions have tests covering the happy path and at least one failure path
- [ ] No empty catch/except blocks anywhere in the diff
- [ ] No hardcoded secrets, URLs, or magic numbers
- [ ] No `console.log` / `print` left in production code — use the project logger
- [ ] All new environment variables are documented in `.env.example`
- [ ] The README is updated if setup or usage changed
- [ ] **JS/TS:** TypeScript types are present on all function signatures
- [ ] **Python:** Type hints are present on all function signatures; `ruff` and `mypy` pass with no new errors
- [ ] No new dependencies added without a reason documented in the PR description

---

## Logging standard

Use the project's shared logger — never `console.log` in JS/TS or bare `print` in Python.

**JavaScript/TypeScript:** Import from `src/lib/logger.ts` (wraps `pino` or equivalent). Log at `info` for normal operations, `warn` for recoverable issues, `error` for failures. Always pass a context object as the second argument: `logger.error({ fn: 'fetchOrder', orderId }, err.message)`.

**Python:** Use the standard `logging` module. Obtain a logger per module: `logger = logging.getLogger(__name__)`. Configure handlers only at the application entry point — never in library code. Always pass structured context via the `extra` kwarg: `logger.error("fetch failed", extra={"fn": "fetch_order", "order_id": order_id})`.

Both: log levels are `debug`, `info`, `warning`/`warn`, `error`. Never use `critical` for routine errors. Never log inside a tight loop.

---

## CI enforcement

The following checks run automatically on every pull request and must pass before merge:

| Check | Tool | Scope |
|---|---|---|
| Lint | `ruff` (Python) / `eslint` (JS/TS) | All source files |
| Type check | `mypy` (Python) / `tsc --noEmit` (TS) | All source files |
| Tests | `pytest` / `jest` | Full suite |
| Dependency audit | `pip-audit` / `npm audit` | Production deps |
| Secret scan | `gitleaks` | Full repo |

Everything else in this document is a human review responsibility. If a rule is not in the table above, it is enforced at code review, not automatically.

---

## What professional output looks like

Every piece of code produced in this project should be something a senior developer with ten years of experience would be comfortable putting their name on. That means:

- A new reader can understand any function in under thirty seconds without asking questions.
- Errors are never hidden and always give the next developer enough context to fix the problem.
- Tests exist, and they prove the code works under the conditions that matter.
- The diff is clean — no leftover debug code, no commented-out blocks, no TODOs without a ticket reference.
- Security and correctness are not afterthoughts.
