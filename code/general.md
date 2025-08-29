# General Coding Convention

This convention is opinionated and pragmatic. It favors one clear default per concern, consistent structure, and secure, reliable practices that scale across languages. Deviation is exceptional, justified in review, and documented near the exception.

## 1. Philosophy

- Opinionated minimalism: pick a single default per concern (formatter, linter/static analysis, test framework, logger, HTTP/client, dependency manager) for each language; stick to it.
- Clarity over cleverness: optimize for readers; prefer explicit types/units, descriptive names, straightforward control flow, and shallow nesting.
- Separation of concerns: keep domain logic pure and testable; isolate I/O (CLI, network, filesystem, environment) behind thin adapters; handle errors at boundaries.
- Reliability by default: apply timeouts, safe retries, and input validation on every external call; deterministic builds; pinned dependencies; UTC and lossless serialization; secure defaults over options.
- Human-first ergonomics: predictable CLIs; readable local logs with opt-in structured output; fail fast with actionable errors; sensible verbosity.
- Constraints as leverage: limit supported platforms and "blessed" patterns so solutions converge; consistency beats personal preference.

## 2. Foundations

- Supported languages and minimum versions are explicit per repository; prefer one active major version per language.
- Supported platforms: macOS and Linux only (Windows not supported).
- Tooling is single-choice per language and enforced in CI: one formatter, one linter/static analyzer, one type checker (if applicable), one test framework, one dependency manager.
- Line length: choose a single target (e.g., 100-120) and enforce via the formatter.

## 3. Repository Layout

```
<repo>/
  bin/                # CLI entrypoints (executable; shebang where applicable)
  src/ or <module>/   # application/library source (one root per language)
  tests/              # mirrors src/ structure for tests
  .github/workflows/  # CI/CD definitions (GitHub Actions)
  README.md           # usage, setup, decisions
  CHANGELOG.md        # SemVer-aligned changes
  LICENSE             # license file (if applicable)
  <lockfiles>         # dependency lock (e.g., *lock)
```

- Executables: live in `bin/`, extensionless where the platform allows; include a proper shebang.
- Imports/modules: prefer explicit, stable paths; avoid fragile relative imports.

## 4. Code Structure

- Entrypoints: a single `main`/bootstrap per binary; orchestration at top, helpers below; group related code.
- Function and method size: keep small (~40-50 lines) or low complexity; extract helpers over deep nesting.
- Types and models: separate domain models from external/IO models; use enums for discrete sets.
- Control flow: prefer early returns and simple patterns; avoid nested comprehensions and overly clever lambdas.

## 5. CLI & Configuration

- Arg parsing: pick one library per language; flat flags for simple tools, subcommands for complex trees.
- Configuration precedence: CLI > environment variables > config files (only when necessary).
- Output: default human-readable text; support `--verbose/--quiet`. Only tools expected to be automated must implement `--output-format json` (otherwise text-only is fine).

## 6. Logging & Errors

- Logging: human-friendly locally; switchable to structured JSON. Include UTC ISO‑8601 timestamps with `Z` suffix. Use consistent log levels.
- Error handling: define custom error types for known failure modes. Let exceptions propagate to boundaries (CLI, web handlers, worker loops). Only catch broad exceptions at boundaries; exit with consistent codes.

## 7. Data, Time, and Serialization

- Units in names: suffix variables with ISO/standard units (e.g., `_seconds`, `_bytes`, `_meters`). Use standard abbreviations only (e.g., `_ms`, `_km`).
- Time: store/process in UTC; prefer timezone-aware types. Serialize timestamps as ISO‑8601 with `Z`.
- Serialization: lossless by default; text for humans, JSON (or equivalent) for machines.

## 8. Networking & External I/O

- HTTP/remote clients: centralize per service with sane defaults:
  - Timeout: 10 seconds baseline.
  - TLS verification: enabled.
  - Retries: 3 with exponential/backoff for transient statuses (e.g., 429, 500, 502, 503, 504).
  - Descriptive `User-Agent` and correlation identifiers where applicable.
- Filesystem/env: explicit encodings (UTF‑8); avoid implicit reliance on CWD; validate inputs; least-privilege file permissions.
 - External systems (Slack, RabbitMQ, etc.): encapsulate each integration behind a dedicated Facade (SOLID) that exposes a narrow, domain‑oriented interface; forbid direct SDK/client usage outside the Facade; handle timeouts/retries, request/response mapping, and error translation inside the Facade.

## 9. Style & Documentation

- Naming follows language idioms; do not impose cross-language defaults.
- Comments/docs: explain intent, constraints, tradeoffs, and invariants; document units and pre/postconditions; avoid restating code; include minimal runnable examples where useful.

## 10. Testing

- Framework: single choice per language. Tests live in `tests/` and mirror `src/` structure.
- Test design: parametrize edge cases; use fixtures/builders; avoid global/shared mutable state; deterministic by default.
- Markers/tags: support `slow` and `network` (skipped unless explicitly opted-in via env/flag).
- Coverage: not a hard gate by default; use as a signal, not a target.

## 11. Dependencies

- Management: one tool per language; declare direct dependencies; maintain a lockfile for reproducibility; deterministic builds.
- Policy: minimize runtime dependencies; review transitive risk; vendor only with justification; update cadence is team-defined with changelog review recommended.

## 12. CI/CD

- CI provider: GitHub Actions only; workflows in `.github/workflows/`.
- Baseline pipeline:
  - Install dependencies with the project's dependency manager.
  - Run formatter in check mode.
  - Run linter/static analysis.
  - Run type checks (when applicable).
  - Run unit tests; cache dependencies where possible.
- Gates: fail on formatting/lint errors and failing tests; optional security scans (SCA, secret detection).
- Reproducibility: fix tool versions; strive for hermetic builds.

## 13. Versioning & Workflow

- Versioning: Semantic Versioning (`MAJOR.MINOR.PATCH`) for libraries and CLIs.
- Commits: Conventional Commits (e.g., `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `perf:`, `build:`).
- Branching: short-lived feature branches off `main`; protect `main`.
- Releases: tag with SemVer; generate changelogs from commits.

## 14. Security

- Secrets: never in code or logs. Load from environment or a secret manager; redact by default.
- Data handling: least privilege for tokens/keys; avoid logging PII; review external interfaces regularly.
- Supply chain: enable dependency alerts; pin/verify checksums when supported.

## 15. Deviation Policy

- Exceptions require rationale, a local note (README/comment) near the code, and a linked issue. Assign an owner and a timebox to revisit.

## 16. Per‑Language Mapping (Guidance)

- No pre-filled tool choices: teams select and document per repository; always defer to language idioms.
- For each language used in the repo, document in `README.md` the chosen:
  - Formatter
  - Linter/static analyzer
  - Type checker (if applicable)
  - Test framework and runner
  - Dependency manager and lockfile
  - Logger/observability approach (text vs JSON switch)
- Enforce these via CI and minimal editor config.

## 18. Org Defaults (This Repository)

- Platforms: macOS and Linux only; Windows not supported.
- Languages & tooling: do not pre-fill tool choices; teams choose per repo; always defer to language idioms.
- CLI output: only tools expected to be automated must implement JSON mode; others may be text-only.
- Testing: coverage is advisory only; do not gate on a minimum by default.
- Dependencies: no org-wide update cadence; teams decide and review changelogs.
- CI/CD: GitHub Actions only; store workflows in `.github/workflows/`.
- Compliance: no SOC2-like constraints at this time.
- Repos: no multi-language monorepos; prefer single-language repositories.
