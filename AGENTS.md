# Agent entry point

- Use English for all repository-facing text, including code comments,
  docstrings, documentation, fixtures, commit messages, issues, PRs and reviews.
  Private planning conversations may use other languages.
- Check the current branch and baseline, linked task, and approved scope.
  Read [the working documentation](docs/development/README.md),
  [the active development plan](docs/development/plan.md), and applicable
  decisions before implementation.
- Start focused work from the current `dev` and use a PR targeting `dev`.
  Do not push directly to protected branches or publish releases as part of
  ordinary development.
- Preserve public behavior unless a task explicitly approves a change.
  Prefer simple changes to existing modules; introduce abstractions and
  dependencies only to solve a demonstrated problem. Escalate unresolved
  scientific contracts, rights assumptions, breaking API changes and
  material scope changes.
- Keep HTTP and parsing separable. Use synthetic fixtures and independently
  specified expected results. Preserve source and rights evidence, distinguish
  absence of data from failed extraction, and do not silently classify unknown
  rights as permission. Do not commit downloaded WebBook data or bulk archives.
- Install with `python -m pip install -e ".[dev]"`; run focused tests and
  `python -m pytest -q` as appropriate. Default checks must be offline;
  live network tests are opt-in. Report actual commands, outcomes, skips
  and limitations. Never claim unobserved CI success.
- Keep issues and PRs as the progress tracker; update the plan for substantive
  scope or acceptance changes. Record ADRs only when necessary. Do not mark
  unfinished work or unapproved decisions as accepted.
- Defer documentation migration and redesign to A5. Maintain the correctness
  of affected descriptions and examples during implementation.
