# Baseline: v1.3.1, against `6e99cae`

Round for the changes v1.3.1 makes after a review of v1.3.0 against the Spec Kit docs merged in github/spec-kit#4945 and #4948: flow-forward constitutions now stop all writes, living spec and a silent constitution reconcile normally, open Convergence tasks that trace to an amended entry are named, a clarification never withdraws an approval, and `Approved:` gets a dated entry on any re-run write. Expectations were amended and committed in `6e99cae`, before these runs, so pre-registration holds.

Eight runs, all against the final command: **A, C, E, H, I** (re-run because v1.3.0's rounds left A, E, H and I graded only against an earlier command) and the new **M1, M2, M3**. Each runner was a fresh agent given only `commands/reconcile.md`, its arguments and its own working copy, with Step 2 answers scripted in advance. C ran on A's end state, committed in the copy first. M1 to M3 ran on variant copies whose changes were part of the baseline commit.

| Case | Result | Notes |
|---|---|---|
| A | Pass | approval first; A1 raised; no route path invented; T008 to T010 with the mandatory test; sweep closed by plan amendments with no task; Next Step implement, converge, archive |
| C | Pass | tree byte-clean; earlier judgments not revisited; implement-then-converge branch and stale-archive warning fired |
| E | Pass | conflict still withholds an approved item; `withheld: 1` with the M = 2 split stated; `Approved:` names only the item written |
| H | Pass | REQ IDs, retired REQ-003 untouched, Assumptions placed by role, Principle II task, A1 self-satisfied, digest test |
| I | Pass | R26: both pre-existing notes gained `Approved: 2026-10-13: approve all`; R13, R17, R19, R22, R23 and the retry path without a path all hold |
| M1 | Pass | R28: flow-forward wrote nothing, asked nothing, showed `No change (flow-forward)`, recommended `/speckit.specify` and no archive |
| M2 | Pass | R29: living spec reconciled exactly as Case F, with no "wrong remedy" note |
| M3 | Pass | R30: `FR-003` amended; T008 byte-identical, named under `## Outstanding Items` and in the Next Step before implement |

## Deviations

- **C used two `None` rows** for targets an earlier run had already applied. The Step 3 action list still has no "already applied" action; the v1.3.0 round's C wrote an empty table instead. Both wrote the right bytes (none). This is the gap round 4 of v1.2.1 first recorded, still open.
- **A added no task for the sweep relocation** and recorded test coverage as "not stated" in the plan. R16 accepts either a task without a path or no task when plan amendments close the item.
- **I amended FR-005 only** and added no acceptance scenario, and appended T007 to T009 rather than amending the open T005. Both choices are permitted and were stated under `## Defaults Applied`.

## Round 2: after the multi-model review, against `8f01462`

A five-model review (opus, fable, terra, sol, flash; terra and sol needed a retry after an expired SSO session) found nine should-fix items: the `Approved:` append was undefined for a rewrite, the Convergence warning could come after an implement recommendation, flow-forward runs could still recommend converge and fail the success criteria, the README still called every run interactive, a re-run that wrote nothing dropped the Convergence warning, a constitution only mentioning flow-forward could block writes, and two v1.3.1 rules had no graded case. They were fixed in `8f01462` together with amended expectations for M1 and M3 and the new M4 and M5, committed before these runs. M4 and M5 ran on M3's end state, committed in each copy first.

| Case | Result | Notes |
|---|---|---|
| M1 | Pass | the constitution sentence is quoted under `## Approval`; converge is recommended nowhere; tree byte-clean |
| M3 | Pass | the T008 warning is the first Next Step bullet; T008 byte-identical |
| M4 | Pass | R31: identical re-run wrote nothing, left `Approved:` alone, and still named T008 first |
| M5 | Pass | R32: slug reused, `FR-003` rewritten, `Approved:` became `approve all; 2026-10-13: approve all` |

Cases A, C, E, H and I were not re-run after `8f01462`. The changes there touch flow-forward runs, Convergence tasks, rewrites under an existing slug and success-criteria wording, none of which those cases exercise.

M4 listed its already-applied target as an `Amend` row while writing nothing. This is the same missing "already applied" action the round-1 C run met.
