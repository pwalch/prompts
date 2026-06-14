# Python Coding Convention

## Summary
This convention is opinionated and pragmatic: it trades breadth for consistency so common decisions are made once and encoded as defaults. The goal is to free attention for domain work by removing bikeshedding and making good practices effortless.

- Opinionated minimalism: choose a single default per concern (formatter, linter, test runner, HTTP client) and use it everywhere. Deviation is exceptional, requires rationale in review, and is documented next to the code that needs it.
- Clarity over cleverness: optimize for readers, not authors. Favor explicit types and units, straightforward control flow, shallow nesting, and names that state intent. When in doubt, pick the version that is easier to understand at 6 months distance.
- Separation of concerns: keep domain logic pure and testable; push I/O (CLI, network, filesystem, environment) to the edges behind thin adapters. Errors propagate upward and are handled at boundaries; models distinguish domain data from external representations.
- Reliability by default: protect every external call with timeouts, retries where safe, and input validation. Keep builds deterministic and dependencies pinned; use UTC and lossless serialization; prefer secure defaults over configurability.
- Human-first ergonomics: expose predictable, consistent CLIs; produce readable logs locally and switch to structured output when automation needs it. Fail fast with actionable messages and offer sensible verbosity controls instead of noisy output.
- Constraints as leverage: limit supported platforms and narrow the "blessed" patterns so everyone solves problems the same way. Prefer removing options over adding configuration; consistency beats personal preference across the repo.

## Foundations

- **Python version:** 3.13+
- **Supported platforms:** macOS, Linux (never Windows)
- **Python interpreter management:** [lonesnake](http://github.com/pwalch/lonesnake) with `direnv`
- **Formatter & linter:** [Ruff](https://docs.astral.sh/ruff/) only
  - Line length: **100**
  - Handles formatting, linting, and import sorting (no Black, no isort).
- **Typing:** Enforced via [Astral `ty`](https://github.com/astral-sh/ty) through Ruff.
  - No untyped function defs.
  - Require return type annotations.
  - No untyped `Any` except where unavoidable.
- never use `__future__`


## Repository Layout

- <repo_name>/
- bin/
  - <tool-name> # CLI executables (extensionless, shebang, chmod +x)
- <repo_name>/ # root Python package (valid module name)
  - init.py
  - cli.py
  - logging_cfg.py
  - http.py
  - models.py
  - utils_datetime.py
- tests/
  - test_*.py
- pyproject.toml
- requirements.in
- requirements.txt
- README.md


- **Executables:**
  - Located in `bin/`, **extensionless**, with `#!/usr/bin/env python` shebang.
- **Imports:**
  - **Absolute imports only**.
  - Group order: stdlib → third-party → local.
  - Blank line between groups.
- keep `__init__.py` files empty


## Code Structure

- Each script:
  - Begins with `#!/usr/bin/env python`.
  - Defines `main()` **at the top**.
  - Calls it at bottom with `if __name__ == "__main__": main()`.
- Always import `from argparse import ArgumentParser`.
  - CLI args are **kebab-case**.
- Functions: ≤ 40-50 lines.
- Functions ordered from top-level to deeper stack; group related functions together.
- **Dataclasses** for domain models.
- **Pydantic BaseModel** for external/IO models.
- **Enums** for discrete sets of choices.
- **Control flow:**
  - Prefer comprehensions (but **no nested comprehensions**).
  - Prefer early returns to nested conditionals.
  - Structural pattern matching (`match`) permitted.
 - **External integrations:** encapsulate all calls to third‑party systems (e.g., Slack, RabbitMQ, payment gateways) behind a small Facade that presents a narrow, domain‑oriented API. Do not import vendor SDKs outside the Facade. The Facade owns timeouts/retries, request/response mapping (Pydantic models), and error translation; expose a `Protocol`/interface for DI and testing.


## CLI & I/O

- Every tool exposes a CLI with `argparse`.
- **Flat flags** for simple CLIs; **subparsers** for complex hierarchies.
- **Configuration precedence:** CLI > environment variables (no dotenv, no config files).
- **Output:**
  - Hhuman-readable text.
  - If necessary, `--verbose/--quiet` or `--output-format {text,json}` can be added later, as needed

## Logging & Errors

- **Logger:** [structlog](https://www.structlog.org/).
- **Rendering:** human-readable.
- Timestamps included (UTC, ISO-8601 with `Z`).
- Log levels uppercase (`INFO`, `ERROR`, etc.).
- **Errors:**
  - Define custom exception classes for known failure modes.
  - Exceptions bubble up (no catching in `main()`).
  - Allow broad `except Exception` only at **CLI entrypoints** or **web boundaries**, never deeper.


## Data & Models

- **Naming:**
  - Avoid generic names like `Meta` or `Info`.
- **Domain models:** `@dataclass` (immutable where sensible, use `slots=True`).
- **IO models:** Pydantic `BaseModel`.
- **No TypedDicts**.
- **Nested dataclasses/models** for >5 fields, with meaningful names.
- **Units in variables:**
  - Always suffix with ISO units (e.g., `_seconds`, `_meters`, `_celsius`).
  - Looser abbreviations allowed **only if ISO-based** (e.g., `_km`, `_km_h`, `_mm`).
- **Datetime:**
  - Always timezone-aware (`datetime` with `timezone.utc`).
  - Serialize as ISO-8601 with `Z` (e.g., `2025-08-30T12:00:00Z`).


## HTTP

- Use `requests` only.
- Centralize session logic in helper (with retries).
- Defaults:
  - Timeout = **10 seconds**.
  - TLS verify = True.
  - Retries: 3 (backoff factor 0.5) for status `[429, 500, 502, 503, 504]`.
  - Include descriptive `User-Agent`.


## Style & Documentation

- **Naming conventions:**
  - Modules/packages: `snake_case`
  - Classes/Enums/Dataclasses: `PascalCase`
  - Functions/variables: `snake_case`
  - Constants: `UPPER_SNAKE_CASE`
  - Private: `_prefix`
- **Docstrings:**
  - Style: reStructuredText.
  - Only required when intent or types are unclear.
  - Explain **how/why**, not what.


## Testing

- Framework: `pytest`.
- Layout: `tests/` at repo root; files named `test_*.py`.
- Parametrize edge cases; use fixtures.
- Markers:
  - `slow`
  - `network` (skipped unless opted in via env flag).
- **No coverage target enforced**.


## Dependencies

- Dependency management: **pip-tools**.
- Declare base deps in `requirements.in`, pin with `pip-compile` to `requirements.txt`.
- Sync with `pip-sync`.
- Minimal runtime deps:
  - `requests`, `structlog`, `pydantic`, `pytest`, `ruff`, `ty`.


## CI/CD

No CI, but respect the following if a CI is requested:

**GitHub Actions** baseline:
- Install deps with pip-tools.
- Run Ruff (`check` + `format --check`).
- Run `ty` type checker.
- Run tests with `pytest`.


## Versioning & Workflow

- **Versioning:** Semantic Versioning (`vMAJOR.MINOR.PATCH`).
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `perf:`, `build:`).
- **Branches:** short-lived feature branches off `main`.
- **Releases:** tag with SemVer-aligned tags.


## Security & Misc

- No secrets in code.
- Secrets loaded only from environment variables.
- Sensitive values must not appear in logs.
- No internationalization requirements.
