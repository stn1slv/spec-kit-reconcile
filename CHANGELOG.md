# Changelog

All notable changes to the Reconcile extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-08-09

### Fixed

- Remediation tasks are no longer marked `[P]`. The command described `[P]` as a priority flag
  for blocking or high-urgency work, but in Spec-Kit `[P]` means "can run in parallel: different
  files, no dependencies", and `/speckit.implement` reads it to decide which tasks to run
  together. Blocking tasks were therefore being written into `tasks.md` as safe to parallelize.
  A gap report cannot establish file-level independence, so the marker is now never emitted.
- New tasks are placed under the phase heading that already exists in `tasks.md`
  (`## Phase N: User Story N - [Title] (Priority: PN)`). The previous example showed
  `## [US2] Settings Dashboard`, which is the inline task tag rather than a heading, so
  following it created a second, differently named section instead of matching the existing one.
- Spec edits now target the sections the canonical `spec-template.md` actually defines.
  The command referred to an "Acceptance Criteria" section and an `AC-04` style ID, neither of
  which exists in the template; the real structure is `**Acceptance Scenarios**` in
  Given/When/Then form under each `### User Story N`. The command now also detects the
  project's own section names and ID convention instead of assuming them.
- Dropped `--include-tasks` / `-IncludeTasks` from the script invocation. With `--paths-only`,
  `check-prerequisites.sh` returns before the flag has any effect, and the tasks path is
  emitted regardless.

## [1.0.0] - 2026-03-14

### Added

- Initial release of the Reconcile extension
- Command: `/speckit.reconcile.run` — post-implementation gap closer for feature artifacts
- Natural-language gap report parsing and normalization
- Gap categories: Wiring & Navigation, Contracts, Acceptance Criteria, Test Coverage, Logic/UX
- Surgical updates to feature's own `spec.md`, `plan.md`, and `tasks.md`
- Remediation task generation with auto-incremented `T###` IDs and exact file paths
- Mandatory integration test tasks for wiring/navigation gaps
- Constitution compliance validation against `.specify/memory/constitution.md`
- Scope modifiers (`--spec-only`, `--plan-only`, `--tasks-only`)
- Sync Impact Report with conditional next-step routing
