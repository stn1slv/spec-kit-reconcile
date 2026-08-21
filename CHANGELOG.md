# Changelog

All notable changes to the Reconcile extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.1] - 2026-08-21

### Compatibility with Spec Kit 1.0.0

A pass over the current Spec Kit conventions, prompted by the 1.0.0 release. The first item is a live defect rather than a modernisation: it broke the command on every agent that does not use the `/speckit.name` invocation form, which now includes the defaults for both Claude and Copilot.

- **Command references are agent-neutral.** The prompt hard-coded thirteen `/speckit.…` invocations, four of them inside the ERROR strings a user is meant to retype. Agents differ: Claude and Copilot install skills and invoke `/speckit-plan`, Codex uses `$speckit-plan`, Kimi uses `/skill:speckit-plan`. Spec Kit resolves the `__SPECKIT_COMMAND_<NAME>__` token per agent, and the prompt now uses it throughout, so every reference renders in the reader's own syntax.
- **The `py` script runtime is declared.** The frontmatter offered only `sh` and `ps`, so a project initialised with `--script py` fell back to the bash script instead of running the Python one it had installed.
- **Templates are read through the resolver, not the directory.** A template is a stack — a project override, then each installed preset by registry priority, then the base — and reading `.specify/templates/` directly returns only the bottom of it. Step 4 creates a missing section at the position the project's template defines, so on a project with a preset layered over the spec template it was placing sections by the wrong order. It now calls `resolve-template.sh`, falls back to the direct read when that is unavailable, and says under `## Sources` which route it took.
- **`requires.speckit_version` is honest.** It claimed `>=0.1.0` while calling scripts and reading state that arrived much later. It is now `>=0.16.2`, the release that introduced command-time template resolution.
- **The installed copy contains only the extension.** Added `.extensionignore`. Installing used to copy the whole repository, so `tests/fixture/` — a complete fake Spec Kit project, nested `.specify/` and `specs/` included — landed inside the user's own `.specify/extensions/`, along with `.git/` on a local install.
- **The manifest declares `category`, `effect` and `homepage`**, which until now existed only in the community catalog entry, and registers an **optional `after_implement` hook** so `/speckit.implement` offers the reconcile step when it finishes. It stays a prompt: the command needs a gap report and will not invent one. The command frontmatter also declares `handoffs` to `speckit.implement` and `speckit.plan`, matching the report's own Next Step routing.

### Fixed after multi-model review

A four-model review (Opus, Fable, Gemini Pro, Gemini Flash) over the branch, plus a documentation audit against Spec Kit 1.0.1. Every upstream integration claim checked out; what follows is internal contradictions the reviewers found, three of which this repository's own `BASELINE-v1.2.1.md` had already named as over-corrections from the previous pass.

- **The action list was short one action.** "Four actions exist, and each row takes exactly one" was followed by four rows, and the next paragraph then required a fifth, `None`, for a no-target row — the action the `Z` count counts. An agent reading the table as closed could not produce that row at all, which made `Z` permanently 0 and the rule that defines it unreachable. `None` is now in the table.
- **`Add` versus `Amend` is decided, and the example obeys it.** The action table said `Add` creates a new entry while the worked example marked a route the plan lacks as `Amend`; two runners published `amend 3; add 2` and `amend 4; add 1` for identical edits. The rule is now explicit — a new entry inside an existing section is `Add`, because the separator is the entry and not the section — and the example row was corrected to match.
- **The counts identity named the wrong cause.** "`M` can be larger than the row count (several items with no target)" cannot happen, since a no-target item takes exactly one row. The case that makes `M` larger is an item whose only output is a remediation task, which takes none. As written the two statements disagreed, and reconciling them invited the reading that breaks `Z`.
- **Wiring & Navigation covers a second clause.** The reachability test alone could not classify a route that ships, works, and is undeclared in the plan — the fixture's own Case H — so a run following the rule was graded a miss by an expectation that predated it. The category now has two clauses, reachability and undeclared routing, and the worked examples no longer duplicate the fixture's gap reports, which had reduced that trap to matching the command's own wording.
- **The resolver's three filenames are given rather than guessed.** Only the bash form was shown; the Python file is `resolve_template.py` with an underscore, and a guessed `resolve-template.py` fails silently into the direct-read fallback, defeating the fix. `plan-template` is named too, since the missing-section rule covers `plan.md`.
- **Block-allocated IDs have no undefined branches.** A task landing in `## Remediation: Gaps` belongs to no phase and so had no block, which is the common case on such a file, and a full block had no overflow rule. Both now take the next unused block.
- **Retirement markers have a recognition test** as precise as the bugfix markers beside them: upper-case, attached to the entry as an annotation, and prose when ambiguous — because a false marker silently drops an amendment.
- **The counts note has a defined slot** (a second line beginning `note:`), the counts line's appearance in both Step 3 and Step 5 is now stated as a deliberate preview/outcome pair whose disagreement is itself the divergence to report, and the `Skipped` action no longer contradicts a target definition that said "will write".
- **`specify extension update` does not work here and the README no longer claims it does.** The community catalog is discovery-only by design (`install_allowed: false`), so `update` skips every entry in it; `--from` with an explicit URL is the only supported upgrade path.
- **The `after_implement` hook prompt states the required arguments.** An optional hook renders "To execute" with no argument slot, so a user following it literally hit `ERROR: No feature spec directory provided`.
- Fixture: a `SUPERSEDED by FR-005` trap in 003 (R23) grades the redirect rule, which was shipping ungraded; Case E now grades the negative half of R19; and the block-allocation and resolves-to-nothing branches are recorded under `## Registered gaps` rather than left to look covered.

### Added

- **Retirement and supersession markers are read, never reversed.** Section 1.2 covered a bugfix extension's annotations; it now covers the `RETIRED` / `SUPERSEDED by [ID]` / `CANCELLED` markers a requirement-change command leaves behind, and is renamed to **Annotations left by other commands**. A retired entry is never revived, a superseded one redirects the amendment to its replacement, and a cancelled task is neither edited nor reused — each is reported under `## Outstanding Items` instead. On a project that has never run such a command the rule applies to nothing.
- **A Convergence phase is never a placement target.** `/speckit.converge` appends `## Phase N: Convergence` sections and treats them as an append-only record of its own findings. A remediation task written into one would be indistinguishable from a convergence finding on the next run, so 4.3 rule 2 now skips those headings and falls through to `## Remediation: Gaps`.
- **README states the boundary** against `/speckit.converge` and `/speckit.analyze`, which take opposite views of which side of the gap is wrong.

### Changed

- **Task IDs follow the file's own allocation convention.** "Highest `T###` plus one" assumed a dense sequence. Where a project allocates in blocks per phase, that rule puts a new task inside a later phase's block; the rule now reads the convention and takes the next free slot in the right block. ID width is read from the file rather than assumed to be three digits, and every existing ID counts toward the maximum — including completed, reopened and cancelled ones, since a retired ID is never handed out again.
- The test fixture's `.specify/extensions.yml` now matches what Spec Kit actually writes, and names the `bugfix` extension by its real id and hook.
- Three fixture traps cover the rules above, registered before the round that grades them: a `## Phase 3: Convergence` section in 001 whose ID counts but whose heading is not a placement target (R20), a `RETIRED` `REQ-003` in 002 that is the closest match to Case H's report and must therefore not be touched (R21), and a `CANCELLED` `T006` in 003 whose ID is still retired (R22). Cases A and I now expect new tasks from `T008` and `T007` respectively.

The fixes below came from **executing** the command against the test fixture v1.2.0 shipped, in five runs by five fresh agents. v1.2.0 had already been through three review rounds with three reviewers each — 64 findings, all applied — and none of those rounds found the first three items below. The round is recorded in `tests/fixture/BASELINE-v1.2.0.md`.

### Fixed

- **The impact map's row unit is the target, not the item-artifact pair.** All five runs hit this,
  each from a different direction: an item that adds a route and amends the testing strategy in one
  file (one row cannot be both `Amend` and `Add`), an item reaching an artifact a scope modifier
  excluded (the formula subtracted only withheld and no-target rows), a withheld item whose row had
  no defined cells, and — the ordinary case — an item touching routing, contracts and testing
  together, which the old unit could not represent at all. Every runner produced correct edits and a
  different published number, which is precisely what the counts exist to prevent. A row is now one
  entry in one artifact, every row carries exactly one of five actions, and `K + N + S + W + Z = R`
  holds exactly with every count but the item total in the same unit.
- **`Wiring & Navigation` is defined rather than illustrated.** The category was three examples, and
  it silently decides whether 4.3 rule 4's integration test is mandatory. Two runners split on it:
  a missing table column was read as Logic/UX and produced no test, while a progress component that
  never mounts was read as Wiring and produced one. The test is now **reachability of something that
  exists** — the capability is in the shipped code and a user cannot get to it — with both cases
  worked through, and a tie-break that classifies as Wiring when genuinely undecidable, because an
  unnecessary test costs a task and a missing one costs the guarantee.
- **Revision notes no longer nest inside whatever section happens to be last.** The form was fixed at
  `###` and the placement at "the bottom of the file", which makes the note render as a subsection of
  `## Assumptions` in a spec and `## Constitution Check` in a plan. The notes now live under a single
  `## Revisions` heading, and an artifact carrying bare entries from an older version has that heading
  added above them — the one retroactive edit this command makes, permitted because it moves no note,
  changes no text and changes no date.

### Testing

- Case A's A1 expectation was **wrong** and is corrected. It asserted that the drift adds a client
  route rather than an API route, so no constitution finding should appear; two runners independently
  flagged A1 anyway, and they were right, because the gap report's own words say the backend exists
  while the plan declares no such route. Review had already warned that this trap graded an inference,
  and the first fix asserted the opposite inference instead of removing the dependency on one — which
  turned an ambiguous expectation into a confidently wrong one that scored correct runs as failures.
- 003's fixture spec is made self-consistent: its revision note claimed to have reconciled a cancel
  affordance that `FR-003` never mentioned. `FR-003` now states what the note says was written.
- Three traps registered for the rules above (R17 revision-note nesting, R18 the target row unit,
  R19 the reachability test), with 003's two artifacts deliberately carrying the old and new note
  forms so the retroactive-add rule is exercised on one and not the other.

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
  the task names its target without one and the omission is reported. A task that creates a new file
  is the one case a not-yet-existing path is correct, and only when the directory it sits in is one
  the plan declares or the repository already contains: that path is derived, not invented. Git
  history, deleted files,
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
  missing statement is never invented by this command — the honest remedy is a remediation task
  directing the author to make it. The one exception is an obligation this run's own amendment
  satisfies: where the gap report supplies the content and the run will write it where the rule
  names, the rule is met by that amendment rather than flagged, because a rule asking for a route to
  be declared in the plan describes exactly the edit the run is already making. An action rule is
  reported as unverified and never flagged: a
  claim is not a verification, so a plan saying nothing, a plan claiming coverage, and a list that
  omits one route are equally unverified.
- **A target table in the impact map, and counts.** 4.1 said to amend "the `FR-XXX` that states the
  changed capability, or add one", which is a judgment made silently while writing; the same input
  could land on a different entry twice. The map now carries one row per item-artifact pair naming
  the target it will amend — by ID, else by quoted heading or opening phrase, never by a bare section
  name — and Step 4 writes exactly that table. `tasks.md` takes no rows, since a remediation task is
  always new and has no existing entry to preview. The counts make a zero legible as "examined and found
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

  **Two invocation shapes change behaviour**, both the price of rejecting input that used to be
  improvised. First, a `--` token sitting immediately after the feature path that is not one of the
  three modifiers is now a fatal error, where v1.1.0 read it as gap report prose: `specs/007 --force
  is now required by the deploy script` ran before and stops now; write it as `specs/007 the deploy
  script now requires --force` instead. Everywhere else a `--` word is still prose, trailing
  position included. Second, a `specs/###` path **anywhere** in the input is now read as a second
  feature and rejected, so a gap report that referred to another feature by path — `the digest route
  from specs/002-notifications is missing here too` — ran before and stops now; refer to it by name
  in prose instead. Both are deliberate, and both are the cost of the one-feature-per-run rule
  being enforced rather than assumed.
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
