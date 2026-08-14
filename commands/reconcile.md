---
description: "Reconcile implementation drift by updating the feature's own spec, plan, and tasks"
argument-hint: "specs/###-feature-name [--spec-only|--plan-only|--tasks-only] <gap report>"
scripts:
  sh: ../../scripts/bash/check-prerequisites.sh --json --paths-only
  ps: ../../scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---
Act as the **Chief Software Architect** and **Implementation Auditor**.
A feature implementation has landed, but "artifact drift" has been discovered (e.g., missing routes, updated behavior, or unlinked UI). Your goal is to **reconcile** this drift by surgically amending the feature's own specification, plan, and task artifacts.

**Surgical means two things**: only the sections the gap report actually reaches are touched, and every word written traces to something in **Allowed Sources** below. Nothing is inferred from the shipped code beyond the one narrow lookup that section defines.

## User Input

```text
$ARGUMENTS
```

### Input Parsing

**This command reconciles exactly one feature per run.** There is no batch or range mode. To reconcile several features, run the command once per feature.

`$ARGUMENTS` names the feature, then optional scope modifiers, then a **Gap Report**: a natural language description of what is missing or changed in the implementation versus the documentation.

**Examples:**
- `specs/007-invoice-settings Backend + tests exist; React screen scaffolded. Users can't navigate to it. Need sidebar link + route.`
- `specs/007-invoice-settings --spec-only The /api/v1/settings endpoint now requires an 'org_id' header not in the original plan.`

**Grammar** — path, then modifiers, then prose:

1. **First token: the feature directory, required.** A path of the form `specs/###-feature-name`, naming the feature to reconcile. It must resolve to **exactly one** existing directory under `REPO_ROOT`; a numeric prefix such as `specs/007` may expand to `specs/007-invoice-settings` when exactly one directory matches.
2. **Then: scope modifiers**, optional, recognised **only until the first non-flag token**:
   - `--spec-only` — update only `spec.md`
   - `--plan-only` — update only `plan.md`
   - `--tasks-only` — update only `tasks.md`
3. **Then: the gap report**, free text. From the first non-flag token onward everything is gap report, taken verbatim — including words that start with `--`, such as a report about a `--force` flag.

If **several** modifiers are supplied, the scope is their **union**: `--spec-only --tasks-only` updates both `spec.md` and `tasks.md` and nothing else.

**Trailing modifiers are still recognised**, because earlier versions of this command accepted only that form. The **trailing run of modifier tokens** — the unbroken sequence of them at the very end of the input, one or several — scopes the run exactly as leading ones do, and several combine into the same union: `…report text --spec-only --tasks-only` writes both files, as it did before this version. It is the *run* that must reach the end, not each modifier individually.

The leading position is canonical and unambiguous; the trailing one is a compatibility path and carries the ambiguity that motivated the change. When the report's own last words are **one of the three modifiers**, take the scoping reading — not "prefer" it, since a parser that hedges gives two agents two answers — and state the interpretation under `## Scoping`, so a user who meant it as prose can see it and re-quote. A trailing token that starts with `--` but is **not** one of the three (`--force`, say) is **prose**, never an error: rule 3's rejection is leading-position only.

**Only the middle is prose.** Both ends scope: the leading position per grammar rule 2, and the trailing run per this paragraph. A modifier is prose only when it sits **after the gap report has begun and before the trailing run** — `specs/001 fix the --spec-only screen` scopes nothing, while `specs/001 --spec-only fix the screen` scopes normally. When modifiers appear in **both** positions, the scope is the union of all of them, exactly as several modifiers in one position combine.

**Validate everything else.** The first four checks are textual and run before Step 0; the fifth needs `REPO_ROOT` and so runs as soon as 0.1 has resolved it, still ahead of every write. **No file is written when any of them fails** — a rejected invocation must leave the repository exactly as it found it.

1. **Empty input, or no feature at all.** If `$ARGUMENTS` is empty, **or the first token starts with `--`** (the feature path must come first, before any modifier), output `ERROR: No feature spec directory provided. Usage: /speckit.reconcile.run specs/###-feature-name [--scope-modifier] [gap report]` and stop.
2. **More than one feature.** This check comes **before** the flag check, so a range or a second path gets the guidance below rather than a generic parse error. Reject when the input covers more than one feature. The checks key on **feature-shaped tokens** — a token whose path contains a `specs/###` segment in any form, with or without the trailing name (`specs/002-x`, `specs/001`, `../specs/002-x`, `/repo/specs/002-x`; a glob character may stand in for digits, as in `specs/00*`), or a bare feature number or name such as `007` or `007-invoice-settings`. A `specs/` path counts **anywhere** in the input; a bare number or name counts only **in the leading region** — the first token, any modifiers, and the token immediately following them, which is where a second feature would be written if one were meant (`specs/001-task-manager 002-notifications …`). Defining it positionally matters: "before the first prose token" would be circular, since grammar rule 3 makes that very token the start of the gap report. A bare number or name also counts **beside a range marker whose other side is a feature reference** (the `008` in `specs/001 thru 008`) — digits inside later prose (`handle 404 errors`, `max 500 items`, `3 screens are unreachable`) are gap report, and so is ordinary punctuation (`double-checked?`, `billing/invoicing`, `*emphasis*`):
   - two or more feature-shaped tokens
   - a glob character (`*` or `?`) **inside a feature-shaped token** (`specs/00*`, `0??-export`)
   - a **word** range marker — `thru`, `through`, or `to` — appearing as a whole token between two feature references (`specs/001 thru specs/008`, `specs/001 thru 008`)
   - a `..` **separating two feature references inside a single token** (`specs/001..specs/008`, `001..008`)

   A word marker only counts as a whole token, never as part of a directory name, so `specs/003-import-to-csv` and `specs/012-through-put` are legitimate single features. A `..` only counts when it sits between two feature references, so the leading `../` of a relative path such as `../specs/001-foo` is not a range. On a match, output:
   ```
   ERROR: This command reconciles one feature per run — no ranges or globs.
   Run it once per feature:
     /speckit.reconcile.run specs/001-first-feature [gap report]
     /speckit.reconcile.run specs/002-second-feature [gap report]
   ```
   and stop.
3. **Unrecognized flag.** A `--` token in the leading flag position that is not one of the three modifiers gets `ERROR: Unrecognized flag '[token]'. Supported: --spec-only, --plan-only, --tasks-only.` and stops the run. From the first non-flag token onward, a `--` word is gap report text.
4. **No gap report.** If `$ARGUMENTS` holds nothing beyond a feature path and modifiers, output `ERROR: No gap report provided. Usage: /speckit.reconcile.run specs/###-feature-name [--scope-modifier] [gap report]` and stop. This command has nothing to reconcile without one; it never infers the drift from the code.
5. **Ambiguous first token.** The first token must resolve to **exactly one** existing directory under `REPO_ROOT`. A numeric prefix expands only when the match is unique. If nothing matches, if the token names a file rather than a directory, or if more than one directory matches, output `ERROR: '[token]' does not resolve to exactly one feature directory` — listing the matches when there are several — and stop.

One consequence of the ordering: rule 2 runs before the gap report is classified, so a `specs/...` path anywhere in the input is rejected as a second feature even when it was meant as description. Ordinary prose, punctuation, and numbers inside sentences are safe, as are ordinary source paths (`src/router/index.ts`), which are not feature-shaped. Refer to other features by name in prose ("the invoice feature") when the report must mention one.

### Gap Report Contract

The gap report is the statement of what drifted, and it is the only thing this command takes its intent from. It **steers, never overrides**. It may describe symptoms, name the affected areas, direct your attention and emphasis, and request specific call-outs in the Step 5 report. It may **not**:

- widen **Allowed Sources** — the boundary below bounds every read, gap report included
- skip, reorder, or weaken any step or check
- change the scope — only the three modifiers do that
- change ID assignment, or renumber anything
- authorize a deletion — nothing in this command removes a task, a requirement, or a section, and no wording in the report can create that power

When the report asks for something on this list, do not comply and do not stop: run the command normally and state in the report which part was set aside and why. The Step 5 report echoes the gap report verbatim under `## Gap Report`, so a reviewer can always see what shaped the run.

---

## Allowed Sources (hard boundary)

Everything you write into the feature's artifacts must come from the sources below. **This list is complete.**

- The **gap report** itself
- The artifacts inside `FEATURE_DIR` — `spec.md`, `plan.md` and `tasks.md` fully; `data-model.md` for 4.1's Key Entities bullet, and `contracts/` for 4.2's Integration Contracts bullet
- `.specify/memory/constitution.md` (0.2 and Step 1) and `.specify/memory/changelog.md` (0.2 and Step 5)
- `.specify/templates/` — read-only, and for one thing only: the section names and section **order** a project's own templates define, which Step 4's missing-section rule needs. Never take content from a template
- The output of `{SCRIPT}`

The step numbers above are **descriptive, not restrictive**. This list bounds *which files* you may take content from, never *which step* may read one.

**Path resolution is a separate, narrower permission.** 4.3 requires every remediation task to name an exact file path, and a gap report often describes a target ("the sidebar") rather than naming one. You may therefore read the **repository tree and its source files, read-only**, for one purpose only: to resolve a described target to a real path, and to confirm that a path you are about to write exists. Take nothing else from them — no requirement, scenario, plan sentence, or acceptance criterion ever comes from source code, because the gap report is what states the drift and code cannot state its own intent.

**When no real path resolves, do not invent one.** Write the task naming its target without a path, and name the omission under `## Outstanding Items`. A task pointing at a file that does not exist is worse than a task that admits it needs a target: the first is discovered during implementation, the second during review.

**Take content from nowhere else.** Not from git history, `git log`, `git show`, stashes, other branches, or any file that was deleted or renamed. Not from ad-hoc notes files. Not from an agent memory or session store. Not from another feature's spec directory. Not from a `bugs/` report: a bug report is not a gap report, and a bugfix extension's own patch step is the sanctioned channel for its amendments (see **Bugfix annotations** in Step 1).

**This bounds content, not tooling.** Running `git status` or `git diff` to verify what you just wrote is fine. Reading git to *obtain* the drift, or to reconstruct an artifact's earlier contents, is not.

Report compliance under `## Sources` in Step 5.

---

## Step 0: Discovery & Setup (Gate)

### 0.1 Resolve Paths

Resolve paths **in this order** — each depends on the one before it, so do not reorder them.

**1. `REPO_ROOT`.** Run `{SCRIPT}` and take `REPO_ROOT` from its output.

- **If the script is missing**, stop and inform the user. It ships with Spec-Kit, so its absence means this is not an initialised Spec-Kit project.
- **If the script runs but exits non-zero** — commonly `Feature directory not found` on a checkout with no `.specify/feature.json` — this is **not** fatal. Its feature directory is not used anyway (see step 2). Recover `REPO_ROOT` by resolving the first token of `$ARGUMENTS` against the **current working directory** and walking up to the nearest ancestor containing `.specify/`. Note the fallback in the Step 5 report. Stop only if no such ancestor exists.

**2. `FEATURE_DIR` — the argument always wins.** Resolve the first token **under `REPO_ROOT`**, not under the current working directory, even when step 1's fallback started from cwd — the walk-up has already established `REPO_ROOT` by then, and a run invoked from a subdirectory would otherwise reject a perfectly valid `specs/001-x`. Apply the **ambiguous first token** check from Input Parsing at this point.

**A token written relative to the cwd resolves to the same directory.** `../specs/001-x`, passed from a subdirectory, means the feature that `specs/001-x` names under `REPO_ROOT`; it does not mean a path outside the repository. Strip any leading `../` segments before resolving, and resolve what remains under `REPO_ROOT`. A first token that still points outside `REPO_ROOT` after that fails the ambiguous-first-token check like any other non-match.

Ignore whatever feature directory `{SCRIPT}` reports. It resolves the project's own state (`SPECIFY_FEATURE_DIRECTORY`, then `.specify/feature.json`) — whichever feature was last worked on, which is not necessarily the one that drifted. When the two differ, report both in Step 5, because this command rewrites three files in place and a user who passed the wrong path needs to see it.

**3. Remaining paths.**
- `FEATURE_SPEC` (`FEATURE_DIR/spec.md`)
- `IMPL_PLAN` (`FEATURE_DIR/plan.md`)
- `TASKS_FILE` (`FEATURE_DIR/tasks.md`)

Use absolute paths for all file operations.

**Validation**: Ensure `spec.md` and `plan.md` exist. If either is missing, stop with:
> ⚠️ Missing required files in `FEATURE_DIR`. Expected: spec.md, plan.md.
> Run `/speckit.specify` and `/speckit.plan` first.

**Step 0 is a gate and writes nothing**, including `TASKS_FILE`. 4.3 creates that file when it needs it.

### 0.2 Load Context

Read `FEATURE_SPEC`, `IMPL_PLAN`, and `TASKS_FILE` (the last one if it exists). **Read regardless of scope**: a modifier says what may be written, never what may be read.

Also read `.specify/memory/changelog.md` if it exists, and note whether this feature already has an entry in the Merged Features Log. Step 5 uses this.

Also read `.specify/memory/constitution.md` if it exists. Extract the MUST-level constraints from its Core Principles, Architecture Standards and Quality Gates.

**A rule is MUST-level only when it says so.** Those three headings say where to look, not that everything under them binds: a constitution often carries aspirations, rationale and house style beside its rules, and a section heading does not promote them. Extract a rule when it states an obligation in binding terms (`MUST`, `MUST NOT`, `is not allowed`, `never`); leave the rest, and take no finding from it. Reading every bullet under a heading as binding turns Step 1 into a survey and produces a finding on every run.

For each extracted rule, note which shape it takes, because Step 1 checks all three and a shape not recorded here is invisible later:

- it **forbids** something ("shared or unassigned active tasks are not allowed");
- it **requires a statement** from features meeting a condition ("every feature that stores user data MUST state its retention rule"), together with any location the rule names for that statement;
- it **requires an action** ("all API routes MUST have automated tests before merge") — something to be *done* rather than written;
- or **both** of the last two, which one sentence often does. Record both halves; filing it under one loses the other.

---

## Step 1: Gap Normalization

Analyze the user's **Gap Report** and normalize it into structured remediation items.

Classify only; Step 4 owns which artifact each category is written to.

| Category | Typical Issues |
|----------|----------------|
| **Wiring & Navigation** | Missing routes, menu items, sidebar links |
| **Contracts** | API field mismatches, missing headers |
| **Requirements** | Shipped code adds, drops or redefines a capability |
| **Behavior** | Implementation behaves differently than planned |
| **Data Model** | New or changed entity, field, or validation rule |
| **Outcomes & Assumptions** | A measurable target or a stated assumption no longer holds |
| **Test Coverage** | New wiring/navigation without verification |
| **Logic/UX** | Success toasts missing, error handling gaps |

### 1.1 Constitution Check (three shapes)

A MUST rule can fail in more than one way, matching the three records from 0.2. **Check only the rules the normalized remediation items actually reach** — this is a surgical command, and a constitution rule the drift never touches is not its business. That bound is what keeps this a check rather than a project-wide audit.

1. **Conflict** — a remediation item would write content contradicting a MUST rule. Flag it CRITICAL:
   ```
   🔴 CONSTITUTION CONFLICT: [remediation item] conflicts with [Principle N / Architecture Standard / Quality Gate]: "[rule text]"
   → Raised in Step 2; an unresolved conflict withholds this item only.
   ```
2. **Unmet obligation** — a rule requires a statement, the drift meets its condition, and no statement exists in the feature's own artifacts (or not in the location the rule names, when it names one). Nothing contradicts anything here; the required content is simply absent, and it is exactly as binding:
   ```
   🔴 CONSTITUTION OBLIGATION UNMET: [Principle N / ...]: "[rule text]"
     Triggered by: [what this drift does that meets the rule's condition]
     Missing:      [the statement the rule requires, and whether it is absent everywhere or only from the named location]
   → Raised in Step 2. This never withholds anything.
   ```
   **An obligation this run's own amendment satisfies is not unmet.** Where the gap report itself supplies the required content and Step 4 will write it into the location the rule names, the rule is met by that amendment: do not flag it, and do not ask about it. A constitution rule saying "any new API route MUST be declared in the plan's Routing & Navigation section", against a drift reporting a route the plan lacks, describes exactly the edit 4.2 is about to make — flagging it would raise a mandatory question about the work in progress. Judge shape 2 against the **end state this run will produce**, which the Step 3 target table names, not against the artifacts as they stand at 1.1. This is the same escape hatch shape 3 carries, and it exists for the same reason.

   **Never write the missing statement yourself** in every other case. The gap report states what the implementation does, not what the author intends a rule to say, and a rule asking for a judgment the report does not make — a retention period, a threshold, a policy — is unmet however obviously it is missing. The legitimate remedy there is a remediation task directing the author to make the statement, which 4.3 writes like any other task — **unless `tasks.md` is out of scope**, in which case write nothing and name the un-written task under `## Scoping`, exactly as 4.3 rule 4 does for the mandatory integration test. A scope modifier is never overridden to satisfy an obligation.

   The line between the two is whether the gap report already contains the answer. It does for "the digest endpoint was added after the plan was written"; it does not for "state your retention rule".
3. **Action-requiring** — the rule asks for something to be *done*. This command reads artifacts and cannot inspect a codebase, a test run, or a CI result, so it can never establish that the action happened, and never that it did not. **Report it as unverified under `## Outstanding Items`; never flag it and never ask about it.** A claim is not a verification: a plan saying nothing, a plan claiming "tests for both routes", and a Testing Strategy whose list omits one route are all equally unverified. The one exception is an artifact **explicitly stating the action was skipped** ("no tests for this route yet"), which is ordinary content contradicting a rule and goes through shape 1.

   Where the drift itself closes such a rule, the remediation task is the right output — a **Wiring & Navigation** gap under a "routes MUST have tests" rule already produces the mandatory integration test of 4.3.

**Your judgment belongs in deciding whether a rule's condition is met, not in deciding whether a qualifying finding is worth raising.** Once a conflict or an obligation qualifies, it goes to Step 2 and the user decides. An argument for why a finding is too minor to raise is an answer to the Step 2 question, not a reason to skip it.

**The feature's own `## Constitution Check` is input, not a verdict.** A `plan.md` usually carries one. Read it, and quote it in the Step 2 question when it bears on the answer, but **never close a finding on it alone**: it records what the author believed at planning time, before the work was done, and re-checking that belief against what actually shipped is the reason this check exists. "No violations" in a plan is evidence about intent, not a finding.

**But distinguish a verdict from a statement of fact.** What that rule bars is treating the author's *opinion about compliance* as the answer. It does not bar the section from containing content a rule actually asks for. Where a MUST rule requires the feature to **state or record something**, and the Constitution Check is where the feature states it, that statement satisfies the obligation like any other. The test is what the sentence does: **"we checked and it is fine" is a verdict and closes nothing; "here is what we changed and why" is the record the rule demanded.**

### 1.2 Bugfix annotations

A bugfix extension (such as `spec-kit-bugfix`) patches the same three files this command edits, and leaves markers behind. Read them as follows, so a reconciliation never undoes a patch:

- Text struck through with `~~...~~` counts as **superseded by a patch** when a `**Bugfix**:` marker or a live replacement wording sits in or beside the same entry. Treat the replacement as the current text. **Never restore struck wording while amending an entry**, and never carry it into a new one.
- `**Bugfix**: [DATE] — [BUG-NNN] ...` lines are patch metadata, not requirement text. Leave them in place; do not amend them and do not treat them as the entry's content.
- A task annotated `(reopened — BUG-NNN)` is **incomplete**, whatever its checkbox shows. It is therefore not "completed work" for the never-edit rule in Step 4 — but do not edit or repurpose it either, because it belongs to a bugfix cycle in progress. Append the remediation task and name the overlap under `## Outstanding Items`.
- Struck text with neither a marker nor a replacement is not a patch artifact you can interpret. Leave it exactly as it stands and name it under `## Outstanding Items`.

---

## Step 2: Clarify (At Most Once; Max 5 Questions)

If the gap report is ambiguous (e.g., "the button doesn't work" without saying which button), ask targeted questions. Ask **only questions that materially change scope or correctness**, and skip the step entirely if everything is unambiguous.

**Both sentences above govern discretionary questions only.** The **always-ask** questions below are exempt from the materiality filter and from the skip-the-step clause alike: their value is the record of the user's decision, so "the answer would not change what gets written" is never a reason to drop one.

**Always ask** about the CRITICAL findings from 1.1. Ask **conflicts and obligations as separate questions**, bundling all of each category into one, because the two take different answers: an obligation may be closed as an accepted gap and a conflict may not.

- **Conflict question.** Legal options: **declare it not actually a conflict** (the rule does not say what it appeared to say, or the item does not do what it appeared to do), which clears the flag and writes the item normally; **revise the gap report and re-run**; or **leave it unresolved**. "Write it anyway" is **not** an option. Only the first option keeps the item on this run — revising and re-running withholds it now exactly as leaving it unresolved does, and the difference is only what the user intends to do next. Say so, so nobody is left believing the item lands this time.
- **Obligation question.** Legal options: **record it as an accepted gap**; **add a remediation task** directing the author to make the statement (subject to 1.1's scope carve-out); or **declare the obligation not triggered** because the condition is not actually met. Under every option the drift's own content is reconciled normally.

Use this format and **wait for answers**:

```markdown
## Question [N]: [Topic]
**Context**: [Relevant implementation detail, and the rule text when the question is a constitution finding]
**Decision Needed**: [1 sentence]
**Suggested Answers**: [Table with Option A/B/C]

**Your choice**: _[Wait for user response]_
```

**What an unresolved conflict does.** It **withholds that item and only that item**: the conflicting remediation item is written into no artifact, generates no task, and is named in the report under `## Constitution Compliance` and `## Outstanding Items` with the recommendation to resolve it and re-run. Every other item in the gap report is reconciled normally and the run completes. **Nothing in Steps 2 to 5 aborts a run**; withholding is the only way an item is left out, and an unmet obligation never causes it.

**Rules:**
- Max 5 questions total. Bundle discretionary questions into one; if the budget is spent, report them under `## Outstanding Items` rather than dropping an always-ask question to make room.
- Max 3 unresolved `NEEDS CLARIFICATION` markers in output — beyond that, pick reasonable defaults and note them under `## Defaults Applied`.

---

## Step 3: Impact Map

Before making any edits, produce a brief impact map. It is the user's preview of this run, so it must promise only what the run will actually do: mark any artifact a scope modifier excluded as `Skipped (out of scope)`, and any artifact an earlier run already reconciled under this slug as `No change (already applied)`. Derive the slug per Step 4 before producing this map.

This table's rows are **artifacts**. An item withheld by an unresolved conflict is not an artifact and has no row here; it is recorded in the Targets table below, which is the table with a row for every item.

```markdown
### Sync Impact Map
| Artifact | Changes | Tasks Generated |
|----------|---------|-----------------|
| `spec.md` | Amend Acceptance Scenario under User Story 2, add Edge Case | None |
| `plan.md` | Add Route `/settings`, update API contract | None |
| `tasks.md` | Append remediation tasks | T045, T046, T047 |
```

**Then the target table.** It records *which existing item each amendment will land on* before any edit, rather than leaving that choice implicit in the writing — the same choice made twice can otherwise come out two different ways. Identify each target by the **citation ladder** defined in Step 4.

**One row per item-artifact pair**, not per item: a single remediation item that reaches both `spec.md` and `plan.md` takes two rows, because it amends a different entry in each and each of those choices needs its own preview. An item that reaches no artifact takes exactly one row, with `—` for the artifact and `None` for the action. An item withheld by an unresolved conflict takes one row with the action `Withheld`.

**`tasks.md` takes no rows here.** This table previews *which existing entry an amendment lands on*, and a remediation task is always new, so it has no target to preview; the Sync Impact Map above already names the tasks this run will generate. `R`, `K` and `N` therefore count rows against `spec.md` and `plan.md` only.

```markdown
### Targets
| # | Item (from the gap report) | Category | Artifact | Target | Action |
|---|---------------------------|----------|----------|--------|--------|
| 1 | Settings screen unreachable | Wiring & Navigation | `plan.md` | Routing & Navigation → "GET /settings" | Amend |
| 2 | Settings screen unreachable | Wiring & Navigation | `spec.md` | FR-014 | Amend |
| 3 | org_id header now required | Contracts | `spec.md` | *new* | Add |
| 4 | org_id header now required | Contracts | `plan.md` | Integration Contracts → "Settings API" | Amend |
```

**State the counts** below the table, and **state the unit of each**, because a number whose unit is guessed is a number two runs will disagree on:

```
normalized items: M (items); rows: R (item-artifact pairs); amending: K (rows); new: N (rows); reaching no artifact: Z (items); withheld: W (items)
```

`K + N` counts rows and must equal `R` minus the rows for withheld items and for items reaching no artifact. `M` counts items, and `Z + W` are subsets of it. The two units are never added together. A zero must be legible as "examined and found nothing to amend", never as "did not look" — an artifact left unchanged is a finding about the drift, not an absence of work.

Step 4 writes exactly this table. An item that turns out to have no viable target is added as new and the divergence is named in the report.

---

## Step 4: Reconciliation (Surgical Edits)

**Constraint**: Operate strictly in place. Do not create branches, switch branches, or run feature-creation scripts. All edits target existing files in `FEATURE_DIR`.

**Scope**: skip any artifact excluded by a scope modifier, and name it in the Sync Impact Report. Out of scope means **not written**, never not read. This rule governs 4.1, 4.2 and 4.3 alike.

**Withheld items are written nowhere.** An item whose 1.1 conflict went unresolved is skipped by 4.1, 4.2 and 4.3 alike, exactly as if the gap report had not named it, and appears in the report instead.

**The citation ladder.** Wherever this command has to identify an existing item — in the Step 3 target table, in a revision note, in the report — name it by its **ID** (`FR-014`, `SC-003`, `T045`); if it has none, by its **quoted heading or opening phrase** (`"Card declined mid-checkout"`); and only if no stable phrase exists, by its file alone. **A bare section name is never acceptable** (`Edge Cases` names a section, not an item, so it identifies nothing).

**Idempotency — the key is a slug**, a short hyphenated name for the drift being fixed (`settings-nav-link`). Derive it from the gap report, but first read the `[Sync: ...]` tags already in `TASKS_FILE` and the revision notes already in `spec.md` and `plan.md`: **if one of them names the same drift, reuse that slug rather than minting a new one.** A refined report describing the same problem must produce the same key, otherwise the whole mechanism misses in the one case it exists for. The date is metadata, never part of the match.

**Idempotency is judged per artifact, not per run.** Scope modifiers mean a slug can be present in `tasks.md` while `spec.md` has never been touched under it, so check the artifact you are about to write rather than another on its behalf.

When the slug is already present in the artifact you are writing, update what that earlier run wrote — the revision note's `Items:` line names exactly which entries those were — rather than appending beside it. Two limits on "update":

- **Never edit a task marked `[X]` or `[x]`.** `/speckit.implement` marks completed work that way, and rewriting a finished task silently changes the record of what was done. If the refined report changes such a task, leave it and append a new one describing the remaining work — unless a later open task already carries this slug and covers that work, in which case it was appended on an earlier run and nothing more is needed. A task annotated `(reopened — BUG-NNN)` is not completed work (1.2), but it is not yours to edit either.
- Never delete a task an earlier run created. If the refined report drops it, leave it in place and say so in the report; removing work items is the user's call.

A re-run that finds everything already applied is a valid outcome. Report it and change nothing.

**A section the drift reaches but the artifact lacks** is created, in the position the project's own template puts it (`## Assumptions` after Success Criteria, matching `spec-template.md`'s order — `.specify/templates/` is readable for exactly this, per Allowed Sources).

**When the artifact renames its sections, map by role, not by name.** 4.1 already requires following the project's own headings, so a spec whose Success Criteria section is called `## How We'll Know It Works` puts the new section **after that section's content, before the next heading of equal or higher level** — after the section, never immediately after its heading line, which would nest the new section inside it. Match on what a section is *for*, never on its wording. When no section in the artifact plays the role the template's neighbour plays, append the new section at the end of the file and say so in the report, so the placement is a stated choice rather than a silent one.

Name every section created this way in the report — a new heading in someone's spec should never be a surprise.

**Revision notes.** Their place and form are fixed, because later runs **read** them: the slug match above depends on finding one an earlier run wrote, and the `Items:` line is what makes a re-run amend the right entries instead of guessing from prose.

```markdown
### Revision: Implementation Sync [YYYY-MM-DD] [Sync: slug]
- Reason: [Summary of drift reconciled]
- Items: [each entry this run amended or added, by the citation ladder]
```

- Written at the **bottom** of `spec.md` and `plan.md`, after the last content section. Consecutive notes read **newest last**, so they read in run order.
- **Never rewrite an earlier note**, with one exception: the note carrying **this run's slug**, which is amended in place rather than duplicated. Its date records the first sync for that slug and is never updated, so a re-run that changes nothing leaves no diff.
- Where the artifact already carries revision notes in some other place or form (an older `## Revision History` section, trailing comment lines), **leave them exactly where they are**, start this block anyway, and name the split under `## Outstanding Items` so a reader knows to look in both.
- `tasks.md` gets no revision note: the `[Sync: ...]` tag on each task is the equivalent record.

### 4.1 Update Specification (`spec.md`)

**Detect the spec's actual section names and ID convention before editing, and follow what you find.** The sections below use the canonical names; a project may use its own.

**Touch only the sections the gap report actually implicates.** This is a surgical amendment, not a spec rewrite.

- **Functional Requirements**: Amend the `FR-XXX` that states the changed capability, or add one continuing from the highest existing ID. Never reuse or renumber an existing ID. This is where behaviour drift belongs when it changes *what the system must do*, as opposed to how a scenario reads.
- **Acceptance Scenarios**: Amend an existing scenario, or add one, under the relevant `### User Story N`. Keep the Given/When/Then form the file already uses.
- **Edge Cases**: Add cases discovered during implementation, folding into an existing entry that describes the same failure mode rather than restating it.
- **Key Entities**: Add a new entity, or extend an existing one with new fields, rather than restating the entity.
- **Success Criteria**: Amend an `SC-XXX` whose target the shipped behaviour changed, or add one continuing from the highest existing ID.
- **Assumptions**: Correct an assumption the implementation invalidated, and add one the implementation now depends on.
- **User Scenarios**: Add a missing user story only when the drift is not covered by any existing one.
- **Revision Note**: **Only if one of the sections above was actually modified**, add or amend the note per the rules above. A spec this gap report did not touch gets no note.

### 4.2 Update Plan (`plan.md`)

**Detect the plan's actual section names before editing, and follow what you find**, exactly as 4.1 requires for the spec. The names below are the canonical ones and are **illustrative, not exhaustive**: a plan holds Summary, Technical Context, Project Structure, Configuration and others, and a drift that reaches one of those amends it like any other. What bounds this step is the drift, not this list.

**Touch only the sections the gap report actually implicates**, as in 4.1. A drift that reaches no plan section leaves this file alone entirely.

- **Routing & Navigation**: Add any missing routes, endpoints, or UI wiring details.
- **Integration Contracts**: Update API schemas, request/response headers, or payloads.
- **Testing Strategy**: Ensure the strategy covers the newly identified gaps.
- **Revision Note**: Add or amend a revision note if one of the sections above was modified, using the same rules as 4.1.

### 4.3 Update Tasks (`tasks.md`)
Create remediation tasks to close the drift.

If `TASKS_FILE` does not exist, create it now with a `## Remediation: Gaps` heading, and number from `T001`. Create it only at this point, and only if this step will actually write tasks into it.

**Task Formatting**:
`- [ ] T{NNN} [{story}] {action verb} {what} in {exact/file/path.ext} [Sync: slug]`

Use the user story tag the task belongs to; omit it for tasks landing in `## Remediation: Gaps`, which belong to no story. The `[Sync: ...]` tag is always appended and holds this run's slug and nothing else (for example `[Sync: settings-nav-link]`), so the same key appears in every artifact and it is everything after `Sync: `.

**Do not emit the `[P]` marker, and never strip it from a task you did not write.** In Spec-Kit it means "can run in parallel: different files, no dependencies", and `/speckit.implement` reads it to decide what to run together. A gap report cannot establish that a remediation is independent of the others, so omitting it on new tasks is always correct: they then run sequentially. Tasks written by `/speckit.tasks` carry it legitimately, and removing it would change how they execute.

**Rules for Tasks**:
1. **Increment IDs**: Find the highest `T###` in `tasks.md`. Start new tasks from `max + 1`. Never reuse or renumber.
2. **Phase Placement**: Place new tasks under the **existing** phase heading that covers the affected user story. Core `tasks.md` files write these as `## Phase N: User Story N - [Title] (Priority: PN)`. Match the heading that is already in the file; never invent a new one from the `[USn]` task tag, which is an inline marker and not a heading. If no existing phase fits, create a `## Remediation: Gaps` section at the end.
3. **Exact Paths**: Every task MUST include an exact file path where the change is needed. The path comes from the gap report, from `plan.md`'s project structure, or from the read-only repository lookup **Allowed Sources** permits — in that order of preference, and never from invention.

   **A task that creates a file names the file it will create.** Most remediation edits an existing file, but a new test or a new module does not exist yet, and "confirm the path exists" cannot apply to it. Such a path is **derived, not invented, when the directory it sits in is one the plan's Project Structure declares or the repository already contains** — `tests/integration/navigation.test.ts` is derived when `tests/integration/` exists; `src/services/settings.py` is invented when nothing declares `src/services/`.

   When neither an existing file nor a declared directory gives a path, write the task naming its target without one and list it under `## Outstanding Items`.
4. **Mandatory Integration Test**: If you identified a **Wiring & Navigation** gap, you MUST add a task for an Integration Test to verify it — unless `tasks.md` is out of scope, in which case write nothing and name the omitted test under `## Scoping`.

---

## Step 5: Sync Impact Report

Output the final report. Use **absolute paths** for all file references.

```markdown
# Sync Impact Report

## Changed Files
[Only files this run actually wrote. Omit any that were skipped or unchanged; `## Scoping` accounts for those.]
| File (absolute path) | Change Summary |
|----------------------|----------------|
| `/absolute/path/to/spec.md` | Amended FR-014, one Acceptance Scenario |
| `/absolute/path/to/plan.md` | Updated Routing/Contracts |
| `/absolute/path/to/tasks.md` | Added [N] remediation tasks |

## Counts
[The Step 3 counts line, reproduced with its units exactly as Step 3 defines it — `normalized items: M (items); rows: R (item-artifact pairs); amending: K (rows); new: N (rows); reaching no artifact: Z (items); withheld: W (items)` — plus any divergence between the target table and what was actually written. A zero here means examined and found nothing to amend.]

## Sources
[Confirm every change came only from the Allowed Sources. Declare the read-only repository lookup whenever it happened, naming what it resolved. Name anything you needed but could not find, and state that you did not reconstruct it or invent a path. If you consulted git to verify your own writes rather than to obtain content, say so here.]

## Path Resolution
[`FEATURE_DIR` and how it was resolved. Note it when `{SCRIPT}` reported a different feature directory, or when the script failed and `REPO_ROOT` was derived by walking up from the argument. Otherwise "Resolved from argument".]

## Constitution Compliance
[Every 1.1 finding with its disposition: the user's Step 2 answer, or why it is still unresolved. Name each withheld item, the rule it conflicts with, and the recommendation to resolve it and re-run. A finding the user closed ("not actually triggered", "accepted gap") belongs here too — a closed finding that appears nowhere is indistinguishable from one that was never raised. State plainly that an accepted gap is recorded nowhere but here, so a re-run will detect and ask again. Or "None".]

## Scoping
[Which artifacts were skipped due to scope modifiers, and which were left unchanged because an earlier run under this slug had already applied them. **For each skipped artifact, name what this drift would have written into it** — a skipped artifact is not "nothing happened", it is a known, named omission, and a reader who cannot see what was left out cannot decide whether to re-run. That includes any mandatory integration test not written, and any remediation task an obligation asked for that 1.1's carve-out suppressed. State the interpretation if the report's final words were one of the three modifiers. `REPO_ROOT` derivation belongs in `## Path Resolution`, not here.]

## New Remediation Tasks
[List the new tasks added, e.g.]
- **T045**: Add sidebar link in `src/components/Sidebar.tsx`
- **T046**: Update router in `src/router/index.ts`
- **T047**: Integration test: navigate to Settings in `tests/integration/navigation.test.ts`

[On a re-run, list what was already present and left unchanged, or "Already applied; no changes".]

## Outstanding Items
[Everything this run noticed and did not act on, in one place — the section formerly called `Outstanding Decisions`, widened. Any item withheld by an unresolved constitution conflict, with the rule it conflicts with and the recommendation to resolve it and re-run. Any remaining `NEEDS CLARIFICATION` markers. Sections created because the artifact lacked them, and any section appended at the end of a file because no template position could be mapped. Tasks written without a path because none resolved. Struck-through text left as it stands because it had neither a Bugfix marker nor a replacement, and any overlap with a task reopened by a bugfix cycle. Any revision notes found in another place or form, with the split named. Each action-requiring constitution rule reported as unverified, with what would settle it — these are unverified, not violated. Any discretionary question the budget could not hold. Or "None".]

## Defaults Applied
[Any decision made with a reasonable default instead of asking, or "None"]

## Gap Report
[The gap report verbatim, and any part set aside because it asked for something the Gap Report Contract forbids, with the reason.]

## Next Step
[Recommend based on what changed:]
- If remediation tasks were added → `/speckit.implement` to execute them
- If plan was significantly updated → `/speckit.plan` to review architecture
- If only spec was updated → Review changes and proceed with implementation
- **If this feature already has an entry in `.specify/memory/changelog.md`** (noted in 0.2) → `/speckit.archive.run specs/###-feature-name` to refresh project memory. The archived copy predates these edits and is now stale. Archiving is idempotent per feature, so re-running it updates the existing record rather than duplicating it.
```

---

## Done Criteria
Each criterion below applies only to artifacts in scope; an artifact a modifier excluded is named in the report instead.

- A rejected invocation wrote nothing: ranges, globs, a second feature, an unrecognized leading flag, a missing gap report, and a first token matching zero or several directories all stop the run before any edit.
- Gap report parsed and categorized against a feature directory that resolved to exactly one existing path under `REPO_ROOT`.
- All content taken only from the Allowed Sources, and the report says so. Every task path came from the gap report, the plan, or the read-only lookup — none invented, and any task left without one is named.
- Gap report applied within its limits: echoed verbatim in the report, with any part that asked to widen sources, skip a step, change scope or IDs, or authorize a deletion named as refused.
- All three constitution shapes checked against the rules the drift reaches, each finding carried to Step 2 or reported as unverified, and every finding's disposition recorded. No content withheld except by an unresolved conflict, and no statement written that the gap report did not supply — an obligation the run's own amendment satisfies from the report is met, not invented.
- Edits match the Step 3 target table, and any divergence is named.
- Every spec section the drift reaches is amended, and no section it does not reach is touched. Any section created is named in the report.
- `tasks.md` updated with incremented `T###` IDs, exact file paths, and no `[P]` marker **on the tasks this run added**. Tasks written by `/speckit.tasks` legitimately carry `[P]`; never strip it from them.
- Integration test task added for wiring gaps.
- Revision note added or amended on each modified `spec.md` / `plan.md`, carrying this run's slug and its `Items:` line; for `tasks.md` the `[Sync: ...]` tag is the equivalent record. No earlier note rewritten.
- No task marked `[X]` was edited, no task reopened by a bugfix cycle was repurposed, no struck-through wording was restored, and no task from an earlier run was deleted.
- Re-running the same gap report changes nothing further.
- Sync Impact Report printed with absolute paths.
