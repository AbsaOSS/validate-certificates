# Copilot Review Rules

Purpose

- Define consistent review behavior and response formatting for Copilot code reviews.
- Keep reviews concise and action-oriented.

Writing style

- Use short headings and bullet lists.
- Prefer do/avoid constraints over prose.
- Make checks verifiable (point to code and impact).
- Keep responses concise; avoid long audit reports unless requested.

Review modes

- Default review: standard PR risk.
- Double-check review: elevated risk PRs.

Mode: Default review

- Scope
  - Single PR, normal risk.
- Priorities (in order)
  - correctness → security → tests → maintainability → style
- Checks
  - Correctness
    - Do highlight logic bugs, missing edge cases, regressions, and contract changes.
    - Do flag bash quoting issues, unbound variable risks, and missing `set -uo pipefail`.
  - Security & data handling
    - Do flag unsafe input handling, secrets exposure, and injection risks via `inputs.*` in shell.
  - Tests
    - Do check that manual smoke-test steps or reproduction cases exist for changed logic.
  - Maintainability
    - Do point out unnecessary complexity, duplication, and unclear naming/structure.
  - Style
    - Avoid style notes unless they reduce readability or break repo conventions.
- Response format
  - Use short bullet points.
  - Reference files + line ranges where possible.
  - Group comments by severity: Blocker (must fix), Important (should fix), Nit (optional).
  - Do provide minimal, actionable suggestions (what to change), not rewrites.
  - Avoid rewriting the whole PR or producing long reports.

Mode: Double-check review

- Scope
  - Higher-risk PRs (security, infra/CI, wide refactors, cert parsing changes).
- Additional focus
  - Do confirm previous review comments were addressed (when applicable).
  - Do re-check high-risk areas: shell injection via inputs, date parsing portability, exit code contract.
  - Do look for hidden side effects: backward compatibility, failure modes on malformed certs.
  - Do validate safe defaults: warning_days default, empty array handling, openssl error paths.
- Response format
  - Only add comments where risk/impact is non-trivial.
  - Avoid repeating minor style notes already covered by default review.
  - If leaving something as-is, call out risk acceptance explicitly:
    - what risk
    - why acceptable
    - mitigation

Commenting rules (all modes)

- Always include:
  - What is the issue (1 line)
  - Why it matters (impact/risk)
  - How to fix (minimal actionable suggestion)
- Prefer linking to existing patterns in the repo over introducing new ones.
- If uncertain (missing context), ask a targeted question instead of assuming.

Non-goals

- Avoid requesting refactors unrelated to the PR's intent.
- Avoid bikeshedding formatting if conventions are already established.
- Avoid architectural rewrites unless explicitly requested.

Repo additions

- Domain-specific high-risk areas
  - Shell injection: `inputs.certificates` is passed directly into bash — verify it is only used via `jq` parsing, not `eval` or direct interpolation in sensitive contexts.
  - Date portability: GNU `date -d` vs BSD `date -j -f` — both must be present with `||`.
  - Exit code contract: exit 1 only on unrecoverable failure (no valid replacement); exit 0 with warnings otherwise.
  - Step Summary: changes to summary format are user-visible and should be treated as contract changes.
- Required checks
  - Verify smoke test on ubuntu-latest and macos-latest.
  - Verify empty-array guard path.
