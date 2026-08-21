# Spec-Kit Reconcile

A Spec-Kit extension to reconcile documentation with implementation drift.

## Overview

`speckit.reconcile.run` closes the gap that opens after a feature ships. You describe what drifted in plain language; the command resolves the feature's paths, edits the sections of `spec.md` and `plan.md` the drift actually reaches, and appends remediation tasks to `tasks.md`.

It is built for the PR phase, where the code has settled and the feature's own artifacts have fallen behind it.

## Features

- **Gap report input**: Free-form notes about what was missed or changed during implementation. The report steers the run without overriding it. It can direct attention and emphasis, but it cannot widen the source boundary, skip a check, change the scope, renumber anything, or authorize a deletion. Ask for one of those and the command refuses that part, finishes the run anyway, and names the refusal in its report. The report is also echoed verbatim, so every run stays auditable.
- **Bounded inputs**: The command declares every file it may take content from. Git history, deleted files, another feature's spec directory, bug reports and agent memory stores are not sources. It may read the repository's own source files, read-only, for exactly one purpose: turning a described target ("the sidebar") into a real path. No task path is ever invented. When none resolves, the task names its target without a path and the omission is reported, because a task pointing at a file that does not exist gets discovered during implementation, while an honest one gets discovered during review.
- **Full spec coverage**: Amends whichever sections the drift reaches, among Functional Requirements, Acceptance Scenarios, Edge Cases, Key Entities, Success Criteria and Assumptions, and leaves the rest alone. Where the drift reaches a section the file lacks, the command creates it in the position the template puts it and reports the creation.
- **Previewed targets**: Before any edit, the run prints a target table naming the existing item each amendment will land on, by ID or by quoted phrase, with the counts behind it. Without that preview the same choice made twice can come out two different ways, and a zero stays ambiguous when it needs to read as "examined and found nothing to amend" rather than "did not look".
- **Remediation tasks**: New `T###` entries in `tasks.md` with incremented IDs and exact file paths, placed under a phase heading already in the file, and never marked `[P]`.
- **Safe to re-run**: Tasks carry a `[Sync: ...]` tag, and revision notes carry the same slug plus the list of items they touched. A refined version of the same gap report updates the earlier result instead of duplicating it, and a re-run amends the same entries instead of finding them again by prose. Idempotency is judged per artifact, so a scope-limited run is not mistaken for a completed one.
- **Enforced verification**: Any wiring or navigation gap the run discovers gets a mandatory integration test task.
- **Constitution compliance**: Checks the rules the drift reaches, in the three ways a MUST rule can fail. A *conflict* is a remediation item that contradicts a rule; you are asked about it, and one left unresolved withholds that item alone while the rest of the gap report is reconciled and the run completes. An *unmet obligation* is a rule requiring a statement the feature never makes. You are asked, it never withholds anything, and the command will not write the statement for you, since a remediation task directing the author to make it is the honest remedy. An *action-requiring* rule ("all API routes MUST have automated tests") is reported as unverified rather than flagged, because this command reads artifacts and cannot inspect a test run.
- **Aware of other writers**: Other commands patch the same three files, and this one reads what they leave behind. Wording struck through by a bugfix patch is never restored, `**Bugfix**:` lines stay as metadata, and a task reopened by a bug counts as incomplete but is never repurposed. Entries a revision marked `RETIRED`, `SUPERSEDED by [ID]` or `CANCELLED` are read the same way: the marker records a decision somebody already made, so the amendment lands on the live entry and the marked one is reported instead of revived.
- **Reporting**: Absolute paths throughout, a per-finding constitution disposition, an explicit sources declaration, and a conditional "Next Step" that routes to `/speckit.archive.run` when the reconciled feature has already been archived. The [`archive` extension](https://github.com/stn1slv/spec-kit-archive) is an optional companion, not a requirement. It owns both that command and the `.specify/memory/changelog.md` this check reads, so a project without it simply never sees the recommendation.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add reconcile --from https://github.com/stn1slv/spec-kit-reconcile/archive/refs/tags/v1.2.1.zip
```
*(Note: Replace `v1.2.1` with the latest release version)*

To upgrade an existing installation, add `--force`. Without it the CLI refuses to overwrite the installed version:

```bash
specify extension add reconcile --from https://github.com/stn1slv/spec-kit-reconcile/archive/refs/tags/v1.2.1.zip --force
```

`specify extension update` does not work for this extension, and will not in future: the community catalog is deliberately discovery-only (`install_allowed: false`), so `update` skips everything in it rather than pulling unvetted third-party code. `--from` with an explicit URL is the supported upgrade path. To find that URL without leaving the CLI:

```bash
specify extension info reconcile   # prints a "Candidate archive" URL for review
```

Requires Spec Kit 0.16.2 or newer. The command resolves templates through the override stack that release introduced, so an older CLI would read the base template and ignore any preset layered over it.

## Usage

Name the feature, then describe the drift in plain text:

```bash
/speckit.reconcile.run specs/007-invoice-settings Backend exists, but the React screen is unreachable; need sidebar link and route
```

The feature path is required and must resolve to exactly one existing directory; a unique numeric prefix such as `specs/007` is accepted. This command rewrites `spec.md`, `plan.md` and `tasks.md` in place, so it never infers which feature you meant.

**One feature per run.** There is no batch or range mode. `specs/001 thru specs/008` and `specs/00*` are rejected, and so is a second `specs/` path anywhere in the input. Refer to another feature by name in prose ("the invoice feature") when the report has to mention one. Ordinary prose, punctuation, numbers inside sentences and source paths such as `src/router/index.ts` are all fine.

Re-running the same gap report is safe: it updates what it already wrote instead of appending a second set of tasks, and it never edits a task `/speckit.implement` has already marked complete.

**On a brownfield feature, keep the gap report narrow.** A spec written for an existing codebase deliberately covers only the change it introduces, not a retroactive account of everything already there. So "the code does X and the spec never mentions it" is the expected state for anything outside that scope, not drift. Scope the report to the change the spec claims to cover; a broad one will back-fill out-of-scope behaviour into a spec that was meant to stay narrow.

The extension registers optional hooks on `after_implement` and `after_converge`, so both commands offer the reconcile step when they finish. They are prompts, never automatic runs, since the command needs a gap report and will not invent one.

The command also declares `handoffs`, which render as follow-on buttons. Those appear only on agents that install commands as files. The twenty or so agents using the skills layout, Claude Code and Copilot among them, build their skill frontmatter from scratch and drop the key. Nothing is lost either way, because the run's own `## Next Step` section does the same routing in text on every agent.

You can optionally restrict the scope of the updates, placing the modifiers immediately after the feature path:
- `--spec-only`: update only `spec.md`
- `--plan-only`: update only `plan.md`
- `--tasks-only`: update only `tasks.md`

Several modifiers combine as a union: `--spec-only --tasks-only` writes both and nothing else. Modifiers are also still recognised as trailing tokens at the very end of the input, which is how earlier versions accepted them. That position is ambiguous when the report's own last words look like a flag, so the leading position is preferred and the run states its interpretation when it has to guess.

## How this differs from `/speckit.converge` and `/speckit.analyze`

Two core commands sit near this one, and the three take different views of which side of the gap is wrong.

| Command | Treats as true | Reads | Writes |
|---|---|---|---|
| `/speckit.converge` (core) | the **artifacts** | spec, plan, tasks, plus the codebase | appends unbuilt work to `tasks.md`, append-only |
| `/speckit.analyze` (core) | neither | spec, plan, tasks | nothing; it reports inconsistency |
| `/speckit.reconcile.run` | the **shipped code**, as the gap report states it | spec, plan, tasks, the constitution | amends `spec.md` and `plan.md`, appends remediation tasks |

Use `converge` when the code lags a settled specification: it finds what was never built. Use this command for the mirror case, when the code shipped and the artifacts were left behind. They are not alternatives, and a feature often needs both, in either order. This command never writes into a `## Phase N: Convergence` section, so their outputs stay separable.

Where a requirement is *changing* rather than drifting, meaning the product decided differently and an existing requirement is now wrong, neither command is the right one. That is a revision, and this command will not revive an entry a revision has already marked `RETIRED` or `SUPERSEDED`.

## Workflow

1.  **Parse the gap report** to determine what drift occurred, rejecting ranges, globs, a second feature, and unknown flags before anything is written.
2.  **Resolve paths**: read `REPO_ROOT` from the core `check-prerequisites.sh` script, then resolve the feature under it from the path you passed. A disagreement with the script's own feature is reported.
3.  **Normalize gaps** into categories (Wiring & Navigation, Contracts, Test Coverage, and so on) and check them against the constitution rules they reach.
4.  **Preview**: an impact map and a target table naming the item each amendment will land on, before any edit.
5.  **Edit** the feature's own `spec.md`, `plan.md`, and `tasks.md`, and nothing else.
6.  **Output a Sync Impact Report** detailing the created tasks and next steps, such as routing to `/speckit.implement`.

## Tests

`tests/fixture/` holds a minimal spec-kit project with deliberate traps and a pre-registered `EXPECTATIONS.md`. A test runner is a fresh agent given only `commands/reconcile.md` and its own working copy; its output is compared to the expectations. Expectations for a round must land in a commit **before** that round's runs, so the claim "written first" is auditable.
