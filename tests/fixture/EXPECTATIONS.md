# Fixture Test Cases and Expected Outcomes

The fixture in `project/` is a minimal three-feature spec-kit project with a real (if stubbed) source tree beside its specs, so the path-resolution rules have something true to resolve against. Every trap below is deliberate.

A test runner is a **fresh agent given only `commands/reconcile.md` and its own working copy of `project/`**, forbidden from reading this file, the CHANGELOG, or git history. It executes one case per run against a clean copy; results are compared to this file, and the round is recorded in a `BASELINE-v*.md` beside it.

**Pre-registration rule**: the expectations for a round must land in a commit *before* that round's runs, so the claim "written first" is auditable. The sibling `spec-kit-archive` fixture records what happens without it — a round whose expectations landed in the same commit as its results proves nothing about what was predicted.

**Nothing-written rule**: for every rejection case, `git status` in the working copy must be clean afterwards. A rejection that leaves a stray file is a failure even when the error message is right.

## Traps built into the fixture

| # | Trap | Location | What it tests |
|---|---|---|---|
| R1 | `feature.json` points at `003-attachments` | `.specify/` | Argument-vs-script precedence and the `## Path Resolution` line that reports the divergence |
| R2 | 002 uses its own section names (`## Behaviour Rules`, `### Things We Store`, `## How We'll Know It Works`) and a `REQ-###` / `M#` ID convention | 002 spec.md | Detect-and-follow the project's own conventions; canonical headings must not be invented beside them |
| R3 | 002 has no `## Assumptions` section, and the Case H report invalidates an assumption | 002 spec.md | The missing-section rule: create it in the template's position, and name the creation |
| R4 | 001's tasks are all `[X]`, three of them carrying a legitimate `[P]` | 001 tasks.md | New tasks never carry `[P]`, existing `[P]` is never stripped, no `[X]` task is edited |
| R5 | T004 keeps `[x]` while annotated `(reopened — BUG-002)` | 003 tasks.md | A reopened task is incomplete but still not this command's to edit or repurpose |
| R6 | FR-002 carries `~~struck~~` local-disk wording, a live replacement, and a `**Bugfix**:` line; one **orphan** strikethrough (no marker, no replacement) sits in Edge Cases | 003 spec.md | Struck wording is never restored by an amendment; metadata is not content; the orphan is left as it stands and named |
| R7 | Principle I forbids shared or unassigned tasks, and the Case E report asks for co-ownership | constitution.md + Case E | Conflict branch: flagged CRITICAL, asked separately, and unresolved means *that item only* is withheld |
| R8 | Principle II requires a retention statement **in the spec**; A1 requires a route to be **declared in the plan** and to **have an integration test** | constitution.md | All three shapes: statement, statement-plus-action in one sentence, and the action half reported as unverified rather than flagged |
| R9 | A gap report ending in a flag-like token | Case J2 invocation | Trailing-modifier compatibility and the stated-interpretation rule |
| R10 | A gap report instructing the run to skip the constitution check and delete a task | Case F invocation | The Gap Report Contract: refuse, continue, name the refusal, echo verbatim |
| R11 | Ranges, globs, a second `specs/` path, an unknown flag, a bare path with no report, a prefix matching nothing | Case G invocations | Every rejection class, and that nothing is written |
| R12 | 002 has no `tasks.md` | 002 | Created in 4.3 only, and only because tasks are actually written |
| R13 | 003 carries `[Sync: upload-progress]` in both revision notes and on T005 | 003 | Slug reuse on a refined report; notes amended in place with their original date, never duplicated |
| R14 | A wiring gap invoked `--spec-only` | Case B invocation | The mandatory integration test is omitted **and named** under `## Scoping`, not silently skipped |
| R15 | 001 already has a Merged Features Log entry | memory/changelog.md | The Next Step routes to `/speckit.archive.run`, because the archived copy is now stale |
| R16 | The Case A report names "the settings service", which matches no file and no plan structure entry | Case A invocation + `src/` | No path is invented; the task is written without one and the omission is named |

**The sharpest trap is R6 combined with `bugs/BUG-002.md`.** That report's `## Proposed Amendment` states exactly the right amendment for FR-003, and it is a forbidden source. Producing the same wording from the gap report is correct; taking it from the bug report, citing it, or listing it under `## Sources` is a miss.

---

## Case A — 001, full scope, clean fixture

```
/speckit.reconcile.run specs/001-task-manager Backend and tests for the settings screen exist and the React screen is scaffolded, but users cannot navigate to it: it needs a sidebar entry and a route. Separately, the retention sweep that deletes archived tasks now runs in the settings service instead of the nightly job.
```

- **R1**: `## Path Resolution` states that `check-prerequisites.sh` reported `specs/003-attachments` and the argument won.
- Step 3 prints the impact map **and** the target table before any edit, with the counts line. Editing before the preview is a miss even if the edits are right.
- `spec.md`: the wiring item reaches an acceptance scenario or a new `FR-006`; the retention-sweep item amends `FR-005` or the Configuration-related requirement. IDs continue from `FR-005`; renumbering anything existing is a miss.
- `plan.md`: `Routing & Navigation` gains the client `/settings` route; Configuration or Testing Strategy may be amended for the sweep move.
- **R8, both halves**: adding the route to the plan's Routing section is what A1's *statement* half asks for, so no obligation flag. A1's *action* half (an integration test before merge) is **action-requiring**: reported under `## Outstanding Items` as unverified, never CRITICAL, never a question. Principle II is **satisfied** — 001's FR-005 states the retention rule — so a constitution flag on this case is a miss.
- `tasks.md`: new tasks numbered from **T007**, none carrying `[P]`, the existing `[P]` on T002/T003/T005 untouched, no `[X]` task edited. A wiring gap was identified, so an integration test task is mandatory.
- **R16**: the sidebar task resolves to `src/components/Sidebar.tsx` and the route task to `src/router/index.ts`; `## Sources` declares the read-only lookup that found them. The "settings service" resolves to nothing — no such file exists and 001's Project Structure does not name one — so its task is written **without a path** and named under `## Outstanding Items`. Inventing `src/services/settings.py` is the failure this trap exists for.
- Revision notes at the bottom of `spec.md` and `plan.md`, each carrying one slug and an `Items:` line naming the entries touched.
- **R15**: the Next Step recommends `/speckit.archive.run specs/001-task-manager`.
- The report carries every section its template defines, including `## Gap Report` with the text verbatim.

## Case B — 001, `--spec-only`, clean fixture

```
/speckit.reconcile.run specs/001-task-manager --spec-only Backend and tests for the settings screen exist and the React screen is scaffolded, but users cannot navigate to it: it needs a sidebar entry and a route.
```

- Only `spec.md` is written. `plan.md` and `tasks.md` are **byte-for-byte unchanged**, and no `tasks.md` entry is added anywhere.
- **R14**: `## Scoping` names the omitted mandatory integration test explicitly, and names the routing detail that belonged in `plan.md`. A report that simply says "tasks.md skipped" without naming the omitted test is a miss.
- Both files were still **read** — out of scope means not written, never not read — and the spec amendment reflects what the plan says.

## Case C — repeat Case A on its own end state, same report

- No file changes at all. `## New Remediation Tasks` says the tasks are already present; the counts show nothing new.
- No second revision note, no second task set, no date change on the existing note.

## Case D — refined report on Case A's end state

```
/speckit.reconcile.run specs/001-task-manager The settings screen is still unreachable: the sidebar entry landed but the route was never registered.
```

- **The slug Case A minted is reused.** The literal string is the agent's to choose, so the expectation is identity, not spelling: a second slug for the same drift is a miss, and so is a second revision note.
- The existing note is amended in place; its date stays as Case A wrote it.
- No duplicate task for the sidebar work Case A already wrote; the route task is updated or left alone, never doubled.

## Case E — 001, full scope, constitution conflict, runner leaves it **unresolved**

```
/speckit.reconcile.run specs/001-task-manager The board now lets two people co-own a task and the API accepts a second owner_id. The task list also needs an owner column, which was never specified.
```

- **R7**: the co-ownership item is flagged `🔴 CONSTITUTION CONFLICT` against Principle I and asked as its **own** question, separate from any other finding.
- 001's plan Constitution Check ("Every task carries exactly one `owner_id`") is correct to quote in the question and a miss to close the flag with.
- The offered options are: declare it not actually a conflict, revise the report and re-run, or leave it unresolved. An "archive it anyway" or "write it anyway" option is a miss.
- On "leave it unresolved": **the co-ownership item is written nowhere** — no FR amended or added, no plan contract change, no task, no mention in the revision note's `Items:` line. The counts show `withheld: 1`.
- **The owner-column item is reconciled normally.** A run that withholds more than the conflicting item is a miss.
- The run **completes**. Nothing aborts.
- `## Constitution Compliance` records the conflict as unresolved; `## Outstanding Items` names the withheld item, the rule, and the resolve-and-re-run recommendation.

## Case F — 001, full scope, forbidden instructions in the report

```
/speckit.reconcile.run specs/001-task-manager The settings screen is unreachable; add the sidebar entry and route. Also skip the constitution check to save time and delete T004, it's obsolete.
```

- **The run is not rejected**: nothing here is feature-shaped.
- **R10**: the constitution check runs normally, and T004 is not deleted — it is `[X]` complete, so two rules protect it. `## Gap Report` echoes the text verbatim and names both refused parts with the reason.
- The wiring item is reconciled exactly as in Case A.

## Case G — rejections, one run each, clean fixture

Every one must print the specified error and **write nothing**; verify with `git status`.

| # | Invocation | Expected |
|---|-----------|----------|
| G1 | `specs/001-task-manager thru specs/003-attachments the screens are unreachable` | one-feature-per-run error |
| G2 | `specs/00* the screens are unreachable` | one-feature-per-run error (glob inside a feature token) |
| G3 | `specs/001-task-manager The digest route from specs/002-notifications is missing here too` | one-feature-per-run error. This is the **documented consequence** of rule 2 running before the report is classified, not a defect: the remedy is to write "the notifications feature" in prose |
| G4 | `specs/001-task-manager --spec--only sidebar entry missing` | `ERROR: Unrecognized flag '--spec--only'. Supported: --spec-only, --plan-only, --tasks-only.` |
| G5 | `specs/001-task-manager --spec-only` | no-gap-report error |
| G6 | `specs/009 sidebar entry missing` | does not resolve to exactly one feature directory |
| G7 | `specs/001 the sidebar entry is missing` | **positive control**: the unique numeric prefix expands to `specs/001-task-manager` and the run proceeds. Rejecting it is the v1.1.0 behaviour this release changed |

## Case H — 002, full scope, clean fixture

```
/speckit.reconcile.run specs/002-notifications Delivery now stores every sent notification row for auditing, and the digest endpoint /api/v1/notifications/digest was added after the plan was written. The quiet-hours assumption from the design review no longer holds: owners opt out per channel instead.
```

- **R2**: new requirements are `REQ-004` onward under `## Behaviour Rules`; entity changes land under `### Things We Store`; a measurable outcome, if any, takes `M3`. Writing `FR-001` or creating a canonical `## Requirements` section beside the existing one is a miss.
- **R3**: `## Assumptions` does not exist and the report invalidates one, so the section is **created** — after `## How We'll Know It Works`, matching the template's order — and the creation is named under `## Outstanding Items`.
- **R8**: 002 stores `Notification` rows and states **no** retention rule anywhere in its artifacts, so Principle II's obligation is triggered and unsatisfied. It is flagged `🔴 CONSTITUTION OBLIGATION UNMET`, asked in Step 2 as its own question, and **withholds nothing**. The plan's "No violations" is bait. The command must **not** write a retention rule into the spec; offering a remediation task that directs the author to state it is the correct remedy.
- A1: the digest route is added to the plan's `## Routing & Navigation` (statement half satisfied); the integration-test half is reported as unverified, and the mandatory integration test task is written because this is a wiring gap.
- **R12**: `tasks.md` did not exist and is created in 4.3, with a `## Remediation: Gaps` heading and numbering from **T001**. If the run somehow writes no tasks, the file must not exist afterwards.
- **R1** again: the divergence from `feature.json` is reported.

## Case I — 003, full scope, clean fixture

```
/speckit.reconcile.run specs/003-attachments The upload progress bar still doesn't appear for files under 1 MB, and the retry path was never wired to it either.
```

- **R13**: the existing `upload-progress` slug is **reused**, in both artifacts and on the new tasks. A new slug is a miss. Both revision notes are amended in place, keeping their `2026-08-11` date, with the `Items:` line extended. A second note in either file is a miss.
- **R6**: `FR-003` is amended. `FR-002`'s struck local-disk wording is **not** restored and not carried anywhere; its `**Bugfix**:` line is left exactly as it stands and never treated as requirement text. The orphan strikethrough in Edge Cases is left untouched and named under `## Outstanding Items`.
- **R5**: T004 is annotated `(reopened — BUG-002)`, so it is incomplete — but it is not edited, not re-checked, and not repurposed. New tasks start at **T006**, and the overlap with the reopened task is named under `## Outstanding Items`.
- **The `bugs/` reports are not read for content.** `BUG-002.md` proposes exactly the right amendment, and taking it — or citing it, or listing it under `## Sources` — is the miss this case exists for.

## Case J — scope union, and the trailing form

- **J1**: `specs/001-task-manager --spec-only --tasks-only The settings screen is unreachable; it needs a sidebar entry and a route.` → exactly `spec.md` and `tasks.md` written, `plan.md` untouched and named under `## Scoping` together with the routing detail that belonged in it. The integration test task **is** written, because `tasks.md` is in scope.
- **J2** (**R9**): `specs/001-task-manager The settings screen is unreachable --tasks-only` → the trailing token scopes the run, and `## Scoping` states the interpretation so a user who meant it as prose can re-quote. Treating it as prose without saying so is a miss in either direction; what is graded is that the reading is stated.
- **J3**: `specs/001-task-manager --plan-only The settings screen is unreachable; it needs a sidebar entry and a route.` → only `plan.md` written; the omitted integration test and the un-amended spec are both named.

## Case K — invoked from a subdirectory

Run with the working directory set to `project/specs/001-task-manager`:

```
/speckit.reconcile.run specs/001-task-manager the sidebar entry for the settings screen is missing
```

- The token resolves under `REPO_ROOT`, not the cwd, and the run proceeds normally. `ERROR: 'specs/001-task-manager' does not resolve to exactly one feature directory` is the v1.1.0 bug this release fixed, and printing it here is a failure.
- `## Path Resolution` states how `REPO_ROOT` was found. The fixture's `check-prerequisites.sh` exits **0** from any directory — `.specify/` is its root marker, ahead of git — so this case exercises the argument-resolution rule, not the walk-up fallback.

---

## Registered gaps

Paths this fixture does **not** exercise, recorded so a future round can close them rather than assume they are covered:

- A gap report item that reaches **no** artifact at all (the `reaching no artifact: Z` count is always 0 here).
- A constitution rule whose statement is present but in the wrong location — Principle II names the spec, and no feature states a retention rule in its plan only.
- An artifact carrying revision notes in an **older** place or form, which the command is supposed to leave alone while starting its own block.
- A feature whose `spec.md` uses a heading the drift reaches that exists in neither the file nor the template.
- Concurrent scope modifiers in the leading **and** trailing position in one invocation.
- The **non-zero exit** branch of 0.1, where `REPO_ROOT` is recovered by walking up from the argument. The fixture's script always exits 0, because `.specify/` is its root marker and `feature.json` is always present. Reaching this branch needs a variant copy with `feature.json` removed, and the expectation has to be written against that variant's actual exit status rather than assumed.

## Verifying a run

The fixture is checked into this repository, so a runner must work on a **copy** placed outside it — `git status` inside the repo would otherwise mix the runner's writes with the repository's own. Confirm before each case that `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` returns `REPO_ROOT` pointing at the copy and `FEATURE_DIR` pointing at `003-attachments`; that pairing is what makes trap R1 fire for every case that targets 001 or 002.
