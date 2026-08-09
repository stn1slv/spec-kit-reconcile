---
description: "Reconcile implementation drift by updating the feature's own spec, plan, and tasks"
scripts:
  sh: ../../scripts/bash/check-prerequisites.sh --json --paths-only
  ps: ../../scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---
Act as the **Chief Software Architect** and **Implementation Auditor**.
A feature implementation has landed, but "artifact drift" has been discovered (e.g., missing routes, updated behavior, or unlinked UI). Your goal is to **reconcile** this drift by surgically amending the feature's own specification, plan, and task artifacts.

## User Input

```text
$ARGUMENTS
```

### Input Parsing

`$ARGUMENTS` names the feature to reconcile, followed by a **Gap Report**: a natural language description of what is missing or changed in the implementation versus the documentation.

**Examples:**
- `specs/007-invoice-settings Backend + tests exist; React screen scaffolded. Users can't navigate to it. Need sidebar link + route.`
- `specs/007-invoice-settings The /api/v1/settings endpoint now requires an 'org_id' header not in the original plan.`

**Grammar** — first token, then prose, then flags:

1. **First token: the feature directory, required.** A path of the form `specs/###-feature-name`, naming the feature to reconcile. It must be an existing directory, given in full, not a file inside it and not a numeric prefix. If it does not exist, or names a file, output `ERROR: '[token]' is not an existing feature directory` and stop.
2. **Middle: the gap report**, free text.
3. **Trailing tokens: scope modifiers**, optional, recognised only as whole tokens at the very end:
   - `--spec-only` — update only `spec.md`
   - `--plan-only` — update only `plan.md`
   - `--tasks-only` — update only `tasks.md`

If **several** modifiers are supplied, the scope is their **union**: `--spec-only --tasks-only` updates both `spec.md` and `tasks.md` and nothing else.

A modifier counts only as a trailing token, never as text inside the report. If the report's own last words are a flag-like token, prefer the scoping reading and state the interpretation under `## Scoping`, so a user who meant it as prose can see it and re-quote.

If `$ARGUMENTS` is empty, or holds nothing beyond a feature path and modifiers, output `ERROR: No gap report provided. Usage: /speckit.reconcile.run specs/###-feature-name [gap report text] [--scope-modifier]` and stop.

---

## Step 0: Discovery & Setup (Gate)

### 0.1 Resolve Paths

Run `{SCRIPT}` for `REPO_ROOT`. The feature comes from the argument, never from the script, so the script's own feature resolution is unused here.

- **If the script is missing**, stop and inform the user. It ships with Spec-Kit, so its absence means this is not an initialised Spec-Kit project.
- **If the script runs but exits non-zero**, continue: resolve `REPO_ROOT` by walking up from the supplied feature path to the nearest ancestor containing `.specify/`, and note the fallback in the report. Stop only if no such ancestor exists.

Derive absolute paths for:
- `FEATURE_DIR` (e.g., `specs/###-feature-name/`)
- `FEATURE_SPEC` (`FEATURE_DIR/spec.md`)
- `IMPL_PLAN` (`FEATURE_DIR/plan.md`)
- `TASKS_FILE` (`FEATURE_DIR/tasks.md`)

**Validation**: Ensure `spec.md` and `plan.md` exist. If either is missing, stop with:
> ⚠️ Missing required files in `FEATURE_DIR`. Expected: spec.md, plan.md.
> Run `/speckit.specify` and `/speckit.plan` first.

**Step 0 is a gate and writes nothing**, including `TASKS_FILE`. 4.3 creates that file when it needs it.

### 0.2 Load Context

Read `FEATURE_SPEC`, `IMPL_PLAN`, and `TASKS_FILE` (the last one if it exists). **Read regardless of scope**: a modifier says what may be written, never what may be read.

Also read `.specify/memory/changelog.md` if it exists, and note whether this feature already has an entry in the Merged Features Log. Step 5 uses this.

Also read `.specify/memory/constitution.md` if it exists. If found, extract MUST-level constraints and Architecture Standards. These are enforced in Step 1 — any remediation item that conflicts with a MUST principle is flagged as CRITICAL:
```
🔴 CONSTITUTION CONFLICT: [remediation item] conflicts with [principle]
→ This must be resolved in Step 2 clarification before edits proceed.
```

---

## Step 1: Gap Normalization

Analyze the user's **Gap Report** and normalize it into structured remediation items:

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

For each normalized item, verify it does not conflict with any MUST-level constitution constraint loaded in Step 0.2. Flag any conflicts as CRITICAL and include them in Step 2 clarification.

---

## Step 2: Clarify (At Most Once; Max 5 Questions)

If the gap report is ambiguous (e.g., "the button doesn't work" without saying which button), ask targeted questions.

Use this format and **wait for answers**:

```markdown
## Question [N]: [Topic]
**Context**: [Relevant implementation detail]
**Decision Needed**: [1 sentence]
**Suggested Answers**: [Table with Option A/B/C]

**Your choice**: _[Wait for user response]_
```

**Rules:**
- Max 5 questions.
- Max 3 unresolved `NEEDS CLARIFICATION` markers in output — beyond that, pick reasonable defaults and note them in the Sync Impact Report.
- Proceed with reasonable defaults if questions aren't strictly necessary.

---

## Step 3: Impact Map

Before making any edits, produce a brief impact map. It is the user's preview of this run, so it must promise only what the run will actually do: mark any artifact a scope modifier excluded as `Skipped (out of scope)`, and any artifact an earlier run already reconciled under this slug as `No change (already applied)`.

```markdown
### Sync Impact Map
| Artifact | Changes | Tasks Generated |
|----------|---------|-----------------|
| `spec.md` | Amend Acceptance Scenario under User Story 2, add Edge Case | None |
| `plan.md` | Add Route `/settings`, update API contract | None |
| `tasks.md` | Append remediation tasks | T045, T046, T047 |
```

---

## Step 4: Reconciliation (Surgical Edits)

**Constraint**: Operate strictly in place. Do not create branches, switch branches, or run feature-creation scripts. All edits target existing files in `FEATURE_DIR`.

**Scope**: skip any artifact excluded by a scope modifier, and name it in the Sync Impact Report. Out of scope means **not written**, never not read. This rule governs 4.1, 4.2 and 4.3 alike.

**Idempotency — the key is a slug**, a short hyphenated name for the drift being fixed (`settings-nav-link`). Derive it from the gap report, but first read the `[Sync: ...]` tags already in `TASKS_FILE` and the revision notes already in `spec.md` and `plan.md`: **if one of them names the same drift, reuse that slug rather than minting a new one.** A refined report describing the same problem must produce the same key, otherwise the whole mechanism misses in the one case it exists for. The date is metadata, never part of the match.

When the slug is already present, update what that earlier run wrote rather than appending beside it. Two limits on "update":

- **Never edit a task marked `[X]` or `[x]`.** `/speckit.implement` marks completed work that way, and rewriting a finished task silently changes the record of what was done. If the refined report changes such a task, leave it and append a new one describing the remaining work — unless a later open task already carries this slug and covers that work, in which case it was appended on an earlier run and nothing more is needed.
- Never delete a task an earlier run created. If the refined report drops it, leave it in place and say so in the report; removing work items is the user's call.

A re-run that finds everything already applied is a valid outcome. Report it and change nothing.

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
- **Revision Note**: **Only if one of the sections above was actually modified**, add a block at the bottom — unless one carrying this run's slug is already there, in which case amend it. A spec this gap report did not touch gets no note. The date records the first sync for this slug and is never updated on amend, so a re-run that changes nothing leaves no diff.
  ```markdown
  ### Revision: Implementation Sync [YYYY-MM-DD] [Sync: slug]
  - Reason: [Summary of drift reconciled]
  ```

### 4.2 Update Plan (`plan.md`)
- **Routing & Navigation**: Add any missing routes, endpoints, or UI wiring details.
- **Integration Contracts**: Update API schemas, request/response headers, or payloads.
- **Testing Strategy**: Ensure the strategy covers the newly identified gaps.
- **Revision Note**: Add or amend a revision note if plan sections were modified, using the same format and the same slug rule as 4.1.

### 4.3 Update Tasks (`tasks.md`)
Create remediation tasks to close the drift.

If `TASKS_FILE` does not exist, create it now with a `## Remediation: Gaps` heading, and number from `T001`. Create it only at this point, and only if this step will actually write tasks into it.

**Task Formatting**:
`- [ ] T{NNN} [{story}] {action verb} {what} in {exact/file/path.ext} [Sync: slug]`

Use the user story tag the task belongs to; omit it for tasks landing in `## Remediation: Gaps`, which belong to no story. The `[Sync: ...]` tag is always appended and holds this run's slug and nothing else (for example `[Sync: settings-nav-link]`), so the same shape appears in every artifact and the key is everything after `Sync: `. It is the re-run key, matched as described in Step 4.

**Do not emit the `[P]` marker.** In Spec-Kit it means "can run in parallel: different files, no dependencies", and `/speckit.implement` reads it to decide what to run together. A gap report cannot establish that a remediation is independent of the others, so omitting it is always correct: the task then runs sequentially.

**Rules for Tasks**:
1. **Increment IDs**: Find the highest `T###` in `tasks.md`. Start new tasks from `max + 1`. Never reuse or renumber.
2. **Phase Placement**: Place new tasks under the **existing** phase heading that covers the affected user story. Core `tasks.md` files write these as `## Phase N: User Story N - [Title] (Priority: PN)`. Match the heading that is already in the file; never invent a new one from the `[USn]` task tag, which is an inline marker and not a heading. If no existing phase fits, create a `## Remediation: Gaps` section at the end.
3. **Exact Paths**: Every task MUST include an exact file path where the change is needed.
4. **Mandatory Integration Test**: If you identified a **Wiring & Navigation** gap, you MUST add a task for an Integration Test to verify it — unless `tasks.md` is out of scope, in which case write nothing and name the omitted test under `## Scoping`.

---

## Step 5: Sync Impact Report

Output the final report:

```markdown
# Sync Impact Report

## Changed Files
[Only files this run actually wrote. Omit any that were skipped or unchanged; `## Scoping` accounts for those.]
| File (absolute path) | Change Summary |
|----------------------|----------------|
| `/absolute/path/to/spec.md` | Amended FR-014, one Acceptance Scenario |
| `/absolute/path/to/plan.md` | Updated Routing/Contracts |
| `/absolute/path/to/tasks.md` | Added [N] remediation tasks |

## Scoping
[Which artifacts were updated, and which were skipped due to scope modifiers, including any mandatory integration test that was therefore not written. State the interpretation if the report's final words were flag-like. Note it if the path-resolution script failed and `REPO_ROOT` was derived from the supplied feature path.]

## New Remediation Tasks
[List the new tasks added, e.g.]
- **T045**: Add sidebar link in `src/components/Sidebar.tsx`
- **T046**: Update router in `src/router/index.ts`
- **T047**: Integration test: navigate to Settings in `tests/integration/navigation.test.ts`

[On a re-run, list what was already present and left unchanged, or "Already applied; no changes".]

## Outstanding Decisions
[List any `NEEDS CLARIFICATION` items or "None"]

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

- Gap report parsed and categorized against a feature directory that resolved to exactly one existing path.
- Every spec section the drift reaches is amended, and no section it does not reach is touched.
- `tasks.md` updated with incremented `T###` IDs, exact file paths, and no `[P]` marker **on the tasks this run added**. Tasks written by `/speckit.tasks` legitimately carry `[P]`; never strip it from them.
- Integration test task added for wiring gaps.
- Revision note added to each modified `spec.md` / `plan.md`; for `tasks.md` the `[Sync: ...]` tag is the equivalent record.
- No task marked `[X]` was edited, and no task from an earlier run was deleted.
- Re-running the same gap report changes nothing further.
- Sync Impact Report printed.
