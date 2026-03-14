# Changelog

All notable changes to the Reconcile extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
