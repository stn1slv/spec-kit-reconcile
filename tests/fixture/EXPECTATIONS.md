# Fixture Test Cases and Expected Outcomes

The fixture in `project/` is a minimal three-feature spec-kit project with a real (if stubbed) source tree beside its specs, so the path-resolution rules have something true to resolve against. Every trap below is deliberate.

A test runner is a **fresh agent given only `commands/reconcile.md` and its own working copy of `project/`**, forbidden from reading this file, the CHANGELOG, or git history. It executes one case per run against a clean copy; results are compared to this file, and the round is recorded in a `BASELINE-v*.md` beside it.

**Pre-registration rule**: the expectations for a round must land in a commit *before* that round's runs, so the claim "written first" is auditable. The sibling `spec-kit-archive` fixture records what happens without it — a round whose expectations landed in the same commit as its results proves nothing about what was predicted.

**Nothing-written rule**: for every rejection case, `git status` in the working copy must be clean afterwards. A rejection that leaves a stray file is a failure even when the error message is right.

**Preparing a working copy.** The fixture is checked into this repository, so a runner must work on a copy placed **outside** it — otherwise `git status` reports this repository's own state rather than the run's. A copy is not a git working tree, so make it one before the run, or the nothing-written check cannot execute:

```bash
cp -R tests/fixture/project /somewhere/outside/run-NN
cd /somewhere/outside/run-NN && git init -q && git add -A && git commit -qm baseline
```

Then `git status --porcelain` after the run is the check: empty for every rejection case, and the exact set of written files for every other case.

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
| R17 | 003's `spec.md` carries bare `### Revision:` entries with no `## Revisions` heading; its `plan.md` already has the heading | 003 spec.md + plan.md | The v1.2.1 nesting rule: the heading is added above the bare entries (the one permitted retroactive edit, reported), and the artifact that already has it is left alone |
| R18 | One remediation item reaching several entries inside one artifact | Cases A, H, I | The v1.2.1 row unit: one row per **target**, every row carrying exactly one action, and `K + N + S + W + Z = R` |
| R19 | A drift where the capability exists but is unreachable, beside one where the capability does not exist | Cases I and E | The Wiring & Navigation reachability test, which decides whether 4.3 rule 4's integration test is mandatory |
| R20 | 001 carries a `## Phase 3: Convergence` section holding an open `T007` | 001 tasks.md | The v1.2.1 placement rule: a Convergence phase is never a placement target, and its ID still counts toward the highest |
| R21 | 002's `REQ-003` is marked `RETIRED`, and Case H's report describes the replacement behaviour | 002 spec.md + Case H | A retired entry is never amended or revived; the drift lands on a new ID and the retired one is named |
| R22 | 003's `T006` is marked `CANCELLED` | 003 tasks.md | A cancelled task is not edited, not repurposed, and **still counts** when finding the highest ID |

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
- **R8: A1 is triggered, and the obligation is raised.** The report says "**Backend and tests** for the settings screen exist", so a settings API route exists that `plan.md` does not declare — A1's condition is met by the report's own words, and the statement half is unmet because this run cannot write a declaration the report never supplies (it gives the *client* route, not the API path). Expect `🔴 CONSTITUTION OBLIGATION UNMET` on A1, asked as its own Step 2 question, withholding nothing. A1's action half is separately reported as unverified. Principle II is **satisfied** (001's FR-005 states the retention rule) and Principle I is not reached, so a finding on either of those is a miss.
  **Corrected in v1.2.1 after the first baseline round.** This expectation previously asserted the opposite — that a client route is not an API route, so no finding should appear — and two runners independently flagged A1 anyway. They were right and the expectation was wrong. Its history is worth keeping: review had already warned that the trap graded an inference, and the first fix asserted the *opposite* inference rather than removing the dependency on one, which turned an ambiguous expectation into a confidently wrong one that scored correct runs as failures.
- **The Quality Gate is deliberately not MUST-worded**, and this case grades that. "Every merged feature keeps its spec, plan and tasks consistent with the shipped code" states no obligation in binding terms, so 0.2 does not extract it and Step 1 takes no finding from it. An agent that treats every bullet under a `## Quality Gates` heading as binding reports it as an unverified action rule on every run. The gate is the control for 0.2's "a rule is MUST-level only when it says so".
- **The "settings service" and the retention sweep**: the retention item reaches `plan.md` (Configuration, Testing Strategy) and no `spec.md` section, since FR-005's retention rule itself is unchanged — only where the sweep runs. A run that amends FR-005 for a relocation is a miss.
- `tasks.md`: new tasks numbered from **T008**, none carrying `[P]`, the existing `[P]` on T002/T003/T005 untouched, no `[X]` task edited. A wiring gap was identified, so an integration test task is mandatory.
- **R20**: `## Phase 3: Convergence` is the last phase heading in the file and holds the highest ID, which makes it the obvious place to append and the wrong one. New tasks go under `## Remediation: Gaps` — neither of 001's user-story phases covers a settings screen, and a Convergence phase is never a placement target. Writing into it is a miss. Its `T007` still counts, which is why numbering starts at T008 rather than T007.
- **R16**: the sidebar task resolves to `src/components/Sidebar.tsx` and the route task to `src/router/index.ts`; `## Sources` declares the read-only lookup that found them.
  The "settings service" resolves to nothing — no such file exists and 001's Project Structure does not name one. **Two outcomes pass**, because the command does not force a task for an item a plan amendment can close: either the item produces a task written **without a path**, named under `## Outstanding Items`; or it produces **no task at all**, closed by the plan's Configuration and Testing Strategy amendments, which the report says so. **Exactly one outcome fails: a task carrying an invented path** such as `src/services/settings.py`. That is the failure this trap exists for, and it is what the case grades.
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
- The existing note is amended in place **only where this run actually changes something**; its date stays as Case A wrote it. Case A already recorded the sidebar and the route, so a run that finds nothing left to amend in `spec.md` correctly leaves that note untouched — "a re-run that changes nothing leaves no diff". What is graded is consistency: a note amended without an edit, or an edit without its note, is a miss; a second note is a miss either way.
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

## Case H — 002, full scope, clean fixture; runner answers the obligation question **"add a remediation task"**

```
/speckit.reconcile.run specs/002-notifications Delivery now stores every sent notification row for auditing, and the digest endpoint /api/v1/notifications/digest was added after the plan was written. The quiet-hours assumption from the design review no longer holds: owners opt out per channel instead.
```

- **R2**: new requirements are `REQ-004` onward under `## Behaviour Rules`; entity changes land under `### Things We Store`; a measurable outcome, if any, takes `M3`. Writing `FR-001` or creating a canonical `## Requirements` section beside the existing one is a miss.
- **R21**: `REQ-003` is marked `RETIRED`, and the report's "owners opt out per channel instead" is precisely the behaviour that retired entry used to cover. That is the trap: the closest-matching requirement is the one that must **not** be touched. Amending REQ-003, un-retiring it, or rewording it to mean per-channel opt-out is a miss; the per-channel behaviour takes a **new** `REQ-###`, and REQ-003 is named under `## Outstanding Items` as a retired entry the report appeared to reach.
- **R3**: `## Assumptions` does not exist and the report invalidates one, so the section is **created** — **after the whole of `## How We'll Know It Works`, following M1 and M2 and before `## Open Questions`**, which is this project's Success Criteria section by role, matching the template's order. Inserting it between that heading and M1, which would nest the new section inside the old one, is a miss, and so is appending it after `## Open Questions`; both wrong placements are now distinguishable from the right one — and the creation is named under `## Outstanding Items`. 002's spec ends with `## Open Questions`, so "template position" and "appended at the end of the file" produce **different** results here and the trap discriminates; appending after `## Open Questions` is a miss. This is also the map-by-role test: the heading is not named "Success Criteria", and matching on wording rather than on what the section is for gets it wrong.
- **R8**: 002 stores `Notification` rows and states **no** retention rule anywhere in its artifacts, so Principle II's obligation is triggered and unsatisfied. It is flagged `🔴 CONSTITUTION OBLIGATION UNMET`, asked in Step 2 as its own question, and **withholds nothing**. The plan's "No violations" is bait. The command must **not** write a retention rule into the spec; offering a remediation task that directs the author to state it is the correct remedy.
- A1, the **self-satisfied obligation**: the digest route is added to the plan's `## Routing & Navigation`, which *is* the declaration A1's statement half demands, so the rule is **met by this run's own amendment** and must **not** be flagged or asked about. A run that raises `🔴 CONSTITUTION OBLIGATION UNMET` on A1 — because the plan lacked the route at check time — is a miss, and so is one that refuses the 4.2 amendment on the grounds that it would be "writing the missing statement". A1's integration-test half is action-requiring and is reported as unverified.
- The mandatory integration test task is written because the digest endpoint is classified **Wiring & Navigation** ("Missing routes" in the category table). That classification is itself part of what this case grades: reading it as **Contracts** only, and therefore writing no test task, is a miss — and the category table is what makes the reading decidable rather than a judgment call.
- **Task paths**: 002's plan declares `src/notifications/`, which the fixture does not contain. A task creating a file under a declared directory carries a **derived** path (4.3 rule 3), so `src/notifications/…` is correct here; a path under a directory neither declared nor present is invented and is a miss. The retention remediation task the runner's answer asks for names `specs/002-notifications/spec.md`, which exists.
- **R12**: `tasks.md` did not exist and is created in 4.3, with a `## Remediation: Gaps` heading and numbering from **T001**. If the run somehow writes no tasks, the file must not exist afterwards.
- **R1** again: the divergence from `feature.json` is reported.

## Case I — 003, full scope, clean fixture

```
/speckit.reconcile.run specs/003-attachments The cancel affordance still isn't reachable for uploads under 1 MB — the progress component never mounts for them — and the retry path never got one either.
```

The report is deliberately a **refinement of the drift the existing note already records** ("the progress indicator shipped without a cancel affordance"), not a new problem in the same component. That is what makes slug identity decidable rather than a judgment call: an earlier wording of this case described a different failure mode, where minting a new slug would have been a defensible reading and the expectation would have graded an opinion.

- **R13**: the existing `upload-progress` slug is **reused**, in both artifacts and on the new tasks. A new slug is a miss. The existing note is amended in place, keeping its `2026-08-11` date, with the `Items:` line extended; a second note in the same file is a miss.
- **R17**: `spec.md`'s note has no `## Revisions` heading above it, so the run **adds one** and says so in the report; `plan.md` already has the heading and is left alone. Appending a new bare `###` block below the existing entry, or moving or rewording either existing note, is a miss.
- **R19**: this drift is **Wiring & Navigation** — the progress component exists and the user cannot reach it — so the integration test of 4.3 rule 4 is **mandatory**. Classifying it as Logic/UX and writing no test is a miss. Contrast Case E, where the capability does not exist at all.
- **The retry path resolves to nothing**: no file in the repository mentions retry and the plan's Project Structure declares no such module, so that task is written **without a path** and named under `## Outstanding Items`. An invented path is the miss.
  **`spec.md`'s note is amended for certain.** `plan.md`'s note is amended **only if a plan section is actually reached** — 4.2 says a drift reaching no plan section leaves the file alone entirely, and a run that correctly leaves `plan.md` untouched here passes. What is graded is consistency: a plan note amended without a plan edit, or a plan edit without its note, is a miss.
- **R6**: `FR-003` is amended. `FR-002`'s struck local-disk wording is **not** restored and not carried anywhere; its `**Bugfix**:` line is left exactly as it stands and never treated as requirement text. The orphan strikethrough in Edge Cases is left untouched and named under `## Outstanding Items`.
- **R5**: T004 is annotated `(reopened — BUG-002)`, so it is incomplete — but it is not edited, not re-checked, and not repurposed. New tasks start at **T007**, and the overlap with the reopened task is named under `## Outstanding Items`.
- **R22**: `T006` is marked `CANCELLED`. It is not edited and not repurposed, and its ID is **not** handed out again — a run that starts new tasks at T006 because the cancelled one "does not count" is a miss. The overlap is named under `## Outstanding Items`.
- **The `bugs/` reports are not read for content.** `BUG-002.md` proposes exactly the right amendment, and taking it — or citing it, or listing it under `## Sources` — is the miss this case exists for.

## Case J — scope union, and the trailing form

- **J1**: `specs/001-task-manager --spec-only --tasks-only The settings screen is unreachable; it needs a sidebar entry and a route.` → exactly `spec.md` and `tasks.md` written, `plan.md` untouched and named under `## Scoping` together with the routing detail that belonged in it. The integration test task **is** written, because `tasks.md` is in scope.
- **J2** (**R9**): `specs/001-task-manager The settings screen is unreachable --tasks-only` → **both halves are graded, and both must hold**: the trailing token **scopes the run** (only `tasks.md` is written), *and* `## Scoping` states the interpretation so a user who meant it as prose can re-quote. Treating it as prose is a miss even when the reading is announced — the command says to *take* the scoping reading, not to prefer it, and a run that writes all three files while narrating its choice is exactly the two-answer outcome that wording removed.
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
- Concurrent scope modifiers in the leading **and** trailing position in one invocation. The command now defines this (the scope is the union of all of them); no case exercises it.
- A **trailing run of several modifiers** (`… report text --spec-only --tasks-only`), which is the union form v1.1.0 documented. J1 covers the union in the leading position only.
- A `--` token in the leading position that is not a modifier, which is now a fatal error where v1.1.0 read it as prose. G4 covers a *malformed* modifier (`--spec--only`); it does not cover a well-formed unrelated flag such as `--force`.
- The **non-zero exit** branch of 0.1, where `REPO_ROOT` is recovered by walking up from the argument. The fixture's script always exits 0, because `.specify/` is its root marker and `feature.json` is always present. Reaching this branch needs a variant copy with `feature.json` removed, and the expectation has to be written against that variant's actual exit status rather than assumed.

## Verifying a run

Prepare the copy as described at the top of this file, then confirm before each case that `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` returns `REPO_ROOT` pointing at the copy and `FEATURE_DIR` pointing at `003-attachments`; that pairing is what makes trap R1 fire for every case that targets 001 or 002.
