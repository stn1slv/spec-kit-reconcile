# Task Manager Constitution

## Core Principles

### I. Single Ownership

Every task MUST have exactly one owner. Shared or unassigned active tasks are not allowed.

### II. Retention Is Explicit

Every feature that stores user data MUST state its retention rule in its spec.

## Architecture Standards

### A1. Routes Are Declared and Tested

Any new API route MUST be declared in the plan's Routing & Navigation section, and MUST have an automated integration test before merge.

## Quality Gates

- Every merged feature keeps its spec, plan and tasks consistent with the shipped code.
