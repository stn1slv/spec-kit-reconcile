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
- **Bugfix-aware**: Works alongside bugfix extensions that patch the same three files. Struck-through wording superseded by a patch is never restored, `**Bugfix**:` lines are left as metadata, and a task reopened by a bug is treated as incomplete but never repurposed.
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

## Usage

Name the feature, then describe the drift in plain text:

```bash
/speckit.reconcile.run specs/007-invoice-settings Backend exists, but the React screen is unreachable; need sidebar link and route
```

The feature path is required and must resolve to exactly one existing directory; a unique numeric prefix such as `specs/007` is accepted. This command rewrites `spec.md`, `plan.md` and `tasks.md` in place, so it never infers which feature you meant.

**One feature per run.** There is no batch or range mode: `specs/001 thru specs/008` and `specs/00*` are rejected, and so is a second `specs/` path anywhere in the input — refer to another feature by name in prose ("the invoice feature") when the report has to mention one. Ordinary prose, punctuation, numbers inside sentences and source paths such as `src/router/index.ts` are fine.

Re-running the same gap report is safe: it updates what it already wrote instead of appending a second set of tasks, and it never edits a task `/speckit.implement` has already marked complete.

You can optionally restrict the scope of the updates, placing the modifiers immediately after the feature path:
- `--spec-only` — update only `spec.md`
- `--plan-only` — update only `plan.md`
- `--tasks-only` — update only `tasks.md`

Several modifiers combine as a union: `--spec-only --tasks-only` writes both and nothing else. Modifiers are also still recognised as trailing tokens at the very end of the input, which is how earlier versions accepted them; that position is ambiguous when the report's own last words look like a flag, so the leading position is preferred and the run states its interpretation when it has to guess.

## Workflow

1.  **Parse the Gap Report** to determine what drift occurred, rejecting ranges, globs, a second feature, and unknown flags before anything is written.
2.  **Resolve paths**: read `REPO_ROOT` from the core `check-prerequisites.sh` script, then resolve the feature under it from the path you passed. A disagreement with the script's own feature is reported.
3.  **Normalize Gaps** into categories (Wiring & Navigation, Contracts, Test Coverage, etc.) and check them against the constitution rules they reach.
4.  **Preview**: an impact map and a target table naming the item each amendment will land on, before any edit.
5.  **Surgically Edit** the feature's specific `spec.md`, `plan.md`, and `tasks.md`.
6.  **Output a Sync Impact Report** detailing the created tasks and next steps (e.g., routing to `/speckit.implement`).

## Tests

`tests/fixture/` holds a minimal spec-kit project with deliberate traps and a pre-registered `EXPECTATIONS.md`. A test runner is a fresh agent given only `commands/reconcile.md` and its own working copy; its output is compared to the expectations. Expectations for a round must land in a commit **before** that round's runs, so the claim "written first" is auditable.
