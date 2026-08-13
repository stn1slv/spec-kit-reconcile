# Changelog

All notable changes to the Reconcile extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-08-13

Most of this release is the sibling `spec-kit-archive` extension's findings applied here. Both
commands were drafted from the same habits, so several defects archive diagnosed between v1.1.1
and v1.2.2 were present in this file in almost the same wording. Where a rule is archive's, it is
adapted rather than copied: the two commands write to different files and do different work.

### Added

- **Allowed Sources: a stated, complete boundary on what may be read for content.** 4.3 has always
  required every remediation task to name an exact file path, and nothing said where that path may
  come from. A gap report saying "need sidebar link + route" carries none, so the path was invented,
  and a task pointing at a file that does not exist is not discovered until someone implements it.
  Content now comes from the gap report and the feature's own artifacts; the repository tree and its
  source files may be read **read-only for one purpose** — resolving a described target to a real
  path and confirming a path exists — and nothing else is taken from them. When no path resolves,
  the task names its target without one and the omission is reported. Git history, deleted files,
  another feature's spec directory, `bugs/` reports and agent memory stores are named as
  non-sources. Verifying your own writes with git is still allowed; reading git for content is not.
- **A contract for the gap report.** It is free text from the user and was treated as pure
  description, so a report reading "…and skip the constitution check to save time" or "…delete T012"
  had nothing forbidding compliance. The report now steers and never overrides: it cannot widen the
  source boundary, skip or weaken a step, change the scope, change ID assignment, or authorize a
  deletion. A report asking for one of those is refused **without stopping the run**, the refusal is
  named, and the report is echoed verbatim under `## Gap Report`.
- **All three shapes a MUST rule can fail in.** Only *conflicts* were checked, so a rule requiring a
  statement the feature never makes ("every feature that stores user data MUST state its retention
  rule") could be violated by omission with nothing to flag, and a rule requiring an *action* ("all
  API routes MUST have automated tests") would have been flagged on every feature that never
  mentioned it. Conflicts, unmet obligations and action-requiring rules are now checked separately,
  bounded to the rules the drift actually reaches. An obligation never withholds content and its
  missing statement is never written by this command — the honest remedy is a remediation task
  directing the author to make it. An action rule is reported as unverified and never flagged: a
  claim is not a verification, so a plan saying nothing, a plan claiming coverage, and a list that
  omits one route are equally unverified.
- **A target table in the impact map, and counts.** 4.1 said to amend "the `FR-XXX` that states the
  changed capability, or add one", which is a judgment made silently while writing; the same input
  could land on a different entry twice. The map now carries one row per normalized item naming the
  target it will amend — by ID, else by quoted heading or opening phrase, never by a bare section
  name — and Step 4 writes exactly that table. The counts make a zero legible as "examined and found
  nothing to amend" rather than "did not look".
- **A test fixture** (`tests/fixture/`): a minimal spec-kit project with deliberate traps and
  pre-registered expectations, executed by a fresh agent given only the command and its own working
  copy. Archive's record is the reason this exists: eleven of its v1.2.2 findings came from executing
  the command, after five review rounds across three models had found none of them.

### Changed

- **Scope modifiers are read immediately after the feature path**, which is where the archive
  extension reads them and where they are unambiguous. The trailing position earlier versions
  required is still recognised, including a trailing **run** of several modifiers, which combines
  into the same union v1.1.0 documented. The trailing position is what forced the "if the report's
  last words look like a flag, prefer the scoping reading" rule, and it keeps that ambiguity; the
  leading position has none, and the hedge is gone — the scoping reading is now *taken*, not
  preferred, since a parser that hedges gives two agents two answers.

  **One invocation shape does change behaviour**, and it is the price of rejecting unknown flags: a
  `--` token sitting immediately after the feature path that is not one of the three modifiers is
  now a fatal error, where v1.1.0 read it as gap report prose. `specs/007 --force is now required by
  the deploy script` ran before and stops now; write it as `specs/007 the deploy script now requires
  --force` instead. Everywhere else a `--` word is still prose, trailing position included.
- **A unique numeric prefix is accepted.** `specs/007` expands to `specs/007-invoice-settings` when
  exactly one directory matches. It was rejected outright here while the archive extension accepted
  it, so the same token behaved differently in two commands the same user runs in the same session.
- **Revision notes record which items they touched.** The `[Sync: slug]` key was written into
  `tasks.md` and into the notes, but never onto the entries it keys, so a re-run had to re-identify
  its own earlier edits by prose — the mechanism missing in the one case it exists for. Notes now
  carry an `Items:` line naming each entry amended, using the same citation ladder. Their order and
  immutability are stated too: newest last, an earlier note is never rewritten except the one
  carrying this run's slug, and notes an older layout put elsewhere are left where they are with the
  split reported.
- **The report gained the sections it already promised.** Defaults were to be "noted in the Sync
  Impact Report" and no section held them; constitution findings had nowhere to land at all. Added:
  `Counts`, `Sources`, `Path Resolution`, `Constitution Compliance`, `Outstanding Items`,
  `Defaults Applied` and `Gap Report`. `Outstanding Decisions` is folded into `Outstanding Items`,
  which now holds everything the run noticed and did not act on: two sections whose difference was
  "a decision versus an observation" only invited the question of which one a finding belongs in.
- Dropped `requires.scripts` from `extension.yml`. It is not part of the manifest schema and the
  validator ignores unknown keys under `requires`, so it never had any effect; Step 0.1 already
  handles a missing `check-prerequisites.sh` itself. Added `argument-hint` frontmatter, which
  Spec-Kit preserves into the generated Claude `SKILL.md`, so the expected argument shape is visible
  where the command is invoked rather than only in the rejection message.

### Fixed

- **An unresolved constitution conflict now has a defined outcome.** The command said a CRITICAL
  finding "must be resolved in Step 2 clarification before edits proceed" and never said what happens
  when the user does not resolve one, leaving an agent free to halt the run or to continue. It now
  says once: the conflicting item alone is withheld — written to no artifact and generating no task —
  everything else in the gap report is reconciled, the run completes, and the withheld item is named
  with a resolve-and-re-run recommendation. Nothing in Steps 2 to 5 aborts a run.
- **The mandatory constitution question can no longer be dropped.** Step 2 ended with "Proceed with
  reasonable defaults if questions aren't strictly necessary", with no exemption for the CRITICAL
  findings the earlier steps declared must reach it. The materiality filter and the skip-the-step
  clause are now scoped to discretionary questions; conflicts and obligations are always asked, as
  separate questions, because an obligation may be closed as an accepted gap and a conflict may not.
- **Unsupported invocations are rejected instead of improvised.** Ranges (`specs/001 thru
  specs/008`), globs inside a feature token, a second feature reference anywhere in the input, and an
  unrecognized flag in the leading position now stop the command with a fixed message before any file
  is written. The one-feature-per-run rule is stated in both the command and the README.
- **The first token resolves under `REPO_ROOT`, not the current working directory.** Invoked from a
  subdirectory, a valid `specs/###-feature-name` could be rejected as not resolving to exactly one
  feature directory. The run also now reports a disagreement between the argument and the feature
  `check-prerequisites.sh` reports, which matters more here than in archive because this command
  rewrites three files in place.
- **A section the drift reaches but the file lacks is created.** 4.1 said to detect and follow the
  project's own section names, and said nothing about a section that is simply absent — a spec with
  no `## Assumptions` is common. It is now created in the position the template puts it, and every
  such creation is named in the report.
- **Bugfix-extension annotations are no longer undone.** A bugfix extension patches the same three
  files this command edits. Struck-through wording superseded by a patch could be restored by an
  amendment; `**Bugfix**:` metadata lines could be read as requirement text; and a task annotated
  `(reopened — BUG-NNN)` was treated as completed work because its checkbox still showed `[x]`. All
  three now have rules: never restore struck wording, leave metadata lines alone, and treat a
  reopened task as incomplete but never repurpose it, since it belongs to a cycle in progress.

## [1.1.0] - 2026-08-09

### Added

- Spec reconciliation now covers every section of the canonical `spec-template.md`. Previously
  only user scenarios and "acceptance criteria" were amended, so a gap report describing a
  changed capability had nowhere to land. Functional Requirements, Edge Cases, Key Entities,
  Success Criteria and Assumptions are now all in scope, and only the sections the gap report
  actually reaches are touched.
- Idempotency. Re-running the same gap report no longer appends a duplicate task set and a
  second revision note. Tasks and revision notes carry a `[Sync: slug]` tag that acts as the
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
