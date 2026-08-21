# Spec-Kit Reconcile

A Spec-Kit extension to reconcile documentation with implementation drift.

## Overview

The `speckit.reconcile.run` command is a **Post-Implementation Gap Closer**. It analyzes a natural-language gap report, resolves paths, surgically updates the feature's `spec.md` and `plan.md`, and appends actionable remediation tasks to `tasks.md`.

This extension acts as the "Inner Loop" of the Double-Loop Parity framework: it ensures that during the PR phase, the *feature artifacts* are continuously aligned with the shipped code.

## Features

- **Gap Report Input**: Accepts free-form natural language observations about what was missed or changed during implementation. The report steers the run and cannot override it: it may direct attention and emphasis, but it may not widen the source boundary, skip a check, change the scope, renumber anything, or authorize a deletion. A report asking for one of those is refused, the run continues, and the refusal is named in the report — which also echoes the report verbatim, so every run stays auditable.
- **Bounded inputs**: Declares the complete list of files it may take content from. Git history, deleted files, another feature's spec directory, bug reports and agent memory stores are not sources. The repository's own source files may be read read-only for exactly one purpose: resolving a described target ("the sidebar") to a real path. **No task path is ever invented** — when none resolves, the task names its target without a path and the omission is reported, because a task pointing at a file that does not exist is discovered during implementation while an honest one is discovered during review.
- **Full Spec Coverage**: Amends whichever sections the drift actually reaches — Functional Requirements, Acceptance Scenarios, Edge Cases, Key Entities, Success Criteria and Assumptions — and leaves the rest alone. A section the drift reaches but the file lacks is created in the position the template puts it, and the creation is reported.
- **Previewed targets**: Before any edit, the run prints a target table naming the existing item each amendment will land on, by ID or by quoted phrase, and the counts behind it. The same choice made twice can otherwise come out two different ways, and a zero has to be legible as "examined and found nothing to amend" rather than "did not look".
- **Remediation Engine**: Appends new tasks (`T###`) to `tasks.md` with auto-incremented IDs and exact file paths, under the phase heading already in the file, and never marked `[P]`.
- **Safe to Re-run**: Tasks carry a `[Sync: ...]` tag and revision notes carry the same slug plus the list of items they touched, so a refined version of the same gap report updates the earlier result instead of duplicating it, and a re-run amends the same entries instead of re-finding them by prose. Idempotency is judged per artifact, so a scope-limited run is not mistaken for a completed one.
- **Enforced Verification**: Automatically mandates integration test tasks for any discovered wiring or navigation gaps.
- **Constitution compliance**: Checks the rules the drift actually reaches, in the three ways a MUST rule can fail. A **conflict** is a remediation item contradicting a rule; you are asked, and an unresolved one withholds *that item only* while the rest of the gap report is reconciled and the run completes. An **unmet obligation** is a rule requiring a statement the feature never makes — asked, never a reason to withhold anything, and never written for you, since a remediation task directing the author to make it is the honest remedy. An **action-requiring** rule ("all API routes MUST have automated tests") is reported as unverified and never flagged, because this command reads artifacts and cannot inspect a test run.
- **Aware of other writers**: Works alongside the other commands that patch the same three files. Struck-through wording superseded by a bugfix patch is never restored, `**Bugfix**:` lines are left as metadata, and a task reopened by a bug is treated as incomplete but never repurposed. Entries a revision marked `RETIRED`, `SUPERSEDED by [ID]` or `CANCELLED` are read the same way: the marker records a decision somebody already made, so the amendment lands on the live entry and the marked one is reported instead of revived.
- **Actionable Reporting**: Absolute paths throughout, a per-finding constitution disposition, an explicit sources declaration, and a conditional "Next Step" that routes to `/speckit.archive.run` when the reconciled feature has already been archived.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add reconcile --from https://github.com/stn1slv/spec-kit-reconcile/archive/refs/tags/v1.2.1.zip
```
*(Note: Replace `v1.2.1` with the latest release version)*

To upgrade an existing installation, add `--force` — without it the CLI refuses to overwrite the installed version:

```bash
specify extension add reconcile --from https://github.com/stn1slv/spec-kit-reconcile/archive/refs/tags/v1.2.1.zip --force
```

Once the extension is listed in the community catalog at this version, `specify extension update reconcile` does the same thing without the URL.

Requires Spec Kit **0.16.2 or newer**: the command resolves templates through the override stack that release introduced, so an older CLI would read the base template and ignore any preset layered over it.

## Usage

Name the feature, then describe the drift in plain text:

```bash
/speckit.reconcile.run specs/007-invoice-settings Backend exists, but the React screen is unreachable; need sidebar link and route
```

The feature path is required and must resolve to exactly one existing directory; a unique numeric prefix such as `specs/007` is accepted. This command rewrites `spec.md`, `plan.md` and `tasks.md` in place, so it never infers which feature you meant.

**One feature per run.** There is no batch or range mode: `specs/001 thru specs/008` and `specs/00*` are rejected, and so is a second `specs/` path anywhere in the input — refer to another feature by name in prose ("the invoice feature") when the report has to mention one. Ordinary prose, punctuation, numbers inside sentences and source paths such as `src/router/index.ts` are fine.

Re-running the same gap report is safe: it updates what it already wrote instead of appending a second set of tasks, and it never edits a task `/speckit.implement` has already marked complete.

The extension also registers an **optional** `after_implement` hook, so `/speckit.implement` offers the reconcile step when it finishes. It is a prompt, never an automatic run: the command needs a gap report and will not invent one.

You can optionally restrict the scope of the updates, placing the modifiers immediately after the feature path:
- `--spec-only` — update only `spec.md`
- `--plan-only` — update only `plan.md`
- `--tasks-only` — update only `tasks.md`

Several modifiers combine as a union: `--spec-only --tasks-only` writes both and nothing else. Modifiers are also still recognised as trailing tokens at the very end of the input, which is how earlier versions accepted them; that position is ambiguous when the report's own last words look like a flag, so the leading position is preferred and the run states its interpretation when it has to guess.

## How this differs from `/speckit.converge` and `/speckit.analyze`

Three commands sit near this one, and they take opposite views of which side of the gap is wrong.

| Command | Treats as true | Reads | Writes |
|---|---|---|---|
| `/speckit.converge` (core) | the **artifacts** | spec, plan, tasks, plus the codebase | appends unbuilt work to `tasks.md`, append-only |
| `/speckit.analyze` (core) | neither | spec, plan, tasks | nothing; it reports inconsistency |
| `/speckit.reconcile.run` | the **shipped code**, as the gap report states it | spec, plan, tasks, the constitution | amends `spec.md` and `plan.md`, appends remediation tasks |

Use `converge` when the code lags a settled specification: it finds what was never built. Use this command for the mirror case, when the code shipped and the artifacts were left behind. They are not alternatives and a feature often needs both, in either order; this command never writes into a `## Phase N: Convergence` section, so their outputs stay separable.

Where a requirement is *changing* rather than drifting — the product decided differently, and an existing requirement is now wrong — neither command is the right one. That is a revision, and this command will not revive an entry a revision has already marked `RETIRED` or `SUPERSEDED`.

## Workflow

1.  **Parse the Gap Report** to determine what drift occurred, rejecting ranges, globs, a second feature, and unknown flags before anything is written.
2.  **Resolve paths**: read `REPO_ROOT` from the core `check-prerequisites.sh` script, then resolve the feature under it from the path you passed. A disagreement with the script's own feature is reported.
3.  **Normalize Gaps** into categories (Wiring & Navigation, Contracts, Test Coverage, etc.) and check them against the constitution rules they reach.
4.  **Preview**: an impact map and a target table naming the item each amendment will land on, before any edit.
5.  **Surgically Edit** the feature's specific `spec.md`, `plan.md`, and `tasks.md`.
6.  **Output a Sync Impact Report** detailing the created tasks and next steps (e.g., routing to `/speckit.implement`).

## Tests

`tests/fixture/` holds a minimal spec-kit project with deliberate traps and a pre-registered `EXPECTATIONS.md`. A test runner is a fresh agent given only `commands/reconcile.md` and its own working copy; its output is compared to the expectations. Expectations for a round must land in a commit **before** that round's runs, so the claim "written first" is auditable.
