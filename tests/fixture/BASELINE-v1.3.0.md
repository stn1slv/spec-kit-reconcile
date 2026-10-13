# Baseline: v1.3.0, against `9c3add4`

Targeted round for the two changes in this release: the always-asked Step 2 approval question (with the `Approved:` line on revision notes and the `## Approval` report section) and the Next Step routing back to converge. Expectations were amended and committed in `9c3add4`, before these runs, so pre-registration holds.

Seven runs: **A, C, E, H, I** (existing cases the approval rule touches) and the new **L1, L2**. Each runner was a fresh agent given only `commands/reconcile.md`, its arguments and its own working copy, with the Step 2 answers scripted in advance and an instruction to print every question before its scripted answer. C ran on A's end state, committed in the copy first so `git status` shows only C's writes. Every other case used a clean copy, and `check-prerequisites.sh` pointed `FEATURE_DIR` at `003-attachments` in each, so R1 fired where it should.

| Case | Result | Notes |
|---|---|---|
| A | Pass | approval asked first and alone; both notes carry `Approved:`; Next Step is implement, then converge, then archive |
| C | Pass | answer worded differently on purpose ("Approve all items."); `Approved:` line untouched; tree byte-clean |
| E | Pass | approved items still withheld by the unresolved conflict; `withheld 2` from a stated M = 3 split |
| H | Pass | approval first, then the Principle II obligation; all earlier traps hold |
| I | Pass | R26: both pre-existing notes gained `Approved:` because, and only because, this run amended them |
| L1 | Pass | R24: `FR-003` byte-for-byte unchanged; sort item withheld with a converge recommendation |
| L2 | Pass | R25: approve none wrote nothing, did not abort, routed to converge, no archive recommendation |

## What the round shows

**The approval question is asked in every case, first and on its own,** including the re-run (C) and the case with no other question (L2). No runner folded it into a constitution question or skipped it because the scripted answer was known.

**Approval and conflicts stay separate.** In E the runner approved every item and still withheld the co-ownership items for the unresolved Principle I conflict, reporting them as approved under `## Approval` and withheld under `## Constitution Compliance`. That is the case the new "approval never overrides a conflict" sentence exists for.

**The `Approved:` line behaves like the date.** A and L1 wrote it fresh, I added it to two notes written before it existed, and C left it alone although its answer was worded differently. L1's line quotes the partial answer, so the note records which item the team accepted and which it did not.

**The converge routing is consistent.** Every case that changed an artifact routes to `/speckit.converge` after implement; L1 and L2 also recommend it for the withheld item. L2 and C correctly drop the archive recommendation because nothing changed.

## Deviations and new findings

- **E's withheld count is 2, not 1.** The runner split the co-ownership sentence into board and API items and stated the split under `## Defaults Applied`. This is the registered `M` granularity gap; the stated choice is what is graded.
- **C's Next Step had no matching branch.** No artifact changed and nothing was withheld, so none of the listed conditions applied; the runner said so and recommended implement then converge for the open tasks. The command has no branch for "already applied, tasks still open". Registered, not fixed: the output was reasonable and no byte was wrong.
- **C's Targets table was empty with all-zero counts and a `note:` line.** This is the "already applied" gap round 4 recorded for v1.2.1; it is unchanged.
- **I amended open task T005 in place** under the reused slug instead of appending a new task. The same-slug rule permits this for an open task the earlier run wrote.
- **H marked the digest endpoint's HTTP method `NEEDS CLARIFICATION`** while writing the path the report named. A1 was still correctly treated as met, since the rule asks for the route's declaration and the path is present.

## Round 2: after review round 1, against `f6a2414`

A five-model review (fable, opus, terra and sol; pro failed to start) found eight should-fix items, fixed in `f6a2414` together with amended expectations, committed before these runs. Three cases re-ran: **C**, **L1**, and the new **L3**.

| Case | Result | Notes |
|---|---|---|
| C | **Fail** | the new Next Step branch and the restored archive warning both fired, but the runner asked a discretionary question and added a path to Case A's open T011: an identical re-run produced a diff |
| L1 | Pass, one deviation | `Approved:` names the settings item only; `FR-003` untouched; deviation: no FR added to `spec.md`, which Case A's inherited bullet expects |
| L3 | Pass | approve none on 003 left the bare `### Revision:` note without the retroactive heading (R27) |

The C failure was not caused by the round-1 fixes: the runner revisited a judgment Case A had made and the report did not touch. Round 1's C runner had not done that, so the command was relying on runner restraint rather than a rule.

## Round 3: after review round 2, against `ab21db3`

A two-model review of the fix diff (opus, terra) found four should-fix items, including a contradiction the round-1 fix introduced ("listed only under `## Approval`" against the moot-finding rule). They were fixed in `ab21db3` with a new re-run rule (an earlier run's entry changes only where the current report changes what it says), and Case C's expectation was amended before the run.

| Case | Result | Notes |
|---|---|---|
| C | Pass | tree byte-clean; the runner cited the re-run rule for leaving T011 alone; implement-then-converge branch and stale-archive warning both fired |

L1's missing spec FR is recorded here rather than fixed: whether an unreachable-screen drift adds a requirement or only plan entries has split across rounds before, like the user-story question in round 4 of v1.2.1.
