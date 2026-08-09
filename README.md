# Spec-Kit Reconcile

A Spec-Kit extension to reconcile documentation with implementation drift.

## Overview

The `speckit.reconcile.run` command is a **Post-Implementation Gap Closer**. It analyzes a natural-language gap report, resolves paths, surgically updates the feature's `spec.md` and `plan.md`, and appends actionable remediation tasks to `tasks.md`.

This extension acts as the "Inner Loop" of the Double-Loop Parity framework: it ensures that during the PR phase, the *feature artifacts* are continuously aligned with the shipped code.

## Features

- **Gap Report Input**: Accepts free-form natural language observations about what was missed or changed during implementation.
- **Full Spec Coverage**: Amends whichever sections the drift actually reaches — Functional Requirements, Acceptance Scenarios, Edge Cases, Key Entities, Success Criteria and Assumptions — and leaves the rest alone.
- **Remediation Engine**: Appends new tasks (`T###`) to `tasks.md` with auto-incremented IDs and exact file paths.
- **Safe to Re-run**: Tasks carry a `[Sync: ...]` tag, so submitting a refined version of the same gap report updates the earlier result instead of duplicating it.
- **Enforced Verification**: Automatically mandates integration test tasks for any discovered wiring or navigation gaps.
- **Compliance Checks**: Includes lightweight validations against the project's `constitution.md` to prevent violating core "MUSTs".
- **Actionable Reporting**: Provides a conditional "Next Step" in the Sync Impact Report.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add reconcile --from https://github.com/stn1slv/spec-kit-reconcile/archive/refs/tags/v1.1.0.zip
```
*(Note: Replace `v1.1.0` with the latest release version)*

## Usage

Provide a plain-text gap report to the command describing the implementation drift:

```bash
/speckit.reconcile.run "Backend exists, but React screen is unreachable; need sidebar link and route"
```

By default the command reconciles the feature you worked on last. Pass a feature path first to reconcile a different one:

```bash
/speckit.reconcile.run specs/007-invoice-settings "The /api/v1/settings endpoint now requires an org_id header"
```

Re-running the same gap report is safe: it updates what it already wrote instead of appending a second set of tasks.

You can optionally restrict the scope of the updates:
- `--spec-only` — update only `spec.md`
- `--plan-only` — update only `plan.md`
- `--tasks-only` — update only `tasks.md`

## Workflow

1.  **Parse the Gap Report** to determine what drift occurred.
2.  **Run the core Spec-Kit `check-prerequisites.sh` script** to identify target feature artifacts.
3.  **Normalize Gaps** into categories (Wiring & Navigation, Contracts, Test Coverage, etc.).
4.  **Surgically Edit** the feature's specific `spec.md`, `plan.md`, and `tasks.md`.
5.  **Output a Sync Impact Report** detailing the created tasks and next steps (e.g., routing to `/speckit.implement`).
