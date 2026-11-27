# Development workspace

This is a working section for the current NistChemPy development cycle, not
part of the published user documentation.

- [Development plan](plan.md): approved goals, scope, work packages and
  acceptance criteria for NistChemPy and NistChemData.
- [GitHub issues](https://github.com/muCommons/NistChemPy/issues): concrete
  tasks, discussions and tracking.
- [Pull requests](https://github.com/muCommons/NistChemPy/pulls): implementation
  and review.
- [Existing development workflows](../source/development_workflows.rst):
  local checks, optional live tests, documentation builds and artifact checks.

## Working process

`main` is the stable default branch; `dev` is the integration branch.
Start each focused development task from the current `dev` and submit a PR
against `dev`. Both integration branches are protected. Prefer squash merges
for focused work; keep the history linear. The required deterministic check
is `offline` once enabled in repository settings. Live tests remain opt-in.

Before starting a task, read the relevant part of the approved plan and
any linked issue or applicable decisions. Record concrete scope, tests and
acceptance criteria in the issue or PR; avoid duplicate tracking documents.

A0 establishes the source catalog and synthetic fixtures with independently
specified expected outputs for the entire agreed scope. Add durable decision
records only when a consequential scientific, API or architecture decision
cannot be adequately recorded in the plan or an existing contract. Do not
create ADRs or additional layers merely for organizational symmetry.

Keep the existing Sphinx site and public documentation structure until A5.
Update affected technical descriptions when necessary for correctness, but
defer Zensical migration, navigation redesign and the custom light/dark
landing page until functional acceptance. Working materials will then be
integrated into the documentation or archived. No intermediate public
releases are planned for this development cycle.
