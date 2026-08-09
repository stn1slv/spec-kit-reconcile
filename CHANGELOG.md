# Changelog

All notable changes to the Reconcile extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-08-09

### Added

- Spec reconciliation now covers every section of the canonical `spec-template.md`. Previously
  only user scenarios and "acceptance criteria" were amended, so a gap report describing a
  changed capability had nowhere to land. Functional Requirements, Edge Cases, Key Entities,
  Success Criteria and Assumptions are now all in scope, and only the sections the gap report
  actually reaches are touched.
- Idempotency. Re-running the same gap report no longer appends a duplicate task set and a
  second revision note. Tasks and revision notes carry a `[Sync: ... slug]` tag that acts as the
  re-run key; the slug is reused when an existing tag names the same drift, so a refined report
  still matches. A task `/speckit.implement` has marked `[X]` is never edited, and no task from
  an earlier run is ever deleted.
- The Sync Impact Report gained a `Scoping` section, and recommends re-running
  `/speckit.archive.run` when the reconciled feature has already been archived, since these
  edits leave project memory stale.

### Changed

- **Breaking:** the feature directory is now a required first argument
  (`/speckit.reconcile.run specs/###-feature-name "gap report"`). It previously came from
  `check-prerequisites.sh`, which resolves whichever feature was worked on last. Since this
  command rewrites three files in place, it no longer infers which feature you meant. The path
  must resolve to exactly one existing directory.

### Fixed

- Scope modifiers are now enforced. `--spec-only`, `--plan-only` and `--tasks-only` were
  declared in the input parsing and then never referenced again, so they had no defined effect
  on what was written. They are now honored across all edits, combine as a union, are matched
  as whole trailing tokens rather than as substrings of the gap report, and are reported.
- `tasks.md` is no longer created during the Step 0 gate. It was created before the clarification
  pause and before the impact map that promises a preview before any edit, so abandoning the run
  left a stray file behind. It is now created in 4.3, only when in scope and only when there are
  tasks to write.
- A non-zero exit from `check-prerequisites.sh` is handled instead of being undefined.

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
