Copilot instructions

Purpose
- Must define a consistent style for .github/copilot-instructions.md.
- Must keep externally-visible behavior stable unless intentionally changing the contract.

Structure
- Must keep sections in the same order across repos.
- Prefer bullet lists over paragraphs.
- Must use constraint words: Must / Must not / Prefer / Avoid.
- Must keep one blank line at end of file.

Context
- Must support a GitHub Action / CI automation context.
- Must treat environment and GitHub I/O as boundary concerns.
- Action is a bash composite action — no compiled artifacts.

Coding guidelines
- Must keep changes small and focused.
- Prefer clear, explicit bash over clever one-liners.
- Must keep pure logic free of environment access where practical.
- Must keep externally-visible strings, summary output, and exit codes stable unless intentional.

Output discipline (reduce review time)
- Must default to concise responses (aim ≤ 10 lines in the final recap).
- Must not paste large file contents/configs/checklists; link and summarize deltas.
- Prefer actionable bullets over prose.
- When making code changes, must end with:
  - What changed
  - Why
  - How to verify (commands/tests)
- Avoid deep rationale unless explicitly requested.

PR Body Management
- Must treat PR description as a changelog.
- Must not rewrite or replace the entire PR body.
- Must append updates chronologically under a new heading.
- Prefer headings: "## Update [YYYY-MM-DD]" or "## Changes Added".
- Must reference the commit hash that introduced the change.

Language and style (Bash)
- Must target bash 5+ (associative arrays, mapfile, etc.).
- Must use `set -uo pipefail` at script start.
- Must quote all variable expansions unless intentionally word-splitting.
- Must use `local` for all function-scoped variables.
- Must not use `eval` or dynamic variable names.
- Prefer `[[ … ]]` over `[ … ]`.
- Prefer `$(…)` over backticks.
- Must include the standard YAML copyright/license header in action.yml.

Documentation
- Must keep README.md up to date with all inputs and their defaults.
- Must keep usage examples pointing to `AbsaOSS/validate-certificates@v1`.
- Must keep the status-legend table accurate.

Quality gates
- Manual smoke test: run action locally with a sample cert and verify log + exit code.
- Verify on ubuntu-latest (GNU date) and macos-latest (BSD date).

Common pitfalls to avoid
- `date` portability: GNU `-d` vs BSD `-j -f`; always provide both forms with `||`.
- jq availability: always assume jq is present on GitHub-hosted runners.
- Empty cert array: must guard with length check and exit 1.
- Stability: avoid changing externally-visible summary strings or exit codes.

Repo additions
- Project name: validate-certificates
- Entry point: action.yml
- Core logic: inline bash in action.yml `runs.steps[0].run`
- Contract-sensitive outputs: GitHub Step Summary table; exit code 0/1; `::warning` annotations

